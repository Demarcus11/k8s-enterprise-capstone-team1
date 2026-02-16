

Tainting Nodes:
- Taints are applied to nodes and allow a node to repel a set of pods.
- Taints are defined by a key, value, and effect. The key and value are used to identify the taint, while the effect determines how the taint will affect pods that do not tolerate it.
- I used "kubectl taint nodes kind-worker dedicated=experimental:NoSchedule" This will taint the node with the NoSchedual tag
![taint](screenshots\displayTaints.png)