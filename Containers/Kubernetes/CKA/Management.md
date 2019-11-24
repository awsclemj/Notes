- [Managing the Kubernetes Cluster](#managing-the-kubernetes-cluster)
  - [Upgrading the Cluster](#upgrading-the-cluster)
  - [Upgrading/Decommissioning Nodes Within a Cluster](#upgradingdecommissioning-nodes-within-a-cluster)
  - [Backing up/Restoring the Cluster](#backing-uprestoring-the-cluster)

# Managing the Kubernetes Cluster
Notes taken via Linux Academy's Certified Kubernetes Admin course.

## Upgrading the Cluster
You can use `kubeadm` to upgrade your cluster components. This can be helpful if you want to take advantage of new features or bug fixes for your cluster. 

There are several different things that need to be considered when talking about k8s versions. There is a version for each of the components in k8s. You can use the following commands for discovery:

* `kubectl version --short` shows you a quick breakdown of your client and server versions
* `kubectl get nodes` will show you the version of **kubelet** you're running on each node
* `kubectl describe nodes` will show you addition details about versions of other running components, such as **kube-proxy**
* You can see the versions of other system components by listing the pods they're running in. For example: `kubectl get po <controller manager name> -o yaml -n kube-system` 

Earlier, we held the kubeadm and kubectl versions so they wouldn't auto-upgrade. To unhold them:
* `sudo apt-mark unhold kubeadm kubelet`
* To install the new version: `sudo apt-get install kubeadm=1.14.1-00`
* Hold the version again: `sudo apt-mark hold kubeadm`
* To upgrade the other components using kubeadm:
    * `sudo kubeadm upgrade plan` -- plans the upgrade before executing it (will show you what components will be upgraded)
    * `sudo kubeadm upgrade apply v1.14.1` -- pulls new images and deploys them intelligently to avoid downtime

Similarly, we will want to upgrade kubectl:
* `sudo apt-mark unhold kubectl`
* `sudo apt install -y kubectl=1.14.1-00`
* `sudo apt-mark hold kubectl`

Finally, install kubelet:
* `sudo apt install -y kubelet=1.14.1-00`
* `sudo apt-mark hold kubelet`

**NOTE:** You will need to install the new versions of kubeadm, kubelet, and kubectl on your master as well as your worker nodes. `kubeadm upgrade` will only need to be run on the master. 

## Upgrading/Decommissioning Nodes Within a Cluster
You may run into a situation where you need to take down a node to upgrade OS components, or otherwise decommission it from the cluster. You can easily remove nodes and re-add them to the cluster, or add new nodes to the cluster. 

In order to remove running pods from a worker node, run the following:
* `kubectl drain <node DNS/IP> --ignore-daemonsets`
    * This removes all running pods that aren't a managed part of k8s (such as your network overlay and kube-proxy)
    * If your pods are running as part of a Deployment or ReplicaSet, they will be scheduled on another node to avoid downtime
* Running `kubectl get nodes` after running a drain will show the drained node's status as **SchedulingDisabled**, indicating it is ready for maintenance.

If you want to place your node back into service:
* `kubectl uncordon <node DNS/IP>`

To entirely delete your node from the cluster:
* `kubectl delete node <node DNS/IP>`

To add a new node to the cluster:
* Install the required components on the new server
* Generate a new token from the master: `sudo kubeadm token generate`
* To print the full join command to use on your new worker: `sudo kubeadm token create <token name> --ttl 2h --print-join-command`

## Backing up/Restoring the Cluster
**etcd** is the persistent data store for the state of your cluster, and thus this is all that will need to be backed up. You will need to ensure you have the correct components in place to back it up. This process is made simple in `kubeadm` managed clusters using a tool called `etcdctl`

Gathering dependencies and backing up:
* Grab the etcd binaries: `wget https://github.com/etcd-io/etcd/releases/download/v3.3.12/etcd-v3.3.12-linux-amd64.tar.gz`
* Unzip them: `tar xvf etcd-v3.3.12-linux-amd64.tar.gz`
* Move them to `/usr/local/bin`: `sudo mv etcd-v3.3.12-linux-amd64/etcd* /usr/local/bin`
* Take a snapshot of your cluster: `sudo ETCDCTL_API=3 etcdctl snapshot save snapshot.db --cacert /etc/kubernetes/pki/etcd/server.crt --cert /etc/kubernetes/pki/etcd/ca.crt --key /etc/kubernetes/pki/etcd/ca.key`
* How to view the `etcdctl` help page: `ETCDCTL_API=3 etcdctl --help`
* Browse the folder the contains your certificates: `cd /etc/kubernetes/pki/etcd/`
* View if your snapshot was successful: `ETCDCTL_API=3 etcdctl --write-out=table snapshot status snapshot.db`
* Zip the contents of your etcd directory: `sudo tar -zcvf etcd.tar.gz /etc/kubernetes/pki/etcd`
* Copy to another server using a utility such as `scp`