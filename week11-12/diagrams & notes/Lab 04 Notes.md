# Lab 04 Notes

## Observability Tooling | Incident Report

### Observability Tooling

According to the article [How to Setup Prometheus Monitoring on Kubernetes Cluster](https://devopscube.com/setup-prometheus-monitoring-on-kubernetes/) published by DevOpsCube, Prometheus is a high-scalable open-source monitoring framework that provides out-of-the-box monitoring capabilities for Kubernetes clusters. In order to get the cluster configured with observability tooling, we first ran the following command:

```console

PS D:\Source\k8s-enterprise-capstone-team1\week11-12\manifests> helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
"prometheus-community" already exists with the same configuration, skipping
PS D:\Source\k8s-enterprise-capstone-team1\week11-12\manifests> helm repo update 
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "prometheus-community" chart repository
Update Complete. ⎈Happy Helming!⎈

```

This sets up the parent repository, to which we ran the following command:

```console

PS D:\Source\k8s-enterprise-capstone-team1\week11-12\manifests> helm install prometheus prometheus-community/kube-prometheus-stack --namespace monitoring --create-namespace
NAME: prometheus
LAST DEPLOYED: Tue Apr 14 20:37:47 2026
NAMESPACE: monitoring
STATUS: deployed
REVISION: 1
DESCRIPTION: Install complete
TEST SUITE: None
NOTES:
kube-prometheus-stack has been installed. Check its status by running:
  kubectl --namespace monitoring get pods -l "release=prometheus"

Get Grafana 'admin' user password by running:

  kubectl --namespace monitoring get secrets prometheus-grafana -o jsonpath="{.data.admin-password}" | base64 -d ; echo

Access Grafana local instance:

  export POD_NAME=$(kubectl --namespace monitoring get pod -l "app.kubernetes.io/name=grafana,app.kubernetes.io/instance=prometheus" -oname)
  kubectl --namespace monitoring port-forward $POD_NAME 3000

Get your grafana admin user password by running:

  kubectl get secret --namespace monitoring -l app.kubernetes.io/component=admin-secret -o jsonpath="{.items[0].data.admin-password}" | base64 --decode ; echo


Visit https://github.com/prometheus-operator/kube-prometheus for instructions on how to create & configure Alertmanager and Prometheus instances using the Operator.

```

The parent repository is what houses all of the observability tooling that was installed above. Moreover, we housed all of the relevant tooling in the monitoring namespace. Once the tooling was installed, we ran the command:

```console

helm upgrade prometheus prometheus-community/kube-prometheus-stack --namespace monitoring -f prometheus-scrape-config.yaml

```

This was done in order to set configuration guidelines for Prometheus with regards to the information being scraped, in this case Prometheus was targeting every pod. In the Prometheus portal, we were most concerned with the status of the cAdvisor target. As explained in the article, [What is cAdvisor? How Does it Work? Explained](https://scaleyourapp.com/what-is-cadvisor-how-does-it-work-explained/) published by scaleyourapp.com, cAdvisor stands for container advisor and provides information with regards to resource usage, performance characteristics, and other related information. As shown in the image below, the cAdvisor target status was up, indicating what performance information was being served properly.

![Lab 04 Prometheus Running](/week11-12/screenshots/Lab04_Prometheus_Running_cAdvisor.PNG)

According the the GeeksForGeeks article, Monitoring Kubernetes Clusters with Prometheus and Grafana, Grafana is an online utility for visualization and analytics. In this lab, we created and uploaded a custom Grafana dashboard designed to capture pod CPU and Memory usage, and leveraged the Prometheus data store in order to visualize performance data as illustrated in the incident report.

### Dashboard Monitoring Flow

The article [What is cAdvisor? How Does it Work? Explained](https://scaleyourapp.com/what-is-cadvisor-how-does-it-work-explained/) provides a great image visualizing the Dashboard Monitoring Architectural Flow between Grafana, Prometheus, and cAdvisor as shown below.

![Dashboard Monitoring Flow](/week11-12/diagrams%20&%20notes/Dashboard%20Monitoring%20Flow.jpg)

### Incident Report

In the case of the Lab 04 execution, I chose to simulate resource saturation across pods. Before beginning simulated resource and memory exhaustion, I first collected a performance baseline under normal operating conditions.

![Performance Baseline](/week11-12/screenshots/Lab04_Grafana_Dashboard_Baseline_Resources.PNG)

Next, resource exhaustion was introduced by running the following command:

![Introduced Exhaustion](/week11-12/screenshots/Lab04_CPU_Resource_Exhaustion.PNG)

Subsequently, the Grafana dashboard illustrated a jump in both CPU and Memory resources, as shown below:

![Resource Exhaustion](/week11-12/screenshots/Lab04_Grafana_Dashboard_Resource_Exhaustion.PNG)

Once the command was stopped and resources were no longer being exhausted, we can observe resources decrease and stabilize once again.

![Post Fix](/week11-12/screenshots/Lab04_Grafana_Dashboard_Post_Fix.PNG)

By alleviating the resource exhaustion targeting the pods we can see both the CPU and Memory resource usage decrease and begin to normalize over time. While the root cause resolution was alleviating the stress process, much of the performance impact seen could have been mitigated through the use of memory limits and automatic scaling.
