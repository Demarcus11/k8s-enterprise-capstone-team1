# Lab 02 Notes - Multi-Container Pods

## Sidecar Usage

According to the official Kubernetes documentation [(Kubernetes | Sidecar Containers)](https://kubernetes.io/docs/concepts/workloads/pods/sidecar-containers/), Sidecar containers are secondary containers that are designed to run alongside the main application within the same Pod. These containers are designed to support, enhance, or extend the functionality of the primary container by providing additional services without altering application code. To illustrate, in the event by which individuals wish to monitor incoming traffic to the main application, a sidecar container could be deployed alongside the main application that shares the same resources as the primary container without being housed within the main application.

## Container Observation

### Containers Starting  

As observed in the kubectl describe logs below, we can see that upon deployment the containers are able to spin up successfully without issue.

```console
PS D:\Source\k8s-enterprise-capstone-team1\week07-08\manifests> kubectl describe pod multi-container-pod
Name:             multi-container-pod
Namespace:        default
Priority:         0
Service Account:  default
Node:             kind-cluster-control-plane/172.18.0.2
Start Time:       Tue, 17 Mar 2026 16:13:26 -0400
Labels:           <none>
Annotations:      <none>
Status:           Running
IP:               10.244.0.11
IPs:
  IP:  10.244.0.11
Containers:
  app:
    Container ID:   containerd://18303ee192a5076aa0063dc140fead747384745dff28cc094186164de8ed07f4
    Image:          nginx
    Image ID:       docker.io/library/nginx@sha256:dec7a90bd0973b076832dc56933fe876bc014929e14b4ec49923951405370112
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Tue, 17 Mar 2026 16:13:27 -0400
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /usr/share/nginx/html from shared-data (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-74jp9 (ro)
  sidecar:
    Container ID:  containerd://2911c412eedc53df5a4dfb69ade2b16ebbb7e9d19a5fe50408db28c5676196af
    Image:         busybox
    Image ID:      docker.io/library/busybox@sha256:b3255e7dfbcd10cb367af0d409747d511aeb66dfac98cf30e97e87e4207dd76f
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      while true; do date >> /data/index.html; sleep 5; done
    State:          Running
      Started:      Tue, 17 Mar 2026 16:13:29 -0400
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /data from shared-data (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-74jp9 (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       True
  ContainersReady             True
  PodScheduled                True
Volumes:
  shared-data:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:
    SizeLimit:  <unset>
  kube-api-access-74jp9:
    SizeLimit:  <unset>
  kube-api-access-74jp9:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
    Optional:                false
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  2m33s  default-scheduler  Successfully assigned default/multi-container-pod to kind-cluster-control-plane
  Normal  Pulling    2m33s  kubelet            Pulling image "nginx"
  Normal  Pulled     2m32s  kubelet            Successfully pulled image "nginx" in 396ms (396ms including waiting). Image size: 62956815 bytes.
  Normal  Created    2m32s  kubelet            Container created
  Normal  Started    2m32s  kubelet            Container started
  Normal  Pulling    2m32s  kubelet            Pulling image "busybox"
  Normal  Pulled     2m30s  kubelet            Successfully pulled image "busybox" in 2.083s (2.083s including waiting). Image size: 2222260 bytes.
  Normal  Created    2m30s  kubelet            Container created
  Normal  Started    2m30s  kubelet            Container started
```

### Containers Restart

In order to force restarts, the sidecar command was updated to:  

```console  
command: ["sh", "-c", "exit 1"]
```

This triggered an instant exit from the sidecar container, triggering consecutive container restarts whilst maintaining running status within the main application, illustrating how the functionality between these two containers is decoupled from one another despite being defined in the same manifest.

![Container Restarts](/week07-08/screenshots/Lab02_Restart_Observation.PNG)

### Containers Termination

In order to force container termination, we first opened a new terminal window and ran the following command:

```console
kubectl get pod multi-container-pod -w
```

In order to observe the status of the containers. Next, we deleted the multi-contianer pod by running the command:

```console
kubectl delete pod multi-container-pod
```

As observed in the screenshot below, the containers terminate together and reach the same ERROR status illustrating how despite the containers are designed to serve two different functions logically, they share the same outer lifecycle.

![Container Termination](/week07-08/screenshots/Lab02_Terminate_Together.PNG)
