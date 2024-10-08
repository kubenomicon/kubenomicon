# Exposed Sensitive interfaces
Some services deployed in a kubernetes cluster are meant to only be accessed by Kubenetes admins. Having them exposed and/or having weak credentials on them can allow an attacker to access them and gain controol over them. Depending on the service, this can allow the attacker to do many different things. [Microsoft calls out the following as sensitive interfaces they've seen exploited](https://www.microsoft.com/en-us/security/blog/2021/03/23/secure-containerized-environments-with-updated-threat-matrix-for-kubernetes/): Apache NiFi, Kubeflow, Argo Workflows, Weave Scope, and the Kubernetes dashboard.

This is essentially a management interface for kubernetes.

![](../images/Pasted%20image%2020240328225334.png)

## Defending
Ensure the sensitive interfaces are not accessible by those who do not need them.

![](../images/Pasted%20image%2020240328225356.png)

Limiting access to specific namespaces as shown in the YAML file below:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: kubernetes-dashboard
  name: saeds-dashboard-viewer
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```
This Kubernetes Role grants a user or service account named "saeds-dashboard-viewer" read-only access to pods within the "kubernetes-dashboard" namespace. This likely allows "saed" to view pod information via the Kubernetes Dashboard but not modify them.

 A simple way to check is by running `kubectl get pods -A` and look for the dashboard a more nuanced way `kubectl auth can-i list --as saeds-dashboard-viewer -n kubernetes-dashboard`.
