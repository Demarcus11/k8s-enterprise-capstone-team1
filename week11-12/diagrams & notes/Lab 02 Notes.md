# Lab 02 Notes

## NetworkPolicy | Allowed v. Blocked Traffic Exhibits

### NetworkPolicies

According to the article [Kubernetes Network Policy - Guide with Examples](https://spacelift.io/blog/kubernetes-network-policy) published by Spacelift, NetworkPolicies are a mexhanism for controlling network traffic flow in Kubernetes clusters. These policies allow developers to define which pods are allowed to exchange network traffic, thereby assisting in limiting the blast radius in the event an application is compromised.  

### Allowed v. Blocked Traffic

In the Lab 02 Execution, we had two NetworkPolicy manifests, [allow-frontend-backend.yaml](/week11-12/manifests/allow-frontend-backend.yaml) which allows a frontend application to interface with a backend application, and [default-deny.yaml](/week11-12/manifests/default-deny.yaml) which denys all incoming traffic. As illustrated in the images below, prior to these NetworkPolicies being applied, all pods could interact with the backend pod and receive information.

![No Policy Frontend](/week11-12/screenshots/Lab02_Frontend_Traffic_No_Network_Policy.PNG)
![No Policy Test](/week11-12/screenshots//Lab02_Testing_Traffic_No_Network_Policy.PNG)

However, once the NetworkPolicies were applied as illustrated in the image below:

![Apply Policy Frontend](/week11-12/screenshots/Lab02_Apply_Frontend_Backend_Network_Policy.PNG)
![Apply Policy Deny](/week11-12/screenshots/Lab02_Apply_Default_Deny_Network_Policy.PNG)
![Policies Running](/week11-12/screenshots/Lab02_Network_Policies_Running.PNG)

The network behavior immediately changes, with the testing pod no longer being able to receive information from the backend pod due to the NetworkPolicy, while the frontend pod could still communicate with the backend pod without issue.

![Policy Frontend](/week11-12/screenshots/Lab02_Frontend_Traffic_Network_Policy.PNG)
![Policy Deny](/week11-12/screenshots/Lab02_Testing_Traffic_Network_Policy.PNG)
