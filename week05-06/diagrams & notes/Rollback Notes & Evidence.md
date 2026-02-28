# Rollback Notes & Evidence

## Rollback Notes

As described in the [*Kubernetes Rollbacks*](https://learnkube.com/kubernetes-rollbacks) article by Gergely Risko, Kubernetes and the Kubectl command line client offer a simple mechanizm to undo changes pertaining to resources such as Deployments, StatefulSets, and DeamonSets in the form of rollbacks. In the Lab 04 executions shown below, we updated the application image to point towards both an updated valid nginx version and an invalid image reference.

**Valid Image Shift:**

```console
PS D:\Source\k8s-enterprise-capstone-team1\week05-06\manifests> kubectl set image deployment/liveness-readiness-probe-deployment app=nginx:1.28
deployment.apps/liveness-readiness-probe-deployment image updated
```

**Invalid Image Shift:**

```console
PS D:\Source\k8s-enterprise-capstone-team1\week05-06\manifests> kubectl set image deployment/liveness-readiness-probe-deployment app=badImage
deployment.apps/liveness-readiness-probe-deployment image updated
```

When monitoring the rollout statuses for the aformentioned image updates we observe the following:

**Valid Image Shift Rollout Status:**

```console
PS D:\Source\k8s-enterprise-capstone-team1\week05-06\manifests> kubectl rollout status deployment/liveness-readiness-probe-deployment
deployment "liveness-readiness-probe-deployment" successfully rolled out
```

**Invalid Image Shift Rollout Status:**

```console
PS D:\Source\k8s-enterprise-capstone-team1\week05-06\manifests> kubectl rollout status deployment/liveness-readiness-probe-deployment
Waiting for deployment "liveness-readiness-probe-deployment" rollout to finish: 1 old replicas are pending termination...
```

Since the one of the image references is invalid, we are met with an error message communicating that the deployment has yet to finish and that an old replica is pending termination. When faced with this sort of issue, we as the developer can roll back the failed deployment as shown below:

```console
PS D:\Source\k8s-enterprise-capstone-team1\week05-06\manifests> kubectl rollout undo deployment/liveness-readiness-probe-deployment
deployment.apps/liveness-readiness-probe-deployment rolled back
```

## Rollback Evidence

**Updating Application Images:**
![Valid](/week05-06/screenshots/Lab04_Update_Application_Image.PNG)
![Invalid](/week05-06/screenshots/Lab04_Update_Application_Image_Invalid.PNG)

**Monitoring Rollout Statuses:**
![Valid](/week05-06/screenshots/Lab04_Rollout_Status_Monitor.PNG)
![Invalid](/week05-06/screenshots/Lab04_Rollout_Status_Monitor_Invalid.PNG)

**Rollback Failed Deployment:**
![FailedDeploymentRollback](/week05-06/screenshots/Lab04_Rollback_Failed_Deployment.PNG)
