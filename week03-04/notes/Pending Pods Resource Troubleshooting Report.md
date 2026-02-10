# Troubleshooting Report

## Issue Summary

pending-pod-resource workload stuck with status pending.

## Symptoms

Pod stuck with status pending.

## Investigation

Apply workload: kubectl apply -f week03-04/broken-workloads/pending-pod-resource.yaml
View pods: kubectl get pods
Status column: Pending

kubectl describe pod pending-pod-resource returns:

```
Name:             pending-pod-resource-5fb86fdcdb-jslfd
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
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-586l5 (ro)
Conditions:
  Type           Status
  PodScheduled   False
Volumes:
  kube-api-access-586l5:
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
  Type     Reason            Age    From               Message
  ----     ------            ----   ----               -------
  Warning  FailedScheduling  3m49s  default-scheduler  0/3 nodes are available: 1 node(s) had untolerated taint(s), 2 Insufficient cpu. no new claims to deallocate, preemption: 0/3 nodes are available: 3 Preemption is not helpful for scheduling.
```

## Root Cause

In the deployment workload, the deployment is requesting 10 CPU cores which way more than any node has available, so the scheduler can't schedule the pod to a node.

The output from kubectl describe pod pending-pod-resource: "0/3 nodes are available: 1 node(s) had untolerated taint(s), 2 Insufficient cpu" tells us that 1 node has a taint and the deployment is requesting insufficeint cpu. The control plane always has a taint so there's always a node with a taint. The insufficent cpu is why the scheduler can't schedule the pod to a node.

## Resolution

Change the requested cpu to a reasonable amount such as 1 or 100m (1/10 of the CPU core)
