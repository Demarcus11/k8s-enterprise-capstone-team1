# Troubleshooting Report

## Issue Summary

crashloop-demo workload failing to run.

## Symptoms

Pod starts with status Error, transitions to status CrashLoopBackOff, and number of restarts increase.

## Investigation

Apply workload: kubectl apply -f week03-04/broken-workloads/crashloop.yaml
View pods: kubectl get pods

Status column: Error then transistions to CrashLoopBackOff
Restarts column: Each restart is delayed exponentially up to 5 minutes

## Root Cause

In the deployment workload there's a line "command: ["/bin/false"]" that causes the process to immediately exit with an error, so the app can't start.

## Resolution

Removing the line "command: ["/bin/false"]" in the crashloop.yaml file then reapplying the file using kubectl apply -f week03-04/fixed-workloads/crashloop.yaml.
