# GitOps Workflow

1. Write YAML file. This is the desired state of the cluster.

2. Push YAML file to a folder on GitHub.

3. ArgoCD will notice that the current cluster state doesn't match the desired state from GitHub and update the cluster automatically.

The application.yaml file is ArgoCD's configs.

- The targetRevision field tells ArgoCD which branch in the GitHub repo holds the cluster configs to use as the source of truth.

- The path is which folder holds the cluster configs in the branch.

- Prune tells ArgoCD to delete a file from the cluster if its deleted from GitHub.

- Self-Heal tells ArgoCD to change the cluster to match the cluster configs in GitHub.
