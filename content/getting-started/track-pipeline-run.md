---
title: "2 - Track your pipeline run"
description: "Track your pipeline run"
weight: 30
---

#### Track your pipeline run

You successfully compiled your first pipeline and already started it. Awesome! {{<icon fa-smile-o>}} <br />

Now it's time to track the status of your pipeline run. You should already see the detailed overview. If <br />
this is not the case, you can go to the detailed overview by click on the name of your pipeline in the overview <br />
view.

![pipeline-detail](/images/pipeline-detail.png?width=450px)

At the top you can start another run of your pipeline if you want by clicking the button "Start Pipeline". <br />
Next to this button you should see another button "Show Logs". If you click on this button you will be redirected <br />
to the log view of your pipeline run. 

The next part is the graphical view of your pipeline. It's a graph that has been automatically drawn from your <br />
developed pipeline and will be automatically updated when one job of your pipeline has been completed. <br />

{{% notice tip %}}
Point your mouse pointer over it and scroll in and out. Use it to zoom in/out of your graph. <br />
This is really helpful when you have a quite complex graph. 
{{% /notice %}}

{{% notice tip %}}
You can click on a completed job-node. If you click afterwards on the "Show Logs" button you just see the output <br />
of this single job instead of a correlation of all jobs.
{{% /notice %}}

At the bottom of the detailed pipeline view you should see a history table of all your runs. <br />
You can click on the ID to select this run which will be shown in the detailed view.