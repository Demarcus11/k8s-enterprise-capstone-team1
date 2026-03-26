# Secrets Best Practices

In the lab, we created a secret using different methods:

- secret.yaml: This file creates a secret using YAML. In the real world, you would never commit that file because it contains sensitive data. Even though it gets encoded in base64, its not secure. Encoded doesn't mean encrypted, so anyone can easily decode it. To get the secret in the cluster, you would keep the file local and manually run `kubectl apply -f`. The advantage to this method is it helps keep a visible trail to recreate the cluster. The big disadvantage is the file may accidently be commited and the password could be exposed.

- Using `kubectl create secret` allows you to apply a secret to the cluster without it ever touching a file meaning it won't be on your hard drive. The advantage is a file can't accidently be commited. The disadvantage is there's no trail on how to recreate the cluster. You would have to remember that the cluster had a secret and what key was. Also, ArgoCD can't track it.
