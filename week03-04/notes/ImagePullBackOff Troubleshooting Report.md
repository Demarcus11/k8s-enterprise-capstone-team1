# Troubleshooting Report

## Issue Summary

image-pull-loop-demo workload failing to run.

## Symptoms

Pod starts with status ErrImagePull, transitions to status ImagePullBackOff, number of restarts don't increase.

## Investigation

Apply workload: kubectl apply -f week03-04/broken-workloads/image-pull-loop.yaml
View pods: kubectl get pods

Status column: ErrImagePull then transistions to ImagePullBackOff

## Root Cause

In the deployment workload, the image defined isn't an image on DockerHub, so the worker node can't pull it. The app never starts thus restarts is never incremented.

## Resolution

Replacing the line "image: fakeimage" in image-pull-loop.yaml with a real image such as nginx.
