# Troubleshooting Report

## Issue Summary

service-selector-mismatch deployment workload can't be accessed because it has no endpoints.

## Symptoms

Pod stuck with status pending.

## Investigation

Apply workloads: kubectl apply -f week03-04/broken-workloads/service-selector-mismatch.yaml and kubectl apply -f week03-04/broken-workloads/broken-service.yaml
View pods: kubectl get pods
Status column: Running

kubectl get endpoints demo-app-service returns:

```
Warning: v1 Endpoints is deprecated in v1.33+; use discovery.k8s.io/v1 EndpointSlice
NAME               ENDPOINTS   AGE
demo-app-service   <none>      2m42s
```

## Root Cause

The service has no endpoints because it couldn't find the deployment workload pods. The selector in the service file must match the label in the deployment file.

## Resolution

Change the selector in the service file to match the label in the deployment file.
