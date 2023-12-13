# Lab 03 - Namespaces

Kubernetes supports multiple virtual clusters backed by the same physical cluster. These virtual clusters are called namespaces.

Namespaces make it possible to run different environments on a single Kubernetes cluster, such as DEV, TST and UAT.  Namespace can also be used to set different access controls per namespace.

Most objects in Kubernetes can be namespaced (pods, services, pvc,...), keep in mind however that some objects however cannot be namespaced and are cluster-wide (pv for example).

## Task 1: Listing namespaces

To see which namespaces are available use the `kubectl get namespaces` command:

```
kubectl get namespaces

---

NAME              STATUS   AGE
default           Active   15m
kube-node-lease   Active   15m
kube-public       Active   15m
kube-system       Active   15m
```

## Task 2: Creating a new namespace

Creating a namespace is easy, `kubectl create namespace <namespacename>`:

```
kubectl create namespace test

---

namespace/test created
```

Check that your namespace has been created:

```
kubectl get ns

---

NAME              STATUS   AGE
default           Active   15m
kube-node-lease   Active   15m
kube-public       Active   15m
kube-system       Active   15m
test              Active   10s
```

> NOTE: as you can see we abreviated `namespace` to `ns`, most of the objects in
> Kubernetes have abreviations that you can use on the command line

## Task 3: Specifying namespaces

When working with namespaces it is important to know that if you do not specify
a specific namepace when issuing a `kubectl` command the default namespace of
your current context is assumed.  In our case this will be `default` but you can
of course create new and/or customize your context.

This means that running the following command:

```
kubectl get all

---

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   15m
```

Is exactly the same as running:

```
kubectl get all -n default

---

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   15m
```

But if we do this for a different namespace we of course get a completely
different result:

```
kubectl get all -n kube-system

---

NAME                                   READY   STATUS    RESTARTS      AGE
pod/coredns-5d78c9869d-wfjs4           1/1     Running   0             15m
pod/etcd-minikube                      1/1     Running   0             15m
pod/kube-apiserver-minikube            1/1     Running   0             15m
pod/kube-controller-manager-minikube   1/1     Running   0             15m
pod/kube-proxy-nxh74                   1/1     Running   0             15m
pod/kube-scheduler-minikube            1/1     Running   0             15m
pod/storage-provisioner                1/1     Running   0             15m

NAME               TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
service/kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   15m

NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/kube-proxy   1         1         1       1            1           kubernetes.io/os=linux   15m

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/coredns   1/1     1            1           15m

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/coredns-5d78c9869d   1         1         1       15m
```

## Task 4: All namespaces

It could happen that you do not really know in which namespace a Kubernetes
object is running.  If that is the case you can always add the
`--all-namespaces` option to your command to list objects from all of the
namespaces.

```
kubectl get pods --all-namespaces

---

NAMESPACE     NAME                               READY   STATUS    RESTARTS      AGE
kube-system   coredns-5d78c9869d-wfjs4           1/1     Running   0             15m
kube-system   etcd-minikube                      1/1     Running   0             15m
kube-system   kube-apiserver-minikube            1/1     Running   0             15m
kube-system   kube-controller-manager-minikube   1/1     Running   0             15m
kube-system   kube-proxy-nxh74                   1/1     Running   0             15m
kube-system   kube-scheduler-minikube            1/1     Running   0             15m
kube-system   storage-provisioner                1/1     Running   0             15m
```

## Task 5: Changing the default context

As mentioned above, we can can change the behaviour that when no namespace is
specified, the default namespace is assumed.  This is done by changing the
context.

Standard behaviour is that default namespace is automatically assumed, let us
verify this first:

```
kubectl get all

---

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   15m
```

Now change the "default" namespace to `kube-system`:

```
kubectl config set-context --current --namespace=kube-system

---

Context "minikube" modified.
```

If we now run `kubectl get all` without specifying a namespace we get all the
objects of the `kube-system` namespace:

```
kubectl get all

---

NAME                                   READY   STATUS    RESTARTS      AGE
pod/coredns-5d78c9869d-wfjs4           1/1     Running   0             15m
pod/etcd-minikube                      1/1     Running   0             15m
pod/kube-apiserver-minikube            1/1     Running   0             15m
pod/kube-controller-manager-minikube   1/1     Running   0             15m
pod/kube-proxy-nxh74                   1/1     Running   0             15m
pod/kube-scheduler-minikube            1/1     Running   0             15m
pod/storage-provisioner                1/1     Running   0             15m

NAME               TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
service/kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   15m

NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/kube-proxy   1         1         1       1            1           kubernetes.io/os=linux   15m

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/coredns   1/1     1            1           15m

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/coredns-5d78c9869d   1         1         1       15m
```

Of course we can still get the objects from the default namespace by specifying
it specifically:

```
kubectl get all -n default

---

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   15m
```

Now change the context again to its normal behavior:

```
kubectl config set-context --current --namespace=default

---

Context "minikube" modified.
```

## Task 6: Deleting namespaces

Deleting a namespace is very easy, keep in mind however that when you delete a
namespace *all* the objects in that namespace will be deleted. So always verify
that all the objects in that namespace can be deleted:

```
kubectl delete ns test

---

namespace "test" deleted
```
