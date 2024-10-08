# Pod Name Similarity
While fairly simple, just naming a pod something that doesn't stand out is a great way to hide among "known good" pods.

In this example, a secondary `etcd` pod was created that is actually just an Ubuntu image.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: etcd 
  namespace: kube-system
spec:
  containers:
  - name: ubuntu
    image: ubuntu:latest
    command: ["sh","-c","sleep 100000000000000"]
```
![](Pasted%20image%2020240331201050.png)

# Defending
> Pull requests needed ❤️ 