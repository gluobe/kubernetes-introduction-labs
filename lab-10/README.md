# Lab 10 - Helm & Operators

## 1 Helm

Helm is a package manager for Kubernetes. It contains pre-configured Kubernetes resources that are called Charts.

These charts can be seen a templates for complete software setups on Kubernetes. These templates can include multiple pods, containers, ingresses, services and so on. Usually, the templates can be further configured using a values file.

In this lab we will look at the very basics of a Helm Chart.

### Task 0: Examine a basic Helm Chart

The most interesting parts of a Helm Chart are the templates directory and the values file.

The templates directory contains yaml files that look a lot like the Kubernetes yaml files you already know. But using helm, these files follow the standard conventions for writing Go templates.

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: deis-database
  namespace: deis
  labels:
    app.kubernetes.io/managed-by: deis
spec:
  replicas: 1
  selector:
    app.kubernetes.io/name: deis-database
  template:
    metadata:
      labels:
        app.kubernetes.io/name: deis-database
    spec:
      serviceAccount: deis-database
      containers:
        - name: deis-database
          image: {{ .Values.imageRegistry }}/postgres:{{ .Values.dockerTag }}
          imagePullPolicy: {{ .Values.pullPolicy }}
          ports:
            - containerPort: 5432
          env:
            - name: DATABASE_STORAGE
              value: {{ default "minio" .Values.storage }}
```

In this example the fields `image` and `imagePullPolicy` are replaced using the Go templating engine with the values specified in the values.yaml file. As an end user you ussually create a custom values file with the configuration you require.

```yaml
imageRegistry: docker.io
dockerTag: latest
pullPolicy: IfNotPresent
```

## 2 Operators

Operators are software extensions to Kubernetes that make use of CRDs (Custom Resource Definitions) to manage applications and their components. These Operators can be compared to their human counterpart, a technician with deep knowlegde of the software and someone that knows how to deploy, maintain and fix it.

Operators come in handy when dealing with complex software such as multi-node database setups. Some example automations that operators can privide are:
- Deploying on demand
- Taking and restoring backups
- Handling upgrades
- Simulate failures
- Choose a leader for distributed software
- ...

In this lab we will install the MongoDB Community Operator into the cluster using Helm and then deploy a database using a CRD.

### Task 0: Create a namespace

Create a namespace for this lab:
```
kubectl create ns lab-10
```

### Task 1: Get the MongoDb Operator Helm chart

First, you will need to install Helm v3. You can follow the instructions on this page https://helm.sh/docs/intro/install/.

Add the MongoDB Helm Chart repository:
```
helm repo add mongodb https://mongodb.github.io/helm-charts
```

Now you can list all the available Charts in the repository and searh for the `mongodb/community-operator` chart:
```
helm search repo mongodb
```

To view the default values (from the values.yaml file) you can execute the following command:
```
helm show values mongodb/community-operator
```

### Task 2: Deploy the Operator

You can deploy the Operator using the `helm install` command. We will not be changing any values so we only need to specify the name of our deployment, the chart to use and the namespace:
```
helm install community-operator-demo mongodb/community-operator --namespace lab-10
```

The MongoDB Operator Helm Chart will by default install the CRDs. Verify the CRDs are available inside the cluster:
```
kubectl get crd

----

NAME                                            CREATED AT
mongodbcommunity.mongodbcommunity.mongodb.com   2023-12-07T13:31:54Z
```

Now verify the operator is running:
```
kubectl -n lab-10 get all
```

### Task 3: Deploy a MongoDB database using the CRD

Now that the operator is running and the CRDs are available we can easily deploy a MongoDB database inside the cluster.

Copy the following manifest into the file `lab-10-mongodb.yaml` and deploy it to the namespace.

```yaml
apiVersion: mongodbcommunity.mongodb.com/v1
kind: MongoDBCommunity
metadata:
  name: mongodb-demo
spec:
  type: ReplicaSet
  members: 1
  version: "6.0.5"
  security:
    authentication:
      modes: ["SCRAM"]
  users:
    - name: my-user
      db: admin
      passwordSecretRef:
        name: my-user-password
      roles:
        - name: clusterAdmin
          db: admin
        - name: userAdminAnyDatabase
          db: admin
      scramCredentialsSecretName: my-scram
  statefulSet:
    spec:
      template:
        spec:
          containers:
            - name: mongod
              resources:
                requests:
                  cpu: "0.1"
                  memory: 100M
            - name: mongodb-agent
              resources:
                requests:
                  cpu: "0.1"
                  memory: 100M
---
apiVersion: v1
kind: Secret
metadata:
  name: my-user-password
type: Opaque
stringData:
  password: asupersecurepassword
```

```
kubectl -n lab-10 apply -f lab-10-mongodb.yaml
```

Verify that the database is deployed:
```
kubectl -n lab-10 get all
```

We should see (apart from the Operator resources) that the CRD from the manifest not only deployed a Pod `mongodb-demo-0` but also a Service `mongodb-demo-svc`.

### Task 4: Clean up

Delete the namespace:
```
kubectl delete ns lab-10
```
