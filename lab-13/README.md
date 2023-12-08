# Lab 13 - Open excercise

## Task 0: Create a namespace

Create a namespace for this lab:
```
kubectl create ns lab-13
```

## Task 1: NGINX webserver

Create a webserver Deployment, make sure to configure the following:
- Deployment of an NGINX webserver
- One replica
- Use container image `nginxinc/nginx-unprivileged:latest`
- Use port `8080`
- Service of type NodePort with port `30000`
- Use a ConfigMap to supply the index file to the directory `/usr/share/nginx/html/`
- Make sure the container is running rootless
- Limit the container resources to 250Mi for memory and 100m for CPU

You can find the Kubernetes documentation [here](https://kubernetes.io/docs/concepts/).

Deploy the manifests to the namespace:
```
kubectl -n lab-13 apply -f manifests.yaml
```

Use minikube to connect to the Service named `webserver`:
```
minikube service -n lab-13 webserver
```

## Task 2: Clean up

Delete the namespace:
```
kubectl delete ns lab-13
```
