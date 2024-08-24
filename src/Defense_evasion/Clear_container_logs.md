# Clear Container Logs
If they are not mounted into a volume or otherwise collected, logs saved to a Pod are wiped when a Pod is destroyed. Because of this, it's common to store logs on a volume mount which can be persistent and/or collected by other services. To clear the logs from the container/pod, an attacker with access to the pod can simply `rm -rf /path/to/logs`. 

# Defending
> Pull requests needed ❤️ 