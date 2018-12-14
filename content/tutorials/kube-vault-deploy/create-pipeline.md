---
title: "1 - Create Pipeline"
description: "Create Gaia Pipeline in Go"
weight: 20
---

## Create Gaia Pipeline in Go

The full pipeline code is available on github: <a href="https://github.com/gaia-pipeline/tutorial-k8s-deployment-go" target="_blank">k8s deployment tutorial code</a>
If you have go installed locally you can run the following command in your terminal to get the pipeline code:

```
go get github.com/gaia-pipeline/tutorial-k8s-deployment-go
```

Otherwise simply clone the source code:

```
git clone https://github.com/gaia-pipeline/tutorial-k8s-deployment-go
```

Open now the pipeline in your favorite editor.
Let's go through the code step by step from the `main.go` file:

```go
// GetSecretsFromVault retrieves all information and credentials
// from vault and stores it in cache.
func GetSecretsFromVault(args sdk.Arguments) error {
	// Get vault credentials
	for _, arg := range args {
		switch arg.Key {
		case "vault-token":
			vaultToken = arg.Value
		case "vault-address":
			vaultAddress = arg.Value
		}
	}

	// Create new Vault client instance
	vaultClient, err := vaultapi.NewClient(vaultapi.DefaultConfig())
	if err != nil {
		log.Printf("Error: %s\n", err.Error())
		return err
	}

	// Set vault address
	err = vaultClient.SetAddress(vaultAddress)
	if err != nil {
		log.Printf("Error: %s\n", err.Error())
		return err
	}

	// Set token
	vaultClient.SetToken(vaultToken)

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

	// Create file
	f, err := os.Create(kubeLocalPath)
	if err != nil {
		log.Printf("Error: %s\n", err.Error())
		return err
	}
	defer f.Close()

	// Write to file
	_, err = f.Write(kubeConf)

	log.Println("All data has been retrieved from vault!")
	return nil
}
```

This function (we could also call it a job now) uses the vault go-client to retrieve our saved Kube-Config from vault. It will replace the hostname from the API-Server which should be "localhost" with the magic DNS name "host.docker.internal". 

```go
// PrepareDeployment prepares the deployment by setting up
// the kubernetes client and caching all manual input from user.
func PrepareDeployment(args sdk.Arguments) error {
	// Setup kubernetes client
	config, err := clientcmd.BuildConfigFromFlags("", kubeLocalPath)
	if err != nil {
		return err
	}

	clientSet, err = kubernetes.NewForConfig(config)
	if err != nil {
		return err
	}

	// Cache given arguments for other jobs
	for _, arg := range args {
		switch arg.Key {
		case "vault-address":
			vaultAddress = arg.Value
		case "image-name":
			imageName = arg.Value
		case "replicas":
			rep, err := strconv.ParseInt(arg.Value, 10, 64)
			if err != nil {
				log.Printf("Error: %s\n", err)
				return err
			}
			replicas = int32(rep)
		case "app-name":
			appName = arg.Value
		}
	}

	return nil
}
```

`PrepareDeployment` prepares the connection to the Kubernetes cluster and parses all given arguments. All given arguments are passed in via Gaia's UI. 

```go
// CreateNamespace creates the namespace for our app.
// If the namespace already exists nothing will happen.
func CreateNamespace(args sdk.Arguments) error {
	// Create namespace object
	ns := &v1.Namespace{
		ObjectMeta: metav1.ObjectMeta{
			Name: appName,
		},
	}

	// Lookup if namespace already exists
	nsClient := clientSet.Core().Namespaces()
	_, err := nsClient.Get(appName, metav1.GetOptions{})

	// namespace exists
	if err == nil {
		log.Printf("Namespace '%s' already exists. Skipping!", appName)
		return nil
	}

	// Create namespace
	_, err = clientSet.Core().Namespaces().Create(ns)
	if err != nil {
		return err
	}

	log.Printf("Service '%s' has been created!\n", appName)
	return err
}
```

`CreateNamespace` creates the namespace and will be skipped if the namespace already exists.

```go
// CreateDeployment creates the kubernetes deployment.
// If it already exists, it will be updated.
func CreateDeployment(args sdk.Arguments) error {
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
						Image:           imageName,
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
	deployClient := clientSet.ExtensionsV1beta1().Deployments(appName)
	_, err := deployClient.Get(appName, metav1.GetOptions{})

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
func CreateService(args sdk.Arguments) error {
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
	serviceClient := clientSet.Core().Services(appName)
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
The last part, the main function, we do register all jobs with their respective priority and start the serve function as usual.

Let's have a look at the `main_test.go` file:

```go
func TestMain(m *testing.M) {
	hostDNSName = "localhost"
	err := GetSecretsFromVault(sdk.Arguments{
		sdk.Argument{
			Type:  sdk.VaultInp,
			Key:   "vault-token",
			Value: "root-token",
		},
		sdk.Argument{
			Type:  sdk.VaultInp,
			Key:   "vault-address",
			Value: "http://localhost:8200",
		}})
	if err != nil {
		fmt.Printf("Cannot retrieve data from vault: %s\n", err.Error())
		os.Exit(1)
	}

	err = PrepareDeployment(sdk.Arguments{
		sdk.Argument{
			Type:  sdk.TextFieldInp,
			Key:   "image-name",
			Value: "nginx:1.13",
		},
		sdk.Argument{
			Type:  sdk.TextFieldInp,
			Key:   "app-name",
			Value: "nginx",
		},
		sdk.Argument{
			Type:  sdk.TextFieldInp,
			Key:   "replicas",
			Value: "1",
		},
	})
	if err != nil {
		fmt.Printf("Cannot prepare the deployment: %s\n", err.Error())
		os.Exit(1)
	}

	err = CreateNamespace(sdk.Arguments{})
	if err != nil {
		fmt.Printf("Cannot create namespace: %s\n", err.Error())
		os.Exit(1)
	}

	r := m.Run()
	os.Exit(r)
}

func TestCreateService(t *testing.T) {
	err := CreateService(sdk.Arguments{})
	if err != nil {
		t.Error(err)
	}
}

func TestCreateDeployment(t *testing.T) {
	err := CreateDeployment(sdk.Arguments{})
	if err != nil {
		t.Error(err)
	}
}
```

These are basic tests but in reality you can test whatever you want. There is basically no limitation!

Let's continue with the next chapter: [{{%icon circle-arrow-right%}}2 - Run tests and compile pipeline]({{%relref "tutorials/kube-vault-deploy/run-tests-compile.md"%}})
