基本上，deployment 的 yaml 跟 replicaset 的非常像，初步只要複製 replicaset 的 yaml 檔後，將 kind 改為 Deployment 後，metadata 底下的 name 跟著改為適當的名稱即可。

deployment-definition.yml：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
  labels:
    app: myapp
    tier: front-end

spec:
  template:
    metadata:
      name: myapp-pod
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
      tier: front-end
```

```bash
$ kubectl create -f deployment-definition.yml
deployment.apps/myapp-deployment created

$ kubectl get replicasets
NAME                          DESIRED   CURRENT   READY   AGE
myapp-deployment-658d769dcd   3         3         3       26s

$ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
myapp-deployment-658d769dcd-f2vf2   1/1     Running   0          37s
myapp-deployment-658d769dcd-f9pxs   1/1     Running   0          37s
myapp-deployment-658d769dcd-ngdpq   1/1     Running   0          37s

$ kubectl describe deployments
Name:                   myapp-deployment
Namespace:              default
CreationTimestamp:      Wed, 12 Feb 2020 22:45:24 +0800
Labels:                 app=myapp
                        tier=front-end
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               tier=front-end
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
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
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   myapp-deployment-658d769dcd (3/3 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  71s   deployment-controller  Scaled up replica set myapp-deployment-658d769dcd to 3
```

一次顯示所有資源：

```bash
$ kubectl get all
NAME                                    READY   STATUS    RESTARTS   AGE
pod/myapp-deployment-658d769dcd-f2vf2   1/1     Running   0          2m4s
pod/myapp-deployment-658d769dcd-f9pxs   1/1     Running   0          2m4s
pod/myapp-deployment-658d769dcd-ngdpq   1/1     Running   0          2m4s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   7d13h

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/myapp-deployment   3/3     3            3           2m4s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/myapp-deployment-658d769dcd   3         3         3       2m4s
```

編輯 deployment-definition.yml 檔案，將 nginx image 版本更新為 1.7.1

```bash
$ cat deployment-definition.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
  labels:
    app: myapp
    tier: front-end

spec:
  template:
    metadata:
      name: myapp-pod
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
      tier: front-end

```

再下 `kubectl apply`：

```bash
$ kubectl apply -f deployment-definition.yml
Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
deployment.apps/myapp-deployment configured

Doraemon@HOST-NB MINGW64 ~/Documents/GitHub/learn-kubernetes (master)
$ kubectl rollout status deployment/myapp-deployment
Waiting for deployment "myapp-deployment" rollout to finish: 1 out of 3 new replicas have been updated...
Waiting for deployment "myapp-deployment" rollout to finish: 1 out of 3 new replicas have been updated...
Waiting for deployment "myapp-deployment" rollout to finish: 1 out of 3 new replicas have been updated...
Waiting for deployment "myapp-deployment" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "myapp-deployment" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "myapp-deployment" rollout to finish: 2 old replicas are pending termination...
Waiting for deployment "myapp-deployment" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "myapp-deployment" rollout to finish: 1 old replicas are pending termination...
deployment "myapp-deployment" successfully rolled out

$ kubectl rollout history deployment/myapp-deployment
deployment.apps/myapp-deployment
REVISION  CHANGE-CAUSE
1         <none>
2         <none>


```

---

```bash
$ kubectl create -f deployment-definition.yml --record
deployment.apps/myapp-deployment created

$ kubectl rollout status deployment/myapp-deployment
deployment "myapp-deployment" successfully rolled out

$ kubectl rollout history deployment/myapp-deployment
deployment.apps/myapp-deployment
REVISION  CHANGE-CAUSE
1         kubectl.exe create --filename=deployment-definition.yml --record=true


```

```bash
$ cat deployment-definition.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
  labels:
    app: myapp
    tier: front-end

spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        tier: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx:1.12

  replicas: 6

  selector:
    matchLabels:
      tier: front-end

```

```bash
$ kubectl apply -f deployment-definition.yml
Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
deployment.apps/myapp-deployment configured

$ kubectl rollout status deployment/myapp-deployment
Waiting for deployment "myapp-deployment" rollout to finish: 3 out of 6 new replicas have been updated...
Waiting for deployment "myapp-deployment" rollout to finish: 3 out of 6 new replicas have been updated...
Waiting for deployment "myapp-deployment" rollout to finish: 3 out of 6 new replicas have been updated...
Waiting for deployment "myapp-deployment" rollout to finish: 3 out of 6 new replicas have been updated...
Waiting for deployment "myapp-deployment" rollout to finish: 3 out of 6 new replicas have been updated...
Waiting for deployment "myapp-deployment" rollout to finish: 4 out of 6 new replicas have been updated...
Waiting for deployment "myapp-deployment" rollout to finish: 4 out of 6 new replicas have been updated...
Waiting for deployment "myapp-deployment" rollout to finish: 5 out of 6 new replicas have been updated...
Waiting for deployment "myapp-deployment" rollout to finish: 5 out of 6 new replicas have been updated...
Waiting for deployment "myapp-deployment" rollout to finish: 5 out of 6 new replicas have been updated...
Waiting for deployment "myapp-deployment" rollout to finish: 5 out of 6 new replicas have been updated...
Waiting for deployment "myapp-deployment" rollout to finish: 5 out of 6 new replicas have been updated...
Waiting for deployment "myapp-deployment" rollout to finish: 5 out of 6 new replicas have been updated...
Waiting for deployment "myapp-deployment" rollout to finish: 2 old replicas are pending termination...
Waiting for deployment "myapp-deployment" rollout to finish: 2 old replicas are pending termination...
Waiting for deployment "myapp-deployment" rollout to finish: 2 old replicas are pending termination...
Waiting for deployment "myapp-deployment" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "myapp-deployment" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "myapp-deployment" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "myapp-deployment" rollout to finish: 5 of 6 updated replicas are available...
deployment "myapp-deployment" successfully rolled out

```

```bash
$ kubectl get deployments.apps
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
myapp-deployment   6/6     6            6           8m25s

$ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
myapp-deployment-6558bdcb4d-2gqgp   1/1     Running   0          2m7s
myapp-deployment-6558bdcb4d-c8m7v   1/1     Running   0          2m59s
myapp-deployment-6558bdcb4d-rf9cz   1/1     Running   0          2m23s
myapp-deployment-6558bdcb4d-tltgx   1/1     Running   0          2m59s
myapp-deployment-6558bdcb4d-vv56f   1/1     Running   0          2m7s
myapp-deployment-6558bdcb4d-xgnxt   1/1     Running   0          2m59s

$ kubectl describe deployments.apps
Name:                   myapp-deployment
Namespace:              default
CreationTimestamp:      Thu, 13 Feb 2020 11:49:34 +0800
Labels:                 app=myapp
                        tier=front-end
Annotations:            deployment.kubernetes.io/revision: 2
                        kubectl.kubernetes.io/last-applied-configuration:
                          {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"labels":{"app":"myapp","tier":"front-end"},"name":"myapp-deploym...
                        kubernetes.io/change-cause: kubectl.exe create --filename=deployment-definition.yml --record=true
Selector:               tier=front-end
Replicas:               6 desired | 6 updated | 6 total | 6 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=myapp
           tier=front-end
  Containers:
   nginx-container:
    Image:        nginx:1.12
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   myapp-deployment-6558bdcb4d (6/6 replicas created)
Events:
  Type    Reason             Age                From                   Message
  ----    ------             ----               ----                   -------
  Normal  ScalingReplicaSet  27m                deployment-controller  Scaled up replica set myapp-deployment-658d769dcd to 3
  Normal  ScalingReplicaSet  21m                deployment-controller  Scaled up replica set myapp-deployment-658d769dcd to 6
  Normal  ScalingReplicaSet  21m                deployment-controller  Scaled up replica set myapp-deployment-6558bdcb4d to 2
  Normal  ScalingReplicaSet  21m                deployment-controller  Scaled down replica set myapp-deployment-658d769dcd to 5
  Normal  ScalingReplicaSet  21m                deployment-controller  Scaled up replica set myapp-deployment-6558bdcb4d to 3
  Normal  ScalingReplicaSet  21m                deployment-controller  Scaled down replica set myapp-deployment-658d769dcd to 4
  Normal  ScalingReplicaSet  21m                deployment-controller  Scaled up replica set myapp-deployment-6558bdcb4d to 4
  Normal  ScalingReplicaSet  21m                deployment-controller  Scaled down replica set myapp-deployment-658d769dcd to 3
  Normal  ScalingReplicaSet  21m                deployment-controller  Scaled up replica set myapp-deployment-6558bdcb4d to 5
  Normal  ScalingReplicaSet  20m (x4 over 21m)  deployment-controller  (combined from similar events): Scaled down replica set myapp-deployment-658d769dcd to 0

$ kubectl rollout history deployment myapp-deployment
deployment.apps/myapp-deployment
REVISION  CHANGE-CAUSE
1         kubectl.exe create --filename=deployment-definition.yml --record=true
2         kubectl.exe create --filename=deployment-definition.yml --record=true


```

```bash
$ kubectl set image deployment/myapp-deployment nginx-container=nginx:1.12-perl
deployment.apps/myapp-deployment image updated

$ kubectl rollout status deployment myapp-deployment
deployment "myapp-deployment" successfully rolled out

Doraemon@HOST-NB MINGW64 ~/Documents/GitHub/learn-kubernetes (master)
$ kubectl rollout history deployment myapp-deployment
deployment.apps/myapp-deployment
REVISION  CHANGE-CAUSE
1         kubectl.exe create --filename=deployment-definition.yml --record=true
2         kubectl.exe create --filename=deployment-definition.yml --record=true
3         kubectl.exe create --filename=deployment-definition.yml --record=true

$ kubectl describe deployments.apps
Name:                   myapp-deployment
Namespace:              default
CreationTimestamp:      Thu, 13 Feb 2020 11:49:34 +0800
Labels:                 app=myapp
                        tier=front-end
Annotations:            deployment.kubernetes.io/revision: 3
                        kubectl.kubernetes.io/last-applied-configuration:
                          {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"labels":{"app":"myapp","tier":"front-end"},"name":"myapp-deploym...
                        kubernetes.io/change-cause: kubectl.exe create --filename=deployment-definition.yml --record=true
Selector:               tier=front-end
Replicas:               6 desired | 6 updated | 6 total | 6 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=myapp
           tier=front-end
  Containers:
   nginx-container:
    Image:        nginx:1.12-perl
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   myapp-deployment-7f96bb849d (6/6 replicas created)
Events:
  Type    Reason             Age                 From                   Message
  ----    ------             ----                ----                   -------
  Normal  ScalingReplicaSet  35m                 deployment-controller  Scaled up replica set myapp-deployment-658d769dcd to 3
  Normal  ScalingReplicaSet  29m                 deployment-controller  Scaled up replica set myapp-deployment-658d769dcd to 6
  Normal  ScalingReplicaSet  29m                 deployment-controller  Scaled up replica set myapp-deployment-6558bdcb4d to 2
  Normal  ScalingReplicaSet  29m                 deployment-controller  Scaled down replica set myapp-deployment-658d769dcd to 5
  Normal  ScalingReplicaSet  29m                 deployment-controller  Scaled up replica set myapp-deployment-6558bdcb4d to 3
  Normal  ScalingReplicaSet  29m                 deployment-controller  Scaled down replica set myapp-deployment-658d769dcd to 4
  Normal  ScalingReplicaSet  29m                 deployment-controller  Scaled up replica set myapp-deployment-6558bdcb4d to 4
  Normal  ScalingReplicaSet  28m                 deployment-controller  Scaled down replica set myapp-deployment-658d769dcd to 3
  Normal  ScalingReplicaSet  28m                 deployment-controller  Scaled up replica set myapp-deployment-6558bdcb4d to 5
  Normal  ScalingReplicaSet  20m (x15 over 28m)  deployment-controller  (combined from similar events): Scaled down replica set myapp-deployment-6558bdcb4d to 0

$ kubectl rollout undo deployment myapp-deployment
deployment.apps/myapp-deployment rolled back

$ kubectl rollout status deployment myapp-deployment
deployment "myapp-deployment" successfully rolled out

$ kubectl rollout history deployment/myapp-deployment
deployment.apps/myapp-deployment
REVISION  CHANGE-CAUSE
1         kubectl.exe create --filename=deployment-definition.yml --record=true
3         kubectl.exe create --filename=deployment-definition.yml --record=true
4         kubectl.exe create --filename=deployment-definition.yml --record=true


```

```bash
$ kubectl apply -f deployment-definition.yml --record
deployment.apps/myapp-deployment configured

$ kubectl rollout history deployment myapp-deployment
deployment.apps/myapp-deployment
REVISION  CHANGE-CAUSE
1         kubectl.exe create --filename=deployment-definition.yml --record=true
3         kubectl.exe create --filename=deployment-definition.yml --record=true
4         kubectl.exe create --filename=deployment-definition.yml --record=true
5         kubectl.exe apply --filename=deployment-definition.yml --record=true

$ kubectl get deployments.apps
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
myapp-deployment   5/6     3            5           27m

$ kubectl get pods
NAME                                READY   STATUS             RESTARTS   AGE
myapp-deployment-6558bdcb4d-dnshc   1/1     Running            0          7m36s
myapp-deployment-6558bdcb4d-f8pwr   1/1     Running            0          7m36s
myapp-deployment-6558bdcb4d-lnqnc   1/1     Running            0          7m30s
myapp-deployment-6558bdcb4d-mntqg   1/1     Running            0          7m36s
myapp-deployment-6558bdcb4d-w9wpc   1/1     Running            0          7m29s
myapp-deployment-65785b9b68-9pg2p   0/1     ImagePullBackOff   0          3m14s
myapp-deployment-65785b9b68-9r7wj   0/1     ImagePullBackOff   0          3m14s
myapp-deployment-65785b9b68-gddm2   0/1     ImagePullBackOff   0          3m14s

$ kubectl rollout undo deployment myapp-deployment
deployment.apps/myapp-deployment rolled back


```


