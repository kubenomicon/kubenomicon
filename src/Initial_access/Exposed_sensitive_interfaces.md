# Exposed Sensitive interfaces
Some services deployed in a kubernetes cluster are meant to only be accessed by Kubenetes admins. Having them exposed and/or having weak credentials on them can allow an attacker to access them and gain controol over them. Depending on the service, this can allow the attacker to do many different things. [Microsoft calls out the following as sensitive interfaces they've seen exploited](https://www.microsoft.com/en-us/security/blog/2021/03/23/secure-containerized-environments-with-updated-threat-matrix-for-kubernetes/): Apache NiFi, Kubeflow, Argo Workflows, Weave Scope, and the Kubernetes dashboard.

This is essentially a management interface for kubernetes.

![](../images/Pasted%20image%2020240328225334.png)
## Defending
Ensure the sensitive interfaces are not accessible by those who do not need them. A simple way to check is by running `kubectl get pods -A` and look for the dashboard. 

![](../images/Pasted%20image%2020240328225356.png)

> Pull requests needed ❤️ 