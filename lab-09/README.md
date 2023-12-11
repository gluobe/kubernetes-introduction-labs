# Lab 09 - Dashboard

The dashboard serves different functions, primarily it is used to visualize your Kubernetes cluster, however it can also be used to deploy new objects into your cluster.  In this small lab we will explore both functions.

## Task 1: Opening the dashboard

Opening the Kubernetes dashboard with minikube is very easy, simply run the following command and the dashboard will open automatically in your browser:

```
minikube dashboard

---

🔌  Enabling dashboard ...
    ▪ Using image docker.io/kubernetesui/dashboard:v2.7.0
    ▪ Using image docker.io/kubernetesui/metrics-scraper:v1.0.8
💡  Some dashboard features require the metrics-server addon. To enable all features please run:

	minikube addons enable metrics-server


🤔  Verifying dashboard health ...
🚀  Launching proxy ...
🤔  Verifying proxy health ...
🎉  Opening http://127.0.0.1:38567/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/ in your default browser...
```

## Task 2: Creating a namespace

We will now create a namespace using the dashboard UI.

Click the `+` link at the top right corner of the dashboard.  Now select the `Create from input` tab.  Copy the content below and paste it into the form:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: lab-09
```

Finally click the `Upload` button.

In the left menu select the `Namespaces` item and verify that your namespace has been created succesfully.

## Task 3: Deploying an app

To deploy an app using the dashboard UI click the `+` link at the top right corner of the dashboard.  Now select the `Create from form` tab.

Fill in the following details:

* `App name`: container-info
* `Container image`: gluobe/container-info:blue
* `Service`: External
* `Port`: 8888
* `Target port`: 80
* `Protocol`: TCP
* `Namespace`: lab-09

Click the `Deploy` button at the bottom.

## Task 4: Opening your app

To open you app, go back to your terminal and enter the following command:

```
minikube service container-info -n lab-09

🎉  Opening kubernetes service lab-09/container-info in default browser...
```

You should see a familiar website.

## Task 5: Deleting a namespace

While you can easily add namespaces (or other objects) from the dashboard, you 
cannot delete objects from there.  So delete the namespace, head back over to 
your terminal and run the following command:

```
kubectl delete ns lab-09

---

namespace "lab-09" deleted
```