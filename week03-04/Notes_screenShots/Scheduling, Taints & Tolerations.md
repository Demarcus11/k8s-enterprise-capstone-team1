

Tainting Nodes:
- Taints are applied to nodes and allow a node to repel a set of pods.
- Taints are defined by a key, value, and effect. The key and value are used to identify the taint, while the effect determines how the taint will affect pods that do not tolerate it.
- I used "kubectl taint nodes kind-worker dedicated=experimental:NoSchedule" This will taint the node with the NoSchedual tag
![taint](screenshots\displayTaints.png)'
Deploying pods without toleration:
- When a node is tainted, pods that do not have a matching toleration will not be scheduled on that node.
- I created a pod without toleration and it was not scheduled on the tainted node.
![no toleration](screenshots\noToleration.png)
Adding toleration to the pod:
- Tolerations are applied to pods and allow them to be scheduled on nodes with matching taints.
- I added a toleration to the pod that matches the taint on the node, and it was successfully scheduled on the tainted node.
![toleration](screenshots\tolerations.png)
selecter pods:
- Node selectors are a way to constrain pods to only be scheduled on nodes that have specific labels.
- Note: Make sure all constraints are accounted for or it will not run
- I created a pod with a node selector that matches the label on the tainted node, and it was successfully scheduled on that node.
![selector](screenshots\selectorPod.png)