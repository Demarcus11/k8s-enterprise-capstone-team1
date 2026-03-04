# Deployment Notes

## Lab 1 Deployment

- Created a stateless application that was deployed using a Deployment with 2 replicas.

  ```
  PS C:\Users\Jose Montalvo\Documents\GitHub\k8s-enterprise-capstone-team1> kubectl get pods
  NAME                        READY   STATUS    RESTARTS   AGE
  app-demo-5d845c9876-5stbg   1/1     Running   0          3m21s
  app-demo-5d845c9876-qgfvz   1/1     Running   0          30m
  ```

- Created a service.yaml to provide a stable IP and load balance across the pods.

  ```
  PS C:\Users\Jose Montalvo\Documents\GitHub\k8s-enterprise-capstone-team1> kubectl get svc
  NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
  app-demo        ClusterIP   10.96.241.74   <none>        80/TCP    21m  
  ```

- Changed the number of replicas in the deployment file to 5.

  ```
  PS C:\Users\Jose Montalvo\Documents\GitHub\k8s-enterprise-capstone-team1> kubectl get pods
  NAME                        READY   STATUS    RESTARTS   AGE
  app-demo-5d845c9876-5stbg   1/1     Running   0          3m35s
  app-demo-5d845c9876-c2fk8   1/1     Running   0          2s
  app-demo-5d845c9876-p72ml   1/1     Running   0          2s
  app-demo-5d845c9876-qgfvz   1/1     Running   0          31m
  app-demo-5d845c9876-ssgpz   1/1     Running   0          2s
  ```
- Tested Replicas by adding a print output
  ```
  command: ["/bin/sh"]
          args: 
            - "-c"
            - echo "Hello from $(hostname)" > /usr/share/nginx/html/index.html && nginx -g 'daemon off;'
  ```
- Used kubectl run curl-test --rm -it --image=busybox -- sh to test the output
  ```
  / # wget -qO- app-demo
  Hello from app-demo-7df55896c9-22qtf
  / # wget -qO- app-demo
  Hello from app-demo-7df55896c9-znnwt
  / # wget -qO- app-demo
  Hello from app-demo-7df55896c9-znnwt
  / # wget -qO- app-demo
  Hello from app-demo-7df55896c9-ljg5v
  / # wget -qO- app-demo
  Hello from app-demo-7df55896c9-dvttz
  / # wget -qO- app-demo
  Hello from app-demo-7df55896c9-znnwt
  / # wget -qO- app-demo
  Hello from app-demo-7df55896c9-ljg5v`
  ```




- Created and injected ConfigMap and secret env vars and volumes.
  ```
  PS C:\Users\Jose Montalvo\Documents\GitHub\k8s-enterprise-capstone-team1> kubectl exec -it app-demo-6f5c469cb-4d5ts -- sh
  # echo $ENV
  dev
  -------------------------------------------------------------------------------------------------------------------------
  PS C:\Users\Jose Montalvo\Documents\GitHub\k8s-enterprise-capstone-team1> kubectl describe pod app-demo-5f45df76cd-5gwkz
  Environment:
      ENV:          <set to the key 'ENV' of config map 'app-config'>   Optional: false
      DB_PASSWORD:  <set to the key 'PASSWORD' in secret 'app-secret'>  Optional: false
  ```


## Probe behavior

- Liveness probe: Pods passed liveness probe when livenessProbe path was /

```
PS C:\Users\demar\Desktop\Capstone Project\k8s-enterprise-capstone-team1> kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
app-probes-6cd55b66c6-6nx92   1/1     Running   0          14s
```

- Liveness probe: Changed livenessProbe path to /fail and the pod's restarts started increasing.

```
PS C:\Users\demar\Desktop\Capstone Project\k8s-enterprise-capstone-team1> kubectl get pods
NAME                          READY   STATUS    RESTARTS      AGE
app-probes-5bccc6cd64-r8nnr   1/1     Running   1 (39s ago)   79s
```

- Readiness probe: Pods passed readiness probe when readinessProbe path was /

```
PS C:\Users\demar\Desktop\Capstone Project\k8s-enterprise-capstone-team1> kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
app-probes-6cd55b66c6-g7z98   1/1     Running   0          16s
```

- Readiness probe: Changed readinessProbe path to /fail and the new pod is stuck at 0/1 ready and the old pod is still running meaning k8s keeps the old pod running until the new pod is ready for traffic.

```
PS C:\Users\demar\Desktop\Capstone Project\k8s-enterprise-capstone-team1> kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
app-probes-6cccf95f6d-ksv5r   0/1     Running   0          6s
app-probes-6cd55b66c6-g7z98   1/1     Running   0          72s
```

## Rollout and rollback observations

- Rollout: Changed image from nginx:1.27 to nginx:1.26 then used `kubectl rollout status deployment/app-probes` to see rollout status and used `kubectl rollout history deployment/app-probes` to see rollout history and it output 2 revisions.

Initially:

```
PS C:\Users\demar\Desktop\Capstone Project\k8s-enterprise-capstone-team1> kubectl rollout history deployment/app-probes
deployment.apps/app-probes
REVISION  CHANGE-CAUSE
1         <none>
```

Once image was changed:

```
PS C:\Users\demar\Desktop\Capstone Project\k8s-enterprise-capstone-team1> kubectl rollout status deployment/app-probes
Waiting for deployment "app-probes" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "app-probes" rollout to finish: 1 old replicas are pending termination...
deployment "app-probes" successfully rolled out

PS C:\Users\demar\Desktop\Capstone Project\k8s-enterprise-capstone-team1> kubectl rollout history deployment/app-probes
deployment.apps/app-probes
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

- Rollback: Changed the image to a fake nginx:fake image and the deployment failed then applied `kubectl rollout undo deployment/app-probes` to rollback the deployment and the new pod was killed.

```
PS C:\Users\demar\Desktop\Capstone Project\k8s-enterprise-capstone-team1> kubectl get pods
NAME                          READY   STATUS             RESTARTS   AGE
app-probes-59c68cf9b5-wpcvq   1/1     Running            0          5m42s
app-probes-f9f777675-lfvwb    0/1     ImagePullBackOff   0          91s

PS C:\Users\demar\Desktop\Capstone Project\k8s-enterprise-capstone-team1> kubectl rollout undo deployment/app-probes
deployment.apps/app-probes rolled back

PS C:\Users\demar\Desktop\Capstone Project\k8s-enterprise-capstone-team1> kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
app-probes-59c68cf9b5-wpcvq   1/1     Running   0          7m4s
```
