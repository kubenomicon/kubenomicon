# Initial access
Initial Access into a Kubernetes cluster is usually the most difficult stage and for the most part, is not specific to Kubernetes and relies on one of the following:

- Finding a weakness in an application being hosted in a Kubernetes cluster
    - [Application Vulnerability](./Initial_access/Application_vulnerability.md)
- Supply chain compromise
    - [Compromised Images In Registry](./Initial_access/Compromised_image_in_registry.md)
- Abusing Developer Resources
    - [Kubeconfig File](./Initial_access/Kubeconfig_file.md)
    - [Exposed Sensitive Interfaces](./Initial_access/Exposed_sensitive_interfaces.md)
    - [Using Cloud Credentials](./Initial_access/Using_cloud_credentials.md)