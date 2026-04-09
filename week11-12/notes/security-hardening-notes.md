# Security Hardening

- RBAC design

  Role Based Access Control uses the concept of Least Priviledge. By default, users and services get the least amount of permissions needed for their function. The readonly-sa we defined in the dev namespace only gets permissions to "get" and "list". This allows devs to view the environment but not modify or delete resources.

  Components like frontend-sa and backend-sa are given identities (service accounts). If a container is compromised, the attack is limited to the permissions of that service account which prevents the entire cluster from being compromised.

  Then the permissions are binded at the namespace level.

- NetworkPolicies and their intent

  The default deny network policy denies all ingress to the dev namespace, so no communication. Allow rules give exceptions which allows things like the frontend pod to reach the backend pod. Attackers can't exploit internal services due to the paths being blocked.

- Pod security constraints applied

  The Pod Security Standards is applied at the namespace level to harden the environment. This disables `allowPrivilegeEscalation` and makes sure containers don't run as the root user. It also restricts the use of `hostPath` volumes. By labeling the dev namespace with pod-security.kubernetes.io/enforce=restricted, the cluster rejects deployments that don't meet the requirements.

- Remaining known risks / future improvements

  The current NetworkPolicies focus on ingress (incoming traffic). Restricting egress (outgoing traffic) to only trusted external APIs would limit risks.
