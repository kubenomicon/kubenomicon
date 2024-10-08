# Using Cloud Credentials
Gaining access to the web application interface of managed Kubernetes services such as [GKE](https://cloud.google.com/kubernetes-engine?hl=en), [AKS](https://azure.microsoft.com/en-us/products/kubernetes-service), or [EKS](https://aws.amazon.com/eks/) is extremely powerful. It should go without saying that if you're able to login to the management interface of a cloud provider, you can access the cluster and cause chaos. Typically this would be done via phishing.

![](../images/Pasted-image-20240320232454.png)



# Defending

Defending your cluster from compromised cloud credentials would require one to plan, considering mandatory training for the wider team about Social Engineering is crucial. Mistakes made by legitimate users are much less predictable, making it harder for defensive systems to identify and thwart threat actors as opposed to a malware-based intrusion.

Utilising password managers like Vault have proven to be a more secure way of handling sensitive passwords, this prevents threat actors from scanning for keystrokes in an already compromised machine/ env making it harder for them to make lateral movements.


