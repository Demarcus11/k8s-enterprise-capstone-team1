# Troubleshooting Report

## Issue Summary

crashloop-demo workload failing to run.

## Symptoms

Pod starts with status Error, transitions to status CrashLoopBackOff, and number of restarts increase.

## Investigation

Apply workload: kubectl apply -f week03-04/broken-workloads/crashloop.yaml

kubectl describe pod crashloop-demo outputs:

```
Name:             crashloop-demo-98f65959f-frjrx
Namespace:        dev
Priority:         0
Service Account:  default
Node:             capstone-project-worker/172.18.0.3
Start Time:       Wed, 11 Feb 2026 09:43:10 -0500
Labels:           app=crashloop-demo
                  pod-template-hash=98f65959f
Annotations:      <none>
Status:           Running
IP:               10.244.1.31
IPs:
  IP:           10.244.1.31
Controlled By:  ReplicaSet/crashloop-demo-98f65959f
Containers:
  app:
    Container ID:   containerd://b3505a18844ec26f9ee71a72833c215ccf7a2ef2a3d4f8eb83bbf7a9036c7204
    Image:          nginx
    Image ID:       docker.io/library/nginx@sha256:341bf0f3ce6c5277d6002cf6e1fb0319fa4252add24ab6a0e262e0056d313208
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Wed, 11 Feb 2026 09:43:12 -0500
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-v75gw (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       True
  ContainersReady             True
  PodScheduled                True
Volumes:
  kube-api-access-v75gw:
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
  Normal  Scheduled  119s  default-scheduler  Successfully assigned dev/crashloop-demo-98f65959f-frjrx to capstone-project-worker
  Normal  Pulling    118s  kubelet            Pulling image "nginx"
  Normal  Pulled     118s  kubelet            Successfully pulled image "nginx" in 700ms (700ms including waiting). Image size: 62939286 bytes.
  Normal  Created    118s  kubelet            Container created
  Normal  Started    117s  kubelet            Container started

Name:             crashloop-demo-c6dd8b4d4-ghsw8
Namespace:        dev
Priority:         0
Service Account:  default
Node:             capstone-project-worker/172.18.0.3
Start Time:       Wed, 11 Feb 2026 09:44:55 -0500
Labels:           app=crashloop-demo
                  pod-template-hash=c6dd8b4d4
Annotations:      <none>
Status:           Running
IP:               10.244.1.32
IPs:
  IP:           10.244.1.32
Controlled By:  ReplicaSet/crashloop-demo-c6dd8b4d4
Containers:
  app:
    Container ID:  containerd://c474273c11ace8acbd9114caf1460b8217262b1c1503856992f5ad7b7d0aadac
    Image:         nginx
    Image ID:      docker.io/library/nginx@sha256:341bf0f3ce6c5277d6002cf6e1fb0319fa4252add24ab6a0e262e0056d313208
    Port:          <none>
    Host Port:     <none>
    Command:
      /bin/false
    State:          Waiting
      Reason:       CrashLoopBackOff
    Last State:     Terminated
      Reason:       Error
      Exit Code:    1
      Started:      Wed, 11 Feb 2026 09:44:58 -0500
      Finished:     Wed, 11 Feb 2026 09:44:58 -0500
    Ready:          False
    Restart Count:  1
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-blqgh (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       False
  ContainersReady             False
  PodScheduled                True
Volumes:
  kube-api-access-blqgh:
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
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  13s                default-scheduler  Successfully assigned dev/crashloop-demo-c6dd8b4d4-ghsw8 to capstone-project-worker
  Normal   Pulling    12s (x2 over 13s)  kubelet            Pulling image "nginx"
  Normal   Pulled     12s                kubelet            Successfully pulled image "nginx" in 779ms (779ms including waiting). Image size: 62939286 bytes.
  Normal   Created    11s (x2 over 12s)  kubelet            Container created
  Normal   Started    11s (x2 over 12s)  kubelet            Container started
  Normal   Pulled     11s                kubelet            Successfully pulled image "nginx" in 348ms (348ms including waiting). Image size: 62939286 bytes.
  Warning  BackOff    10s (x2 over 11s)  kubelet            Back-off restarting failed container app in pod crashloop-demo-c6dd8b4d4-ghsw8_dev(11f69d02-9af4-4904-b5fb-3f4a0bff15ba)
```

kubectl get events outputs:

```
LAST SEEN   TYPE      REASON                         OBJECT                                     MESSAGE
5m47s       Normal    Scheduled                      pod/crashloop-demo-98f65959f-96f2g         Successfully assigned dev/crashloop-demo-98f65959f-96f2g to capstone-project-worker
5m46s       Normal    Pulling                        pod/crashloop-demo-98f65959f-96f2g         Pulling image "nginx"
5m45s       Normal    Pulled                         pod/crashloop-demo-98f65959f-96f2g         Successfully pulled image "nginx" in 901ms (901ms including waiting). Image size: 62939286 bytes.
5m44s       Normal    Created                        pod/crashloop-demo-98f65959f-96f2g         Container created
5m44s       Normal    Started                        pod/crashloop-demo-98f65959f-96f2g         Container started
69s         Normal    Killing                        pod/crashloop-demo-98f65959f-96f2g         Stopping container app
5m48s       Normal    Killing                        pod/crashloop-demo-98f65959f-rnww4         Stopping container app
5m48s       Normal    SuccessfulCreate               replicaset/crashloop-demo-98f65959f        Created pod: crashloop-demo-98f65959f-96f2g
69s         Normal    SuccessfulDelete               replicaset/crashloop-demo-98f65959f        Deleted pod: crashloop-demo-98f65959f-96f2g
112s        Normal    Scheduled                      pod/crashloop-demo-c6dd8b4d4-ldnxh         Successfully assigned dev/crashloop-demo-c6dd8b4d4-ldnxh to capstone-project-worker
18s         Normal    Pulling                        pod/crashloop-demo-c6dd8b4d4-ldnxh         Pulling image "nginx"
111s        Normal    Pulled                         pod/crashloop-demo-c6dd8b4d4-ldnxh         Successfully pulled image "nginx" in 744ms (744ms including waiting). Image size: 62939286 bytes.
15s         Normal    Created                        pod/crashloop-demo-c6dd8b4d4-ldnxh         Container created
15s         Normal    Started                        pod/crashloop-demo-c6dd8b4d4-ldnxh         Container started
109s        Normal    Pulled                         pod/crashloop-demo-c6dd8b4d4-ldnxh         Successfully pulled image "nginx" in 646ms (646ms including waiting). Image size: 62939286 bytes.
14s         Warning   BackOff                        pod/crashloop-demo-c6dd8b4d4-ldnxh         Back-off restarting failed container app in pod crashloop-demo-c6dd8b4d4-ldnxh_dev(23d3d3bc-f24c-49bd-ada3-71fbcea3b325)
95s         Normal    Pulled                         pod/crashloop-demo-c6dd8b4d4-ldnxh         Successfully pulled image "nginx" in 2.558s (2.558s including waiting). Image size: 62939286 bytes.
70s         Normal    Pulled                         pod/crashloop-demo-c6dd8b4d4-ldnxh         Successfully pulled image "nginx" in 595ms (595ms including waiting). Image size: 62939286 bytes.
16s         Normal    Pulled                         pod/crashloop-demo-c6dd8b4d4-ldnxh         Successfully pulled image "nginx" in 1.657s (1.657s including waiting). Image size: 62939286 bytes.
113s        Normal    SuccessfulCreate               replicaset/crashloop-demo-c6dd8b4d4        Created pod: crashloop-demo-c6dd8b4d4-ldnxh
113s        Normal    ScalingReplicaSet              deployment/crashloop-demo                  Scaled up replica set crashloop-demo-c6dd8b4d4 from 0 to 1
69s         Normal    ScalingReplicaSet              deployment/crashloop-demo                  Scaled down replica set crashloop-demo-98f65959f from 1 to 0
```

kubectl logs crashloop-demo outputs nothing.

1. kubectl describe pod crashloop-demo
   This shows us a healthy pod vs the failed pod. The first pod shows state: Running, ready: True, restarts: 0. The second pod shows command:
   /bin/false, state: Waiting (reason: CrashLoopBackOff), last state: terminated (reason: error), exit code 1, started: 09:44:58 -0500
   finished: 09:44:58 -0500 (the container lived for less than a second). In the events section it shows the pod is scheduled, image is pulled, container created, started, then fails.

2. kubectl get events
   Tells us that the pod was scheduled, pulled the image, container created and started. Then the container dies. It tries the entire process again, then outputs "Back-off restarting failed container app in pod crashloop-demo-c6dd8b4d4-ldnxh_dev".

3. kubectl logs crashloop-demo
   Output nothing because the container lived for less than a second.

## Root Cause

In the deployment workload there's a line "command: ["/bin/false"]" that causes the process to immediately exit with an exit code 1, so once the container starts it dies.

## Resolution

Removing the line "command: ["/bin/false"]" in the crashloop.yaml file then reapplying the file using kubectl apply -f week03-04/fixed-workloads/crashloop.yaml.

kubectl describe pod crashloop-demo outputs:

```
Name:             crashloop-demo-98f65959f-2b4pq
Namespace:        dev
Priority:         0
Service Account:  default
Node:             capstone-project-worker/172.18.0.3
Start Time:       Wed, 11 Feb 2026 09:56:07 -0500
Labels:           app=crashloop-demo
                  pod-template-hash=98f65959f
Annotations:      <none>
Status:           Running
IP:               10.244.1.33
IPs:
  IP:           10.244.1.33
Controlled By:  ReplicaSet/crashloop-demo-98f65959f
Containers:
  app:
    Container ID:   containerd://8ec511d526abda6c0a33fa3b2391c19ae7416dfbd04646602be1db87b973c21e
    Image:          nginx
    Image ID:       docker.io/library/nginx@sha256:341bf0f3ce6c5277d6002cf6e1fb0319fa4252add24ab6a0e262e0056d313208
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Wed, 11 Feb 2026 09:56:09 -0500
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-pv5lv (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       True
  ContainersReady             True
  PodScheduled                True
Volumes:
  kube-api-access-pv5lv:
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
  Normal  Scheduled  5s    default-scheduler  Successfully assigned dev/crashloop-demo-98f65959f-2b4pq to capstone-project-worker
  Normal  Pulling    5s    kubelet            Pulling image "nginx"
  Normal  Pulled     4s    kubelet            Successfully pulled image "nginx" in 617ms (617ms including waiting). Image size: 62939286 bytes.
  Normal  Created    4s    kubelet            Container created
  Normal  Started    4s    kubelet            Container started
```

Since its healthy, it shows one pod and with state: Running.

kubecl get events outputs:

```
LAST SEEN   TYPE      REASON                         OBJECT                                     MESSAGE
17m         Normal    Scheduled                      pod/crashloop-demo-c6dd8b4d4-ghsw8         Successfully assigned dev/crashloop-demo-c6dd8b4d4-ghsw8 to capstone-project-worker
6m42s       Normal    Pulling                        pod/crashloop-demo-c6dd8b4d4-ghsw8         Pulling image "nginx"
17m         Normal    Pulled                         pod/crashloop-demo-c6dd8b4d4-ghsw8         Successfully pulled image "nginx" in 779ms (779ms including waiting). Image size: 62939286 bytes.
14m         Normal    Created                        pod/crashloop-demo-c6dd8b4d4-ghsw8         Container created
14m         Normal    Started                        pod/crashloop-demo-c6dd8b4d4-ghsw8         Container started
```

This shows the new workload was scheduled, pulled, created and started.

kubectl logs rashloop-demo-98f65959f-2b4pq -n dev outputs:

```
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2026/02/11 14:56:09 [notice] 1#1: using the "epoll" event method
2026/02/11 14:56:09 [notice] 1#1: nginx/1.29.5
2026/02/11 14:56:09 [notice] 1#1: built by gcc 14.2.0 (Debian 14.2.0-19)
2026/02/11 14:56:09 [notice] 1#1: OS: Linux 6.6.87.2-microsoft-standard-WSL2
2026/02/11 14:56:09 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2026/02/11 14:56:09 [notice] 1#1: start worker processes
2026/02/11 14:56:09 [notice] 1#1: start worker process 33
2026/02/11 14:56:09 [notice] 1#1: start worker process 34
2026/02/11 14:56:09 [notice] 1#1: start worker process 35
2026/02/11 14:56:09 [notice] 1#1: start worker process 36
```

This shows the container started, is running, and printing things.
