# Troubleshooting Report

## Issue Summary

image-pull-loop-demo workload failing to run.

## Symptoms

Pod starts with status ErrImagePull, transitions to status ImagePullBackOff, number of restarts don't increase.

## Investigation

Apply workload: kubectl apply -f week03-04/broken-workloads/image-pull-loop.yaml

kubectl describe pod image-pull-loop-demo outputs:

```
Name:             image-pull-loop-demo-7684dd69dd-n7gnf
Namespace:        dev
Priority:         0
Service Account:  default
Node:             capstone-project-worker2/172.18.0.2
Start Time:       Tue, 10 Feb 2026 11:37:07 -0500
Labels:           app=image-pull-loop-demo
                  pod-template-hash=7684dd69dd
Annotations:      <none>
Status:           Running
IP:               10.244.2.18
IPs:
  IP:           10.244.2.18
Controlled By:  ReplicaSet/image-pull-loop-demo-7684dd69dd
Containers:
  app:
    Container ID:   containerd://912c2e6c1839f9d46226c372e08376e591f9483928f3c981516e69c998d63ac6
    Image:          nginx
    Image ID:       docker.io/library/nginx@sha256:341bf0f3ce6c5277d6002cf6e1fb0319fa4252add24ab6a0e262e0056d313208
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Tue, 10 Feb 2026 11:37:09 -0500
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-rb8k2 (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       True
  ContainersReady             True
  PodScheduled                True
Volumes:
  kube-api-access-rb8k2:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:                      <none>

Name:             image-pull-loop-demo-84846c8df4-mlcvv
Namespace:        dev
Priority:         0
Service Account:  default
Node:             capstone-project-worker/172.18.0.3
Start Time:       Wed, 11 Feb 2026 10:21:42 -0500
Labels:           app=image-pull-loop-demo
                  pod-template-hash=84846c8df4
Annotations:      <none>
Status:           Pending
IP:               10.244.1.34
IPs:
  IP:           10.244.1.34
Controlled By:  ReplicaSet/image-pull-loop-demo-84846c8df4
Containers:
  app:
    Container ID:
    Image:          fakeimage
    Image ID:
    Port:           <none>
    Host Port:      <none>
    State:          Waiting
      Reason:       ImagePullBackOff
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-f9hz6 (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       False
  ContainersReady             False
  PodScheduled                True
Volumes:
  kube-api-access-f9hz6:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason     Age                   From               Message
  ----     ------     ----                  ----               -------
  Normal   Scheduled  4m37s                 default-scheduler  Successfully assigned dev/image-pull-loop-demo-84846c8df4-mlcvv to capstone-project-worker
  Normal   Pulling    92s (x5 over 4m37s)   kubelet            Pulling image "fakeimage"
  Warning  Failed     91s (x5 over 4m36s)   kubelet            Failed to pull image "fakeimage": failed to pull and unpack image "docker.io/library/fakeimage:latest": failed to resolve reference "docker.io/library/fakeimage:latest": pull access denied, repository does not exist or may require authorization: server message: insufficient_scope: authorization failed
  Warning  Failed     91s (x5 over 4m36s)   kubelet            Error: ErrImagePull
  Warning  Failed     36s (x15 over 4m35s)  kubelet            Error: ImagePullBackOff
  Normal   BackOff    11s (x17 over 4m35s)  kubelet            Back-off pulling image "fakeimage"
```

Shows a healthy pod vs the failed pod. The healthy pod shows the image, image id, state: running, ready: true. The failed pod shows the image, no image id, state: waiting (reason ImagePullBackOff) and ready: false. No image id is a good indicator that there's an issue with the image being pulled.

kubectl get events outputs:

```
LAST SEEN   TYPE      REASON              OBJECT                                       MESSAGE
5m18s       Normal    Pulling             pod/image-pull-loop-demo-84846c8df4-mlcvv    Pulling image "fakeimage"
5m17s       Warning   Failed              pod/image-pull-loop-demo-84846c8df4-mlcvv    Failed to pull image "fakeimage": failed to pull and unpack image "docker.io/library/fakeimage:latest": failed to resolve reference "docker.io/library/fakeimage:latest": pull access denied, repository does not exist or may require authorization: server message: insufficient_scope: authorization failed
5m17s       Warning   Failed              pod/image-pull-loop-demo-84846c8df4-mlcvv    Error: ErrImagePull
3m8s        Normal    BackOff             pod/image-pull-loop-demo-84846c8df4-mlcvv    Back-off pulling image "fakeimage"
```

This shows the image trying to be pulled then fails leading to BackOff.

kubectl logs image-pull-loop-demo-7684dd69dd-ktmgw outputs nothing because the container can't start.

## Root Cause

In the deployment workload, the image defined isn't an image on DockerHub, so the worker node can't pull it. If the image can't be pulled then the container can't start.

## Resolution

Replacing the line "image: fakeimage" in image-pull-loop.yaml with a real image such as nginx.

kubectl describe pod image-pull-loop-demo outputs:

```
Name:             image-pull-loop-demo-7684dd69dd-n7gnf
Namespace:        dev
Priority:         0
Service Account:  default
Node:             capstone-project-worker2/172.18.0.2
Start Time:       Tue, 10 Feb 2026 11:37:07 -0500
Labels:           app=image-pull-loop-demo
                  pod-template-hash=7684dd69dd
Annotations:      <none>
Status:           Running
IP:               10.244.2.18
IPs:
  IP:           10.244.2.18
Controlled By:  ReplicaSet/image-pull-loop-demo-7684dd69dd
Containers:
  app:
    Container ID:   containerd://912c2e6c1839f9d46226c372e08376e591f9483928f3c981516e69c998d63ac6
    Image:          nginx
    Image ID:       docker.io/library/nginx@sha256:341bf0f3ce6c5277d6002cf6e1fb0319fa4252add24ab6a0e262e0056d313208
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Tue, 10 Feb 2026 11:37:09 -0500
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-rb8k2 (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       True
  ContainersReady             True
  PodScheduled                True
Volumes:
  kube-api-access-rb8k2:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:                      <none>
```

Shows the image, image id, state: running, ready: true.

kubectl get events outputs:

```
1s         Normal    Scheduled           pod/image-pull-loop-demo-7684dd69dd-ktmgw    Successfully assigned dev/image-pull-loop-demo-7684dd69dd-ktmgw to capstone-project-worker2
10s         Normal    Pulling             pod/image-pull-loop-demo-7684dd69dd-ktmgw    Pulling image "nginx"
9s          Normal    Pulled              pod/image-pull-loop-demo-7684dd69dd-ktmgw    Successfully pulled image "nginx" in 1.805s (1.805s including waiting). Image size: 62939286 bytes.
9s          Normal    Created             pod/image-pull-loop-demo-7684dd69dd-ktmgw    Container created
8s          Normal    Started             pod/image-pull-loop-demo-7684dd69dd-ktmgw    Container started
19s         Normal    Killing             pod/image-pull-loop-demo-7684dd69dd-n7gnf    Stopping container app
12s         Normal    SuccessfulCreate    replicaset/image-pull-loop-demo-7684dd69dd   Created pod: image-pull-loop-demo-7684dd69dd-ktmgw
```

Shows the pod being schedule, image pulled, container created and started.

kubectl logs image-pull-loop-demo-7684dd69dd-ktmgw outputs:

```
2026/02/11 15:41:58 [notice] 1#1: using the "epoll" event method
2026/02/11 15:41:58 [notice] 1#1: nginx/1.29.5
2026/02/11 15:41:58 [notice] 1#1: built by gcc 14.2.0 (Debian 14.2.0-19)
2026/02/11 15:41:58 [notice] 1#1: OS: Linux 6.6.87.2-microsoft-standard-WSL2
2026/02/11 15:41:58 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2026/02/11 15:41:58 [notice] 1#1: start worker processes
2026/02/11 15:41:58 [notice] 1#1: start worker process 32
2026/02/11 15:41:58 [notice] 1#1: start worker process 33
2026/02/11 15:41:58 [notice] 1#1: start worker process 34
2026/02/11 15:41:58 [notice] 1#1: start worker process 35
```

This shows the container is running and printing.
