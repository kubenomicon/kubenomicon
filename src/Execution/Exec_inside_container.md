# Exec Inside Container

The ability to "exec" into a container on the cluster. This is a very powerful privilege that allows you to run arbitrary commands within a container. You can ask the API server if you're allowed to exec into pods with kubectl by running: `kubectl auth can-i create pods/exec`. If you're allowed to exec into pods, the response will be `yes`.  See [RBAC](RBAC.md) for more information

Exec-ing into a pod is simple: `kubectl exec <pod_name> -- <command_to_run>`. There are a few things to know. 
1. The binary you're running must exist in the container. Containers that have been minimized using a tool such as [SlimToolkit](https://github.com/slimtoolkit/slim) will only have binaries that are needed for the application to run. This can be frustrating for an attacker as you may need to bring any tooling you need to execute. If you're attacking a pod that doesn't seem to have anything inside it, you can try utilizing [shell builtins](https://www.gnu.org/software/bash/manual/html_node/Bourne-Shell-Builtins.html) to execute some commands.
![](../images/Pasted%20image%2020240329001449.png)
2. If you can exec into a pod, you can upload files to a pod as well using `kubectl cp <local_file> <podname>:<location_in_pod>`
![](../images/Pasted%20image%2020240321102339.png)
3. When Exec-ing into a pod, you will by default exec into the first container listed in the pod manifest . If there are multiple containers in a pod you can list them using `kubectl get pods <pod_name> -o jsonpath='{.spec.containers[*].name}'` which will output the names of each container. Once you have the name of a container you can target it using kubectl with the `-c` flag: `kubectl exec -it <pod_name> -c <container_name> -- sh`
		![](../images/Pasted%20image%2020240321103439.png)
		
> Note. This is an instance where I've diverged from [Microsoft's Threat Matrix](https://microsoft.github.io/Threat-Matrix-for-Kubernetes/). I've combined the [Exec into container](https://microsoft.github.io/Threat-Matrix-for-Kubernetes/techniques/Exec%20into%20container/)  and [bash/cmd inside container](https://microsoft.github.io/Threat-Matrix-for-Kubernetes/techniques/bash%20or%20cmd%20inside%20container/) techniques into one.


# Defending
> Pull requests needed ❤️ 