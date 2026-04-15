# Lab 03 Notes

## Pod Security Standards and Safety Implications | Non-Compliant v. Compliant Exhibits

### Pod Security Standards

As described in the article, [Kubernetes Pod Security Standards: Profiles, Enforcement & Best Practices](https://www.groundcover.com/learn/security/pod-security-standards) published by groundcover, Pod Security Standards (PSS) are a Kubernetes feature for assigning varying levels of security to Pods and enable administrators to assign the right level of security to various workloads. PSS security levels correspond to "profiles", those being Privileged, Baseline, and Restricted. As outlined in the official Kubernetes documentation [(Pod Security Standards | Kubernetes)](https://kubernetes.io/docs/concepts/security/pod-security-standards/), security profile details are as follows:

- Privileged: The Privileged policy is purposely-open, and entirely unrestricted. This type of policy is typically aimed at system- and infrastructure-level workloads managed by privileged, trusted users.
- Baseline: The Baseline policy is aimed at ease of adoption for common containerized workloads while preventing known privilege escalations. This policy is targeted at application operators and developers of non-critical applications.
- Restricted: The Restricted policy is aimed at enforcing current Pod hardening best practices, at the expense of some compatibility. It is targeted at operators and developers of security-critical applications, as well as lower-trust users.

It is important to leverage these security profiles in order to manage workload access and keep applications secure.

### Non-Compliant v. Compliant Exhibits

Before applying non-compliant and compliant workloads, we first applied and validated a restricted security configuration as shown in the image below.

![PSS](/week11-12/screenshots/Lab03_Apply_Pod_Security_Configuration.PNG)
![Validation](/week11-12/screenshots/Lab03_Enforced_Pod_Security.PNG)

After this Pod Security Standard was applied the following non-compliant manifest, [insecure-pod.yaml](/week11-12/manifests/insecure-pod.yaml) was applied. As illustrated in the manifest, this deployment ran as root, used privileged: true, and utilized hostPath volumes. As such, the workload was rejected and unable to be applied.

![Non-Compliant](/week11-12/screenshots/Lab03_Apply_Insecure_Pod_Rejection.PNG)

On the other hand, the compliant manifest that adhered the Restricted Pod Security Standards was able to be deployed without issue.

![Compliant](/week11-12/screenshots/Lab03_Apply_Compliant_Restricted_Pod.PNG)

It is safer to add guardrails to deployments and utilize Pod Security Standards in order to prevent bad actors from resource misuse, maintain resource security, and protect applications.
