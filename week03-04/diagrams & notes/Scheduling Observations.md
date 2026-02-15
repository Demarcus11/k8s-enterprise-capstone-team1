# Scheduling Observations

## Scheduling Analysis

According to the official Kubernetes documentation, a taint is a property applied to a node that allows a node to repel a set of pods. On the other hand, tolerations are properties applied to pods that allow the scheduler to schedule pods with matching taints. NoSchedule restricts the taint and ensures that no new Pods will be scheduled on the tainted node unless they have a matching toleration, however, Pods running on the node prior to the taint being applied will not be evicted [(Kubernetes | Taints and Tolerations)](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/).

As illustrated in the pod deployments in the manifest folder ([no-toleration-pod.yaml](/week03-04/manifests/no-toleration-pod.yaml) and respectively [toleration-pod.yaml](/week03-04/manifests/toleration-pod.yaml)), the pod without a toleration is listed as Pending and will remain Pending until a node without a taint becomes available, on the other hand, the pod with the corresponding toleration is running on the node without issue. This behavior can be seen in the screenshot below. Note that [limit-deployment.yaml](/week03-04/manifests/limit-deployment.yaml) is not affected by the taint, as it was running on the node prior to the taint being applied.

![Pods Running Taint Example](/week03-04/screenshots/Lab02_Toleration_Pod_Running.PNG)

NodeSelector, according to the Kubernetes documentation, is the simplest recommended form of a node selection constraint. The nodeSelector field can be added to a Pod specification where you specify the node labels you want the target node to have. Kubernetes will then only schedule the Pod onto nodes that have each of the labels you specify [(Kubernetes | Assigning Pods to Nodes)](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/). This can be seen when observing [node-selector-pod.yaml](/week03-04/manifests/node-selector-pod.yaml), as well as the following screenshots where the label is added, the pod is deployed, and the pod can be seen running.

![NodeSelector Label Added](/week03-04/screenshots/Lab02_Node_Selector_Apply_Label.PNG)
![NodeSelector Pod Deploy](/week03-04/screenshots/Lab02_Node_Selector_Pod_Deploy.PNG)
![NodeSelector Pod Running](/week03-04/screenshots/Lab02_Node_Selector_Pod_Running.PNG)
