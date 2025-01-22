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

## seccomp Profiles
We can restrict the system calls available to the container through seccomp profiles, which will prevent many of the container breakout methods used due to the availability of risky system calls (e.g. SYS_PTRACE used to inject malicious processes, SYS_MODULE to load malicious kernel modules into the container).
The default seccomp profile [blocks a significant number](https://docs.docker.com/engine/security/seccomp/#significant-syscalls-blocked-by-the-default-profile) of system calls used in container breakouts that will typically be present within privileged containers.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: seccomp-protected-pod
  labels:
    app: seccomp-protected-pod
spec:
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: seccomp-container
    image: nginx:latest
```

> Pull requests needed ❤️ 