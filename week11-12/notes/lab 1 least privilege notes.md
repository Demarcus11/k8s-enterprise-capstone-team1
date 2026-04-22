# Why This is Least Privilege

It is diffined that Least privilege is the limited permission given so that just a spesific task can be completed. The task given to us is just the ability to view pods.
This is done by:
1. Creating a service account named readonly-sa in the dev namespace.
2. Creating a role named pod-reader in the dev namespace that allows read-only access to pods. 
3. Creating a role binding named readonly-binding in the dev namespace that binds the pod-reader role to the readonly-sa service account.

We know this follows least privilege because access is given by allowing only pod resources to be viewed. And only the get and list verbs to be used. Verbs like delete are restricted and cannot be used

When testing the authority of the dev space of readonly-sa we are given proof of the authority to view but not change pods
```
PS C:\Users\Jose Montalvo\Documents\GitHub\k8s-enterprise-capstone-team1> kubectl auth can-i list pods --as=system:serviceaccount:dev:readonly-sa -n dev
yes
PS C:\Users\Jose Montalvo\Documents\GitHub\k8s-enterprise-capstone-team1> kubectl auth can-i delete pods --as=system:serviceaccount:dev:readonly-sa -n dev
no
```