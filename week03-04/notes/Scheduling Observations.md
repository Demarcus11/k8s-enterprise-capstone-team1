# Scheduling Observations

1. Pod tries to run with no toleration with all nodes tainted

   A taint repels pods. If a pod doesn't have a toleration then it can't run on a node with a taint. The pod will remain in status: Pending until there's a available node with no taint.

2. Pod tries to run with a toleration with nodes tainted

   Once pods have a toleration, the pods change from status: Pending to status: Running on the tainted node. The pods with tolerations don't have to run on the tainted node, but they are allowed to.

3. Pod and node with NodeSelector

   NodeSelector attracts pods. By giving a node a label and adding nodeSelector to the pod, the pod will only run on the node with a matching label. If no node exists with the label then the pod will be stuck at status: Pending even if non of the nodes are tainted.
