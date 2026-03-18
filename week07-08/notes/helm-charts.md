# Helm Charts

## values.yaml

values.yaml is the config file for the helm chart. Without helm, if you wanted to change the number of replicas from 2 to 3 then you would have to find that line in deployment.yaml, change it, thn save. With helm, you only have to change the line in values.yaml. The lines in values.yaml will be injected into templates/deployment.yaml where the brackets are.

This is useful for reusability because you can use the same helm chart for different evironments (dev, staging, prod). You just have to change the values.yaml for each environment rather than creating a new deployment.yaml for each environment.

## Templates vs Raw YAML

Raw YAML is static, so the values are hardcoded. if you wanted to change something, you have to edit the file manually and if you wanted to use the deployment for a different environment, you'd have to create two files. A template is dynamic, so the values aren't hardcoded. The values are injected from a values.yaml file. This allows you to be able to use the same deployment file for different environments, you only need to change the values in the values.yaml file.

## Rollback behavior

When you run helm upgrade or helm install, helm creates a new revision. A revision is a template + values used for the template. When you rollback, it tells Kubernetes to update the resources to match that revision. A rollback creates a new revision.
