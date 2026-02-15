# Troubleshooting Report

## Issue Summary

pending-pod deployment is unable to reach ready state.

## Symptoms

Pod unable to transition out of Pending state, no restarts and no status transitions imposed by Kubernetes.

## Investigation

### Commands Used

**Command:**  kubectl apply -f pending-pod.yaml  
**Analysis:** Applies the pending pod deployment.  
**Response:** deployment.apps/pending-pod created  

**Command:** kubectl describe pod pending-pod-7bb67bb584-tsjjs  
**Analysis:** The describe command describes the state of the pod and provides a detailed description of selected resources [(kubectl | describe)](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_describe/). As shown in the describe response below, the Pod Status is listed as Pending and the PodScheduled condition is listed as False.  
**Response:**  

```console
Name:             pending-pod-7bb67bb584-tsjjs
Namespace:        default
Priority:         0
Service Account:  default
Node:             <none>
Labels:           app=pending-pod
                  pod-template-hash=7bb67bb584
Annotations:      <none>
Status:           Pending
IP:
IPs:              <none>
Controlled By:    ReplicaSet/pending-pod-7bb67bb584
Containers:
  app:
    Image:      nginx
    Port:       <none>
    Host Port:  <none>
    Requests:
      cpu:        100
      memory:     128Gi
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-r8pj4 (ro)
Conditions:
  Type           Status
  PodScheduled   False
Volumes:
  kube-api-access-r8pj4:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              <none>
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason            Age    From               Message
  ----     ------            ----   ----               -------
  Warning  FailedScheduling  4m32s  default-scheduler  0/1 nodes are available: 1 Insufficient cpu, 1 Insufficient memory. no new claims to 
deallocate, preemption: 0/1 nodes are available: 1 Preemption is not helpful for scheduling.
```

**Command:** kubectl get events  
**Analysis:** The kubectl events command prints a table of the most important information concerning events within a cluster [(kubectl | events)](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_events/). As shown in the event response below, the Pod was created, but was issued a FailedScheduling warning due to insufficient cpu and memory.  
**Response:**  

```console
LAST SEEN   TYPE      REASON              OBJECT                             MESSAGE
6m6s        Warning   FailedScheduling    pod/pending-pod-7bb67bb584-tsjjs   0/1 nodes are available: 1 Insufficient cpu, 1 Insufficient memory. no new claims to deallocate, preemption: 0/1 nodes are available: 1 Preemption is not helpful for scheduling.
46s         Warning   FailedScheduling    pod/pending-pod-7bb67bb584-tsjjs   0/1 nodes are available: 1 Insufficient cpu, 1 Insufficient memory. no new claims to deallocate, preemption: 0/1 nodes are available: 1 Preemption is not helpful for scheduling.
6m6s        Normal    SuccessfulCreate    replicaset/pending-pod-7bb67bb584  Created pod: pending-pod-7bb67bb584-tsjjs
6m6s        Normal    ScalingReplicaSet   deployment/pending-pod             Scaled up replica set pending-pod-7bb67bb584 from 0 to 1
```

**Command:** kubectl logs pending-pod-7bb67bb584-tsjjs  
**Analysis:** The kubectl logs command is utilized in order to print the logs for a container in a pod or a specified resource [(kubectl | logs)](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_logs/). Since the Pod cannot be scheduled due to insufficient memory, no container can be created and mounted to the pod, as such no logs can be collected concerning the container.  
**Response:**  No Response  

## Root Cause

As described in the article, [*8 Reasons Why Kubernetes Pod is in a Pending State*](https://www.howtouselinux.com/post/kubernetes-pod-pending#Check_Resource_Availability) by David Cao, Nodes might lack the necessary CPU or memory resources required by the pod. It is important to compare resource requests and limits to the node's allocatable resources in order to ensure that the Pod will have the required resources in order to be deployed correctly. Looking at the broken manifest [pending-pod.yaml](/week03-04/manifests/broken-workloads/pending-pod.yaml), the CPU request limit is 100 while the Memory request limit is 128Gi, meaning that the resource requires 100 full CPU cores and 128Gi in order to be assigned to a node.

## Resolution

To resolve this issue, adjust the request parameters to a level that can be reasonably accommodated by a cluster node, as demonstrated in the fixed workload [pending-pod.yaml](/week03-04/manifests/fixed-workloads/pending-pod.yaml) manifest, where we used the same numeric values but with smaller units.

**Command:** kubectl apply -f pending-pod.yaml  
**Analysis:** Applies the pending pod deployment.  
**Response:** deployment.apps/pending-pod created  

**Command:** kubectl describe pod pending-pod-b8468f865-qlcqr  
**Analysis:** The describe command describes the state of the pod and provides a detailed description of selected resources [(kubectl | describe)](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_describe/). Unlike the broken workload where the Status was listed as Pending and the PodScheduled condition is False, the Status of the Pod with the proper resource constraints is Running and the Conditions all read True.  
**Response:**  

```console
Name:             pending-pod-b8468f865-qlcqr
Namespace:        default
Priority:         0
Service Account:  default
Node:             kind-control-plane/172.19.0.2
Start Time:       Sat, 14 Feb 2026 18:50:25 -0500
Labels:           app=pending-pod
                  pod-template-hash=b8468f865
Annotations:      <none>
Status:           Running
IP:               10.244.0.11
IPs:
  IP:           10.244.0.11
Controlled By:  ReplicaSet/pending-pod-b8468f865
Containers:
  app:
    Container ID:   containerd://d013fd441719348750e8d16424fc295758fb72a8b7923bc227ea9ab907f1071d
    Image:          nginx
    Image ID:       docker.io/library/nginx@sha256:341bf0f3ce6c5277d6002cf6e1fb0319fa4252add24ab6a0e262e0056d313208
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Sat, 14 Feb 2026 18:50:27 -0500
    Ready:          True
    Restart Count:  0
    Requests:
      cpu:        100m
      memory:     128Mi
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-dhlkp (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       True
  ContainersReady             True
  PodScheduled                True
Volumes:
  kube-api-access-dhlkp:
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
  Normal  Scheduled  21s   default-scheduler  Successfully assigned default/pending-pod-b8468f865-qlcqr to kind-control-plane
  Normal  Pulling    20s   kubelet            Pulling image "nginx"
  Normal  Pulled     20s   kubelet            Successfully pulled image "nginx" in 656ms (656ms including waiting). Image size: 62939286 bytes.
  Normal  Created    20s   kubelet            Container created
  Normal  Started    20s   kubelet            Container started
```

**Command:** kubectl get events  
**Analysis:** The kubectl events command prints a table of the most important information concerning events within a cluster [(kubectl | events)](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_events/). Unlike the previous event response, the Pod was created, scheduled, successfully pulled in an image, created a container, and started a container without issue.  
**Response:**  

```console
LAST SEEN   TYPE      REASON              OBJECT                              MESSAGE
27s         Normal    Scheduled           pod/pending-pod-b8468f865-qlcqr     Successfully assigned default/pending-pod-b8468f865-qlcqr to kind-control-plane
26s         Normal    Pulling             pod/pending-pod-b8468f865-qlcqr     Pulling image "nginx"
26s         Normal    Pulled              pod/pending-pod-b8468f865-qlcqr     Successfully pulled image "nginx" in 656ms (656ms including waiting). Image size: 62939286 bytes.
26s         Normal    Created             pod/pending-pod-b8468f865-qlcqr     Container created
26s         Normal    Started             pod/pending-pod-b8468f865-qlcqr     Container started
28s         Normal    SuccessfulCreate    replicaset/pending-pod-b8468f865    Created pod: pending-pod-b8468f865-qlcqr
45m         Normal    ScalingReplicaSet   deployment/pending-pod              Scaled up replica set pending-pod-7bb67bb584 from 0 to 1      
28s         Normal    ScalingReplicaSet   deployment/pending-pod              Scaled up replica set pending-pod-b8468f865 from 0 to 1
```

**Command:**  kubectl logs pending-pod-b8468f865-qlcqr  
**Analysis:** The kubectl logs command is utilized in order to print the logs for a container in a pod or a specified resource [(kubectl | logs)](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_logs/). Since a container was created inside the pod, unlike the broken workload, the log response shows logs corresponding the running container.  
**Response:**  

```console
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
/docker-entrypoint.sh: Configuration complete; ready for start up
2026/02/14 23:50:28 [notice] 1#1: using the "epoll" event method
2026/02/14 23:50:28 [notice] 1#1: using the "epoll" event method
2026/02/14 23:50:28 [notice] 1#1: nginx/1.29.5
2026/02/14 23:50:28 [notice] 1#1: built by gcc 14.2.0 (Debian 14.2.0-19)
2026/02/14 23:50:28 [notice] 1#1: nginx/1.29.5
2026/02/14 23:50:28 [notice] 1#1: built by gcc 14.2.0 (Debian 14.2.0-19)
2026/02/14 23:50:28 [notice] 1#1: OS: Linux 6.6.87.2-microsoft-standard-WSL2
2026/02/14 23:50:28 [notice] 1#1: OS: Linux 6.6.87.2-microsoft-standard-WSL2
2026/02/14 23:50:28 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2026/02/14 23:50:28 [notice] 1#1: start worker processes
2026/02/14 23:50:28 [notice] 1#1: start worker process 33
2026/02/14 23:50:28 [notice] 1#1: start worker process 34
```
