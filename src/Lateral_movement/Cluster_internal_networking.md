# Cluster Internal Networking
By default, Pods in a cluster can communicate with each other if there are no network policies in place preventing this. This allows pods to communicate even across namespaces.

In the following example, the pod IP for `my-nginx-pod` is obtained by running `kubectl get pod my-nginx-pod -o custom-columns=NAME:metadata.name,IP:status.podIP`
![](../images/Pasted%20image%2020240404143403.png)

To demonstrate that we can reach this pod from the `dmz` namespace, the command `kubectl exec -it tcpdump -n dmz -- wget -O - 10.244.0.52` is ran. The returned information is the default nginx webpage. 
![](../images/Pasted%20image%2020240404143448.png)


# Defending
This can be "fixed" by implementing network policies
> Pull requests needed ❤️ 

# References and resources
- [Kubernetes Networking](https://www.tigera.io/learn/guides/kubernetes-networking/)
- [Kubernetes Networking Fundamentals](https://opensource.com/article/22/6/kubernetes-networking-fundamentals)
- [Kubernetes Threat Matrix](https://kubernetes-threat-matrix.redguard.ch/lateral-movement/cluster-internal-networking/)