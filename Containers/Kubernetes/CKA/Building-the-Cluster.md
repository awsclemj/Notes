# Building a Kubernetes Cluster
Notes taken via Linux Academy's Certified Kubernetes Admin course.

## Release Binaries, Provisioning, and Types of Clusters
There are many choices to make when creating a k8s cluster. Depending on your use case, you may want to build a custom solution. On the other hand, many cloud providers offer pre-built solutions that make spinning up a cluster quite simple.

* Custom:
    * Install binaries manually
    * Configure network fabric
    * Locate release binaries
    * Build your own images
    * Secure cluster communications
* Pre-built
    * Minikube
    * Minishift
    * Microk8s
    * Ubuntu on LXD
    * AWS, Azure, Google Cloud

When installing manually, take note that **kubelet** is the only control plane service that needs to run on the system. This is because kubelet is what controls running pods. All other components can run as pods. You can even make your own custom container images using the release binaries. This is especially helpful if using a private registry. 

## Installing k8s on the Master and Worker Nodes
The following details the steps necessary to install k8s on multiple nodes.

* Install the necessary packages through your package manager. Each node must have the following:
    * Docker (or whichever container runtime you'll be using)
    * kubelet
    * kubeadm
    * kubectl

* **Note:** you may want to hold them at their current version so nothing unexpected happens if they were to auto-update. Ubuntu example: `sudo apt-mark hold docker-ce kubelet kubeadm kubectl`

* You will then need to enable network bridging in `sysctl.conf`: `echo "net.bridge.bridge-nf-call-iptables=1" | sudo tee -a /etc/sysctl.conf`

* Initialize the cluster from the master node: `sudo kubeadm init --pod-network-cidr=10.244.0.0/16`

* Create a local kubeconfig on the master node:
```bash
mkdir -p $HOME/.kube

sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

* Finally, apply Flannel, a container network interface (CNI) network overlay: `kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml`

* Join each of your worker nodes to the master: `sudo kubeadm join [your unique string from the kubeadm init command]`

* Now, when running `kubectl get nodes`, you should see all nodes joined in the cluster.

## Building a Highly Available Cluster
You can replicate your cluster components across multiples nodes for high availability. This allows for fault tolerance in your cluster. However, some components require to be run in an active/standby mode to avoid conflicts.

* The **controller manager** and **scheduler** actively monitor the cluster state, and therefore these control plane components can only be active on one master node at a time.
    * This is facilitated by a "leader elect" function that is true by default in these components' YAML manifests. This gives the components a means to know which is the leader at any given time.

### etcd
* Due to the distributed nature of **etcd**, they can be in a stacked or external topology. 
* A **stacked** etcd topology is when each master node has its own local etcd, and each of those etcds only communicate with that node's API Server.
* An **external** etcd topology is when etcd is external to the k8s cluster

* etcd uses a raft consesus algorithm which requires a majority to move to the next state. More than half of the etcd instances must agree on a state change in order for it to happen. It isbest to have an odd number of etcd instances for this reason.

#### Installing a Stacked etcd Topology
* Download the binaries
* Extract binaries to `/usr/local/bin`
* Create two directories `/etc/etcd` and `var/lib/etcd`
* Create a systemd unit file for etcd
* Enable and start the etcd service

Using kubeadm: `kubeadm init --config=kubeadm-config.yaml`

## Configuring Secure Cluster Communications
It is recommended that all communications between the cluster are secured using HTTPS. 

### Using kubectl
`kubectl` is a translator that communicates with the API Server's CRUD (create, read, update, delete) interface which is exposed via a REST API. This is facilitated by POST requests and HTTP(S) handles authentication, authorization, etc. The state is then stored in **etcd**. 

When communicating with the API server, serveral steps are used to validate the entity requesting the state change can perform the action they're requesting:
* Authentication - via TLS certificate, for instance
* Authorization - can this user perform the requested action on the requested resource?
* Admission - request is passed through plugins based on the resource. Only used if the request is to create, modify, or delete a resource. Skipped for reads.

All requests either come from users or pods. For requests from users, Role-Based Access Controls (RBAC) is used for the authorization stage. 

#### Users
* A user can be associated with one or more roles. Each role dictates what actions the users can take on a resource.
* Can be namespace-level or cluster-level:
    * Role - defines the actions that can be performed
    * Role bindings - defines who can perform the actions

#### Pods
* ServiceAccounts are used for Pod-level permissions and are namespace-level permissions. ServiceAccounts serve as the identity of the Pod and a token file is used for auth to the API server. 
* Default permissions for new ServiceAccounts do not allow users to list or modify resources
* To view the ServiceAccounts in your cluster: `kubectl get serviceaccounts`
* If a ServiceAccount is not explicitly stated in the Pod's manifest, the default will be applied

## Testing Cluster Performance
There are various testing suites available for testing a k8s cluster end-to-end. One example is **Kubetest**. However, we will be running through a manual end-to-end test. 

* Run a simple nginx deployment: `kubectl run nginx --image=nginx`
* Check your deployment is ready: `kubectl get deployments`
* Check that your pods are running: `kubectl get pods`
* Check that your pod is directly accessible: `kubectl port-forward nginx 8081:80`
* Open a new terminal window and curl for status: `curl -I http://localhost:8081`
* Can you get logs from the pod?: `kubectl logs nginx`
* Can you run a command inside the container?: `kubectl exec -it nginx -- nginx -v`
* Expose the Deployment with a Service: `kubectl expose deployment nginx --port 80 --type NodePort`
* View services: `kubectl get services`
* Check the health of nodes: `kubectl describe nodes`
* Check the health of pods: `kubectl describe pods`