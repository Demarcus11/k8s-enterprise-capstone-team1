# Resource Behavior

1. Limits and Requests

A request is a guranteed amount of memory and CPU time a container must get each CPU cycle.

A limit is the maximum amount of memory and CPU time a container can get each CPU cycle.

2. OOMKilled Scenario

OOMKilled is a pod status where a container in the pod used more memory than the limit allowed it to have. To generate this scenario, in the deployment workload, a command was added that once the containers are running, it continously allocates more memory exceeding the memory limit. Once the memory limit is reached, the container is killed.

3. Kubernetes response

Kubernetes tries to self-heal and recreate the failed pod. After repeated fails, it turns the status to CrashLoopBackOff which exponentially delays the tries (10s, 20, 40s,...) to avoid wasting CPU cycles on a pod that is broken.

Kubernetes keeps a pod from the old deployment workload running until a pod from the new workload is healthy.
