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

$ kubectl get deployments
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
myapp-deployment   2/3     3            2           13s

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


