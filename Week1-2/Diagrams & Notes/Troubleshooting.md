# Troubleshooting

Issue: ResourceNotFound error when trying to find cluster. Running "aws configure get region" to verify the region.

Issue: Can't find created pods. Checked default namespace instead of dev namespace. Ran "kubectl config set-context --current --namespace-dev" to make the dev namespace the default.
