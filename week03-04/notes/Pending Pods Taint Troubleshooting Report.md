# Troubleshooting Report

## Issue Summary

pending-pod-taint workload stuck with status pending.

## Symptoms

Pod stuck with status pending.

## Investigation

View all nodes: kubectl get nodes
Taint all nodes: kubectl taint nodes <node name> <key>:<value>:<effect>
Apply workload: kubectl apply -f week03-04/broken-workloads/pending-pod-taint.yaml

kubectl describe pod pending-pod-taint outputs:

```
Name:             pending-pod-taint
Namespace:        dev
Priority:         0
Service Account:  default
Node:             <none>
Labels:           <none>
Annotations:      <none>
Status:           Pending
IP:
IPs:              <none>
Containers:
  nginx:
    Image:        nginx
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-zttr8 (ro)
Conditions:
  Type           Status
  PodScheduled   False
Volumes:
  kube-api-access-zttr8:
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
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  75s   default-scheduler  0/3 nodes are available: 3 node(s) had untolerated taint(s). no new claims to deallocate, preemption: 0/3 nodes are available: 3 Preemption is not helpful for scheduling.
```

Shows status: pending, failed to schedule and 0/3 nodes are available because 3 ndoes had untolerated taints.

kubectl get events outputs:

```
3m3s        Warning   FailedScheduling    pod/pending-pod-taint                        0/3 nodes are available: 3 node(s) had untolerated taint(s). no new claims to deallocate, preemption: 0/3 nodes are available: 3 Preemption is not helpful for scheduling.
```

Shows the pod failed to schedule.

kubectl logs pending-pod-taint outputs nothing because the pod is never scheduled.

## Root Cause

All the worker nodes are tainted meaning the scheduler only schedules pod with tolerations for the taint to be scheduled on worker nodes with taints.

The output from kubectl describe pod pending-pod-resource: "0/3 nodes are available: 3 node(s) had untolerated taint(s)." tells us that no nodes are available because they have untolerated taints meaning no pods had tolerations.

## Resolution

Add a toleration to the deployment workload or remove the taint from the worker nodes.

Remove taints: kubectl taint nodes <node name> key=value:<effect>-

kubectl describe pending-pod-taint outputs:

```
Name:             pending-pod-taint
Namespace:        dev
Priority:         0
Service Account:  default
Node:             capstone-project-worker/172.18.0.3
Start Time:       Wed, 11 Feb 2026 11:14:43 -0500
Labels:           <none>
Annotations:      <none>
Status:           Running
IP:               10.244.1.37
IPs:
  IP:  10.244.1.37
Containers:
  nginx:
    Container ID:   containerd://97f0675ed42834e2ca1c15ab4fc7e665f0000fc4c2e8f74c3509705c9d4f445c
    Image:          nginx
    Image ID:       docker.io/library/nginx@sha256:341bf0f3ce6c5277d6002cf6e1fb0319fa4252add24ab6a0e262e0056d313208
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Wed, 11 Feb 2026 11:14:44 -0500
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-zttr8 (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       True
  ContainersReady             True
  PodScheduled                True
Volumes:
  kube-api-access-zttr8:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
                             randomKey=randomValue:NoSchedule
Events:
  Type     Reason            Age    From               Message
  ----     ------            ----   ----               -------
  Normal   Scheduled         8s     default-scheduler  Successfully assigned dev/pending-pod-taint to capstone-project-worker
  Normal   Pulling           8s     kubelet            Pulling image "nginx"
  Normal   Pulled            7s     kubelet            Successfully pulled image "nginx" in 485ms (485ms including waiting). Image size: 62939286 bytes.
  Normal   Created           7s     kubelet            Container created
  Normal   Started           7s     kubelet            Container started
```

Shows status: running, pod scheduled, image pulled, container created and started.

kubectl get events:

```
LAST SEEN   TYPE      REASON              OBJECT                                       MESSAGE
96s         Normal    Scheduled           pod/pending-pod-taint                        Successfully assigned dev/pending-pod-taint to capstone-project-worker
96s         Normal    Pulling             pod/pending-pod-taint                        Pulling image "nginx"
95s         Normal    Pulled              pod/pending-pod-taint                        Successfully pulled image "nginx" in 485ms (485ms including waiting). Image size: 62939286 bytes.
95s         Normal    Created             pod/pending-pod-taint                        Container created
95s         Normal    Started             pod/pending-pod-taint                        Container started
```

Shows pod scheduled, image pulled, container created and started.

kubectl logs pending-pod-taint outputs:

```
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2026/02/11 16:14:44 [notice] 1#1: using the "epoll" event method
2026/02/11 16:14:44 [notice] 1#1: nginx/1.29.5
2026/02/11 16:14:44 [notice] 1#1: built by gcc 14.2.0 (Debian 14.2.0-19)
2026/02/11 16:14:44 [notice] 1#1: OS: Linux 6.6.87.2-microsoft-standard-WSL2
2026/02/11 16:14:44 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2026/02/11 16:14:44 [notice] 1#1: start worker processes
2026/02/11 16:14:44 [notice] 1#1: start worker process 33
2026/02/11 16:14:44 [notice] 1#1: start worker process 34
2026/02/11 16:14:44 [notice] 1#1: start worker process 35
2026/02/11 16:14:44 [notice] 1#1: start worker process 36
```

Shows the container is running and printing.
