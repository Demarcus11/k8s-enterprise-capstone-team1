# Troubleshooting Report

## Issue Summary

service-selector-mismatch deployment workload can't be accessed because it has no endpoints.

## Symptoms

Can't connect to app when port forwarding after running: kubectl port-forward svc/service-selector-demo-service 8080:80

## Investigation

2 files: Service file and Deployment file

Apply workload and service: kubectl apply -f week03-04/broken-workloads/service-selector-mismatch.yaml and kubectl apply -f week03-04/broken-workloads/service.yaml

kubectl describe pod service-selector-demo outputs:

```
Name:             service-selector-demo-66fc4559b6-nnbcs
Namespace:        dev
Priority:         0
Service Account:  default
Node:             capstone-project-worker2/172.18.0.2
Start Time:       Wed, 11 Feb 2026 11:49:25 -0500
Labels:           app=service-selector-demo
                  pod-template-hash=66fc4559b6
Annotations:      <none>
Status:           Running
IP:               10.244.2.29
IPs:
  IP:           10.244.2.29
Controlled By:  ReplicaSet/service-selector-demo-66fc4559b6
Containers:
  nginx:
    Container ID:   containerd://711d041721cec94499ba1daf4b84fa96ad40f00b6fc8f8e3fd259aa7da128e7b
    Image:          nginx
    Image ID:       docker.io/library/nginx@sha256:341bf0f3ce6c5277d6002cf6e1fb0319fa4252add24ab6a0e262e0056d313208
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Wed, 11 Feb 2026 11:49:26 -0500
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-4frgr (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       True
  ContainersReady             True
  PodScheduled                True
Volumes:
  kube-api-access-4frgr:
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
  Normal  Scheduled  53s   default-scheduler  Successfully assigned dev/service-selector-demo-66fc4559b6-nnbcs to capstone-project-worker2
  Normal  Pulling    53s   kubelet            Pulling image "nginx"
  Normal  Pulled     52s   kubelet            Successfully pulled image "nginx" in 422ms (422ms including waiting). Image size: 62939286 bytes.
  Normal  Created    52s   kubelet            Container created
  Normal  Started    52s   kubelet            Container started
```

Shows the pod is running, was scheduled, image pulled, container created and started.

kubectl get events outputs:

```
LAST SEEN   TYPE      REASON              OBJECT                                                MESSAGE
114s        Normal    Scheduled           pod/service-selector-demo-66fc4559b6-nnbcs            Successfully assigned dev/service-selector-demo-66fc4559b6-nnbcs to capstone-project-worker2
114s        Normal    Pulling             pod/service-selector-demo-66fc4559b6-nnbcs            Pulling image "nginx"
113s        Normal    Pulled              pod/service-selector-demo-66fc4559b6-nnbcs            Successfully pulled image "nginx" in 422ms (422ms including waiting). Image size: 62939286 bytes.
113s        Normal    Created             pod/service-selector-demo-66fc4559b6-nnbcs            Container created
113s        Normal    Started             pod/service-selector-demo-66fc4559b6-nnbcs            Container started
3m28s       Normal    Pulling             pod/service-selector-demo-service-66fc4559b6-m6gtw    Pulling image "nginx"
3m27s       Normal    Pulled              pod/service-selector-demo-service-66fc4559b6-m6gtw    Successfully pulled image "nginx" in 363ms (364ms including waiting). Image size: 62939286 bytes.
3m27s       Normal    Created             pod/service-selector-demo-service-66fc4559b6-m6gtw    Container created
3m27s       Normal    Started             pod/service-selector-demo-service-66fc4559b6-m6gtw    Container started
```

Shows both the service and deployment were scheduled and started.

kubectl logs service-selector-demo-66fc4559b6-nnbcs outputs:

```
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2026/02/11 16:49:26 [notice] 1#1: using the "epoll" event method
2026/02/11 16:49:26 [notice] 1#1: nginx/1.29.5
2026/02/11 16:49:26 [notice] 1#1: built by gcc 14.2.0 (Debian 14.2.0-19)
2026/02/11 16:49:26 [notice] 1#1: OS: Linux 6.6.87.2-microsoft-standard-WSL2
2026/02/11 16:49:26 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2026/02/11 16:49:26 [notice] 1#1: start worker processes
2026/02/11 16:49:26 [notice] 1#1: start worker process 33
2026/02/11 16:49:26 [notice] 1#1: start worker process 34
2026/02/11 16:49:26 [notice] 1#1: start worker process 35
2026/02/11 16:49:26 [notice] 1#1: start worker process 36
```

Shows the container is running and printing.

kubectl get endpoints service-selector-demo-service outputs:

```
NAME                            ENDPOINTS   AGE
service-selector-demo-service   <none>      4m33s
```

This shows the service isn't attached to a deployment, so it has no endpoints to access.

## Root Cause

The service has no endpoints because it couldn't find the deployment workload pods. The selector in the service file must match the label in the deployment file.

## Resolution

Change the selector in the service file to match the label in the deployment file.

kubectl get endpoints service-selector-demo-service outputs:

```
NAME                            ENDPOINTS        AGE
service-selector-demo-service   10.244.2.30:80   6m32s
```

Shows the service is now attached to a deployment and has an endpoint to access.
