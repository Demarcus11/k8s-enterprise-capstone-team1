# Validation Logic

When the CI pipeline runs and reaches the "Validate Kubernetes Manifests" stage, it executes a containerized security scan using `ghcr.io/aquasecurity/trivy`. This validates the deployment files against the CIS (Center for Internet Security) benchmarks (:latest tag, no resource limits, missing probes). If a deployment fails a benchmark then the pipeline stage fails with code 1 and the bad deployments never reach the cluster.
