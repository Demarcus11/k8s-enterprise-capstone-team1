# Kubernetes Architecture, Namespace Usage, and Troubleshooting

## Kubernetes Architecture

As outlined in the official Kubernetes documentation a Kubernetes cluster consists of a control plane in addition to worker nodes, where worker nodes host the pods that are components of the application workload, while the control plane manages the worker nodes and the pods in the cluster. The control plane is designed to make decisions about the cluster, and act as a management device for communication, storage, and scheduling [(Cluster Architecture | Kubernetes)](https://kubernetes.io/docs/concepts/architecture/).
The kube-apiserver is a critical component of the Kubernetes control plane that exposes Kubernetes API endpoints to developers, as developers are able to utilize the kubectl command line tool in order to interact with the API server and manage cluster state. Using these endpoints, developers utilize manifest files (deployment.yaml, service.yaml, namespaces.yaml) in order to define the desired state of cluster resources. [(Kubernetes Manifests: Everything You Need to Know)](https://www.vcluster.com/blog/kubernetes-manifest).

The deployment manifest is designed to manage a set of Pods to run an application workload. The deployment manifest defines information including but not limited to the name of the deployment, number of pods to be created by the ReplicaSet, labels for pod management, and pod specification [(Deployments | Kubernetes)](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/).

The service manifest, on the other hand, is a method for exposing a network application that is running across one or more pods within the cluster. By defining a type such as ClusterIP as well as Port, the service is exposed on a cluster-internal IP by which requests can be received and routed to pods within the cluster by a kube-proxy [(Service | Kubernetes)](https://kubernetes.io/docs/concepts/services-networking/service/).

## Namespaces

As described in the official Kubernetes documentation, namespaces are a mechanism for isolating groups of resources within a single cluster [(Namespaces | Kubernetes)](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/). In the instance of our Week01-02 namespaces, we have a development, staging, and production namespace in order to logically separate cluster resources.

## Troubleshooting

No issues encountered whilst performing the Week01-02 lab.
