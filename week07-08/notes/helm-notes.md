#### Week 7 - 8

### Lab 1 Helm
## Steps taken for the Lab

# Creating the Helm
First we create our own templates (config, deployment, service) making sure not to hardcode anydata that should be changed in the values.yaml, then we instal the Helm file

```
PS C:\Users\Jose Montalvo\Documents\GitHub\k8s-enterprise-capstone-team1> helm install myhelmapp-release week07-08\webapp1/
NAME: myhelmapp-release
LAST DEPLOYED: Sun Mar 22 12:37:50 2026
NAMESPACE: default
STATUS: deployed
REVISION: 1
DESCRIPTION: Install complete
TEST SUITE: None
```


Then Check both helm list and helm replicas

```
PS C:\Users\Jose Montalvo\Documents\GitHub\k8s-enterprise-capstone-team1> helm list
NAME                    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
myhelmapp-release       default         8               2026-03-23 22:50:54.168088 -0400 EDT    deployed        webapp1-0.1.0   1.16.0  
```
```
PS C:\Users\Jose Montalvo\Documents\GitHub\k8s-enterprise-capstone-team1> kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
app-probes-5485d57655-xj9ls         1/1     Running   0          2d
myhelmapp-release-f4f4b9448-gjrhl   1/1     Running   0          14h
myhelmapp-release-f4f4b9448-qn7g6   1/1     Running   0          14h
```

# Updating the helm

After changing the Image tag, Replica count, Service type make sure to use ```helm upgrade <helm name> <file path>```

```
PS C:\Users\Jose Montalvo\Documents\GitHub\k8s-enterprise-capstone-team1> helm upgrade myhelmapp-release week07-08\webapp1/
Release "myhelmapp-release" has been upgraded. Happy Helming!
NAME: myhelmapp-release
LAST DEPLOYED: Mon Mar 23 12:12:07 2026
NAMESPACE: default
STATUS: deployed
REVISION: 10
DESCRIPTION: Upgrade complete
TEST SUITE: None
```
Now we can view the updated versions
```
PS C:\Users\Jose Montalvo\Documents\GitHub\k8s-enterprise-capstone-team1> helm history myhelmapp-release
REVISION        UPDATED                         STATUS          CHART           APP VERSION     DESCRIPTION     
1               Sun Mar 22 12:37:50 2026        superseded      webapp1-0.1.0   1.16.0          Install complete
2               Mon Mar 23 12:12:07 2026        superseded      webapp1-0.1.0   1.16.0          Upgrade complete
3               Mon Mar 23 12:17:34 2026        superseded      webapp1-0.1.0   1.16.0          Upgrade complete
4               Mon Mar 23 12:38:50 2026        superseded      webapp1-0.1.0   1.16.0          Upgrade complete
5               Mon Mar 23 22:46:04 2026        superseded      webapp1-0.1.0   1.16.0          Upgrade complete
6               Mon Mar 23 22:47:29 2026        superseded      webapp1-0.1.0   1.16.0          Upgrade complete
7               Mon Mar 23 22:48:41 2026        superseded      webapp1-0.1.0   1.16.0          Upgrade complete
8               Mon Mar 23 22:50:54 2026        superseded      webapp1-0.1.0   1.16.0          Upgrade complete
9               Tue Mar 24 13:56:14 2026        deployed        webapp1-0.1.0   1.16.0          Upgrade complete
10              Tue Mar 24 13:57:41 2026        deployed        webapp1-0.1.0   1.16.0          Upgrade complete
```

Get pods to view how the new update is working. we can see that replicas jumped from 3 to 4 and the old replicas are being terminated
```
PS C:\Users\Jose Montalvo\Documents\GitHub\k8s-enterprise-capstone-team1> kubectl get pods
NAME                                 READY   STATUS        RESTARTS   AGE
app-probes-5485d57655-xj9ls          1/1     Running       0          2d1h
myhelmapp-release-85754fc67c-b5dnp   1/1     Running       0          35s
myhelmapp-release-85754fc67c-bp9jh   1/1     Running       0          27s
myhelmapp-release-85754fc67c-r9tm2   1/1     Running       0          35s
myhelmapp-release-85754fc67c-z7z62   1/1     Running       0          27s
myhelmapp-release-f4f4b9448-gjrhl    1/1     Terminating   0          15h
myhelmapp-release-f4f4b9448-qn7g6    1/1     Terminating   0          15h
myhelmapp-release-f4f4b9448-rkzdd    1/1     Terminating   0          35s
```
Now we can also check the service type to confirm it is changed to nodeport
```
PS C:\Users\Jose Montalvo\Documents\GitHub\k8s-enterprise-capstone-team1> kubectl get svc
NAME                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes          ClusterIP   10.96.0.1       <none>        443/TCP        38d
myhelmapp-release   NodePort    10.96.140.253   <none>        80:30291/TCP   25h
```
And now we can check the image tag
```
PS C:\Users\Jose Montalvo\Documents\GitHub\k8s-enterprise-capstone-team1> kubectl describe pod myhelmapp-release
...
Image:         nginx:1.28
...

```
# Rollback to 1.27, 2 replicas, and clusterIP service type
```
PS C:\Users\Jose Montalvo\Documents\GitHub\k8s-enterprise-capstone-team1> helm rollback myhelmapp-release 9
Rollback was a success! Happy Helming!
```
after checkin the history we can confirmation of a rollback
```
PS C:\Users\Jose Montalvo\Documents\GitHub\k8s-enterprise-capstone-team1> helm history myhelmapp-release        
REVISION        UPDATED                         STATUS          CHART           APP VERSION     DESCRIPTION     
...
12              Tue Mar 24 14:14:35 2026        deployed        webapp1-0.1.0   1.16.0          Rollback to 9
...
```
Now we can check the pods image tag and services to confirm the rollback
```
PS C:\Users\Jose Montalvo\Documents\GitHub\k8s-enterprise-capstone-team1> kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
app-probes-5485d57655-xj9ls         1/1     Running   0          2d1h
myhelmapp-release-f4f4b9448-44rdt   1/1     Running   0          3m46s
myhelmapp-release-f4f4b9448-4fk7v   1/1     Running   0          3m47s

PS C:\Users\Jose Montalvo\Documents\GitHub\k8s-enterprise-capstone-team1> kubectl get svc
NAME                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes          ClusterIP   10.96.0.1       <none>        443/TCP   38d
myhelmapp-release   ClusterIP   10.96.140.253   <none>        80/TCP    25h

PS C:\Users\Jose Montalvo\Documents\GitHub\k8s-enterprise-capstone-team1> kubectl describe pod myhelmapp-release
...
Image:         nginx:1.27
...
```
## Important explanations
- When creating a helm chart, it is important to make sure that any data that should be changed is not hardcoded in the templates but instead is placed in the values.yaml file. this is where all the all the important variables that might get regularly changed will be declared.
- The diferance between a template and raw Yaml file is simple. A raw file will have all the data hard coded into the file, however a template file will take its important information that might be changed 
and and call that information from a file that declares variables like the values.yaml file. This allows for easier updates and maintenance of the helm chart.
- When you create a helm it will store both the kubectl manifests and the values for the deployment. So when the helm is rolled back helm will retrieve the old version and reaply those values and menefests to match the old state. However, after some research I found out that only the kuberneties manifests are managed by helm, any external data and resaurses will not be changed by a helm rollback 


# other notes 
- When updating a helm chart, it is important to use the ```helm upgrade``` command to apply the changes. This will ensure that the changes are properly applied and that the helm chart is updated correctly.
- When rolling back a helm chart, it is important to use the ```helm rollback``` command to revert to a previous version. This will ensure that the rollback is properly applied and that the helm chart is reverted correctly.
- When checking the status of a helm chart, it is important to use the ```helm list``` command to view the current status of the helm chart. This will provide information about the current version, status, and other details about the helm chart.
- When checking the history of a helm chart, it is important to use the ```helm history``` command to view the history of the helm chart. This will provide information about the previous versions, updates, and other details about the helm chart.
- When checking the pods and services of a helm chart, it is important to use the ```kubectl get pods``` and ```kubectl get svc``` commands to view the current status of the pods and services. This will provide information about the current state of the pods and services, including the image tag, service type, and other details.



