# Scenarios

1. Pods not showing after applying deployment workload

   Problem: You run kubectl get pods and see "No resources found" but you just applied the deployment workload.

   This is usually a namespace mismatch issue. You are most likely in the default namespace so only pods defined in the default namespace are found. The deployment workload is most likely defined for a different namespace such as dev or prod.

   Solution: Check all namespaces for pods: kubectl get pods -A then switch to the corrent namespace: kubectl config set-context --current --namespace=<name>.

2. Service applied and pods running, but you can't access the pods

   Problem: You apply the serivce.yaml file and deployment.yaml file but you can't access the pods.

   This is usually because the selector in the service.yaml file doesn't exactly match the label in the deployment.yaml file. The Service can only find the pods if the selector and label match.

   Solution: check service.yaml and deployment.yaml to see if selector and label matches.

3. EKS Worker Node creation failed

   Problem: EKS Worker Node creation failed.

   Solution 1: creating worker nodes with t3.micro which caused resource exhaustion. switched to t3.small instances.

   Solution 2: creating EKS cluster with 1 subnet instead of 2 required subnets for EKS CLuster Creation.
