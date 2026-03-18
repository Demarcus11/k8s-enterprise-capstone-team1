# Pipeline Stages

## Validate YAML

This is the stage where `helm lint` is run which checks the syntax of the helm charts and manifests. This prevents syntax errors, missing fields, and templating issues from being deployed the cluster.

## Build (Mock)

This is the stage where the app source code is turned into a container image. We used a mock for the lab, but in the real world you would use `docker build` and `docker push` and a tagged image would be pushed to a container registry such as Docker Hub.

## Deploy to Kubernetes

This is the last stage where the cluster state is updated using `helm upgrade --install`. This command compares the current cluster configs with the new configs. If they are different then a rolling update is applied (new pods created and old pods gradually removed).
