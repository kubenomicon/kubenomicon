# RBAC
The kubernetes Role-based access conrol (RBAC) system is probably one of the most important security controls available within kubernetes. 
```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: pod-copy
rules:
- apiGroups: [""]
  resources: ["pods/exec"]
  verbs: ["create"]
```

## RBAC
There is one interesting quirk - Kubernetes doesn't have users. Well, at least not in the standard sense. A "user" in Kubernetes is simply a identity. To denote these *identities*, Kubernetes uses certificates and only trusts certificates signed by the Kubernetes CA. To create an identity, you first create a client certificate and set the common name to the identity you wish to create. You then use the Kubernetes Certificate Authority to cryptographically sign the client certificate that represents an identity.
## Creating an RBAC user
1. Generate a key: `openssl genrsa -out dev.key 2048`
2. Next, generate a certificate singing request (CSR) with the identity of "dev" and a group of "developer": `open ssl -new -key dev.key -out dev.csr -subj"/CN=dev/O=developer`
3. Sign the CSR using the Kubernetes CA's certificate with a validity period of 30  days: `openssl x509 -req -in dev.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out dev_cert_signed.crt -days 30`
Now that this certificate has been signed by the Kubernetes CA, it can be embedded in a Kubeconfig file and a user with access to that kubeconfig file can assume the identity of `dev`. This can be used if you, for example, need to create an `Auditor` role that has read only access to the cluster.


RBAC is then applied to an identity using 2 of the 4 following concepts, *Role* and *ClusterRole* or *RoleBinding* and *ClusterRoleBinding*. 
- **Role**: A role is a set of additive permissions. Anything that is not explicitly permitted is denied.
```yaml
# Example modified from: https://kubernetes.io/docs/reference/access-authn-authz/rbac/
# The version of the kubernetes API you're using
apiVersion: rbac.authorization.k8s.io/v1
# The type of object to create. In this case, a role.
kind: Role # Or ClusterRole
metadata:
  # The namespace your role is allowed access to. Omit if ClusterRole
  namespace: default
  # The name of the role
  name: pod-reader
rules:
# Which API group can be accessed. "" indicates the core API group
- apiGroups: [""] 
  # The resource able to be accessed with verbs 
  resources: ["pods"]
  # What the role is allowed to access with kubectl
  verbs: ["get", "watch", "list"]
```
- **ClusterRole**: The same as a role, but is not "namespaced" which means ClusterRoles apply to the entire cluster, making them more powerful.
```yaml
# Example modified from: https://kubernetes.io/docs/reference/access-authn-authz/rbac/
# The version of the kubernetes API you're using
apiVersion: rbac.authorization.k8s.io/v1
# The type of object to create. In this case, a role.
kind: ClusterRole 
metadata:
  name: pod-reader
rules:
# Which API group can be accessed. "" indicates the core API group
- apiGroups: [""] 
  # The resource able to be accessed with verbs 
  resources: ["pods"]
  # What the role is allowed to access with kubectl
  verbs: ["get", "watch", "list"]
```
- **RoleBinding**: Binds the permissions defined in a *Role* to an identity.
```yaml
# Example modified from: https://kubernetes.io/docs/reference/access-authn-authz/rbac/
# The version of the kubernetes API you're using
apiVersion: rbac.authorization.k8s.io/v1
# This role binding allows "jane" to read pods in the "default" namespace.
kind: RoleBinding # Or ClusterRoleBinding
metadata:
  name: read-pods
  # Omit if ClusterRoleBinding
  namespace: default 
subjects:
# You can specify more than one "subject"
- kind: User
  name: jane 
  apiGroup: rbac.authorization.k8s.io
roleRef:
  # this must be Role or ClusterRole
  kind: Role 
  # this must match the name of the Role or ClusterRole you wish to bind to
  name: pod-reader 
  apiGroup: rbac.authorization.k8s.io
```
- **ClusterRoleBinding**: The same as a *RoleBinding*, but for *ClusterRoles*
```yaml
# Example modified from: https://kubernetes.io/docs/reference/access-authn-authz/rbac/
# The version of the kubernetes API you're using
apiVersion: rbac.authorization.k8s.io/v1
# This role binding allows "jane" to read pods in the "default" namespace.
kind: ClusterRoleBinding
metadata:
  name: read-pods
subjects:
# You can specify more than one "subject"
- kind: User
  name: jane 
  apiGroup: rbac.authorization.k8s.io
roleRef:
  # this must be Role or ClusterRole
  kind: ClusterRole 
  # this must match the name of the Role or ClusterRole you wish to bind to
  name: pod-reader 
  apiGroup: rbac.authorization.k8s.io
```

Roles and cluster roles can be queried for using `kubectl get roles,clusterroles | grep -v system`. Note that it might be useful to remove the "system" accounts using `grep -v system`
![](src/images/Pasted%20image%2020240325022307.png)


# RBAC Limitations
RBAC permissions granted to a resource type do not have a way of determining which sub-resources you're targeting. For example, there is no way in RBAC to say "Allow the user to only get the secret named my-secret".  Granting the permissions to get a secret means a user can get *all* secrets within that namespace.

Additionally, there is no field-level access control that would allow a user to edit only one field of a manifest. For example, to edit the namespace field of a manifest, a user would need full write permission to the manifest. 
# References
- [Find RBAC associated with service accounts](/Persistence/Container_service_account.html#find-rbac-associated-with-service-accounts)