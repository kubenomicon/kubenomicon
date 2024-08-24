# CoreDNS poisoning
CoreDNS is the "new" DNS system for Kubernetes which replaced the old KubeDNS system. 

By default, the CoreDNS configuration is stored as a configmap in the `kube-system` namespace

The following is an example of a CoreDNS configuration file. 
```yaml
apiVersion: v1
data:
  Corefile: |
    .:53 {
        log
        errors
        health {
           lameduck 5s
        }
        ready
        kubernetes cluster.local in-addr.arpa ip6.arpa {
           pods insecure
           fallthrough in-addr.arpa ip6.arpa
           ttl 30
        }
        prometheus :9153
        hosts {
           192.168.49.1 host.minikube.internal
           fallthrough
        }
        forward . /etc/resolv.conf {
           max_concurrent 1000
        }
        cache 30
        loop
        reload
        loadbalance
    }
kind: ConfigMap
metadata:
  creationTimestamp: "2024-03-29T04:00:45Z"
  name: coredns
  namespace: kube-system
  resourceVersion: "417"
  uid: 40770875-a1f7-4bf0-aeb5-4b71f60035a1

```

# Attacking 
If an attacker is able to edit this ConfigMap, they could redirect DNS traffic. In the following example, running `kubectl exec -it tcpdump -- nslookup grahamhelton.com` returns the name servers of `grahamhelton.com`
- `18.165.83.26`
- `18.165.83.67`
- `18.165.83.47`
- `18.165.83.124`

Running `nslookup` outside of a pod returns the same results. 

![](../images/Pasted%20image%2020240404133500.png)

The CoreDNS config map file can be queried using `kubectl get configmap coredns -n kube-system -o yaml`.

![](../images/Pasted%20image%2020240404135109.png)
If an attacker can edit this ConfigMap, they can add a `rewrite` rule that redirects traffic from `grahamhelton.com` to `kubenomicon.com` by adding in `rewrite name grahamhelton.com kubenomicon.com` into the config map. 

![](../images/Pasted%20image%2020240404133622.png)

Editing the config map can be accomplished by running `kubectl get configmap coredns -n kube-system -o yaml > poison_dns.yaml`, manually adding the file, and then running `kubectl apply -f poison_dns.yaml`, or by running `kubectl edit configmap coredns -n kube-system` and making changes.

![](../images/Pasted%20image%2020240404133817.png)



Once the ConfigMap has been edited, CoreDNS usually needs to be restarted. To do so run `kubectl rollout restart -n kube-system deployment/coredns`. Finally, we can re-run the previous `nslookup` command inside a pod to prove that our traffic to `grahamhelton.com` will be routed to `kubenomicon.com` by running `kubectl exec -it tcpdump -- nslookup grahamhelton.com`. This time, instead of the name servers being returned being the valid name server for `grahamhelton.com`, they are instead the name server for `kubenomicon.com`.

![](../images/Pasted%20image%2020240404133935.png)

# Defending
> Pull requests needed ❤️ 

# References & Resources
- [The redguard threat matrix has a great writeup on this](https://kubernetes-threat-matrix.redguard.ch/lateral-movement/coredns-poisoning/)! 
- [DNS spoofing on Kubernetes clusters](https://www.aquasec.com/blog/dns-spoofing-kubernetes-clusters/) 