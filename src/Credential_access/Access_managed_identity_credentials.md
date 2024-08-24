# Access managed identity credentials
With access to a Kubernetes cluster running in  a cloud environment, a common way to escalate privileges is by accessing the [IMDS]() endpoint at `169.254.169.254/latest/meta-data/iam/security-credentials/<user>` to obtain tokens that may allow for privielge escalation or lateral movement. 

![](../images/Pasted%20image%2020240329002531.png)

This attack is different depending on the cloud provider.

# Azure
> Pull requests needed ❤️ 

# GCP
> Pull requests needed ❤️ 


# AWS
> Pull requests needed ❤️ 


# Defending
For AWS environments, enforcing the use of [[IMDSv2]] can help mitigate this attack or simply disable the IMDS if it's unneeded. [IMDSpoof](https://github.com/grahamhelton/IMDSpoof) can be used in conjunction with honey tokens to create detection. 

> Pull requests needed ❤️ 

# Resources & References
[Nick Frichette](https://hackingthe.cloud/aws/exploitation/ec2-metadata-ssrf/) has a wonderful resource for pentesting cloud environments.