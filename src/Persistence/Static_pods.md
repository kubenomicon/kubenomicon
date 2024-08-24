Static pods in Kubernetes are interesting from an attacker perspective because the are created without needing the API server. A static pod is instead managed by the kubelet service running on a node.

With that being said, if a static pod is created, the kubelet will try to create a **mirror pod** on the API server, but the pod cannot be controlled by the API server. Static pods have the name of the node they're running on appended to the end of them. By default, the kubelet watches the directory `/etc/kubernetes/manifests` for new manifests. If an attacker is able to somehow place a manifest inside this directory, it will be run (although sometimes you may need to restart the kubelet).

> Note: This bypassess admission controllers

Static pods cannot be used to do things such as mount secrets.

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

![](../images/Pasted%20image%2020240331195846.png)
![](Pasted%20image%2020240331195849.png)

# Defending
> Pull requests needed ❤️ 