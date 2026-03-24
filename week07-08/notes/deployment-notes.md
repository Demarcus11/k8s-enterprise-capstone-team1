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
PS C:\Users\Jose Montalvo\Documents\GitHub\k8s-enterprise-capstone-team1> kubectl get pods -w
NAME                         READY   STATUS    RESTARTS      AGE
app-probes-98b65cfc4-qdkqm   1/1     Running   0             100s
```

- Liveness probe: Changed livenessProbe's path from / to /doesnotexist causing the probe to restart.

```
PS C:\Users\Jose Montalvo\Documents\GitHub\k8s-enterprise-capstone-team1> kubectl get pods -w
NAME                          READY   STATUS    RESTARTS      AGE
app-probes-76c79f8989-7t4n6   0/1     Running   1 (0s ago)    31s
app-probes-76c79f8989-7t4n6   1/1     Running   1 (11s ago)   42s
app-probes-76c79f8989-7t4n6   0/1     Running   2 (0s ago)    61s
app-probes-76c79f8989-7t4n6   1/1     Running   2 (11s ago)   72s
app-probes-76c79f8989-7t4n6   0/1     Running   3 (0s ago)    91s
app-probes-76c79f8989-7t4n6   1/1     Running   3 (11s ago)   102s
app-probes-76c79f8989-7t4n6   0/1     Running   4 (0s ago)    2m1s
app-probes-76c79f8989-7t4n6   1/1     Running   4 (11s ago)   2m12s
```
- To prove liveness prope is working we can use kubectl describe pod app-probes
```
Events:
  Type     Reason     Age                   From               Message
  ----     ------     ----                  ----               -------
  Normal   Scheduled  20m                   default-scheduler  Successfully assigned default/app-probes-76c79f8989-pvncx to kind-worker
  Normal   Started    17m (x6 over 20m)     kubelet            Container started
  Normal   Killing    16m (x6 over 20m)     kubelet            Container app failed liveness probe, will be restarted
  Warning  BackOff    10m (x13 over 16m)    kubelet            Back-off restarting failed container app in pod app-probes-76c79f8989-pvncx_default(49af65ee-55eb-470e-a58f-0f2079abfab9)
  Normal   Pulled     6m29s (x9 over 20m)   kubelet            Container image "nginx" already present on machine and can be accessed by the pod
  Normal   Created    5m50s (x10 over 20m)  kubelet            Container created
  Warning  Unhealthy  5m31s (x28 over 20m)  kubelet            Liveness probe failed: HTTP probe failed with statuscode: 404
```

- Readiness probe: passed when readinessProbe path: /  THere was also no restarts

```
PS C:\Users\Jose Montalvo\Documents\GitHub\k8s-enterprise-capstone-team1> kubectl get pods
NAME                          READY   STATUS    RESTARTS      AGE
app-probes-754c9b6786-r2jrk   1/1     Running   0             18s
```

- Readiness probe: readinessProbe path set to /broken . Now the new pod is stuck at 0/1 ready and the old pod is still running This is thanks to k8s keeping the old pod running until the new pod is ready for traffic.

```
PS C:\Users\Jose Montalvo\Documents\GitHub\k8s-enterprise-capstone-team1> kubectl get pods
NAME                          READY   STATUS    RESTARTS      AGE
app-probes-754c9b6786-r2jrk   1/1     Running   0             8m56s
app-probes-7dff488bbc-fdlsh   0/1     Running   0             2s
```

## Rollout and rollback observations

- Rollout: Changed image from nginx:1.27 to nginx:1.26 then used `kubectl rollout status deployment/app-probes` to see rollout status and used `kubectl rollout history deployment/app-probes` to see rollout history and it output 2 revisions.

Initially:

```
PS C:\Users\Jose Montalvo\Documents\GitHub\k8s-enterprise-capstone-team1> kubectl rollout history deployment/app-probes
deployment.apps/app-probes
REVISION  CHANGE-CAUSE
1         <none>

```

Once image was changed:

```
PS C:\Users\Jose Montalvo\Documents\GitHub\k8s-enterprise-capstone-team1> kubectl rollout status deployment/app-probes
Waiting for deployment "app-probes" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "app-probes" rollout to finish: 1 old replicas are pending termination...
deployment "app-probes" successfully rolled out

PS C:\Users\Jose Montalvo\Documents\GitHub\k8s-enterprise-capstone-team1> kubectl rollout history deployment/app-probes
deployment.apps/app-probes 
REVISION  CHANGE-CAUSE
1         <none>
5         <none>
```

- Rollback: Changed the image to nginx:fake image causing an imagePull error then rolled back the update with `kubectl rollout undo deployment/app-probes` to rollback the deployment and the new pod was killed.

```
PS C:\Users\Jose Montalvo\Documents\GitHub\k8s-enterprise-capstone-team1> kubectl get pods                            
NAME                          READY   STATUS         RESTARTS      AGE    
 
app-probes-5485d57655-rg57m   1/1     Running        0             13m    
app-probes-5bd799486f-4nmrj   0/1     ErrImagePull   0             35s    

-----------------------------------------------------------------------------------------------------------------------

PS C:\Users\Jose Montalvo\Documents\GitHub\k8s-enterprise-capstone-team1> kubectl rollout undo deployment/app-probes
deployment.apps/app-probes rolled back

-----------------------------------------------------------------------------------------------------------------------

PS C:\Users\Jose Montalvo\Documents\GitHub\k8s-enterprise-capstone-team1> kubectl get pods
NAME                          READY   STATUS    RESTARTS      AGE
app-probes-5485d57655-rg57m   1/1     Running   0             16m
```
