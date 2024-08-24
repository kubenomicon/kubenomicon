
# Delete Kubernetes Events
Kubernetes events are essentially logs at the cluster layer. Events are reported to the API server and contain information about state changes such as pods being created or nodes restarting.

There is no _directory_ where events are stored and thus it may be harder to ingest these logs into a SIEM without creating a custom application.

Specific logs can be queried with kubectl: `kubectl get events -o yaml | yq .items.1`

```yaml
apiVersion: v1
count: 298
eventTime: null
firstTimestamp: "2024-03-29T04:05:01Z"
involvedObject:
  apiVersion: v1
  fieldPath: spec.containers{distroless}
  kind: Pod
  name: distroless
  namespace: default
  resourceVersion: "679"
  uid: aa451abc-99dd-4684-b373-75a13faf42a3
kind: Event
lastTimestamp: "2024-03-29T05:10:12Z"
message: Pulling image "istio/distroless"
metadata:
  creationTimestamp: "2024-03-29T04:05:01Z"
  name: distroless.17c12087dd1a32b1
  namespace: default
  resourceVersion: "3958"
  uid: b5919efc-7277-4147-87c0-e515796b7c50
reason: Pulling
reportingComponent: ""
reportingInstance: ""
source:
  component: kubelet
  host: minikube
type: Normal

```

Or simply with `kubectl get events`

![](Pasted%20image%2020240331201017.png)

Logs can be cleared using `kubectl delete events --all`

# Defending
> Pull requests needed ❤️ 