# Best Practices for secret files

- Never expose secrets in plain text or store them where they don’t belong (like Git). In the manifest folder in github you can see the secret.yaml file, however, so can anyone else. Instead you should add the secret directly to your cluster
- Use Kubernetes secrets to store sensitive information, such as passwords, API keys, and certificates. This way, you can keep your secrets separate from your application code and manage them securely.
- Use environment variables to inject secrets into your application at runtime. This way, you can avoid hardcoding secrets in your application code and keep them secure.
- Regularly rotate your secrets to minimize the risk of exposure. This can be done manually or using automated tools that integrate with Kubernetes. 