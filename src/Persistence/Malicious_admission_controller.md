# Malicious Admission Controller
Admission controllers are components that can intercept requests to the API server and make changes to (or validate) manifests. An attacker can intercept and modify the manifests before they are deployed into the cluster. The following code snippet is an example:

```go
// Example from: https://medium.com/ovni/writing-a-very-basic-kubernetes-mutating-admission-webhook-398dbbcb63ec
p := []map[string]string{}for i := range pod.Spec.Containers {    patch := map[string]string{  
        "op": "replace",  
        "path": fmt.Sprintf("/spec/containers/%d/image", i),   
        "value": "debian",  
    }
p = append(p, patch)
}
// parse the []map into JSON  
resp.Patch, err = json.Marshal(p)
```

# Defending

From [microsoft](https://microsoft.github.io/Threat-Matrix-for-Kubernetes/techniques/Malicious%20admission%20controller/): Adhere to least privielge princples by restricting permissions to deploy or modify MutatingAdmissionWebhook and ValidatingAdmissionWebhook objects.

> Pull requests needed ❤️ 