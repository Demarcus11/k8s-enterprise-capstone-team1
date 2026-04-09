# Why its safer

restricted-pod.yaml is safer than insecure-pod.yaml because:

- It uses runAsNonRoot: true which prevents the root execution vulnerability. If a hacker gets into the pod, they are a low-level user and can't install software or change system settings.

- It uses allowPriviledgeEscalation: false which prevents a process from gaining more permissions than its parent.

- It uses capabilities: drop: ["ALL"] which prevents things like changing the system clock or raw networking. It strips the container of everything except the bare minimum needed.

- It doesn't use hostPath which prevents the pod from seeing the physical hard drive of the server. Without this, a pod could read the /etc/shadow file of the host and steal every password on the system.
