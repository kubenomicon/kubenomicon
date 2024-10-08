# Images From A  Private Registry
Images stored in a private container registry cannot be pulled into a Kubernetes cluster unless the cluster has some way to authenticate to the registry. If an attacker is able to gain access to the authentication credentials, they may be able to pull down the images on their own. 

Collecting images may be useful to an attacker who is looking for secrets inside the container. Additionally, if an attacker is able to upload images to the registry, they could compromise the cluster. For more information see [Initial Access -> Compromised images in registry](../Initial_access/Compromised_image_in_registry.md).

![](../images/Pasted%20image%2020240404160908.png)

If authenticated, images can be pulled from a registry by running `docker pull <registry_URL>/REPO/IMAGE:TAG`

![](../images/Pasted%20image%2020240404161723.png)

![](../images/Pasted%20image%2020240404160447.png)


# Defending
> Pull requests needed ❤️ 