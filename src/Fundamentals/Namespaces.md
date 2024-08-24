# Namespaces
Namespaces in Kubernetes are a way to segment a Kubernetes cluster, similar to how subnetting divides a network into smaller sub networks.  Namespaces are designed to share a cluster among different departments or logical groupings such as dev/test/prod. Namespaces are *not* intended to be used for workload isolation. For isolating workloads, multiple clusters should be used (although there are projects aimed at simplifying this). Namespaces provide a few other core features:
- **Resource Isolation**: Resources such as CPU and memory can be limited by namespace. This prevents one team utilizing all the the resources in a cluster. 
- **Access Control**: RBAC can be used to grant "users" access to only specific namespaces. 
- **Enforcing Network Policies**: While not really a namespace specific feature, pods can by default communicate across namespaces, but network policies can be enforced at a namespace level to prevent this.
- **Reduce Naming Confusion**: Resources cannot have the same name within a namespace, but they can have the same name as a resource in another namespace.

By default, kubernetes has 4 different namespaces, including *default*, *kube-node-lease*, *kube-public*, and *kube-system*. You can query namespaces with `kubectl get namespace`

Namespaces are a resource boundary, they should not be considered a strong security boundary. Namespaces allow you to scope [RBAC](RBAC.md). 