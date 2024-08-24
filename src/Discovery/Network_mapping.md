Network mapping in kubernetes is fundamentally the same concept as network mapping _any_ network. There are a few unique challanges when it comes to mapping a network within a kubenetes cluster depending on your level of access. Get IP of pod if for some reason you need to: `k get pod ssh -o custom-columns=NAME:metadata.name,IP:status.podIP`


# Kubernetes IP Address Ranges

- The network plugin is configured to assign IP addresses to Pods.
- The kube-apiserver assigns IP addresses to Services
- The Kubelet (or cloud-controller-manager) assigns IPs to a node. 

## Container to Container Networking
For networking, containers within a pod behave as if they are on the same host. They can all reach each other's ports on localhost since they share some resources (including volumes, cpu, ram, etc). 

# Service Discovery
Services can be discovered in all namspaces with `kubectl get services -A`. Once getting a list of services, you can query the manifest of a services by running `kubectl get service <service_name> -o yaml`. This will give you an idea of what port's the service is running on. In this case, the nginx server was running on port 80. To connect to this service, the command `kubectl port-forward service/<service_name> 8080:80` can be run which maps port 8080 on our local machine to the service's port 80. In this case, it's an nginx webpage that we can reach by navigating to it in our browser at `127.0.0.1:8080`

![](../images/Pasted%20image%2020240402111003.png)

![](../images/Pasted%20image%2020240402111305.png)

# Defending
> Pull requests needed ❤️ 