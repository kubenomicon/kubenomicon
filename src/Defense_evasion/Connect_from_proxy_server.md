# Connect from proxy server


An adequately hardened Kubernetes cluster will have access controls (such as firewalls) in place to limit traffic to the API server. Connecting to the API server from inside a trusted server (or inside an allow listed subnet) can allow an attacker access to resources as well as blend in with legitimate traffic.

The attack path for this would be to compromise a developers machine, then masquerade as the developer's identity to to perform further actions against a Kubernetes cluster.


# Defending
> Pull requests needed ❤️ 