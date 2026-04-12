# Validation Notes

## Rules
- all required fields exist
- the yaml file i properly formated
- valid api versions
- there is a resourse limit
- the lastest tag is not used
- must have a livenessProbe and readinessProbe
- there should be security rules
- every deployment shou have replicas
- All resoursed need to be labeled

## Lalidation logic explanation

The validation step in the ci pipeline first alerts the user that it is scanning the manifests, then it runs the trivy scaner inside the container and mounts the repo into the container. It then scans the folder of my week 09 - 10 manifests and detects:
- Kubernetes misconfigurations
- Security risks
- Best practice violations
The scan is configured to only report high and critical issues, and if any such issues are found, the pipeline will exit with a non-zero code, which will fail the build and prevent the deployment of the manifests until the issues are resolved. Examples of the flags are:
- Missing resource limits
- Using :latest tag
- Running as root
- Missing probes
- Privileged containers
- Open network access