# Troubleshooting Report

## Issue Summary

pending-pod-node-selector workload stuck with status pending.

## Symptoms

Pod stuck with status pending.

## Investigation

Apply workload: kubectl apply -f week03-04/broken-workloads/pending-pod-node-selector.yaml
View pods: kubectl get pods
Status column: Pending

kubectl describe pod pending-pod-node-selector returns:

```
Name:             pending-pod-node-selector-5d6d95bb7d-fh2wd
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
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-68z4f (ro)
Conditions:
  Type           Status
  PodScheduled   False
Volumes:
  kube-api-access-68z4f:
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
  Warning  FailedScheduling  17s   default-scheduler  0/3 nodes are available: 1 node(s) had untolerated taint(s), 2 node(s) didn't match Pod's node affinity/selector. no new claims to deallocate, preemption: 0/3 nodes are available: 3 Preemption is not helpful for scheduling.
```

## Root Cause

In the deployment workload, the deployment is using a nodeSelector which means the workload can only be scheduled to a node with a selector label. None of the nodes have a selector label so the scheduler can't schedule the pod to a node.

The output from kubectl describe pod pending-pod-resource: "0/3 nodes are available: 1 node(s) had untolerated taint(s), 2 node(s) didn't match Pod's node affinity/selector" tells us that 1 node has a taint and 2 nodes didn't match the pod's node selector. The control plane always has a taint so there's always a node with a taint. The pod's selector didn't match any nodes is the reason the scheduler can't schedule the pod to a node.

## Resolution

Remove the node selector or give a node a selector label.

Give a node a selector label: kubectl label nodes capstone-project-worker2 randomSelector=randomValue
