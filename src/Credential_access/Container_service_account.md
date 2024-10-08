# Container service accounts
When a pod makes a request to the API server, the pod authenticates as a Service account. You can inspect which service account a pod is using by querying the API server. `kubectl get pods/<pod_name> -o yaml | yq .spec.serviceAccountName`. 
![](../images/Pasted%20image%2020240322112135.png)

If a service account is not set in the manifest, Kubernetes automatically sets it which can be accessed from inside the pod at one of the following locations:

```bash
/run/secrets/kubernetes.io/serviceaccount
/var/run/secrets/kubernetes.io/serviceaccount
/secrets/kubernetes.io/serviceaccount
```

A manifest can opt out of mounting a service account by specifying in either the Pod or ServiceAccount manifest but the Pod spec takes precedence:
```yaml
# ServiceAccount manifest disabling automounting
# Manifest from: https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/
apiVersion: v1
kind: ServiceAccount
metadata:
  name: build-robot
automountServiceAccountToken: false

```

```yaml
# Pod manifest disabling automounting
# Manifest from: https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  serviceAccountName: build-robot
  automountServiceAccountToken: false
  ...
```

Service accounts are namespace specific and can be listed with `kubectl get serviceaccount -n <namespace>` 

By default, the service account granted to pods in the `kube-system` namespace grants full access to all resources. Service account permissions can be can be verified with the following kubectl command. The below command runs `kubectl auth can-i --list` using the service account tokens/certificates/namespace mounted in the Pod's default locations. You may need to change the location of `--token`,`--certificate-authhority`, and `-n` if the secret is in a non-standard location.
```bash
# Run kubectl and grab the service account tokens/certificate/namespace
# from their default locations. You may need to alter this if they're in
# non standard locations
./kubectl auth can-i --list \
--token=$(cat /run/secrets/kubernetes.io/serviceaccount/token) \
--certificate-authority=/run/secrets/kubernetes.io/serviceaccount/ca.crt \
-n $(cat /run/secrets/kubernetes.io/serviceaccount/namespace)
```

The following RBAC shows that this ServiceAccount in the `kube-system` namespace has full access to all resources and verbs.

![](../images/Pasted%20image%2020240322115345.png)

In contrast, the default ServiceAccount in a namespace that is not `kube-system` does not have any useful RBAC permissions.

![](../images/Pasted%20image%2020240322115957.png)

This ServiceAccount has `get`,`list`, and `watch` permissions for the `pod` resource.
![](../images/Pasted%20image%2020240322122020.png)

# Find RBAC associated with service accounts
Sometimes it's useful to find the RBAC assocaited with a ServiceAccount. Run the following command replacing `REPLACEME` with the name of the service account you wish to view the RBAC verbs for.
```bash
# Remember to replace REPLACEME
kubectl get rolebinding,clusterrolebinding \
--all-namespaces -o \
jsonpath='{range .items[?(@.subjects[0].name=="REPLACEME")]}[{.roleRef.kind},{.roleRef.name}]{end}'
```

The output will show the Role that is applied binded to this service account. The RBAC associated with that rule can be queried with `kubectl describe <name>`

![](../images/Pasted%20image%2020240322122528.png)

Alternatively you can run `kubectl get rolebindings,clusterrolebindings --all-namespaces -o wide | grep <ServiceAccountName>` but the output is very large. 

# ServiceAccount API Tokens
Additionally an API token can be created for a service account that can be used to authenticate. 
![](../images/Pasted%20image%2020240322124414.png)

# Defending
> Pull requests needed ❤️ 