# Privileged container

Privileged containers represent a very dangerous permission that can be applied in a pod manifest and should almost never be allowed. Privileged pods are set under the `securityContext`. Privileged containers essentially share the same resources as the host node and do not offer any security boundary normally provided by a container. Running a privileged pod dissolves nearly all isolation between the container and the host node.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: priv-pod 
spec:
  hostNetwork: true
  containers:
  - name: priv-pod 
    image: nginx 
    securityContext:
            privileged: true
```

# Defending
From microsoft:
- Restrict over permissive containers: Block privileged containers using admission controllers
- Ensure that pods meet defined pod security standards: restrict privileged containers using pod security standards
- Gate images deployed to Kubernetes cluster: Restricted deployment of new containers from trusted supply chains

> Pull requests needed ❤️ 