When trying to complete lab 4. I applied the crashloop.yaml file found in broken-workloads and quickly found errors
when viewing the logs. The command "bin/false" would continously cause the pods to fail when trying to run.
After changing the command to "bin/true", the pod would run and then immediately complete instead of being a 
long-lived process. After getting rid of the command entirely, the pod was able to run a long process and stay in the
running status.