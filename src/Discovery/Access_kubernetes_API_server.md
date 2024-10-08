# Access Kubernetes API server
The kubernetes API server is the central communication channel with the cluster. This is what kubectl interacts with (although you can interact with it directly by sending REST requests using something such as curl). 

The API server can be found by looking for the `KUBERNETES_PORT_443_TCP` environment variable inside of a pod. Without proper permissions you can't do much by accessing the API, but you should still ensure that only trusted machines can talk to it.

![](../images/Pasted%20image%2020240401115238.png)
- Image from: https://kubernetes.io/docs/concepts/security/controlling-access/#api-server-ports-and-ips 

# Transport security 
The official documentation page has a great overview: 

> By default, the Kubernetes API server listens on port 6443 on the first non-localhost network interface, protected by TLS. In a typical production Kubernetes cluster, the API serves on port 443. The port can be changed with the `--secure-port`, and the listening IP address with the `--bind-address` flag.
> 
> The API server presents a certificate. This certificate may be signed using a private certificate authority (CA), or based on a public key infrastructure linked to a generally recognized CA. The certificate and corresponding private key can be set by using the `--tls-cert-file` and `--tls-private-key-file` flags.
>
> If your cluster uses a private certificate authority, you need a copy of that CA certificate configured into your `~/.kube/config` on the client, so that you can trust the connection and be confident it was not intercepted.


## Authentication
Authentication then takes place using client certificates, bearer tokens, or an authenticating proxy to authenticate API requests. Any identity that creates an API call using a valid certificate signed by the cluster's CA is considered authenticated. [The documentaion](https://kubernetes.io/docs/reference/access-authn-authz/authentication/) has much more information on Kubernetes authentication.

## Authorization
All API requests are evaluated using the API server. Permissions are denied by default. There are a few different authorization modes that can be used by the API server:
- [Node](https://kubernetes.io/docs/reference/access-authn-authz/node/):  Special mode used by kubelets
- [RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/): The most common access control method. Grants roles or users access to resources.
- [ABAC](https://kubernetes.io/docs/reference/access-authn-authz/abac/): A more granular (and complex) access control system than RBAC. Can be used with RBAC. Enabled by adding `--authorization-mode=ABAC` to the API server manifest (often found in `/etc/kubernetes/manifests/kube-apiserver.yaml`). [Great overview here](https://overcast.blog/mastering-kubernetes-attribute-based-access-control-abac-bb6a732cd561)
- [Webhook](https://kubernetes.io/docs/reference/access-authn-authz/webhook/): Causes Kubernetes to query an outside service to determine user privileges. 

## Admission Control
After authentication and authorization, any admission controllers that are present act on the request. For example, if an admission controller is put into place that disallows privileged pods from being created, attempting to create a privileged pod will be stopped after authentication and authorization occurs.

# Enumeration & Situational Awareness
During an engagement, it's possible to land on a machine that is using a Kubeconfig (or one is found). To see where the API server is in the context of the Kubeconfig being used, the command `kubectl config view --raw` can be used to view the Kubeconfig file. Additionally, `kubect config current-context` will return the cluster name.

![](../images/Pasted%20image%2020240401131919.png)

# Defending
> Pull requests needed ❤️ 