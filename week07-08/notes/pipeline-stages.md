# Pipeline Stages

## Stage 0 Checkout  
This is the stage where the source code is checked out from version control. In our case, we are using GitHub Actions and the `actions/checkout` action to checkout the code from our repository. This makes your helm chart, YAML, and dockerfile avalable

## Stage 1 Validate YAML
This stage is where the YAML files a validated by using `helm lint` to check the helm chart for syntax errors, any missing fields or bad structure.

## Stage 2 Build (Mock) 
This stage is  simulates when the sourse code is converted to a containter (Docker) image. This would normally run `docker build` and `docker push`, tag the image, then push it to a registry like docker hub. This makes sure the app can be packaged. 

## Stage 3 Deploy to Kubernetes

The final stage that updates the cluster stage using `helm upgrade --install`. Here the file deploys your app to Kubernetes using Helm, then, if the app exists and the current cluster is different, then the cluster will be updated. However if it doesnt exist then it will install the cluster.
