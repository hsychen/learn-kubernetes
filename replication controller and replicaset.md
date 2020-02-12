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

replicaset-definition-1.yml：

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

replicaset-definition-2.yml：

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
    name: myapp
    tier: front-end

spec:
  template:
    metadata:
      name: myaapp-pod
      labels:
        app: myapp
        tier: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx
  replicas: 3

  selector:
    matchLabels:
      app: myapp

```

```bash
$ kubectl create -f replicaset-definition-2.yml
replicaset.apps/myapp-replicaset created

$ kubectl get replicasets
NAME               DESIRED   CURRENT   READY   AGE
myapp-replicaset   3         3         3       20s

$ kubectl describe replicasets
Name:         myapp-replicaset
Namespace:    default
Selector:     app=myapp
Labels:       name=myapp
              tier=front-end
Annotations:  <none>
Replicas:     3 current / 3 desired
Pods Status:  3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=myapp
           tier=front-end
  Containers:
   nginx-container:
    Image:        nginx
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From                   Message
  ----    ------            ----  ----                   -------
  Normal  SuccessfulCreate  53s   replicaset-controller  Created pod: myapp-replicaset-5cqkw
  Normal  SuccessfulCreate  53s   replicaset-controller  Created pod: myapp-replicaset-x48gb
  Normal  SuccessfulCreate  53s   replicaset-controller  Created pod: myapp-replicaset-9mhrh

$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
myapp-replicaset-5cqkw   1/1     Running   0          4m2s
myapp-replicaset-9mhrh   1/1     Running   0          4m2s
myapp-replicaset-x48gb   1/1     Running   0          4m2s

```

測試刪除其中一個 pod，新的 pod 又會再度長回來：

```bash
$ kubectl delete pod myapp-replicaset-5cqkw
pod "myapp-replicaset-5cqkw" deleted

Doraemon@HOST-NB MINGW64 ~/Documents/GitHub/learn-kubernetes (master)
$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
myapp-replicaset-9mhrh   1/1     Running   0          6m53s
myapp-replicaset-d4rq5   1/1     Running   0          15s
myapp-replicaset-x48gb   1/1     Running   0          6m53s

```

如果在 replicaset 外，再獨立建立一個性質相同的 pod，結果是：馬上會被終止：

```bash
$ kubectl create -f pod-definition-1.yml
pod/myapp-pod created

$ kubectl get pod
NAME                     READY   STATUS        RESTARTS   AGE
myapp-pod                0/1     Terminating   0          2s
myapp-replicaset-krmrw   1/1     Running       0          48s
myapp-replicaset-nxpss   1/1     Running       0          48s
myapp-replicaset-szm5t   1/1     Running       0          48s

Doraemon@HOST-NB MINGW64 ~/Documents/GitHub/learn-kubernetes (master)
$ kubectl describe replicaset
Name:         myapp-replicaset
Namespace:    default
Selector:     app=myapp
Labels:       name=myapp
              tier=front-end
Annotations:  <none>
Replicas:     3 current / 3 desired
Pods Status:  3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=myapp
           tier=front-end
  Containers:
   nginx-container:
    Image:        nginx
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age                From                   Message
  ----    ------            ----               ----                   -------
  Normal  SuccessfulCreate  57s                replicaset-controller  Created pod: myapp-replicaset-krmrw
  Normal  SuccessfulCreate  57s                replicaset-controller  Created pod: myapp-replicaset-szm5t
  Normal  SuccessfulCreate  57s                replicaset-controller  Created pod: myapp-replicaset-nxpss
  Normal  SuccessfulDelete  10s (x2 over 25s)  replicaset-controller  Deleted pod: myapp-pod

```

replicaset 建立時，會盡可能地，分散在不同的 node：

```bash
$ kubectl get pod -o wide
NAME                     READY   STATUS    RESTARTS   AGE   IP          NODE          NOMINATED NODE   READINESS GATES
myapp-replicaset-5kkg7   1/1     Running   0          26s   10.32.0.3   k8s-worker1   <none>
 <none>
myapp-replicaset-f7b4h   1/1     Running   0          26s   10.32.0.2   k8s-worker2   <none>
 <none>
myapp-replicaset-kkmhf   1/1     Running   0          26s   10.32.0.2   k8s-worker1   <none>
 <none>
vi
```

修改 replicaset-definition-2.yml 檔案，將 replicas 改為 6

```bash
$ vim replicaset-definition-2.yml

  1 apiVersion: apps/v1
  2 kind: ReplicaSet
  3 metadata:
  4   name: myapp-replicaset
  5   labels:
  6     name: myapp
  7     tier: front-end
  8
  9 spec:
 10   template:
 11     metadata:
 12       name: myaapp-pod
 13       labels:
 14         app: myapp
 15         tier: front-end
 16     spec:
 17       containers:
 18         - name: nginx-container
 19           image: nginx
 20   replicas: 6
 21
 22   selector:
 23     matchLabels:
 24       app: myapp
```

```bash
$ kubectl replace -f replicaset-definition-2.yml
replicaset.apps/myapp-replicaset replaced

$ kubectl get replicasets

NAME               DESIRED   CURRENT   READY   AGE
myapp-replicaset   6         6         6       7m16s

$ kubectl get pod -o wide
NAME                     READY   STATUS    RESTARTS   AGE     IP          NODE          NOMINATED NODE   READINESS GATES
myapp-replicaset-5kkg7   1/1     Running   0          6m20s   10.32.0.3   k8s-worker1   <none>           <none>
myapp-replicaset-f7b4h   1/1     Running   0          6m20s   10.32.0.2   k8s-worker2   <none>           <none>
myapp-replicaset-frgx8   1/1     Running   0          11s     10.32.0.3   k8s-worker2   <none>           <none>
myapp-replicaset-kkmhf   1/1     Running   0          6m20s   10.32.0.2   k8s-worker1   <none>           <none>
myapp-replicaset-nq6lw   1/1     Running   0          11s     10.32.0.4   k8s-worker1   <none>           <none>
myapp-replicaset-vldvt   1/1     Running   0          11s     10.32.0.4   k8s-worker2   <none>           <none>

$ kubectl describe replicasets
Name:         myapp-replicaset
Namespace:    default
Selector:     app=myapp
Labels:       name=myapp
              tier=front-end
Annotations:  <none>
Replicas:     6 current / 6 desired
Pods Status:  6 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=myapp
           tier=front-end
  Containers:
   nginx-container:
    Image:        nginx
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age    From                   Message
  ----    ------            ----   ----                   -------
  Normal  SuccessfulCreate  8m14s  replicaset-controller  Created pod: myapp-replicaset-f7b4h
  Normal  SuccessfulCreate  8m14s  replicaset-controller  Created pod: myapp-replicaset-kkmhf
  Normal  SuccessfulCreate  8m14s  replicaset-controller  Created pod: myapp-replicaset-5kkg7
  Normal  SuccessfulCreate  2m5s   replicaset-controller  Created pod: myapp-replicaset-frgx8
  Normal  SuccessfulCreate  2m5s   replicaset-controller  Created pod: myapp-replicaset-nq6lw
  Normal  SuccessfulCreate  2m5s   replicaset-controller  Created pod: myapp-replicaset-vldvt

```

以 `kubectl scale` 進行 scale：

```bash
$ kubectl scale --replicas=3 -f replicaset-definition-2.yml
replicaset.apps/myapp-replicaset scaled

$ kubectl get pod -o wide
NAME                     READY   STATUS    RESTARTS   AGE   IP          NODE          NOMINATED NODE   READINESS GATES
myapp-replicaset-5kkg7   1/1     Running   0          11m   10.32.0.3   k8s-worker1   <none>
 <none>
myapp-replicaset-f7b4h   1/1     Running   0          11m   10.32.0.2   k8s-worker2   <none>
 <none>
myapp-replicaset-kkmhf   1/1     Running   0          11m   10.32.0.2   k8s-worker1   <none>
 <none>

$ kubectl get replicasets

NAME               DESIRED   CURRENT   READY   AGE
myapp-replicaset   3         3         3       12m

```

如果我們以檔案進行 scale，檔案「並不會」自動更新。

```bash
$ cat replicaset-definition-2.yml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
    name: myapp
    tier: front-end

spec:
  template:
    metadata:
      name: myaapp-pod
      labels:
        app: myapp
        tier: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx
  replicas: 6

  selector:
    matchLabels:
      app: myapp

```

刪除 replicaset：

```bash
$ kubectl delete replicasets myapp-replicaset
replicaset.apps "myapp-replicaset" deleted

$ kubectl get all
NAME                         READY   STATUS        RESTARTS   AGE
pod/myapp-replicaset-f7b4h   0/1     Terminating   0          16m

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   7d14h

```


