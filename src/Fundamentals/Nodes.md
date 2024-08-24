# Nodes

_Worker nodes_ exist within the _data plane_ which is the plane in a Kubernetes cluster that carries out actions given from the control plan. This is where Pods are deployed and thus your applications reside. The data plane typically consists of:

- **Worker Node**: One or more computers responsible for running workloads (such as pods)
- **Kubelet**: Process that runs on each Node. Responsible for executing tasks (such as deploying pods) provided to it by the API server.
- Pod: An object that represents one or more containers (sometimes called workloads)

![](../images/Pasted%20image%2020240331202719.png)

# Master Node

The _Master Node(s)_ exists within the _Control Plane_ and carries out the administrative actions for interacting and managing Kubernetes. The control plane consists of:

- **API Server**: The communication channel for any interaction with Kubernetes. Any interaction with Kubernetes must traverse the API server. `kubectl` is the normal way of interacting with the API server but it can also be communicated with via any tool capable of making API calls such as `curl`.
- **Scheduler**: Watches the API server and listens for Pods that have not been assigned to a worker node. The scheduler is responsible for finding a Node to place the pod on.
- **etcd**: Version controlled key/value store. This holds the current state of the cluster.
- **Controller manager**: Is a collection of _controllers_ each of which have control loops that watch the _API Server_ for state changes to the cluster and make changes if the actual state is different than the desired state.
- **Cloud Controller Manager**: Similar to the _Controller Manager_ but interacts with your cloud provider's APIs.

All of these components together are known as a _cluster_. Clusters can be configured in many ways, but a production environment is likely being run in a "high availability" configuration with at least 3 control plane nodes that are kept in sync and `n` number of worker nodes that Pods are deployed to.

![](https://kubernetes.io/images/kubeadm/kubeadm-ha-topology-stacked-etcd.svg)
Note that the `etcd` server can be configured in a few other ways than show above.

# Attacking Nodes

It is _usually_ true that gaining access root-level access to a node participating in kubernetes is very much the "endgame" for most environments. Root access to a kubernetes node allows an attacker to access information from all pods running on that node by exploring the [overlay2](https://docs.docker.com/storage/storagedriver/overlayfs-driver/) storage driver, deploy [Static Pods](../Persistence/Static_pods.md), plunder `/etc/kubernetes` (discussed below), and more.

# /etc/kubernetes

`/etc/kubernetes` is where kubernetes information is stored in most default configurations.

- `/etc/kubernetes`: This is where kubeconfig files typically live as well as configuation information for control plane components (if on a control-plane node)
- `/etc/kubernetes/manifests` is the path where the Kubelet looks for Pod manifests to deploy [See Persistence -> Static Pods](../Persistence/Static_pods.md). There are a few default Pod manifests:
    - etcd.yaml
    - kube-apiserver.yaml
    - kube-controller-manager.yaml
    - kube-scheduler.yaml

Often times you can find very sensitive information in the `/etc/kubernetes` directory on a node such as [Initial Access -> Kubeconfig file](../Initial_access/Kubeconfig_file.md). Which can be exfiltrated to an attacker machine to gain access to the cluster. 

![](../images/Pasted%20image%2020240325010843.png)