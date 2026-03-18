# Lab 01 Notes - Deployment Strategies

## Blue Green vs. Canary

According to the DevOps Daily article *[Deployment Strategies: Blue-Green, Canary, and Rolling Deployments Explained](https://devops-daily.com/posts/deployment-strategies-guide)*, Blue-Green deployment maintains two identical production environments where one serves traffic while the other sits idle. Come deployment, you deploy to the idle environment and test to ensure that everything is running smoothly, then you route traffic to the idle environment. Blue-Green strives at being safe and simplistic, however it suffers from resource duplication. Canary, like Blue-Green, maintains two environments but instead of switching all at once, small gradual increases in traffic are routed to the new version while most users remain on the previous stable version. Ultimately, Canary serves as a middle ground between speed and safety, but is notably more complex to implement than Blue-Green.

## Rollout and Rollback Evidence

![Deployment](/week07-08/screenshots/Lab04_Rolling_Update_Deployment_And_Rollout.PNG)
![Bad Release](/week07-08/screenshots/Lab04_Rolling_Update_Bad_Release.PNG)
![Bad Release Outcome](/week07-08/screenshots/Lab04_Rolling_Update_Bad_Release_Outcome.PNG)
![Rollback](/week07-08/screenshots/Lab04_Rolling_Update_Safe_Rollback.PNG)
