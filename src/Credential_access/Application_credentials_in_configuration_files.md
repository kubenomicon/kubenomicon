# Application credentials in configuration files
Accessing application credentials is not a Kubernetes specific issue, however, credentials used in a Kubernetes cluster may be visible through manifests. Most notably, gaining access to an Infrastructure as Code repository could lead to sensitive information being identified from manifests.

Additionally, Kubernetes ConfigMaps are frequently used to pass information to a pod. This can be in the form of configuration files, environment variables, etc.

In this example, information is passed via a ConfigMap to a Pod running postgres which sets the environment variables `POSTGRS_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD`, and `PGDATA`. While ConfigMaps are not *supposed*  to be used for sensitive information, they still can be used to pass in information such as passwords.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: postgres-config
  labels:
    app: ecommerce
    tier: postgres
data:
  POSTGRES_DB: prod 
  POSTGRES_USER: prod 
  POSTGRES_PASSWORD: 123graham_is_SO_cool123 
  PGDATA: /var/lib/postgresql/data/pgdata
```

After a config map is created, it can be referenced by a manifest by using `- configMapRef` which will link the config map to the Pod.

```yaml
apiVersion: v1
kind: Pod 
metadata:
  name: postgres 
spec:
  containers:
  - name: postgres
    image: postgres:latest
    envFrom:
      - configMapRef:
          name: postgres-config
```

Once inside the pod, environment variables passed in via ConfigMaps can be listed with `env`.

![](../images/Pasted%20image%2020240331145655.png)

Beyond ConfigMaps, searching for potentially sensitive strings such as `PASSWORD=`, is worthwhile. A tool like [Dredge](https://github.com/grahamhelton/dredge) can be used for this. 

![](../images/Pasted%20image%2020240331152414.png)

# Defending
> Pull requests needed ❤️ 