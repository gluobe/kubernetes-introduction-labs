# Lab 12 - Security Context

A Security Context defines the parameters around privilege, access control and capabilities of a Pod or Container.

This can include:
- File access control based on user and group ID
- Enfore a read-only root filesystem
- Allow privilege escalation
- Allow certain Linux capabilities (eg: CAP_NET_RAW)
- ...

In this lab we will take a look at user and group IDs and permissions inside containers

## Task 0: Create a namespace

Create a namespace for this lab:
```
kubectl create ns lab-12
```

## Task 1: Deploy a rootless Pod

Copy the below manifest `lab-12-rootless.yaml` and deploy it to the namespace.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo1
spec:
  securityContext:
    runAsNonRoot: true
  containers:
    - name: c1
      image: alpine:3.18.5
      command: ["sh", "-c", "sleep 1h"]
      volumeMounts:
        - name: c1-vol
          mountPath: /data
  volumes:
    - name: c1-vol
      emptyDir: {}
```

```
kubectl -n lab-12 apply -f lab-12-rootless.yaml
```

Now describe the deployed Pod:
```
kubectl -n lab-12 describe pod security-context-demo1
```

We see that the container `c1` is not running and an error has occured (`CreateContainerConfigError`). The alpine image by default uses the root user but our SecurityContext forbids it. We will fix this issue in the following task.

Delete the failed Pod:
```
kubectl -n lab-12 delete pod security-context-demo1
```

## Task 2: Deploy a rootless Pod with a specific user and group ID

Now edit the manifest so that the containers in the Pod will run as user `1000` and group `3000`. Also specify a filesystem group with ID `2000`. Apply this modified manifest to the namespace.

```yaml
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
```

```
kubectl -n lab-12 apply -f lab-12-rootless.yaml
```

Once the container has started, execute an interactive shell into the container:
```
kubectl exec -n lab-12 -it security-context-demo1 -- sh
```

Verify that the user has ID `1000` and belongs to the groups `2000` and `3000`:
```
id

----

uid=1000 gid=3000 groups=2000,3000
```

Also verify that all processes in the container run as user `1000`:
```
ps

----

PID   USER     TIME  COMMAND
    1 1000      0:00 sleep 1h
   13 1000      0:00 sh
   21 1000      0:00 ps aux
```

You should be able to create files in the EmptyDir `/data` directory, verify this:
```
echo "hello world" > /data/hello.txt
```

Verify the created file belongs to the correct group and user:
```
ls -alh /data/hello.txt

----

-rw-r--r--    1 1000     2000           0 Dec  6 08:24 /data/demo/tmp.txt
```

Note that the file belongs to the fsGroup ID and not the group ID! The volume and any files created in that volume will be owned by fsGroup. All processes also get this supplementary group ID.

Verify that the user is not able to create files anywhere else:
```
echo "hello world" > /home/hello.txt

----

sh: can't create /home/hello.txt: Permission denied
```

## Task 3: Container SecurityContext

Copy the following manifest into the file `lab-12-user-override.yaml` and deploy it to the namespace.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo2
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  containers:
    - name: c1
      image: alpine:3.18.5
      command: ["sh", "-c", "sleep 1h"]
      volumeMounts:
        - name: c1-vol
          mountPath: /data
      securityContext:
        runAsUser: 4000
  volumes:
    - name: c1-vol
      emptyDir: {}
```

```
kubectl -n lab-12 apply -f lab-12-user-override.yaml
```

Once the container has started, execute an interactive shell into the container:
```
kubectl -n lab-12 exec -it security-context-demo2 -- sh
```

Now verify that the container's processes run as user ID `4000` instead of `1000`.
```
ps

----

PID   USER     TIME  COMMAND
    1 4000      0:00 sleep 1h
    7 4000      0:00 sh
   14 4000      0:00 ps
```

The container's Security Context applies only on itself and overrides the Pod's Security Context.

## Task 4: Clean up

Delete the namespace:
```
kubectl delete ns lab-12
```
