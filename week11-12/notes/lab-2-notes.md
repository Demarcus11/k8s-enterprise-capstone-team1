
# Lab 2 notes:

Before we add these new yaml codes to our cluster we should check if both the frontend and backend can comunicate with each other.
```
PS C:\Users\Jose Montalvo\Documents\GitHub\k8s-enterprise-capstone-team1> kubectl exec -it frontend -n dev -- sh                       
~ $ curl http://10.244.2.12:8080
this is the backend
```
```
PS C:\Users\Jose Montalvo\Documents\GitHub\k8s-enterprise-capstone-team1> kubectl run test --image=busybox -n dev -it -- sh
/ # wget -O- http://10.244.2.12:8080
Connecting to 10.244.2.12:8080 (10.244.2.12:8080)
writing to stdout
this is the backend
-                    100% |*******************************|    20  0:00:00 ETA
written to stdout
```
By running these commands we can see that everything can comunicate with each other. 
----------------------------------------------------------------------------
Now after applying both our allow-frontend-backend and default-deny
```
PS C:\Users\Jose Montalvo\Documents\GitHub\k8s-enterprise-capstone-team1> kubectl apply -f week11-12\manifests\default-deny.yaml       
networkpolicy.networking.k8s.io/default-deny created
PS C:\Users\Jose Montalvo\Documents\GitHub\k8s-enterprise-capstone-team1> kubectl apply -f week11-12\manifests\allow-frontend-backend.yaml
networkpolicy.networking.k8s.io/allow-frontend-to-backend created
```
We can test what traffic is and isnt allowd.


## Test showing allowed traffic
```
PS C:\Users\Jose Montalvo\Documents\GitHub\k8s-enterprise-capstone-team1> kubectl exec -it frontend -n dev -- sh
~ $ curl http://10.244.2.12:8080
this is the backend
```
## Test showing denied traffic
```
PS C:\Users\Jose Montalvo\Documents\GitHub\k8s-enterprise-capstone-team1> kubectl run test --image=busybox -n dev -it -- sh
All commands and output from this session will be recorded in container logs, including credentials and sensitive information passed through the command prompt.
If you don't see a command prompt, try pressing enter.
/ # wget -O- http://10.244.2.12:8080
Connecting to 10.244.2.12:8080 (10.244.2.12:8080)
wget: can't connect to remote host (10.244.2.12): Connection timed out
```

