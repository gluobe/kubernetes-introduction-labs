# Lab 11 - RBAC

Role-based access control (RBAC) is a method of regulating access to computer or network resources based on the roles of individual users within your organization.

The RBAC API uses four types of objects: `Role`, `RoleBinding`, `ClusterRole` and `ClusterRoleBinding`. A Role can contain multiple rules that represent a set of permissions. It is only possible to specify "allow" permissions and a Role by default has no permissions set (eg: A user with an empty role cannot do anything).

A Role always belongs to a specific namespace. A ClusterRole can be used to set permissions on multiple namespaces or cluster-scoped resources.

## Task 0: Create a namespace

```
kubectl create ns lab-11
```

## Task 1: Create a user

Generate an RSA-key for the demo-user:
```
openssl genrsa -out demo-user.key 2048
```

Create a Certificate Signing Reguest associated with the key:
```
openssl req -new -key demo-user.key -out demo-user.csr -subj "/CN=demo-user/O=demo-group"
```

Generate and sign a certificate with the minikube's CA certificate and key:
```
openssl x509 -req -in demo-user.csr -CA ~/.minikube/ca.crt -CAkey ~/.minikube/ca.key -CAcreateserial -out demo-user.crt -days 500
```

Now add the signed certificate and key to your kubeconfig:
```
kubectl config set-credentials demo-user --client-certificate=demo-user.crt --client-key=demo-user.key
kubectl config set-context demo-user-context --cluster=minikube --user=demo-user
```


## Task 2: Create a Role

Copy the following manifest to the file `lab-11-role.yaml` and deploy it to the namespace.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: lab-11
  name: pod-reader-demo
rules:
- apiGroups: [""]
  resources: ["pods", "pods/logs"]
  verbs: ["get", "watch", "list"]
```

```
kubectl -n lab-11 apply -f lab-11-role.yaml
```

Most resources are represented and accessed using a string representation that refers to the same name from the API endpoints. Resources can also have subresources and are delimited by a `/`. For example, the resource `pods/log` corresponds to the API endpoint `GET /api/v1/namespaces/{namespace}/pods/{name}/log` and gives access to the logs of a Pod. For a user to read pods, the role also needs the `pods` resource. The empty `apiGroups` field indicates the core API group.

It is possible to refer to resource names to restrict access to individual instances of a resource. For example, a container named "alpine-demo-container". It is also possible to use a wildcard `*` to allow any current or future action on all current and future resources.

### Task 3: Create a Role Binding

Copy the following manifest to the file `lab-11-rolebinding.yaml` and deploy it to the namespace.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: lab-11
subjects:
- kind: User
  name: demo-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader-demo
  apiGroup: rbac.authorization.k8s.io
```

```
kubectl -n lab-11 apply -f lab-11-rolebinding.yaml
```

With this RoleBinding we assign the user `demo-user` the Role created in the previous task for the namespace lab-11.

## Task 4: Verify the RBAC rules

Right now, you are still using the admin account. To change to the demo-user account change the context with the following command:
```
kubectl config use-context demo-user-context
```

Try to get the pods in the lab-11 namespace. It should succeed without error:
```
kubectl -n lab-11 get pods
```

Now try to get the pods from the default namespace. It should fail with a `forbidden` error:
```
kubectl -n default get pods
```

## Task 5: Clean up

Delete the kubeconfig entries for the demo-user and revert back to the minikube admin user:
```
kubectl config delete-context demo-user-context
kubectl config delete-user demo-user
kubectl config use-context minikube
```

Delete the namespace:
```
kubectl delete ns lab-11
```
