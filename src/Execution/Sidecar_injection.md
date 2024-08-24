# Sidecar injection
Pods are comproised of one or more containers. A sidecar container is a container that can be launched in a pod with other containers. This is commonly used for 3rd party programs that do things such as collect logs or configure network proxies.

In the following scenario there is an nginx server called `main-application`. The main application (in this case  `nginx`) will eventually output some logs to `/var/log/nginx`. The problem is that we don't have a way to collect those logs to send to something such as a SIEM. A solution to this would be to mount the path `/var/log/nginx` and then launch a side car container that is responsible for collecting the logs from `/var/log/nginx`. In this example, a simple `busybox` container is started that prints the log files to the screen every 30 seconds. This is a contrived example, but the sidecar could do any number of things.

```yaml
# Modified from https://www.airplane.dev/blog/kubernetes-sidecar-container
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    app: webapp
spec:
  containers:
    - name: main-application
      image: nginx
      volumeMounts:
        - name: shared-logs
          mountPath: /var/log/nginx
    - name: sidecar-container
      image: busybox
      command: ["sh","-c","while true; do cat /var/log/nginx/access.log; sleep 30; done"]
      volumeMounts:
        - name: shared-logs
          mountPath: /var/log/nginx
  volumes:
    - name: shared-logs
      emptyDir: {}

---

# Service Configuration
apiVersion: v1
kind: Service
metadata:
  name: simple-webapp
  labels:
    run: simple-webapp
spec:
  ports:
  - port: 80
    protocol: TCP
  selector:
    app: webapp
  type: NodePort
```

It's simple to tell how many pods are in a container by seeing the *READY* column.
![](../images/Pasted%20image%2020240321134144.png)

If there are multiple containers in a pod you can list them using `kubectl get pods <pod_name> -o jsonpath='{.spec.containers[*].name}'` which will output the names. Once you have the name of a container you can specifiy it using kubectl with the `-c` flag. `kubectl exec -it <pod_name> -c <container_name> -- sh`

![](../images/Pasted%20image%2020240321134104.png)

#### Defending
From [Microsoft](https://microsoft.github.io/Threat-Matrix-for-Kubernetes/techniques/Sidecar%20Injection/):
- Adhear to least-privielge principles
- Restrict over permissive containers
- Gate images deployed to kubernetes clusters

> Pull requests needed ❤️ 

