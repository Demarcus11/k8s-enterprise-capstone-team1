# Troubleshooting Report

## Issue Summary

pending-pod-resource workload stuck with status pending.

## Symptoms

Pod stuck with status pending.

## Investigation

Apply workload: kubectl apply -f week03-04/broken-workloads/pending-pod-resource.yaml

kubectl describe pod pending-pod-resource outputs:

```
Name:             pending-pod-resource-5fb86fdcdb-lwg6c
Namespace:        dev
Priority:         0
Service Account:  default
Node:             <none>
Labels:           app=pending-pod-resource
                  pod-template-hash=5fb86fdcdb
Annotations:      <none>
Status:           Pending
IP:
IPs:              <none>
Controlled By:    ReplicaSet/pending-pod-resource-5fb86fdcdb
Containers:
  app:
    Image:      nginx
    Port:       <none>
    Host Port:  <none>
    Requests:
      cpu:        10
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-mqn9q (ro)
Conditions:
  Type           Status
  PodScheduled   False
Volumes:
  kube-api-access-mqn9q:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  19s   default-scheduler  0/3 nodes are available: 1 node(s) had untolerated taint(s), 2 Insufficient cpu. no new claims to deallocate, preemption: 0/3 nodes are available: 3 Preemption is not helpful for scheduling.
```

Shows status: pending, failed to schedule, and 0/3 nodes are available: insufficient cpu.

kubectl get events outputs:

```
LAST SEEN   TYPE      REASON              OBJECT                                       MESSAGE
116s        Warning   FailedScheduling    pod/pending-pod-resource-5fb86fdcdb-lwg6c    0/3 nodes are available: 1 node(s) had untolerated taint(s), 2 Insufficient cpu. no new claims to deallocate, preemption: 0/3 nodes are available: 3 Preemption is not helpful for scheduling.
117s        Normal    SuccessfulCreate    replicaset/pending-pod-resource-5fb86fdcdb   Created pod: pending-pod-resource-5fb86fdcdb-lwg6c
```

Shows the pod being created but failing to schedule.

kubectl logs kubectl logs pending-pod-resource-5fb86fdcdb-lwg6c outputs nothing.

## Root Cause

In the deployment workload, the deployment is requesting 10 CPU cores which way more than any node has available, so the scheduler can't schedule the pod to a node.

The output from kubectl describe pod pending-pod-resource: "0/3 nodes are available: 1 node(s) had untolerated taint(s), 2 Insufficient cpu" tells us that 1 node has a taint and the deployment is requesting insufficeint cpu. The control plane always has a taint so there's always a node with a taint. The insufficent cpu is why the scheduler can't schedule the pod to a node.

## Resolution

Change the requested cpu to a reasonable amount such as 1 or 100m (1/10 of the CPU core).

kubectl describe pod pending-pod-resource outputs:

```
Name:             pending-pod-resource-6c9bdf9f6d-hb2vb
Namespace:        dev
Priority:         0
Service Account:  default
Node:             capstone-project-worker/172.18.0.3
Start Time:       Wed, 11 Feb 2026 11:01:36 -0500
Labels:           app=pending-pod-resource
                  pod-template-hash=6c9bdf9f6d
Annotations:      <none>
Status:           Running
IP:               10.244.1.36
IPs:
  IP:           10.244.1.36
Controlled By:  ReplicaSet/pending-pod-resource-6c9bdf9f6d
Containers:
  app:
    Container ID:   containerd://642455423ecb309b414b899aa43128e63f21de58f4be687551bfa73662a36daa
    Image:          nginx
    Image ID:       docker.io/library/nginx@sha256:341bf0f3ce6c5277d6002cf6e1fb0319fa4252add24ab6a0e262e0056d313208
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Wed, 11 Feb 2026 11:01:38 -0500
    Ready:          True
    Restart Count:  0
    Requests:
      cpu:        1
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-j9pg8 (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       True
  ContainersReady             True
  PodScheduled                True
Volumes:
  kube-api-access-j9pg8:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  6s    default-scheduler  Successfully assigned dev/pending-pod-resource-6c9bdf9f6d-hb2vb to capstone-project-worker
  Normal  Pulling    5s    kubelet            Pulling image "nginx"
  Normal  Pulled     5s    kubelet            Successfully pulled image "nginx" in 476ms (476ms including waiting). Image size: 62939286 bytes.
  Normal  Created    5s    kubelet            Container created
  Normal  Started    4s    kubelet            Container started
```

Shows status: running.

kubectl get events output:

```
LAST SEEN   TYPE      REASON              OBJECT                                       MESSAGE
81s         Normal    Scheduled           pod/pending-pod-resource-6c9bdf9f6d-hb2vb    Successfully assigned dev/pending-pod-resource-6c9bdf9f6d-hb2vb to capstone-project-worker
80s         Normal    Pulling             pod/pending-pod-resource-6c9bdf9f6d-hb2vb    Pulling image "nginx"
80s         Normal    Pulled              pod/pending-pod-resource-6c9bdf9f6d-hb2vb    Successfully pulled image "nginx" in 476ms (476ms including waiting). Image size: 62939286 bytes.
80s         Normal    Created             pod/pending-pod-resource-6c9bdf9f6d-hb2vb    Container created
79s         Normal    Started             pod/pending-pod-resource-6c9bdf9f6d-hb2vb    Container started
```

Shows the pod being scheduled, image pulled, container created and started.

kubectl logs pending-pod-resource-6c9bdf9f6d-hb2vb outputs:

```
2026/02/11 16:01:38 [notice] 1#1: using the "epoll" event method
2026/02/11 16:01:38 [notice] 1#1: nginx/1.29.5
2026/02/11 16:01:38 [notice] 1#1: built by gcc 14.2.0 (Debian 14.2.0-19)
2026/02/11 16:01:38 [notice] 1#1: OS: Linux 6.6.87.2-microsoft-standard-WSL2
2026/02/11 16:01:38 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2026/02/11 16:01:38 [notice] 1#1: start worker processes
2026/02/11 16:01:38 [notice] 1#1: start worker process 33
2026/02/11 16:01:38 [notice] 1#1: start worker process 34
2026/02/11 16:01:38 [notice] 1#1: start worker process 35
2026/02/11 16:01:38 [notice] 1#1: start worker process 36
```

Shows the container is running and printing.
