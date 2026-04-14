# Why its least priviledge

Least Priviledge is giving the least amount of permissions necessary to perform a specific task.

- serviceaccount.yaml defines a user ID in the dev namespace. The name is readonly-sa.

- readonly-role.yaml defines the permissions that user ID has. The permissions defined are get and list pods in the dev namespace.

- readonly-rolebinding.yaml attaches the permissions to that user ID. The permissions to get and list pods in the dev namespace are given to readonly-sa.

This follows least priviledge because readonly-sa is bound to the dev namespace (can't see prod or staging), its resources are restricted to pods (can't do anything with configmaps or secrets), and it can only get and list pods (can't modify or delete pods). This gives readonly-sa the least amount of permissions to do its job.
