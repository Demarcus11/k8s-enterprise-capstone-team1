# Lab 01 Notes

## RBAC & ServiceAccounts | Least Privilege Explanation

### RBAC & ServiceAccounts

As outlined in the official Kubernetes documentation, [(Service Accounts | Kubernetes)](https://kubernetes.io/docs/concepts/security/service-accounts/) ServiceAccounts are a type of non-human account that provides a distinct identity within a Kubernetes cluster. These accounts leverage RBAC (Role Based Access Control) in order to manage resource permissions and ensure that pods associated with the ServiceAccount can only use the authenticated permissions it has been granted.

### Least Privilege Explanation

In the case of the Lab 01 Execution, we first applied [readonly-role.yaml](/week11-12/manifests/readonly-role.yaml), [readonly-rolebinding.yaml](/week11-12/manifests/readonly-rolebinding.yaml), and [serviceaccount.yaml](/week11-12/manifests/serviceaccount.yaml) to define resource access as shown below.

![Apply Role](/week11-12/screenshots/Lab01_Apply_ReadOnly_Role.PNG)
![Apply Binding](/week11-12/screenshots/Lab01_Apply_ReadOnly_RoleBinding.PNG)
![Apply Account](/week11-12/screenshots/Lab01_Apply_ServiceAccount.PNG)
![Resources Running](/week11-12/screenshots/Lab01_Resources_Running.PNG)

Once these have been applied within the dev namespace, the [testing-pod.yaml](/week11-12/manifests/testing-pod.yaml) workload was applied by which we defined a serviceAccountName by which the pod would be associated with. The serviceAccount the testingPod was associated with was readonly-sa, which restricted the pod to only being able to read, but not manage resources as illustrated in the image below.

![Restriction](/week11-12/screenshots/Lab01_Pod_Privilege_And_Restriction.PNG)

This is an example of least privilege, as the pod was not granted unnesscessary permissions and could only access data necessary for its role.
