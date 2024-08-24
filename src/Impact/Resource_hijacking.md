# Resource Hijacking
A common "attack" is running cryptominers on compromised infrastructure. Due to the nature of Kubernetes having the ability to scale up machines quickly, an attacker that is able to deploy Pods in a compromised cluster would be able to hijack the Node compute resources to mine cryptocurrency. 

# Resource Exhaustion 
A sub category of Resource Hijacking is Resource exhaustion. If an attacker is able to identify that resources are automatically scaled up based on demand, they could execute an attack that would cost a company a large amount of money by flooding the service and thus causing extra compute to be used to scale with demand.