# Lab 07 - Services

A Kubernetes Service is an abstraction which defines a logical set of Pods and a policy by which to access them - sometimes called a micro-service. The set of Pods targeted by a Service is (usually) determined by a Label Selector.

As an example, consider an image-processing backend which is running with 3 replicas. Those replicas are fungible - frontends do not care which backend they use. While the actual Pods that compose the backend set may change, the frontend clients should not need to be aware of that or keep track of the list of backends themselves. The Service abstraction enables this decoupling.

## Task 0: Creating a namespace

Create a namespace for this lab:

```
kubectl create ns lab-07

---

namespace/lab-07 created
```

## Task 1: Creating your first service

Create a file `lab-07-deployment.yml` with the YAML for the deployment using the
content below:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: container-info
  labels:
    app: container-info
spec:
  replicas: 3
  selector:
    matchLabels:
      app: container-info
  template:
    metadata:
      labels:
        app: container-info
    spec:
      containers:
      - name: container-info
        image: gluobe/container-info:blue
        ports:
        - containerPort: 80
```

Create the deployment using the above file:

```
kubectl apply -f lab-07-deployment.yml -n lab-07

---

deployment.apps/container-info created
```

Verfiy that this is working:

```
kubectl get pods -n lab-07

---

NAME                              READY   STATUS    RESTARTS   AGE
container-info-86bcc46f95-2jbmz   1/1     Running   0          15s
container-info-86bcc46f95-7lplz   1/1     Running   0          15s
container-info-86bcc46f95-szw88   1/1     Running   0          15s
```

Now create a file `lab-07-service.yml` with the YAML for the service using the content below:

```yaml
kind: Service
apiVersion: v1
metadata:
  name: container-info
spec:
  selector:
    app: container-info
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: NodePort
```

Create the service using the above file:

```
kubectl apply -f lab-07-service.yml -n lab-07

---

service/container-info created
```

Check that the service has been created succesfully:

```
kubectl get service -n lab-07

---

NAME             TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
container-info   NodePort   10.108.171.167   <none>        80:30631/TCP   20s
```

## Task 2: Inspecting your first service

To see more information about the service we can, again, use the
`kubectl describe` command:

```
kubectl -n lab-07 describe service container-info

---

Name:                     container-info
Namespace:                lab-07
Labels:                   <none>
Annotations:              <none>
Selector:                 app=container-info
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.108.171.167
IPs:                      10.108.171.167
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  30631/TCP
Endpoints:                10.244.0.10:80,10.244.0.11:80,10.244.0.9:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

From the output you can see that a service looks a lot like a loadbalancer, you can see the VIP (IP) and the different backends (Endpoints).

## Task 3: Testing your first service

In previous labs we always used the `port-forward` method to expose a pod, as this method only allows us to connect to a single pod we will use a different method when wofking with service.

To test your service you can run the command below, it will automatically open a browser with your service/application:

```
minikube service container-info -n lab-07

---

|-----------|----------------|-------------|-----------------------------|
| NAMESPACE |      NAME      | TARGET PORT |             URL             |
|-----------|----------------|-------------|-----------------------------|
| lab-07    | container-info |          80 | http://192.168.59.102:30631 |
|-----------|----------------|-------------|-----------------------------|
🎉  Opening service lab-07/container-info in default browser...
```

> NOTE: when you refresh you should see a different "avatar" for each pod, the
> avatar is generated based on the container name, however browsers tend to
> cache pretty hard, so you might need to clear your cache/cookies before
> hitting refresh to see a different avatar.

If you cannot see different avatars, use can use the command below to test with `curl` instead.  The command will `curl` the application 10 times and will list the container name.  You should see that it hits different pods:

```
for i in {1..10}; do curl -s $(minikube service container-info -n lab-07 --url) | grep -E "<td>container-info"; done

---
          <td>container-info-86bcc46f95-szw88</td>
          <td>container-info-86bcc46f95-7lplz</td>
          <td>container-info-86bcc46f95-szw88</td>
          <td>container-info-86bcc46f95-2jbmz</td>
          <td>container-info-86bcc46f95-2jbmz</td>
          <td>container-info-86bcc46f95-2jbmz</td>
          <td>container-info-86bcc46f95-szw88</td>
          <td>container-info-86bcc46f95-7lplz</td>
          <td>container-info-86bcc46f95-7lplz</td>
          <td>container-info-86bcc46f95-7lplz</td>
```

## Task 4: Cleaning up

Clean up the namespace for this lab:

```
kubectl delete ns lab-07

---

namespace "lab-07" deleted
```