The pod initially failed to schedule because the target node was tainted with a NoSchedule taint, which prevents pods without a matching toleration from being placed on the node. Kubernetes correctly rejected the pod since it did not explicitly tolerate the taint, resulting in the pod remaining in a Pending state. 

```console
PS C:\Users\nucle\OneDrive\Desktop\S26_Classwork\Capstone\k8s-enterprise-capstone-team1\week03-04\manifests\broken-workloads> kubectl describe pod
Name:             no-toleration-pod
Namespace:        default
Priority:         0
Service Account:  default
Node:             <none>
Labels:           <none>
Annotations:      <none>
Status:           Pending
IP:
IPs:              <none>
Containers:
  app:
    Image:        nginx
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-x9kz7 (ro)
Conditions:
  Type           Status
  PodScheduled   False
Volumes:
  kube-api-access-x9kz7:
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
  Warning  FailedScheduling  31s   default-scheduler  0/1 nodes are available: 1 node(s) had untolerated taint {env: prod}. preemption: 0/1 nodes are available: 1 Preemption is not helpful for scheduling.

```

After adding a toleration that matched the node’s taint, the scheduler was allowed to place the pod on the node. 

Adding a node selector ensured the pod was scheduled to the intended node. This demonstrates how taints protect nodes from unintended workloads and how tolerations explicitly allow pods to bypass those restrictions when appropriate.