# Troubleshooting Report

## Issue Summary

pending-pod-taint workload stuck with status pending.

## Symptoms

Pod stuck with status pending.

## Investigation

View all nodes: kubectl get nodes
Taint all nodes: kubectl taint nodes <node name> randomKey=randomValue:NoSchedule
Apply workload: kubectl apply -f week03-04/broken-workloads/pending-pod-taint.yaml
View pods: kubectl get pods
Status column: Pending

kubectl describe pod pending-pod-taint returns:

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
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-vsvfn (ro)
Conditions:
  Type           Status
  PodScheduled   False
Volumes:
  kube-api-access-vsvfn:
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
  Warning  FailedScheduling  28s   default-scheduler  0/3 nodes are available: 3 node(s) had untolerated taint(s). no new claims to deallocate, preemption: 0/3 nodes are available: 3 Preemption is not helpful for scheduling.
```

## Root Cause

All the worker nodes are tainted meaning the scheduler only schedules pod with tolerations for the taint to be scheduled on worker nodes with taints.

The output from kubectl describe pod pending-pod-resource: "0/3 nodes are available: 3 node(s) had untolerated taint(s)." tells us that no nodes are available because they have untolerated taints meaning no pods had tolerations.

## Resolution

Add a toleration to the deployment workload or remove the taint from the worker nodes.

Remove taint: kubectl taint nodes <node name> key=value:<effect>-
