# Troubleshooting

Issue: EKS Cluster creation failed. creating EKS cluster with 1 subnet instead of 2 required subnets for EKS CLuster Creation.

Issue: EKS Cluster creation failed. Not using proper subnet tags for EKS cluster. added proper tags to subnets.

Issue: EKS Worker Node creation failed. creating worker nodes with t3.micro which caused resource exhaustion, switched to t3.small instances.

Issue: ResourceNotFound error when trying to find cluster. Running "aws configure get region" to verify the region.

Issue: Can't find created pods. Checked default namespace instead of dev namespace. Ran "kubectl config set-context --current --namespace-dev" to make the dev namespace the default.
