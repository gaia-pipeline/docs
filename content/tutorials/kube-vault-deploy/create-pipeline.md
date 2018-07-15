---
title: "1 - Create Pipeline"
description: "Create Pipeline written in Go"
weight: 20
---

## Create a Pipeline in Go

Run the following command in your terminal to get the pipeline code:

```
go get github.com/gaia-pipeline/tutorial-k8s-deployment-go
```

Open now the pipeline in your favorite editor:

```
$GOPATH/src/github.com/gaia-pipeline/tutorial-k8s-deployment-go
```

Let's go through the code step by step from the `main.go` file:

```go
// GetSecretsFromVault retrieves all information and credentials
// from vault and saves it to local space.
func GetSecretsFromVault() error {
	// Create vault client
	vaultClient, err := connectToVault()
	if err != nil {
		log.Printf("Error: %s\n", err.Error())
		return err
	}

	// Read kube config from vault and decode base64
	l := vaultClient.Logical()
	s, err := l.Read(kubeConfVaultPath)
	if err != nil {
		log.Printf("Error: %s\n", err.Error())
		return err
	}
	conf := s.Data["data"].(map[string]interface{})
	kubeConf, err := base64.StdEncoding.DecodeString(conf["conf"].(string))
	if err != nil {
		log.Printf("Error: %s\n", err.Error())
		return err
	}

	// Convert config to string and replace localhost.
	// We use here the magical DNS name "host.docker.internal",
	// which resolves to the internal IP address used by the host.
	// If this should not work for you, replace it with your real IP address.
	confStr := string(kubeConf[:])
	confStr = strings.Replace(confStr, "localhost", hostDNSName, 1)
	kubeConf = []byte(confStr)

	// Write kube config to file
	if err = writeToFile(kubeLocalPath, kubeConf); err != nil {
		log.Printf("Error: %s\n", err.Error())
		return err
	}

	// Read app image version from vault
	v, err := l.Read(appVersionVaultPath)
	if err != nil {
		log.Printf("Error: %s\n", err.Error())
		return err
	}

	// Write image version to file
	version := (v.Data["data"].(map[string]interface{}))["version"].(string)
	if err = writeToFile(appVersionLocalPath, []byte(version)); err != nil {
		log.Printf("Error: %s\n", err.Error())
		return err
	}
	log.Println("All data has been retrieved from vault!")
	return nil
}
```

This job uses the vault go-client to retrieve our saved information from vault. It will replace the hostname from the API-Server which should be "localhost" with the magic DNS name "host.docker.internal". Due to a <a href="https://github.com/gaia-pipeline/gaia/issues/28" target="_blank">bug in Gaia</a> it is currently not possible that two jobs share data during a pipeline run.
To work around this issue, we write all information to the local space.

```go
// CreateNamespace creates the namespace for our app.
// If the namespace already exists nothing will happen.
func CreateNamespace() error {
	// Get kubernetes client
	c, err := getKubeClient(kubeLocalPath)
	if err != nil {
		return err
	}

	// Create namespace object
	ns := &v1.Namespace{
		ObjectMeta: metav1.ObjectMeta{
			Name: appName,
		},
	}

	// Lookup if namespace already exists
	nsClient := c.Core().Namespaces()
	_, err = nsClient.Get(appName, metav1.GetOptions{})

	// namespace exists
	if err == nil {
		log.Printf("Namespace '%s' already exists. Skipping!", appName)
		return nil
	}

	// Create namespace
	_, err = c.Core().Namespaces().Create(ns)
	if err != nil {
		return err
	}

	log.Printf("Service '%s' has been created!\n", appName)
	return err
}
```

`CreateNamespace` creates the namespace and will be skipped if the namespace already exists. Helper/Util functions help us to avoid duplicated code and write clean code.

```go
// CreateDeployment creates the kubernetes deployment.
// If it already exists, it will be updated.
func CreateDeployment() error {
	// Get kubernetes client
	c, err := getKubeClient(kubeLocalPath)
	if err != nil {
		log.Printf("Error: %s\n", err.Error())
		return err
	}

	// Load image version from file
	v, err := ioutil.ReadFile(appVersionLocalPath)
	if err != nil {
		log.Printf("Error: %s\n", err.Error())
		return err
	}

	// Create deployment object
	d := &v1beta1.Deployment{}
	d.ObjectMeta = metav1.ObjectMeta{
		Name: appName,
		Labels: map[string]string{
			"app": appName,
		},
	}
	d.Spec = v1beta1.DeploymentSpec{
		Replicas: &replicas,
		Selector: &metav1.LabelSelector{
			MatchLabels: map[string]string{
				"app": appName,
			},
		},
		Template: v1.PodTemplateSpec{
			ObjectMeta: d.ObjectMeta,
			Spec: v1.PodSpec{
				Containers: []v1.Container{
					v1.Container{
						Name:            appName,
						Image:           fmt.Sprintf("%s:%s", appName, v),
						ImagePullPolicy: v1.PullAlways,
						Ports: []v1.ContainerPort{
							v1.ContainerPort{
								ContainerPort: int32(80),
							},
						},
					},
				},
			},
		},
	}

	// Lookup existing deployments
	deployClient := c.ExtensionsV1beta1().Deployments(appName)
	_, err = deployClient.Get(appName, metav1.GetOptions{})

	// Deployment already exists
	if err == nil {
		_, err = deployClient.Update(d)
		if err != nil {
			log.Printf("Error: %s\n", err.Error())
			return err
		}
		log.Printf("Deployment '%s' has been updated!\n", appName)
		return nil
	}

	// Create deployment object in kubernetes
	_, err = deployClient.Create(d)
	if err != nil {
		log.Printf("Error: %s\n", err.Error())
		return err
	}
	log.Printf("Deployment '%s' has been created!\n", appName)
	return nil
}
```

`CreateDeployment` creates the deployment object if it does not exist. If it exists, it will be updated.

```go
// CreateService creates the service for our application.
// If the service already exists, it will be updated.
func CreateService() error {
	// Get kubernetes client
	c, err := getKubeClient(kubeLocalPath)
	if err != nil {
		log.Printf("Error: %s\n", err.Error())
		return err
	}

	// Create service obj
	s := &v1.Service{
		ObjectMeta: metav1.ObjectMeta{
			Name: appName,
		},
		Spec: v1.ServiceSpec{
			Selector: map[string]string{
				"app": appName,
			},
			Type: v1.ServiceTypeNodePort,
			Ports: []v1.ServicePort{
				v1.ServicePort{
					Protocol:   v1.ProtocolTCP,
					Port:       int32(8090),
					TargetPort: intstr.FromInt(80),
				},
			},
		},
	}

	// Lookup for existing service
	serviceClient := c.Core().Services(appName)
	currService, err := serviceClient.Get(appName, metav1.GetOptions{})

	// Service already exists
	if err == nil {
		s.ObjectMeta = currService.ObjectMeta
		s.Spec.ClusterIP = currService.Spec.ClusterIP
		_, err = serviceClient.Update(s)
		if err != nil {
			log.Printf("Error: %s\n", err.Error())
			return err
		}
		log.Printf("Service '%s' has been updated!\n", appName)
		return nil
	}

	// Create service
	_, err = serviceClient.Create(s)
	if err != nil {
		log.Printf("Error: %s\n", err.Error())
		return err
	}
	log.Printf("Service '%s' has been created!\n", appName)
	return nil
}
```

`CreateService` creates the service object if it does not exist. If it exist, it will be updated.
The last part, the main function, we register all jobs with their respective priority and start the serve function as usual.

Let's have a look at the `main_test.go` file:

```go
func TestMain(m *testing.M) {
	// We change the vault address to localhost and the Kube API-Hostname to localhost.
	// This allows us to start the starts locally.
	vaultAddress = "http://localhost:8200"
	hostDNSName = "localhost"
	err := GetSecretsFromVault()
	if err != nil {
		fmt.Printf("Cannot retrieve data from vault: %s\n", err.Error())
		os.Exit(1)
	}

	err = CreateNamespace()
	if err != nil {
		fmt.Printf("Cannot create namespace: %s\n", err.Error())
		os.Exit(1)
	}

	r := m.Run()
	os.Exit(r)
}

func TestCreateService(t *testing.T) {
	err := CreateService()
	if err != nil {
		t.Error(err)
	}
}

func TestCreateDeployment(t *testing.T) {
	err := CreateDeployment()
	if err != nil {
		t.Error(err)
	}
}
```

These are basic tests but in reality you can test whatever you want. There is basically no limitation!

Let's continue with the next chapter: [{{%icon circle-arrow-right%}}2 - Run tests and compile pipeline]({{%relref "tutorials/kube-vault-deploy/run-tests-compile.md"%}})
