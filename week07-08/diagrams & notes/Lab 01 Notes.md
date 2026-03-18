# Lab 01 Notes - Helm Charts

## Values.yaml

As outlined in the official Helm documentation [(Helm | Value Files)](https://helm.sh/docs/chart_template_guide/values_files/), Values files are built in objects that provide default configuration values to be passed to the chart. As opposed to hard coding information pertaining to manifests such as the replica count, image information, and service information, these values can be stored in the Value file instead. This enables default configuration parameters to be changed without editing and redeploying manifests, as illustrated in when upgrading the image:

![Upgrade Release](/week07-08/screenshots/Lab01_Helm_Upgrade_Release.PNG)

Instead of having to edit and redeploy [deployment.yaml](/week07-08/solutions/helm/sample-app/templates/deployment.yaml) to change the image version, the command shown in the previous screenshot allows us to change the image version at runtime.

## Templates vs raw YAML

According to the Medium Article, [Helm Basics — Understanding Charts, Templates, and Repositories](https://medium.com/@bavicnative/helm-basics-understanding-charts-templates-and-repositories-6b8e55f539e0), Templates allow you to create flexible and reusable configurations through the use of {{ }} placeholders. Templates work in conjunction with Values files, as these values are populated using the parameters found in the [values.yaml](/week07-08/solutions/helm/sample-app/values.yaml) file. On the other hand, raw YAML is static such that these values are hardcoded in the deployment or service manifests. As such, in order to make changes to raw YAML files we must edit and redeploy them, while in templated files we can use the command:

```console
helm upgrade
```

In order to make changes at runtime.

## Rollback Behavior

According to the official Helm documentation [(Helm | Rollback)](https://helm.sh/docs/helm/helm_rollback/), the rollback command rolls back a release to a previous version. As observed accross the Lab01 screenshots, the helm chart was installed, the release was upgraded using the upgrade command, and the relase was finally rolled back to the release captured in revision one (in this case when the chart was initially installed), as observed in revision three.  

![Rollback Release](/week07-08/screenshots/Lab01_Helm_Rollback.PNG)
