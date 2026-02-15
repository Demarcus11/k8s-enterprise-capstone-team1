# Troubleshooting Report

## Issue Summary

crashloop-demo deployment is unable to reach ready state; status remains either Error or CrashLoopBackOff with each subsequent restart.

## Symptoms

Successfully creates and starts the container, then immediately exits with an Error (Exit Code 1). After the initial Error, the pod Status is listed as CrashLoopBackOff, with each subsequent restart causing an Error and reentering the CrashLoopBackOff state.  

## Investigation

### Commands Used

**Command:** kubectl apply -f crashloop.yaml  
**Analysis:** Applies the crashloop deployment.  
**Response:** deployment.apps/crashloop-demo created  

**Command:** kubectl describe pod crashloop-demo-c6dd8b4d4-q4nq2  
**Analysis:** The describe command describes the state of the pod and provides a detailed description of selected resources [(kubectl | describe)](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_describe/). As shown in the response below, the Pod's status is listed as Running, however, the state of the container raises concern. The container state is listed as Terminated, with the reason being Error (Exit Code 1). Furthermore, the Started timestamp and Finished timestamp for the container is the same, indicating an instant termination of the container once it was created. Looking further down in the response toward the Events section, it indicates that the pod was scheduled via the kube-scheduler, the image for the pod was successfully pulled, the container within the pod was created, the container was started, then a Back-Off restarts the failed container application.  
**Response:**  

```console
Name:             crashloop-demo-c6dd8b4d4-q4nq2
Namespace:        default
Priority:         0
Service Account:  default
Node:             kind-control-plane/172.19.0.2
Start Time:       Sat, 14 Feb 2026 13:04:40 -0500
Labels:           app=crashloop-demo
                  pod-template-hash=c6dd8b4d4
Annotations:      <none>
Status:           Running
IP:               10.244.0.5
IPs:
  IP:           10.244.0.5
Controlled By:  ReplicaSet/crashloop-demo-c6dd8b4d4
Containers:
  app:
    Container ID:  containerd://648d1ca2d454b45800975abeae84f7c67ab723aadfd9896d4de63444171cce75
    Image:         nginx
    Image ID:      docker.io/library/nginx@sha256:341bf0f3ce6c5277d6002cf6e1fb0319fa4252add24ab6a0e262e0056d313208
    Port:          <none>
    Host Port:     <none>
    Command:
      /bin/false
    State:          Terminated
      Reason:       Error
      Exit Code:    1
      Started:      Sat, 14 Feb 2026 13:07:25 -0500
      Finished:     Sat, 14 Feb 2026 13:07:25 -0500
    Last State:     Terminated
      Reason:       Error
      Exit Code:    1
      Started:      Sat, 14 Feb 2026 13:06:36 -0500
      Finished:     Sat, 14 Feb 2026 13:06:36 -0500
    Ready:          False
    Restart Count:  4
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-44z7b (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       False
  ContainersReady             False
  PodScheduled                True
Volumes:
  kube-api-access-44z7b:
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
  Type     Reason     Age                  From               Message
  ----     ------     ----                 ----               -------
  Normal   Scheduled  3m52s                default-scheduler  Successfully assigned default/crashloop-demo-c6dd8b4d4-q4nq2 to kind-control-plane  
  Normal   Pulled     2m52s                kubelet            Successfully pulled image "nginx" in 59.376s (59.376s including waiting). Image size:
62939286 bytes.
  Normal   Pulled     2m50s                kubelet            Successfully pulled image "nginx" in 622ms (622ms including waiting). Image size: 62939286 bytes.
  Normal   Pulled     2m29s                kubelet            Successfully pulled image "nginx" in 6.649s (6.649s including waiting). Image size: 62939286 bytes.
  Normal   Pulled     117s                 kubelet            Successfully pulled image "nginx" in 1.139s (1.139s including waiting). Image size: 62939286 bytes.
  Normal   Pulling    69s (x5 over 3m51s)  kubelet            Pulling image "nginx"
  Normal   Pulled     68s                  kubelet            Successfully pulled image "nginx" in 1.575s (1.575s including waiting). Image size: 62939286 bytes.
  Normal   Created    67s (x5 over 2m52s)  kubelet            Container created
  Normal   Started    67s (x5 over 2m51s)  kubelet            Container started
  Warning  BackOff    66s (x5 over 2m49s)  kubelet            Back-off restarting failed container app in pod crashloop-demo-c6dd8b4d4-q4nq2_default(bcebf625-0302-4ff8-8372-1070ebebd352)
```

**Command:** kubectl get events  
**Analysis:** The kubectl events command prints a table of the most important information concerning events within a cluster [(kubectl | events)](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_events/). As observed in the console output below, the pod is successfully scheduled, the image is pulled successfully, the container is created, the container is started, and then a Back-off is initiated restarting the failed container.  
**Response:**  

```console
LAST SEEN   TYPE      REASON                    OBJECT                                MESSAGE
4m33s       Normal    Scheduled                 pod/crashloop-demo-c6dd8b4d4-q4nq2    Successfully assigned default/crashloop-demo-c6dd8b4d4-q4nq2 to kind-control-plane
20s         Normal    Pulling                   pod/crashloop-demo-c6dd8b4d4-q4nq2    Pulling image "nginx"
3m33s       Normal    Pulled                    pod/crashloop-demo-c6dd8b4d4-q4nq2    Successfully pulled image "nginx" in 59.376s (59.376s including waiting). Image size: 62939286 bytes.
19s         Normal    Created                   pod/crashloop-demo-c6dd8b4d4-q4nq2    Container created
19s         Normal    Started                   pod/crashloop-demo-c6dd8b4d4-q4nq2    Container started
3m31s       Normal    Pulled                    pod/crashloop-demo-c6dd8b4d4-q4nq2    Successfully pulled image "nginx" in 622ms (622ms including waiting). Image size: 62939286 bytes.
18s         Warning   BackOff                   pod/crashloop-demo-c6dd8b4d4-q4nq2    Back-off restarting failed container app in pod crashloop-demo-c6dd8b4d4-q4nq2_default(bcebf625-0302-4ff8-8372-1070ebebd352)
```

**Command:** kubectl logs crashloop-demo-c6dd8b4d4-q4nq2  
**Analysis:** The kubectl logs command is utilized in order to print the logs for a container in a pod or a specified resource [(kubectl | logs)](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_logs/). Since the container terminates virtually instantly, there are no logs to be created concerning the container.  
**Response:** No Response  

## Root Cause

The reason the container immediately terminates upon creation is due to the command: ["/bin/false"] within the deployment manifest [crashloop.yaml](/week03-04/manifests/broken-workloads/crashloop.yaml). This command does nothing and exits with a non-zero status code, as such, when the container is instantiated this command is run which causes an immediate exit. This is what causes the CrashLoopBackOff restart pattern, as the kubelet continually tries to restart a container which is immediately terminating.

## Resolution

In order to fix this issue, remove the command: ["/bin/false"] line from the deployment as shown in [crashloop.yaml](/week03-04/manifests/fixed-workloads/crashloop.yaml). Once the new deployment is applied, the command responses will show as follows.

**Command:** kubectl apply -f crashloop.yaml  
**Analysis:** Applies the crashloop deployment.  
**Response:** deployment.apps/crashloop-demo created  

**Command:** kubectl describe pod crashloop-demo-98f65959f-lfpjm
**Analysis:** The describe command describes the state of the pod and provides a detailed description of selected resources [(kubectl | describe)](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_describe/). As opposed to the previous broken workload where the Pod's status was listed as Running but the container state was Terminated, the fixed workload response shows that both the Pod and Container are running without issue. Looking at the container information, observe that there is no Finished timestamp, indicating that the container is still running. Moreover, the conditions values for Ready and ContainersReady now read as True, when previously they were False in the broken workload. Looking at the events section, there are no subsequent logs following the pod being scheduled, the image for the pod being pulled, and the container within the pod being created and started, indicating that both the container and pod are running as expected.
**Response:**

```console
Name:             crashloop-demo-98f65959f-lfpjm   
Namespace:        default
Priority:         0
Service Account:  default
Node:             kind-control-plane/172.19.0.2    
Start Time:       Sat, 14 Feb 2026 14:46:15 -0500  
Labels:           app=crashloop-demo
                  pod-template-hash=98f65959f      
Annotations:      <none>
Status:           Running
IP:               10.244.0.6
IPs:
  IP:           10.244.0.6
Controlled By:  ReplicaSet/crashloop-demo-98f65959f
Containers:
  app:
    Container ID:   containerd://d4aa151d5142940f47449ced3e62d61d562d0b274dc27e6a34c43d232fb4d491
    Image:          nginx
    Image ID:       docker.io/library/nginx@sha256:341bf0f3ce6c5277d6002cf6e1fb0319fa4252add24ab6a0e262e0056d313208
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Sat, 14 Feb 2026 14:46:18 -0500
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-f78x6 (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       True
  ContainersReady             True
  PodScheduled                True
Volumes:
  kube-api-access-f78x6:
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
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  72s   default-scheduler  Successfully assigned default/crashloop-demo-98f65959f-lfpjm to kind-control-plane
  Normal  Pulling    71s   kubelet            Pulling image "nginx"
  Normal  Pulled     71s   kubelet            Successfully pulled image "nginx" in 584ms (584ms including waiting). Image size: 62939286 bytes. 
  Normal  Created    70s   kubelet            Container created
  Normal  Started    70s   kubelet            Container started
```

**Command:** kubectl get events  
**Analysis:** The kubectl events command prints a table of the most important information concerning events within a cluster [(kubectl | events)](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_events/). As opposed to the broken workload where a Back-off was initiated restarting the failed container, the events response now shows that the pod is successfully scheduled, the image is pulled successfully, the container is created, the container is started, and the container is running without errors or Back-off events.  
**Response:**  

```console
LAST SEEN   TYPE     REASON              OBJECT                                MESSAGE
86s         Normal   Scheduled           pod/crashloop-demo-98f65959f-lfpjm    Successfully assigned default/crashloop-demo-98f65959f-lfpjm to kind-control-plane
85s         Normal   Pulling             pod/crashloop-demo-98f65959f-lfpjm    Pulling image "nginx"
85s         Normal   Pulled              pod/crashloop-demo-98f65959f-lfpjm    Successfully pulled image "nginx" in 584ms (584ms including waiting). Image size: 62939286 bytes.
84s         Normal   Created             pod/crashloop-demo-98f65959f-lfpjm    Container created
84s         Normal   Started             pod/crashloop-demo-98f65959f-lfpjm    Container started
87s         Normal   SuccessfulCreate    replicaset/crashloop-demo-98f65959f   Created pod: crashloop-demo-98f65959f-lfpjm
87s         Normal   ScalingReplicaSet   deployment/crashloop-demo             Scaled up replica set crashloop-demo-98f65959f from 0 to 1 
```

**Command:** kubectl logs crashloop-demo-98f65959f-lfpjm  
**Analysis:** The kubectl logs command is utilized in order to print the logs for a container in a pod or a specified resource [(kubectl | logs)](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_logs/). As opposed to the broken workload where the container was terminating instantly, the console response now shows logs concerning the running container.  
**Response:**  

```console
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2026/02/14 19:46:18 [notice] 1#1: using the "epoll" event method
2026/02/14 19:46:18 [notice] 1#1: nginx/1.29.5
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2026/02/14 19:46:18 [notice] 1#1: using the "epoll" event method
2026/02/14 19:46:18 [notice] 1#1: nginx/1.29.5
2026/02/14 19:46:18 [notice] 1#1: built by gcc 14.2.0 (Debian 14.2.0-19)
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2026/02/14 19:46:18 [notice] 1#1: using the "epoll" event method
2026/02/14 19:46:18 [notice] 1#1: nginx/1.29.5
2026/02/14 19:46:18 [notice] 1#1: built by gcc 14.2.0 (Debian 14.2.0-19)
2026/02/14 19:46:18 [notice] 1#1: OS: Linux 6.6.87.2-microsoft-standard-WSL2
2026/02/14 19:46:18 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2026/02/14 19:46:18 [notice] 1#1: start worker processes
2026/02/14 19:46:18 [notice] 1#1: start worker process 33
2026/02/14 19:46:18 [notice] 1#1: start worker process 34
```
