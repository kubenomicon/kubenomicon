# Kubeconfig file
The Kubeconfig file is a configuration file that contains all the information `kubectl` needs to access the cluster. This includes information such as where the API server is, which credentials to use to interact with it, default namespaces, etc. You can change which Kubeconfig file you're using by setting the `$KUBECONFIG` environment variable.

Should an attacker gain access to a Kubeconfig file, they can instruct Kubectl to use it to access the cluster. `export KUBECONFIG=/path/to/kubeconfig`. Note that this file is typically just called `config` and stored in `~/.kube/config` but these can be left in many different places so it's worth hunting for them.

![](../images/2024-03-20_23-46.png)

The following is an example of what a Kubeconfig YAML file looks like:
```yaml
apiVersion: v1
# Holds information on how to access the cluster
clusters:
  - cluster:
  # The API server's public key. Does not need to be kept secret
      certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURCakNDQWU2Z0F3SUJBZ0lCQVRBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwdGFXNXAKYTNWaVpVTkJNQjRYRFRJek1EY3lOREU0TXpJMU1Gb1hEVE16TURjeU1qRTRNekkxTUZvd0ZURVRNQkVHQTFVRQpBeE1LYldsdWFXdDFZbVZEUVRDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTUFECitnZDVoUEk5VmorNFk3Q25ZcDRDTnZBVkpGNGE5eWVrYUhkbWJaN0Mzby8zZ0xNT29CWDFEdktMbFh0WVFxaXcKUEpuYk1LMFJFNGI2QzM5K3laN3V4aTdNZGllc2xHYmdPRitLNnMvb2xBOExHdnk4R3V6Zmk3T1RlaFRacFF6VAoraGFzaFlLNFRJYU5KNGtTOUN0dFd6VzJVa243cHNxNWpFa0l0eFpGdnpWblhwYVNPQVZVOEJRSm1rMzhQUXIxCm5nVzdJbkdiNFNQcGZOWVlrVURUOEVzWG10eElJdWU5ZmJ2aThPM0E1eTNFSVB0ZkJDdk45M3paUFRIK0RyVTkKRUduYkhqWlVlQ3hGa1E1QmtMMjVjcTh2UVoyZWhtb3d6a1Z1dVM3SGUyZTFUOHNuc01uanpwaGtoV2NMMDRiWApPTHhmYy8wRER0VVJEV0pYdnRNQ0F3RUFBYU5oTUY4d0RnWURWUjBQQVFIL0JBUURBZ0trTUIwR0ExVWRKUVFXCk1CUUdDQ3NHQVFVRkJ3TUNCZ2dyQmdFRkJRY0RBVEFQQmdOVkhSTUJBZjhFQlRBREFRSC9NQjBHQTFVZERnUVcKQkJTTDQyYkEydVdsWnVzSHFOYUdqd0RwM21CRHNqQU5CZ2txaGtpRzl3MEJBUXNGQUFPQ0FRRUFDVEZjc2FaaAp4bjVNZTI2bkN1VVNjSDEzbFhKSjFSOXJPLzNXRDE0cUZrMERET2ZMVkZBdkR1L0xWU2ZIVkF5N0dSYWJCOWNUCmVXNndDV3JhUy9aczFDYXVMOG8vTVdoWG9VWUtHc0IxNVE0R21VUzBLMXV4L2ZNUUlZczVUNUJmU0UrLzBsQ0EKL2hINWRVaDMraklSa1ZhVVZBbDFxL3VQR0dIRXlqWGNMdlp5TGVmSENTMlJWbFU5SS9xb2FkQTd2ZE5US3VTNwpYOUZhZjdNNUxMYXRzNldraWRXd3BrS3FDQ3Z2YlhNck85SmFobXhrbFZvamhYYUZTQkNuSWpaQUIzQ2JTSWNBClpRWFNBTVlaTWZBSUNEYTF3eW1jM1dXUUZQVlZ0NUpubHd3WWx3TlVpTk9GdUJqZUtMMTUvSDZyS3VRdktHbkcKMmdVRUphUFV4WS93U0E9PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
	  # API Server Address
      server: https://192.168.59.101:8443
    name: dev-cluster
  - cluster:
      certificate-authority: /home/smores/.minikube/ca.crt
      extensions:
        - extension:
            last-update: Mon, 18 Mar 2024 14:44:21 EDT
            provider: minikube.sigs.k8s.io
            version: v1.30.1
          name: cluster_info
      server: https://192.168.49.2:8443
    name: minikube
# Which Cluster, user, and namespace to access by default
contexts:
  - context:
      cluster: minikube
      extensions:
        - extension:
            last-update: Mon, 18 Mar 2024 14:44:21 EDT
            provider: minikube.sigs.k8s.io
            version: v1.30.1
          name: context_info
      namespace: default
      user: minikube
    name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
  # Which user to authenticate to the cluster as
  - name: minikube
    user:
	# Contains a cert for the user signed by the kubernetes CA. This IS sensitive. Sometimes a token is used instead (such as service accounts)
      client-certificate: /home/smores/.minikube/profiles/minikube/client.crt
      client-key: /home/smores/.minikube/profiles/minikube/client.key

```

You can utilize [Dredge](https://github.com/grahamhelton/dredge) to search for Kubeconfig files.

![](../images/Pasted%20image%2020240328224853.png)


# Switching Contexts
Kubeconfig files allow you to set multiple "contexts". Each context may have different RBAC permissions. In the following example, the `admin` user has full admin permissions as denoted by the `kubectl auth can-i --list | head` command displaying all RBAC verbs for all resources (piped to head for brevity). 

Upon switching to the `dev` context using `kubectl config use-context dev`, and re-running `kubectl auth can-i --list | head`, the RBAC permissions for the dev context are displayed which are far less permissive.
![](../images/Pasted%20image%2020240321123344.png)


# Defending

Assuming an attacker has gained access into your cluster, they are still limited to the permissions granted to the credentials within the kubeconfig file. This is your **most crucial line of defense:**

- **Minimize Cluster-Admin Kubeconfigs:** Never have more than a few absolutely necessary kubeconfigs with cluster-admin privileges.

```yaml
apiVersion: v1
kind: Config
clusters:
- cluster:
    certificate-authority-data: <CA_CERTIFICATE_DATA>
    server: <YOUR_KUBERNETES_API_SERVER_ADDRESS>
  name: my-cluster
contexts:
- context:
    cluster: my-cluster
    user: namespace-admin
    namespace: my-namespace 
  name: namespace-admin-context
current-context: namespace-admin-context
users:
- name: namespace-admin
  user:
    token: <SERVICE_ACCOUNT_TOKEN>
```

Above is an example of a really BAD kubeconfig file found by a threat actor, a kubeconfig file with cluster-admin privileges grants essentially unlimited power over your Kubernetes cluster. It's like having the root password for your entire infrastructure.

Let’s take a look at the cluster-role and cluster-rolebinding for the `namespace-admin` user:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: my-namespace
  name: namespace-admin
rules:
- apiGroups: ["", "apps", "extensions", "batch"] # Include common API groups
  resources: ["*"] # Allow access to all resources within the namespace
  verbs: ["*"]     # Allow all actions (get, list, create, update, delete, etc.)

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: namespace-admin-binding
  namespace: my-namespace
subjects:
- kind: ServiceAccount
  name: namespace-admin  # Match the ServiceAccount name in your kubeconfig
  namespace: my-namespace
roleRef:
  kind: Role
  name: namespace-admin
  apiGroup: rbac.authorization.k8s.io
```

**Excessive Permissions:**

* The `namespace-admin` user doesn't need cluster-wide permissions. 
* They should only have access to resources within their designated namespace (`my-namespace`).

**Security Risk:**

* If the `namespace-admin` kubeconfig is compromised, the attacker inherits these excessive permissions, enabling them to:

    * **Manipulate any resource:** Create, delete, or modify pods, deployments, services, etc., disrupting applications or deploying malicious workloads.
    * **Steal sensitive data:** Access secrets containing passwords, API keys, and other confidential information.
    * **Alter cluster settings:** Modify security policies, resource quotas, and network configurations, potentially compromising the entire cluster's integrity.

**Best Practice:**

* **Principle of Least Privilege:** Grant only the minimum necessary permissions. Create Roles and RoleBindings specific to the `my-namespace` namespace, restricting the `namespace-admin` user's access accordingly.
* **Limit Cluster-Admin:** Minimize the number of kubeconfigs with cluster-admin privileges and protect them with utmost care.

By adhering to these principles, you significantly reduce the potential damage from compromised credentials, even if an attacker manages to gain initial access to your cluster.
