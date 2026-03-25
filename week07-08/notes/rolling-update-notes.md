
PS C:\Users\Jose Montalvo\Documents\GitHub\k8s-enterprise-capstone-team1> kubectl set image deployment/rolling-demo app=nginx:1.27
deployment.apps/rolling-demo image updated
PS C:\Users\Jose Montalvo\Documents\GitHub\k8s-enterprise-capstone-team1> kubectl rollout status deployment/rolling-demo
deployment "rolling-demo" successfully rolled out



# Deployment Strategies

## Blue-Green

Two identical production environments are maintained. Blue is the current version and green is the new version. You deploy new code to the green evironment and once its tested, you route traffic from the blue to the green.

Pros: No downtime and if green fails you can switch back to blue.

Cons: You have to maintain two environments (more resource heavy).

## Canary

New releases are only released to a small number of users before releasing it to everyone. You deploy a single canary pod and route a small amount of traffic to it. You slowy increase the traffic if no errors are reported until the old release is phased out.

Pros: Only a little amount of users are affected by potential bugs.

Cons: More advanced traffic routing needed.
