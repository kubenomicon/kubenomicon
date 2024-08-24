# Services
Services are used to expose groups of pods over a network. They connect a set of pods to an service name and IP address. They can almost be thought of as a reverse proxy. Instead of having to directly connect to a Pod (and having to know *how* to connect to it), a client only has to know how to reach the service which will then route the request to the available pod.

![](../images/Pasted%20image%2020240402104130.png)
- Image from [NigelPoulton](https://nigelpoulton.com/explained-kubernetes-service-ports/) (Highly recommend [this explanation](https://nigelpoulton.com/explained-kubernetes-service-ports/))


## ClusterIP
ClusterIP services are only accessible from other pods running in the cluster.
```yaml
apiVersion: v1  
# Defines the type as a Service
kind: Service  
metadata:  
  name: Backend  
spec:  
  # Defines the service type as ClusterIP
  type: ClusterIP  
  ports:  
  # The external port mapping
    - port: 80 
  # The internal port mapping (What port the pod is listening on)
      targetPort: 80  
  # Select which pods to expose with the ClusterIP
  selector:
     name: my-demo-pod   
     type: front-end-app

```

## Nodeport
Exposes an app to the outside world
```yaml
apiVersion: v1
kind: Service
metadata:
  name: test-nodeport
spec:
  type: NodePort
  selector:
    app: web
  ports:
  # The internal cluster port
  - port: 8080
    # Port the pod is listening on 
    targetPort: 80
	# The external port on cluster nodes
    nodePort: 1337 
```

## LoadBalancer