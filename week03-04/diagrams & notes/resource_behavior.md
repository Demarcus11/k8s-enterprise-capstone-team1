a Kubernetes deployment was first created without any CPU or memory requests or limits to observe default resource behavior. Resource usage was monitored using kubectl top.

 which showed that the pod could freely consume available CPU and memory without enforcement. The deployment was then updated to include CPU and memory requests and limits, demonstrating how requests influence pod scheduling while limits enforce runtime constraints. 
 
 Finally, the memory limit was intentionally set too low. which then triggered an OOMKilled event in Kind, confirming that Kubernetes strictly enforces memory limits by terminating containers that exceed them. This lab highlights the importance of properly defining resource requests and limits to ensure stability, predictable performance, and efficient resource utilization in a Kubernetes cluster.

 AWS EKS stops deployments if it will trigger OOMKilled:
 ```console
    PS C:\Users\nucle\OneDrive\Desktop\S26_Classwork\Capstone\k8s-enterprise-capstone-team1\week03-04\manifests> kubectl apply -f wk2_deployment_oom.yaml
    The Deployment "lab01-app" is invalid: spec.template.spec.containers[0].resources.requests: Invalid value: "64Mi": must be less than or equal to memory limit of 32Mi
 ```