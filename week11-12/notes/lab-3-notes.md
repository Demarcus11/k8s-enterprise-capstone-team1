# Lab 3 Process and Why This is Safer

## Process
Enabled restricted like security on dev namespace
```
PS C:\Users\Jose Montalvo\Documents\GitHub\k8s-enterprise-capstone-team1> kubectl label namespace dev pod-security.kubernetes.io/enforce=restricted pod-security.kubernetes.io/enforce-version=latest
namespace/dev labeled
```
Create a pod (restricted-pod.yaml comments) that runs as root and uses privileded: true and applyed it recieving this error
```
PS C:\Users\Jose Montalvo\Documents\GitHub\k8s-enterprise-capstone-team1> kubectl apply -f week11-12\manifests\restricted-pod.yaml
Error from server (Forbidden): error when creating "week11-12\\manifests\\restricted-pod.yaml": pods "insecure-pod" is forbidden: violates PodSecurity "restricted:latest": privileged (container "insecure" must not set securityContext.privileged=true), allowPrivilegeEscalation != false (container "insecure" must set securityContext.allowPrivilegeEscalation=false), unrestricted capabilities (container "insecure" must set securityContext.capabilities.drop=["ALL"]), restricted volume types (volume "host-volume" uses restricted volume type "hostPath"), runAsNonRoot != true (pod or container "insecure" must set securityContext.runAsNonRoot=true), seccompProfile (pod or container "insecure" must set securityContext.seccompProfile.type to "RuntimeDefault" or "Localhost")
```
After fixing the pod so it:
- does not run as root
- does not use privileged: true
- sets allowPrivilegeEscalation to false
- drops all capabilities
- does not use hostPath volumes
- set seccompProfile type to RuntimeDefault

We can see that the Pod will now be correctly applied
```
PS C:\Users\Jose Montalvo\Documents\GitHub\k8s-enterprise-capstone-team1> kubectl apply -f week11-12\manifests\restricted-pod.yaml
pod/secure-pod created

PS C:\Users\Jose Montalvo\Documents\GitHub\k8s-enterprise-capstone-team1> kubectl get pods -n dev                                 
NAME                          READY   STATUS    RESTARTS       AGE
backend                       1/1     Running   1 (28m ago)    1d20h
frontend                      1/1     Running   17 (28m ago)   1d20h
secure-pod                    1/1     Running   0              3m5s
test                          1/1     Running   1 (28m ago)    1d17h

```
## Why this is safer
- These Lables enforce=restricted enforce-version=latest turn on automatic security enforcement at the Kubernetes API level instead of reliying on humans.
- These lables enforce security by default stoping the deployment of risky settings
- runAsNonRoot: true is used  to prevent pods running as root sence the Root inside a container can sometimes escalate to the host
- privileged: true is used to prevent pods from accessing host resources and bypassing security controls
- allowPrivilegeEscalation: false is used to prevent pods from gaining more privileges than the parent process
- capabilities.drop=["ALL"] is used to remove all Linux capabilities from the container, reducing the attack surface
- hostPath volumes are restricted because they can give containers access to the host filesystem, which can be a security risk
- seccompProfile.type set to RuntimeDefault is used to apply a default seccomp profile that restricts system calls, enhancing security by limiting the actions a container can perform.



