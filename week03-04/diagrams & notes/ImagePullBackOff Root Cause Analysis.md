# Troubleshooting Report

## Issue Summary

imagepull-demo deployment is unable to reach ready state; Status transitions between either ErrImagePull or ImagePullBackOff.

## Symptoms

Pod is created with status ErrImagePull. From ErrImagePull, the status continually transitions between ErrImagePull and ImagePullBackOff as kubelet continues to try to pull the container image but fails. The number of restarts remains at zero because the Pod never successfully starts to begin with.

## Investigation

### Commands Used

**Command:** kubectl apply -f imagepull.yaml  
**Analysis:** Applies the imagepull deployment.  
**Response:** deployment.apps/imagepull-demo configured  

**Command:** kubectl describe pod imagepull-demo-76d5646bc9-v589l  
**Analysis:** The describe command describes the state of the pod and provides a detailed description of selected resources [(kubectl | describe)](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_describe/). As shown in the response below, the Pod's status is listed as Pending, indicating that the Pod has yet to run successfully. Moreover, in the Containers section of the response, the Container State is listed as Waiting due to ImagePullBackOff, indicating issues with the deployment's image.  
**Response:**  

```console
Namespace:        default
Priority:         0
Service Account:  default
Node:             kind-control-plane/172.19.0.2     
Start Time:       Sat, 14 Feb 2026 16:49:03 -0500   
Labels:           app=imagepull-demo
                  pod-template-hash=76d5646bc9      
Annotations:      <none>
Status:           Pending
IP:               10.244.0.8
IPs:
  IP:           10.244.0.8
Controlled By:  ReplicaSet/imagepull-demo-76d5646bc9
Containers:
  app:
    Container ID:
    Image:          invalid-image-reference
    Image ID:
    Port:           <none>
    Host Port:      <none>
    State:          Waiting
      Reason:       ImagePullBackOff
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-nrlz2 (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       False
  ContainersReady             False
  PodScheduled                True
Volumes:
  kube-api-access-nrlz2:
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
  Normal   Scheduled  48s                default-scheduler  Successfully assigned default/imagepull-demo-76d5646bc9-v589l to kind-control-plane
  Normal   BackOff    20s (x2 over 46s)  kubelet            Back-off pulling image "invalid-image-reference"
  Warning  Failed     20s (x2 over 46s)  kubelet            Error: ImagePullBackOff
  Normal   Pulling    6s (x3 over 47s)   kubelet            Pulling image "invalid-image-reference"
  Warning  Failed     5s (x3 over 47s)   kubelet            Failed to pull image "invalid-image-reference": failed to pull and unpack image 
"docker.io/library/invalid-image-reference:latest": failed to resolve reference "docker.io/library/invalid-image-reference:latest": pull access denied, repository does not exist or may require authorization: server message: insufficient_scope: authorization failed
  Warning  Failed     5s (x3 over 47s)   kubelet            Error: ErrImagePull
```

**Command:** kubectl get events  
**Analysis:** The kubectl events command prints a table of the most important information concerning events within a cluster [(kubectl | events)](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_events/). As illustrated in the response logs below, immediately after the Pod is scheduled and the image is pulled a warning is issued with the reason being Failed. The error message reads, "Failed to pull image "invalid-image-reference": failed to pull and unpack image "docker.io/library/invalid-image-reference:latest": failed to resolve reference "docker.io/library/invalid-image-reference:latest": pull access denied, repository does not exist or may require authorization: server message: insufficient_scope: authorization failed." This error message indicates that either pull access is denied to the docker image library, the library requires authorization, or the repository doesn't exist.  
**Response:**  

```console
LAST SEEN   TYPE      REASON              OBJECT                                 MESSAGE
53s         Normal    Scheduled           pod/imagepull-demo-76d5646bc9-v589l    Successfully assigned default/imagepull-demo-76d5646bc9-v589l to kind-control-plane
11s         Normal    Pulling             pod/imagepull-demo-76d5646bc9-v589l    Pulling image "invalid-image-reference"
10s         Warning   Failed              pod/imagepull-demo-76d5646bc9-v589l    Failed to pull image "invalid-image-reference": failed to pull and unpack image "docker.io/library/invalid-image-reference:latest": failed to resolve reference "docker.io/library/invalid-image-reference:latest": pull access denied, repository does not exist or may require authorization: server message: insufficient_scope: authorization failed
10s         Warning   Failed              pod/imagepull-demo-76d5646bc9-v589l    Error: ErrImagePull
25s         Normal    BackOff             pod/imagepull-demo-76d5646bc9-v589l    Back-off pulling image "invalid-image-reference"
25s         Warning   Failed              pod/imagepull-demo-76d5646bc9-v589l    Error: ImagePullBackOff
53s         Normal    SuccessfulCreate    replicaset/imagepull-demo-76d5646bc9   Created pod: imagepull-demo-76d5646bc9-v589l
4m37s       Normal    ScalingReplicaSet   deployment/imagepull-demo              Scaled up replica set imagepull-demo-7559986558 from 0 to 153s         Normal    ScalingReplicaSet   deployment/imagepull-demo              Scaled up replica set imagepull-demo-76d5646bc9 from 0 to 1
```

**Command:** kubectl logs imagepull-demo-76d5646bc9-v589l  
**Analysis:** The kubectl logs command is utilized in order to print the logs for a container in a pod or a specified resource [(kubectl | logs)](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_logs/). The response below indicates that the container is currently in waiting status due to the image's inability to be pulled.  
**Response:**  

```console
Error from server (BadRequest): container "app" in pod "imagepull-demo-76d5646bc9-v589l" is waiting to start: image can't be pulled
```  

## Root Cause

According to the article *Fixing Kubernetes Error: ImagePullBackOff or ErrImagePull* published by Sling Academy, ErrorImagePull signifies that the container image could not be pulled for reasons such as incorrect image name, non-existent tag, or authentication issues, while ImagePullBackOff indicates that Kubernetes has failed multiple attempts to pull the container image [(Fixing Kubernetes Error: ImagePullBackOff or ErrImagePull | Sling Academy)](https://www.slingacademy.com/article/fixing-kubernetes-error-imagepullbackoff-errimagepull/#understanding-the-errors). In this case, the reason why the deployment will not work is due to the fact that the container image within the deployment manifest [imagepull.yaml](/week03-04/manifests/broken-workloads/imagepull.yaml) is not a valid image.

## Resolution

In order to fix this issue, replace the image parameter from the deployment with a valid image from [DockerHub](https://hub.docker.com/)as shown in [imagepull.yaml](/week03-04/manifests/fixed-workloads/imagepull.yaml). Once the new deployment is applied, the command responses will show as follows.

**Command:** kubectl apply -f imagepull.yaml  
**Analysis:** Applies the imagepull deployment.  
**Response:** deployment.apps/imagepull-demo configured  

**Command:** kubectl describe pod imagepull-demo-76d5646bc9-v589l  
**Analysis:** The describe command describes the state of the pod and provides a detailed description of selected resources [(kubectl | describe)](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_describe/). Unlike the previous broken workload response where the Pod's status was listed as Pending and the Container State was listed as Waiting due to ImagePullBackOff, the Pod Status is now Running and the Container State now reads True for all parameters, indicating that the Pod is now running successfully and there are no issues concerning the image.  
**Response:**  

```console
Name:             imagepull-demo-c5b6f74-9bh9n
Namespace:        default
Priority:         0
Service Account:  default
Node:             kind-control-plane/172.19.0.2
Start Time:       Sat, 14 Feb 2026 17:36:32 -0500
Labels:           app=imagepull-demo
                  pod-template-hash=c5b6f74
Annotations:      <none>
Status:           Running
IP:               10.244.0.10
IPs:
  IP:           10.244.0.10
Controlled By:  ReplicaSet/imagepull-demo-c5b6f74
Containers:
  app:
    Container ID:   containerd://8a9b00ac278708e771be4ed58b47a95e05e9b2eee79d7d5248f2ffec88f67f6a
    Image:          nginx
    Image ID:       docker.io/library/nginx@sha256:341bf0f3ce6c5277d6002cf6e1fb0319fa4252add24ab6a0e262e0056d313208
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Sat, 14 Feb 2026 17:36:33 -0500
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-wqwc8 (ro)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-wqwc8 (ro)
Conditions:
  Type                        Status
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       True
  ContainersReady             True
  PodScheduled                True
Volumes:
  kube-api-access-wqwc8:
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
  Normal  Scheduled  25s   default-scheduler  Successfully assigned default/imagepull-demo-c5b6f74-9bh9n to kind-control-plane
  Normal  Pulling    26s   kubelet            Pulling image "nginx"
  Normal  Pulled     26s   kubelet            Successfully pulled image "nginx" in 403ms (404ms including waiting). Image size: 62939286 bytes.
  Normal  Created    25s   kubelet            Container created
  Normal  Started    25s   kubelet            Container started
```

**Command:** kubectl get events  
**Analysis:** The kubectl events command prints a table of the most important information concerning events within a cluster [(kubectl | events)](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_events/). As opposed to the broken workload where a warning was issued alongside a message indivating an issue where the image name is incorrect, the tag does not exist, or authentication is invalid, the event response below indicates that the pod was successfully created and scheduled, the image was successfully pulled, and the container was successfully created and started.  
**Response:**  

```console
LAST SEEN   TYPE     REASON      OBJECT                             MESSAGE
6m14s       Normal   Scheduled   pod/imagepull-demo-c5b6f74-9bh9n   Successfully assigned default/imagepull-demo-c5b6f74-9bh9n to kind-control-plane
6m14s       Normal   Pulling     pod/imagepull-demo-c5b6f74-9bh9n   Pulling image "nginx"
6m14s       Normal   Pulled      pod/imagepull-demo-c5b6f74-9bh9n   Successfully pulled image "nginx" in 403ms (404ms including waiting). Image size: 62939286 bytes.
6m13s       Normal   Created     pod/imagepull-demo-c5b6f74-9bh9n   Container created
6m13s       Normal   Started     pod/imagepull-demo-c5b6f74-9bh9n   Container started
```

**Command:** kubectl logs imagepull-demo-c5b6f74-9bh9n  
**Analysis:** The kubectl logs command is utilized in order to print the logs for a container in a pod or a specified resource [(kubectl | logs)](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_logs/). Unlike the previous response which indicated that the container was in waiting status due to the image's inability to be pulled, the command response below provides the corresponding logs for the running Pod.  
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
2026/02/14 22:36:33 [notice] 1#1: using the "epoll" event method
2026/02/14 22:36:33 [notice] 1#1: nginx/1.29.5
2026/02/14 22:36:33 [notice] 1#1: built by gcc 14.2.0 (Debian 14.2.0-19)
2026/02/14 22:36:33 [notice] 1#1: OS: Linux 6.6.87.2-microsoft-standard-WSL2
2026/02/14 22:36:33 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2026/02/14 22:36:33 [notice] 1#1: start worker processes
2026/02/14 22:36:33 [notice] 1#1: start worker process 33
2026/02/14 22:36:33 [notice] 1#1: start worker process 34
```  
