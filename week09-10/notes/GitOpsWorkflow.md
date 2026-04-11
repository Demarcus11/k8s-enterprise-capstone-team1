# The GitOps workflow

1st:    Create a yaml file that will difine the desired state of the cluster
2nd:    Push the yaml file to a git repository
3rd:    ArgoCD will monitor the git repository for changes and automatically apply the changes to the cluster
4th:    If the cluster state drifts from the desired state defined in the yaml file, ArgoCD will automatically revert the changes to maintain the desired state
