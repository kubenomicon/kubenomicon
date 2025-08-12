# Compromised image In registry
A compromised container image in a trusted registry can be used to gain initial access to a Kubernetes cluster if you're able to push images to it. This attack path is fundamentally the same concept as [Persistence -> Backdoor_container](../Persistence/Backdoor_container.html).

A compromised image in a container registry is the logical next step to [Persistence -> Backdoor_container](../Persistence/Backdoor_container.html). If an attacker is able to upload or tamper with the "trusted" images in a registry such as [Harbor](https://github.com/goharbor/harbor), they can fully control the environment the application is operating within. This is analogous downloading an ubuntu ISO that an attacker had tampered with and using it as your base operating system.

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
This attack relies on the Kubernetes cluster being configured to pull container images from the registry without validation. We can add validation in the form of: 

- **Allowlisted Container Images**: Using policy-as-code tools like [Open Policy Agent](https://q4us.dev/kubernetes-security-hardening-image-whitelisting/#:~:text=parameters%3A%0A%20%C2%A0%C2%A0%20images%3A%0A%20%C2%A0%C2%A0%C2%A0%C2%A0%20%23%20Images,weave%2Dnpc%3Alatest) we can define and enforce standards for the workloads that can be run in a cluster. 
- **Kubernetes Admission Controller**: If we are generating provenance during the build phase of our software development lifecycle, we can use [Kubernetes Admission Controllers](https://docs.github.com/en/actions/security-for-github-actions/using-artifact-attestations/enforcing-artifact-attestations-with-a-kubernetes-admission-controller#about-image-verification) to verify a container image was built via your build process, and deny running container images in the registry that cannot prove this. 

