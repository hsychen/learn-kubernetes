rc-definition.yml：

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: myapp-rc
  labels:
    app: myapp
    type: front-end
spec:
  template:
    metadata:
      name: myaapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx

  replicas: 3
```

建立 Replication Controller：

```bash
$ kubectl create -f rc-definition.yml
replicationcontroller/myapp-rc created

$ kubectl get replicationcontrollers
NAME       DESIRED   CURRENT   READY   AGE
myapp-rc   3         3         3       52s

$ kubectl get pods
NAME             READY   STATUS    RESTARTS   AGE
myapp-rc-lnbjn   1/1     Running   0          90s
myapp-rc-r2dzp   1/1     Running   0          90s
myapp-rc-tqjmn   1/1     Running   0          90s

```

replicaset-definition.yml：

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
    app: myapp
    type: front-end

spec:
  template:
    metadata:
      name: myaapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx

  replicas: 3
  selector:
    matchLabels:
      type: front-end
```

```bash
$ kubectl create -f replicaset-definition.yml
replicaset.apps/myapp-replicaset created

$ kubectl get replicasets
NAME               DESIRED   CURRENT   READY   AGE
myapp-replicaset   3         3         3       3m19s

$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
myapp-replicaset-c88l9   1/1     Running   0          5m1s
myapp-replicaset-gwg2x   1/1     Running   0          5m1s
myapp-replicaset-zmx2g   1/1     Running   0          5m1s

```

如何 scale？

修改 replicaset-definition.yml 檔案，將其中 `replicas: 3` 改成 `replicas: 6`，再執行 `kubectl replace -f`：

```bash
$ kubectl replace -f replicaset-definition.yml
replicaset.apps/myapp-replicaset replaced

$ kubectl get replicasets
NAME               DESIRED   CURRENT   READY   AGE
myapp-replicaset   6         6         6       6m

$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
myapp-replicaset-7qph2   1/1     Running   0          5m36s
myapp-replicaset-ghlvq   1/1     Running   0          6m11s
myapp-replicaset-mdj86   1/1     Running   0          6m11s
myapp-replicaset-p6dx7   1/1     Running   0          6m11s
myapp-replicaset-x6wpp   1/1     Running   0          5m36s
myapp-replicaset-xjkwq   1/1     Running   0          5m36s

```

或者 `kubectl scale --replicas=8` 直接指定副本數

```bash
$ kubectl scale --replicas=8 -f replicaset-definition.yml
replicaset.apps/myapp-replicaset scaled

$ kubectl get replicasets
NAME               DESIRED   CURRENT   READY   AGE
myapp-replicaset   8         8         8       50m

$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
myapp-replicaset-7qph2   1/1     Running   0          49m
myapp-replicaset-f68z4   1/1     Running   0          42m
myapp-replicaset-ghlvq   1/1     Running   0          50m
myapp-replicaset-mdj86   1/1     Running   0          50m
myapp-replicaset-nxk5r   1/1     Running   0          42m
myapp-replicaset-p6dx7   1/1     Running   0          50m
myapp-replicaset-x6wpp   1/1     Running   0          49m
myapp-replicaset-xjkwq   1/1     Running   0          49m

```

這樣用也可以：

```bash
$ kubectl scale --replicas=4 replicaset myapp-replicaset
replicaset.apps/myapp-replicaset scaled

$ kubectl get replicasets
NAME               DESIRED   CURRENT   READY   AGE
myapp-replicaset   4         4         4       52m

$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
myapp-replicaset-ghlvq   1/1     Running   0          52m
myapp-replicaset-mdj86   1/1     Running   0          52m
myapp-replicaset-p6dx7   1/1     Running   0          52m
myapp-replicaset-xjkwq   1/1     Running   0          52m

```

刪除，亦會將 pod 連帶刪除：

```bash
$ kubectl delete replicasets myapp-replicaset
replicaset.apps "myapp-replicaset" deleted

$ kubectl get pods
NAME                     READY   STATUS        RESTARTS   AGE
myapp-replicaset-ghlvq   0/1     Terminating   0          55m
myapp-replicaset-mdj86   0/1     Terminating   0          55m
myapp-replicaset-p6dx7   0/1     Terminating   0          55m
myapp-replicaset-xjkwq   0/1     Terminating   0          55m

$ kubectl get pods
No resources found in default namespace.

```


