# Lab 03 Notes - CI Pipeline  

## Pipeline Stage Explanation

As outlined in the Lab 03 Tasks, the CI pipeline [(ci-pipeline.yaml)](/week07-08/manifests/ci-pipeline.yaml) implemented must: 1) Validate the YAML 2) Build the container image 3) Deploy to Kubernetes using kubectl or Helm, and 4) Trigger the CI pipeline on PR merge. These steps are decomposed into jobs, and are defined using a series of steps and environment definitions.  

### Validate YAML

The first job, validate-yaml, runs on ubuntu-latest on GitHub's servers, and triggers if a pull request's merge status is true. If the condition is met, it first checks out the code using the checkout action. Next, kubeval is installed on the CI pipeline instance evaluating the files, allowing the files to be validated. The next step runs kubeval, evaluating files that end with the .yaml extension within the week07-08/manifests folder. Finally, the last step validates the helm chart sample-app.

### Build Container Image

The second job, build-image, runs on ubuntu-latest on GitHub's servers and requires that validate-yaml has run successfully. If validate-yaml has run successfully, it first checks out the code using the checkout action. Next, it runs a mock build in which it simulates building a container image, when in reality no actual container was built.

### Deploy to Kubernetes

The third job, deploy, runs on ubuntu-latest on GitHub's servers and requires that build-image has run successfully. If build-image has run successfully, it first checks out the code using the checkout action. Next, it installs Helm. Finally, a mock deployment is ran to simulate the deployment to Kubernetes using Helm, since no containers are actually being built in the build-image step, there is nothing to actually deploy.

### Trigger CI Pipeline on PR Merge and Result

![Pipeline Outcome](/week07-08/screenshots/Lab03_CI_Pipeline_Working.PNG)
