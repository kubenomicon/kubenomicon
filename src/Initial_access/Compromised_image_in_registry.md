# Compromised image In registry
A compromised container image in a trusted registry can be used to gain initial access to a Kubernetes cluster if you're able to push images to it. This attack path is fundamentally the same concept as [Persistence -> Backdoor_container](http://localhost:3000/Persistence/Backdoor_container.html).

A compromised image in a container registry is the logical next step to [Persistence -> Backdoor_container](http://localhost:3000/Persistence/Backdoor_container.html). If an attacker is able to upload or tamper with the "trusted" images in a registry such as [Harbor](https://github.com/goharbor/harbor), they can fully control the environment the application is operating within. This is analogous downloading an ubuntu ISO that an attacker had tampered with and using it as your base operating system.

![](Pasted%20image%2020240331200054.png)

# Attacking
This attack picks up where [Persistence -> Backdoor Container](../Persistence/Backdoor_container.md) left off. The prerequisites for this attack are:
1. You are able to upload images to a container registry.
2. You know the container image name that will be pulled
3. You have created a backdoor image (see [Persistence -> Backdoor Container](../Persistence/Backdoor_container.md))

First, lets login to the container registry using `docker login <registry_url> -u <username>`. Next, ensure that your backdoored image is available by running `docker image ls | grep <image_name>`. 

Now we have to `tag` the image. `docker tag <image_to_tag> <registry_url>/REPOSITORY/IMAGE_NAME`

Finally, push the backdoored image by running `docker push <registry_url>/REPOSITORY/IMAGE_NAME`. 

![](../images/Pasted%20image%2020240404162125.png)

After that, the image will be pushed to the container registry. Assuming the image is pulled by Kubernetes, your backdoored image will be deployed.

![](../images/Pasted%20image%2020240404162845.png)

# Defending
> Pull requests needed ❤️ 