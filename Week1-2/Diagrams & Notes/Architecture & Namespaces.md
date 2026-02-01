# Architecture & Namespaces

The developer communicates with the cluster by sending manifests (deployment.yaml, service.yaml, and namespaces.yaml) to the control plane’s API server via kubectl. The manifests define the desired state of the cluster. The state is stored in the control plane’s etcd.

The deployment workload (deployment.yaml) defines a pod group. It tells the control plane how many pods to run (replicas), the pod group label, and what images the containers should use. The scheduler assigns the pods to a worker node. The kubelet on the worker node starts the containers. Each pod is assigned a unique, internal IP address. Each pod is given a namespace. Namespaces are a way to organize the pods in a cluster. We set the default namespace to dev, so the pods created will implicitly be put under the dev namespace. We can change the namespace the pods are created under by explicitly writing the namespace in deployment.yaml.

Service.yaml sets up internal routing to a pod group. When the internal IP receives a request, the kube-proxy on the worker nodes uses the selector and namespace to find the correct pod group in the cluster and routes traffic to a pod in the group.
