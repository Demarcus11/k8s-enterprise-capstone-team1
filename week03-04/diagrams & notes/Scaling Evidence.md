# Scaling Evidence

## Scaling Analysis

As outlined in the official Kubernetes documentation, a HorizontalPodAutoscaler (HPA) automatically updates a workload such as a deployment with the aim of automatically scaling capacity to match demand [(Kubernetes | Horizontal Pod Autoscaling)](https://kubernetes.io/docs/concepts/workloads/autoscaling/horizontal-pod-autoscale/). In Lab03 we were tasked to generate an HPA based on CPU, as demonstrated in [hpa.yaml](/week03-04/manifests/hpa.yaml). In this file, we defined the target application by which we wish to scale [(hpa-resource-deployment.yaml)](/week03-04/manifests/hpa-resource-deployment.yaml), the minimum and maximum replicas, the resources in which we wish to scale by type, and the threshold by which scaling should begin. In this instance, scaling was implemented targeting the CPU resource, specifically by means of average CPU utilization. If the average CPU utilization across the replicas exceeds 50%, the HPA enforces the rule and creates additional replica instances to manage utilization, up to a maximum of 5.

## Scaling Screenshots

**Load Generation:**
![Load Generation](/week03-04/screenshots/Lab03_HPA_Generate_Load.PNG)

**Scaling Behavior:**
![Scaling Behavior](/week03-04/screenshots/Lab03_HPA_Scaling_Evidence.PNG)
