# Secrets
A Secret is an object that stores sensitive information such as a password, token, key, etc. Secrets are deployed to a cluster just like any other object in kubernetes by creating a manifest and declaring the `kind` as a `Secret`. By default, secrets are **NOT** encrypted (although this [can be configured](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/)) and are stored as base64 strings. 

Secrets designed to be a better way to pass sensitive information to applications. This means that sensitive information does not need to be hardcoded into an application, and instead the sensitive information can be stored by kubernetes which makes things a bit more secure and makes it easier to do tasks such as rotating credentials. 

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: very-very-secret
data:
  very-secret-file: YWRtaW4gaGVzIGRvaW5nIGl0IHNpZGV3YXlzCg== 
```

![](src/images/Pasted%20image%2020240325013909.png)
# Secret types
There are many different "types" of secrets but the default and most common is Opaque. 
- **Opaque**: Can hold almost any type of general data.
- **Service Account**: Stores.... service account tokens. 
# Pod Considerations
An attacker who is able to create a pod in a namespace can mount any secret in that namespace into the Pod and read it in plain text even if RBAC does not allow for reading of secrets using a manifest simliar to the one below:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: very-very-secret 
spec:
  volumes:
    - name: secret-volume
      secret:
        secretName: very-very-secret 
  containers:
    - name: UWUntu-with-secret
      image: ubuntu 
      volumeMounts:
        - name: secret-volume
          readOnly: true
		  # Where the secret is mounted to inside the Pod
          mountPath: "/etc/secret-volume"
```

For more information [See -> Container Service Account](src/Credential_access/Container_service_account.md).

# Encryption at rest configuration
It is possible to find the raw encryption key used to encrypt secrets within the `Kind: EncrpytionConfiguration` manifest. Ideally keys should be held outside of your kubernetes cluster in a key management system. An attacker who is able to grab the keys can simply decrypt the information.
```yaml
# Pulled from: https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/
# CAUTION: this is an example configuration.
#          Do not use this for your own cluster!
#
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
      - configmaps
      - pandas.awesome.bears.example # a custom resource API
    providers:
      # This configuration does not provide data confidentiality. The first
      # configured provider is specifying the "identity" mechanism, which
      # stores resources as plain text.
      #
      - identity: {} # plain text, in other words NO encryption
      - aesgcm:
          keys:
            - name: key1
              secret: c2VjcmV0IGlzIHNlY3VyZQ==
            - name: key2
              secret: dGhpcyBpcyBwYXNzd29yZA==
      - aescbc:
          keys:
            - name: key1
              secret: c2VjcmV0IGlzIHNlY3VyZQ==
            - name: key2
              secret: dGhpcyBpcyBwYXNzd29yZA==
      - secretbox:
          keys:
            - name: key1
              secret: YWJjZGVmZ2hpamtsbW5vcHFyc3R1dnd4eXoxMjM0NTY=
  - resources:
      - events
    providers:
      - identity: {} # do not encrypt Events even though *.* is specified below
  - resources:
      - '*.apps' # wildcard match requires Kubernetes 1.27 or later
    providers:
      - aescbc:
          keys:
          - name: key2
            secret: c2VjcmV0IGlzIHNlY3VyZSwgb3IgaXMgaXQ/Cg==
  - resources:
      - '*.*' # wildcard match requires Kubernetes 1.27 or later
    providers:
      - aescbc:
          keys:
          - name: key3
            secret: c2VjcmV0IGlzIHNlY3VyZSwgSSB0aGluaw==
```

# Defending
From the [kubernetes documentation](https://kubernetes.io/docs/concepts/configuration/secret/):
1. Enable Encryption at Rest for Secrets.
2. Enable or configure RBAC rules with least-privilege access to Secrets.
3. Restrict Secret access to specific containers.
4. Consider using external Secret store providers.
- [Good practices for Kubernetes Secrets](https://kubernetes.io/docs/concepts/security/secrets-good-practices/)