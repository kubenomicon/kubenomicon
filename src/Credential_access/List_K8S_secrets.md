# List K8S secrets
Listing Kubernetes secrets is a bit unintuitive. A "list" of all secrets (in a namespace) can be created by running `kubectl get secrets` This access is granted by the RBAC `list` verb.

In practice this looks relatively safe, however, an attacker can simply output the metadata associated with the secret as YAML or JSON and access the actual secret.

![](../images/Pasted%20image%2020240331130924.png)

The RBAC `get` verb refers to getting a specific secret. The actual secret can be queried with `kubectl get secret <secret_name> -o yaml | yq .data.<secret_field> | base64 -d`

![](../images/Pasted%20image%2020240331125207.png)

A one liner from, [AntiTree](https://www.antitree.com/2020/11/when-list-is-a-lie-in-kubernetes/) to "dump all the ClusterRoles that have LIST but not GET permissions. The thought is that if you have LIST without GET, you’re attempting to restrict access to secrets but you’re going to be sorely mistaken."

```bash
kubectl get clusterroles -o json |\
    jq -r '.items[] | select(.rules[] |
    select((.resources | index("secrets")) 
    and (.verbs | index("list")) 
    and (.verbs | index("get") | not))) |
    .metadata.name'
```

Like most resources, secrets are namespace scoped.
![](../images/Pasted%20image%2020240331134143.png)

# Defending
> Pull requests needed ❤️ 