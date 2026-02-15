# Kubernetes Architecture & Namespaces

## Kubernetes Architecture

The developer communicates with the cluster by sending manifest files (deployment.yaml, service.yaml, and namespaces.yaml) to the control plane’s API server via kubectl. The manifests define the desired state of the cluster. The desired state is stored in the control plane’s etcd.

The deployment workload (deployment.yaml) defines a pod group:

• It tells the control plane how many pods to run (replicas) in the pod group, the pod group label, and what images the containers should use.

• The control plane's scheduler assigns the pods to a worker node. The kubelet on that node starts the containers. Every pod is assigned a unique IP address and is assigned to a namespace.

Service.yaml defines a ClusterIP that is the entry point for a pod group. When the ClusterIP receives a request, the kube-proxy running on the worker nodes load balances and routes the request to a pod in the pod group (using the label selector in service.yaml and namespace defined).

## Namespaces

Namespaces are a way to virtually organize the pods in a cluster. We set the default namespace to dev, so new pods created will implicitly be put under the dev namespace. We can change which namespace new pods are put under by explicitly writing the namespace in deployment.yaml.
