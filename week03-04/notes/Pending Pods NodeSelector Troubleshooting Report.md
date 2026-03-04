# Troubleshooting Report

## Issue Summary

pending-pod-node-selector workload stuck with status pending.

## Symptoms

Pod stuck with status pending.

## Investigation

Apply workload: kubectl apply -f week03-04/broken-workloads/pending-pod-node-selector.yaml

kubectl describe pod pending-pod-node-selector outputs:

```
Name:             pending-pod-node-selector-5d6d95bb7d-s6n9t
Namespace:        dev
Priority:         0
Service Account:  default
Node:             <none>
Labels:           app=pending-pod-node-selector
                  pod-template-hash=5d6d95bb7d
Annotations:      <none>
Status:           Pending
IP:
IPs:              <none>
Controlled By:    ReplicaSet/pending-pod-node-selector-5d6d95bb7d
Containers:
  nginx:
    Image:        nginx
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-zm9f2 (ro)
Conditions:
  Type           Status
  PodScheduled   False
Volumes:
  kube-api-access-zm9f2:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              randomSelector=randomValue
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  18s   default-scheduler  0/3 nodes are available: 1 node(s) had untolerated taint(s), 2 node(s) didn't match Pod's node affinity/selector. no new claims to deallocate, preemption: 0/3 nodes are available: 3 Preemption is not helpful for scheduling.
```

Shows status: pending, pod failed to schedue (2 node(s) didn't match Pod's node affinity/selector).

kubectl get events outputs:

```
LAST SEEN   TYPE      REASON              OBJECT                                            MESSAGE
2m43s       Warning   FailedScheduling    pod/pending-pod-node-selector-5d6d95bb7d-s6n9t    0/3 nodes are available: 1 node(s) had untolerated taint(s), 2 node(s) didn't match Pod's node affinity/selector. no new claims to deallocate, preemption: 0/3 nodes are available: 3 Preemption is not helpful for scheduling.
```

Shows pod failed to schedule.

kubectl logs pending-pod-node-selector outputs nothing because the pod failed to schedule.

## Root Cause

In the deployment workload, the deployment is using a nodeSelector which means the workload can only be scheduled to a node with a selector label. None of the nodes have a selector label so the scheduler can't schedule the pod to a node.

The output from kubectl describe pod pending-pod-resource: "0/3 nodes are available: 1 node(s) had untolerated taint(s), 2 node(s) didn't match Pod's node affinity/selector" tells us that 1 node has a taint and 2 nodes didn't match the pod's node selector. The control plane always has a taint so there's always a node with a taint. The pod's selector didn't match any nodes is the reason the scheduler can't schedule the pod to a node.

## Resolution

Remove the node selector or give a node a selector label.

Show labels on nodes: kubectl get nodes --show-labels
Give a node a selector label: kubectl label nodes <node name> <key>=<value>
Remove labels: kubectl label nodes <node name> <key>-

pending-pod-node-selector-7fc4b55469-8g89l

kubectl describe pod pending-pod-node-selector:

```
Name:             pending-pod-node-selector-5d6d95bb7d-hmgfx
Namespace:        dev
Priority:         0
Service Account:  default
Node:             capstone-project-worker/172.18.0.3
Start Time:       Wed, 11 Feb 2026 11:31:48 -0500
Labels:           app=pending-pod-node-selector
                  pod-template-hash=5d6d95bb7d
Annotations:      <none>
Status:           Running
IP:               10.244.1.38
IPs:
  IP:           10.244.1.38
Controlled By:  ReplicaSet/pending-pod-node-selector-5d6d95bb7d
Containers:
  nginx:
    Container ID:   containerd://c5408a775f31f70a87707a84cd5aa94a9d688093de9105258fae2bda8d5f502b
    Image:          nginx
    Image ID:       docker.io/library/nginx@sha256:341bf0f3ce6c5277d6002cf6e1fb0319fa4252add24ab6a0e262e0056d313208
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Wed, 11 Feb 2026 11:31:49 -0500
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-bgkjm (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       True
  ContainersReady             True
  PodScheduled                True
Volumes:
  kube-api-access-bgkjm:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              randomSelector=randomValue
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  18s   default-scheduler  Successfully assigned dev/pending-pod-node-selector-5d6d95bb7d-hmgfx to capstone-project-worker
  Normal  Pulling    18s   kubelet            Pulling image "nginx"
  Normal  Pulled     17s   kubelet            Successfully pulled image "nginx" in 360ms (360ms including waiting). Image size: 62939286 bytes.
  Normal  Created    17s   kubelet            Container created
  Normal  Started    17s   kubelet            Container started
```

Shows status: running, pod scheduled, container created and started.

kubectl get events outputs:

```
LAST SEEN   TYPE      REASON              OBJECT                                            MESSAGE
97s         Normal    Scheduled           pod/pending-pod-node-selector-5d6d95bb7d-hmgfx    Successfully assigned dev/pending-pod-node-selector-5d6d95bb7d-hmgfx to capstone-project-worker
97s         Normal    Pulling             pod/pending-pod-node-selector-5d6d95bb7d-hmgfx    Pulling image "nginx"
96s         Normal    Pulled              pod/pending-pod-node-selector-5d6d95bb7d-hmgfx    Successfully pulled image "nginx" in 360ms (360ms including waiting). Image size: 62939286 bytes.
96s         Normal    Created             pod/pending-pod-node-selector-5d6d95bb7d-hmgfx    Container created
96s         Normal    Started             pod/pending-pod-node-selector-5d6d95bb7d-hmgfx    Container started
```

Shows the pod scheduled, image pulled, container created and started.

kubectl logs pending-pod-node-selector-5d6d95bb7d-hmgfx:

```
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2026/02/11 16:31:49 [notice] 1#1: using the "epoll" event method
2026/02/11 16:31:49 [notice] 1#1: nginx/1.29.5
2026/02/11 16:31:49 [notice] 1#1: built by gcc 14.2.0 (Debian 14.2.0-19)
2026/02/11 16:31:49 [notice] 1#1: OS: Linux 6.6.87.2-microsoft-standard-WSL2
2026/02/11 16:31:49 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2026/02/11 16:31:49 [notice] 1#1: start worker processes
2026/02/11 16:31:49 [notice] 1#1: start worker process 33
2026/02/11 16:31:49 [notice] 1#1: start worker process 34
2026/02/11 16:31:49 [notice] 1#1: start worker process 35
2026/02/11 16:31:49 [notice] 1#1: start worker process 36
```

Shows the container is running and printing.
