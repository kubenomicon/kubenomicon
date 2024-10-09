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

To bolster the security of your Kubernetes environment, it's essential to embrace a Zero Trust model. This means inherently trusting no user or service, requiring verification at every access attempt.  This principle should guide your approach when defining roles and role bindings for both automated systems and human users, leveraging the `Roles` and `RoleBindings` resources within Kubernetes.  For a detailed walkthrough of this process, refer to the "Defending" section further down in this document.

While prevention is crucial, a robust detection strategy is equally important.  Fortunately, powerful open-source tools like Falco are readily available to help you monitor and identify suspicious activities within your cluster.

Falco is a runtime security tool that monitors container activity and detects anomalous behavior.

Falco can run as a:

* DaemonSet
* Deployment
* Sidecar container (less common)

The most common and recommended approach is to run as a DaemonSet — a service —  as it allows Falco to observe the activity of all containers across your infrastructure.

The `/etc/falco` directory is a crucial location for configuring and operating. The file you should generally only modify is `falco_rules.local.yaml`.

An important point to know: If a later rule has the same conditions as an earlier rule, the later rule takes precedence. This means it can effectively "override" the earlier rule.

Let’s take a look for ourselves. We’re going to deliberately set off the alerts using Falco:

![](../videos/falco-demo.mov)

**The Setup**

*   **Two terminals:** 
    1. One terminal displays the Falco logs (`journalctl -fu falco`), providing real-time insights into security events. 
    2. The other terminal is used to interact with your Kubernetes cluster.

*   **Falco monitoring:** Falco is running in the background, as a DaemonSet, monitoring your Kubernetes cluster for suspicious activities.

**The Actions**

1.  **Create a pod:** I used `kubectl create` to create a new pod in the cluster. Falco saw this API request to the Kubernetes API server.
2.  **Exec into the pod:** I used `kubectl exec` to get a shell inside the running container. Falco detected this action as potentially risky, as it involves accessing a container's internal environment.
3.  **Delete the pod:** I used `kubectl delete` to remove the pod. Falco observed this action as part of its monitoring of the Kubernetes API server.

**Why Falco reacted**

Falco has built-in rules that trigger on these actions —  as we discussed above — because they can be indicative of malicious activity:

*   **Pod Creation:** Attackers might try to create pods to run malicious workloads.
*   **Exec into a pod:** This is a common technique for attackers to gain access to a container and execute commands.
*   **Pod Deletion:** Attackers might delete pods to disrupt services or cover their tracks.

**The Output**

In the terminal running `journalctl -fu falco`, you see Falco's output, which likely includes:

*   **Timestamp:** When the event occurred.
*   **Rule:** The Falco rule that was triggered (e.g., a rule related to `kubectl exec`).
*   **Severity:** The severity level of the event (e.g., "warning" or "critical").
*   **Details:** Information about the event, such as the pod name, namespace, user, and the specific command executed.

**The Takeaway**

This demo effectively illustrates how Falco can detect potentially risky actions within your Kubernetes environment. By monitoring the Kubernetes API server and container activity, Falco provides valuable visibility and helps you identify and respond to security threats.
