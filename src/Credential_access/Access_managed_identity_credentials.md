# Access managed identity credentials
With access to a Kubernetes cluster running in  a cloud environment, a common way to escalate privileges is by accessing the unauthenticated [IMDS]() endpoint at `169.254.169.254/latest/meta-data/iam/security-credentials/<user>` to obtain tokens that may allow for privilege escalation or lateral movement. 

![](../images/Pasted%20image%2020240329002531.png)

This attack is different depending on the cloud provider.

# Azure
> Pull requests needed ❤️ 

# GCP
Requests can be made to `metadata.google.internal/computeMetadata/v1` from a compromised instance to obtain instance metadata, including service account tokens.
The impact of a request to this endpoint is subject to the IAM permissions of the service account the instance has access to - reducing the risk from AWS' IMDSv1 implementation which does not have the notion of IAM permission-introspection on this endpoint.


# AWS
> Pull requests needed ❤️ 


# Defending
For AWS environments, enforcing the use of [[IMDSv2]] can help mitigate this attack or simply disable the IMDS if it's unneeded. [IMDSpoof](https://github.com/grahamhelton/IMDSpoof) can be used in conjunction with honey tokens to create detection. 
AWS' threat detection platform GuardDuty can be used to identify calls to the instance metadata service, e.g. with the `UnauthorizedAccess:GC2/MetadataDNSRebind` rule that will use DNS logs to identify EC2 instances within your AWS account that are querying a domain resolving to the IMDS IP address.

> Pull requests needed ❤️ 

# Resources & References
[Nick Frichette](https://hackingthe.cloud/aws/exploitation/ec2-metadata-ssrf/) has a wonderful resource for pentesting cloud environments.