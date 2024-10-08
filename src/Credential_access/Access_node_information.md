# Access node information
Kubernetes nodes often store sensitive information that should not be accessible from within a pod. If an attacker has access to files on a node, they may be able to use these information identified for various other techniques such as [Privilege Escalation](../Privilege_escalation.md). 

Accessing node information requires either one of the following:
1. A Container breakout vulnerability
2. Kubernetes misconfiguration

Having full read access to a node's filesystem is dangerous as this gives an attacker access to read the [overlay2](../overlay2.md) storage driver and access other sensitive information stored on nodes. Much of the information of value is stored in `/etc/kubernetes`. If able to access a node, normal Linux privilege escalation techniques apply such as searching for credentials. 

[Dredge](https://github.com/grahamhelton/dredge) can be used to search for secrets on a node.

![](../images/Pasted%20image%2020240331135730.png)

# Interesting node files and directories 
The locations of some potentially interesting files: Note this can vary greatly depending on the implementation details of a cluster. Note this list is not exhaustive:
- `/etc/kubernetes/`: The default place to store Kubernetes specific information
- `/etc/kubernetes/azure.json` When on an AKS cluster, default location where service principles are stored.
- `/etc/kubernetes/manifests`: The default place to store manifests [See Persistence -> Static Pods](../Persistence/Static_pods.md)
- `/var/lib/kubelet/*`:  Files used by the kubelet

# Defending
> Pull requests needed ❤️ 