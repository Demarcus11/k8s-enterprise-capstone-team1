# Security & GitOps Notes

## Lab 01 - Explaining GitOps Workflow | Evidence of Auto-Sync

As outlined in ArgoCD's official documentation [(ArgoCD Documentation)](https://argo-cd.readthedocs.io/en/stable/), ArgoCD is a GitOps delivery tool for Kubernetes in which application deployment and lifecycle management can be audited and automated. In the context of Lab 01, we were tasked with creating an ArgoCD application pointing to our remote Git repository, as evidenced in [application.yaml](/week09-10/manifests/application.yaml). By enabling auto-sync in this manifest and deploying the file to our remote repository, we ensured that our local cluster remains in sync with manifest definitions outlined on our remote repository.  

This is evidenced when creating the [deployment.yaml](/week09-10/manifests/deployment.yaml) file. Instead of applying the file to the local cluster manually, this file was pushed to our remote repository. Since the application file enabled auto-sync and detected changes on the remote repository, the deployment manifest was automatically deployed to our local cluster without having to deploy it manually.

## Evidence of Auto-Sync

As illustrated below, when drift was introduced manually to the deployment, ArgoCD immediately recognized this drift and automatically synced/reconciled the change by immediately creating another pod.

![Drift and Auto-Sync](/week09-10/screenshots/Lab01_Argocd_Drift_And_Reconciliation.PNG)

## Lab 02 - Secrets and Secret Handling Best Practices | Evidence of Secret Injection

As outlined in the official Kubernetes documentation [(Secrets | Kubernetes)](https://kubernetes.io/docs/concepts/configuration/secret/), a Secret is an object that contains a small amount of sensitive data such as a passwrod, token, or key that would otherwise be stored in a Pod specification or container image. Secrets enable developers to exclude confidential data from application code, as Secrets can be created and managed independently from the Pods that rely on them.

It is important to manage these secrets effectively and safely, and the official Kubernetes documentation even has information concerning the best practices for using Secrets [Good Practices for Kubernetes Secrets](https://kubernetes.io/docs/concepts/security/secrets-good-practices/). This page lists core best practices concerning Secret use such as:

- Configure least-privilege access to Secrets
- Restrict access for Secrets using namespaces
- Improve etcd management policies
- Restrict Secret access to specific containers  
- Protect Secret data after reading
- Avoid sharing Secret manifests

## Lab 03 - Vulnerability Management | Evidence of Trivy Vulnerability Scan

As outlined in the official Trivy documentation [(Trivy | Kubernetes)](https://trivy.dev/docs/latest/guide/target/kubernetes/), Trivy is a tool that can connect to a Kubernetes cluster and scan it for security issues. When scanning a cluster, the container image and cluster resources are scanned for vulnerabilities, misconfigurations, and exposed secrets. Upon the completion of a scan, a report summary can be generated in which  

### Evidence of Trivy Scan

This cluster scanning tooling can also be configured directly within CI pipeline workflows in order to validate manifests, as illustrated below:

![Scan Pending](/week09-10/screenshots/Lab03_Trivy_Scan_Stage.PNG)
![Scan Complete](/week09-10/screenshots/Lab03_Pipeline_Failure.PNG)

Moreover, the complete Trivy logs associated with this Lab can be found here: [Lab03_Trivy_Logs.txt](/week09-10/console%20logs/Lab03_Trivy_Logs.txt)

## Lab 04 - Validation Rules and Logic Explanation | Evidence of Blocked Deployment

As defined within the official Kubernetes documentation [(OPA Gatekeeper: Policy and Governance for Kubernetes | Kubernetes)](https://kubernetes.io/blog/2019/08/06/opa-gatekeeper-policy-and-governance-for-kubernetes/), Open Policy Agent Gatekeeper is a project that can be leveraged in order to enforce policies and improve governance within a kubernetes environment. In the article, the authors describe how Kubernetes allows decoupling policy decisions from the API controller by the use of admission controllers. These admission controllers intercept admission requests to the API before they are created and applied. As such, Gatekeeper allows the developer to customize these configuration contros by through the use of policies and constraints.

A constraint template defines the requirements by which the system must meet. Once defined, the policy defines individual constraints as defined by the template and enforces said rules within the system.

When looking at the [gatekeeper-constraint-template.yaml](/week09-10/manifests/gatekeeper-constraint-template.yaml) file, we see that out validation logic is as follows:

```console
|
        package gatekeeperpolicy

        violation[{"msg": msg}] {
          container := input.review.object.spec.template.spec.containers[_]
          endswith(container.image, ":latest")
          msg := sprintf("'%v' uses :latest tag", [container.name])
        }

        violation[{"msg": msg}] {
          container := input.review.object.spec.template.spec.containers[_]
          not container.resources.limits
          msg := sprintf("'%v' is missing resource limits", [container.name])
        }

        violation[{"msg": msg}] {
          container := input.review.object.spec.template.spec.containers[_]
          not container.resources.requests
          msg := sprintf("'%v' is missing resource requests", [container.name])
        }

        violation[{"msg": msg}] {
          container := input.review.object.spec.template.spec.containers[_]
          not container.livenessProbe
          msg := sprintf("'%v' is missing livenessProbe", [container.name])
        }

        violation[{"msg": msg}] {
          container := input.review.object.spec.template.spec.containers[_]
          not container.readinessProbe
          msg := sprintf("'%v' is missing readinessProbe", [container.name])
        }
```

The first violation checks if the container.image ends with ":latest", an indication that the deployment is not using a versioned image and is instead using the latest tag. The second and third violations check to see if requests and limits are defined within the deployment. Finally, the third and fourth violations ensure that deployment contains definitions for bothe liveness and readiness probes.

### Evidence of Blocked Deployment

![Policy and Constraint](/week09-10/screenshots/Lab04_OPA_Gatekeeper_Policy_And_Constraint.PNG)
![Policy and Constraint Pods](/week09-10/screenshots/Lab04_OPA_Gatekeeper_Pods_Running.PNG)
![Constraint Active](/week09-10/screenshots/Lab04_OPA_Gatekeeper_Constraint_Active.PNG)
![Blocked vs Valid](/week09-10/screenshots/Lab04_OPA_Gatekeeper_Valid_VS_Invalid_Deployment.PNG)
