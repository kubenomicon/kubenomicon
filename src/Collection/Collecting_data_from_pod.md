# Collecting data from pod
Collecting data from a Pod is essentially the exfiltrating of data. This can be done in a near infinite amount of ways depending on what kind of tooling is available inside the pod (although you may need to get creative), but the intended way to copy data in and out of a Pod is by using the `kubectl cp` command

- To copy data INTO a pod: `kubectl cp ~/path/to/file.sh podname:/file.sh`
- To copy data OUT of a pod: `kubectl cp podname:etc/passwd passwd`

![](../images/Pasted%20image%2020240404171643.png)

# Defending
> Pull requests needed ❤️ 