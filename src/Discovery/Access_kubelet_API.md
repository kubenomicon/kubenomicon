# Access Kubelet API
The Kubelet is an agent that runs on a Kubernetes node. This is the component that established communication between the node and API server.  The Kubelet doesn't manage containers that were not created by Kubernetes, however, it can create Static Pods. See [staticpods](). Once the pod has been scheduled on a node, the Kubelet running on that node picks it up and takes care of actually starting containers.

Depending on the cluster, the settings defining authorization and authentication to the Kubelet API server can be found in various locations such as `/etc/systemd/system/kubelet.service.d/10-kubeadm.conf`. 
## Kubelet Authentication
By default, requests that are not outright rejected are given the username `system:anonymous` and a group of `system:unauthenticated`. To disable anonymous authentication, start the kubelet with the `--anonymous-auth=false` flag. 
## Kubelet Authorization
The Kubelet can serve a small REST API with read access on port 10250. Make requests to the a kubelet API to: 
	- Run commands (possibly interactively) in a different pod
    - Start a new pod with privileges and node filesystem/resource access

Any request that is authenticated (even anonymous requests) is then authorized. Some information expanded upon from [Liz Rice's Github Gist](https://gist.github.com/lizrice/c32740fac51db2a5518f06c3dae4944f)
- If `--anonymous-auth` is turned off, you will see a `401 Unauthorized` response.  
![](../images/Pasted%20image%2020240401151622.png)
- If `--anonymous-auth=true` but `--authorization-mode`is not set to `AlwaysAllow`, you will get a `Forbidden (user=system:anonymous, verb=get, resource=nodes, subresource=proxy)`  response.
![](../images/Pasted%20image%2020240401151631.png)
- If `--anonymous-auth=true` and `--authorization-mode=AlwaysAllow`  you'll see a list of pods.  
![](../images/Pasted%20image%2020240401152205.png)

When making changes, restart the systemd service with `sudo systemctl daemon-reload ; sudo systemctl restart kubelet.service`


# Attacking 
As an attacker, you can attempt to run a curl command against the Kubelet running on a node. If you can do this, it's pretty much game over for the node. What we're looking for is a kubelet with the both the flags `--anonymous-auth=true` and `--authorization-mode=AlwaysAllow` to be passed to the Kubelet startup command.

Attempt to communicate with kubelet by running: `curl -sk https://<node ip>:10250/runningpods/ | jq`. If successful, a ton of JSON will be returned. 

If  the message returned message is`Forbidden (user=system:anonymous, verb=get, resource=nodes, subresource=proxy)` or simply `Unauthorized`, the Kubelet probably has `--anonymous-auth=false` and/or does not have `--authorization-mode=AlwaysAllow` set and thus you cannot communicate with the Kubelet API.

If pods ARE returned, there is a lot of useful information such as namespace data that can show you new namespaces you previously didn't know about. 

If you don't have JQ and you don't want to upload it, you can do some funky parsing with `sed` to make things more legible `curl -sk https://192.168.49.2:10250/runningpods/ | sed s/,/\\n/g | sed s/^{/\\n/`.
  ![](../images/Pasted%20image%2020240401145250.png)
Using the `pod name`,`namespace`, and `container_name` data from the previous curl command you can attempt to execute commands on the pod by running: `curl -sk https://<master node ip>:10250/run/<namespace>/<pod_name>/<container_name>/ -d "cmd=id"`. This will attempt to run the `id` command (Or any command you wish, just make sure the binary is actually on the pod.)

![](../images/Pasted%20image%2020240401145846.png)


# Defending
> Pull requests needed ❤️ 