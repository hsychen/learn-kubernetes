一開始

```bash
$ kubectl config get-contexts
CURRENT   NAME              CLUSTER      AUTHINFO           NAMESPACE
          minikube          minikube     minikube
*         vagrant-kubeadm   kubernetes   kubernetes-admin

$ kubectl get nodes
NAME          STATUS   ROLES    AGE     VERSION
k8s-master    Ready    master   5d14h   v1.17.2
k8s-worker1   Ready    worker   5d13h   v1.17.2
k8s-worker2   Ready    worker   5d13h   v1.17.2
```

跑一個 nginx 的 pod：

```bash
$ kubectl run nginx --image=nginx
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
deployment.apps/nginx created

$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-6db489d4b7-5rcx7   1/1     Running   0          2m5s
```

```bash
$ kubectl describe pods
Name:         nginx-6db489d4b7-5rcx7
Namespace:    default
Priority:     0
Node:         k8s-worker2/10.0.2.15
Start Time:   Mon, 10 Feb 2020 12:10:55 +0800
Labels:       pod-template-hash=6db489d4b7
              run=nginx
Annotations:  <none>
Status:       Running
IP:           10.32.0.2
IPs:
  IP:           10.32.0.2
Controlled By:  ReplicaSet/nginx-6db489d4b7
Containers:
  nginx:
    Container ID:   docker://1331d2aeab2d262f149fe8d1de3bfd4fd620f568b2191ccc336dffc5674aa594
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:ad5552c786f128e389a0263104ae39f3d3c7895579d45ae716f528185b36bc6f
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Mon, 10 Feb 2020 12:11:15 +0800
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-xrt88 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-xrt88:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-xrt88
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age        From                  Message
  ----    ------     ----       ----                  -------
  Normal  Scheduled  <unknown>  default-scheduler     Successfully assigned default/nginx-6db489d4b7-5rcx7 to k8s-worker2
  Normal  Pulling    2m43s      kubelet, k8s-worker2  Pulling image "nginx"
  Normal  Pulled     2m25s      kubelet, k8s-worker2  Successfully pulled image "nginx"
  Normal  Created    2m24s      kubelet, k8s-worker2  Created container nginx
  Normal  Started    2m24s      kubelet, k8s-worker2  Started container nginx
```

```bash
$ kubectl get pods -o wide
NAME                     READY   STATUS    RESTARTS   AGE     IP          NODE          NOMINATED NODE   READINESS GATES
nginx-6db489d4b7-5rcx7   1/1     Running   0          5m38s   10.32.0.2   k8s-worker2   <none>           <none>
```

環境清空：

```bash
$ kubectl delete deployments nginx
deployment.apps "nginx" deleted

$ kubectl get pods
No resources found in default namespace.

```

pod-definition.yml：

```yaml
apiVersion: v1

kind: Pod

metadata:
  name: mypod-app
  labels:
    app: myapp

spec:
  containers:
    - name: nginx-container
      image: nginx


```

由 pod definition 建立 pod：

```bash
$ kubectl create -f pod-bash: _filedir: command not found
definition.yml

$ kubectl create -f pod-definition.yml
pod/mypod-app created

$ kubectl get pod
NAME        READY   STATUS    RESTARTS   AGE
mypod-app   1/1     Running   0          26s

$ kubectl delete pod mypod-app
pod "mypod-app" deleted

```


