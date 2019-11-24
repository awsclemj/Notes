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
* `sudo apt intstall -y kubelet=1.14.1-00`
* `sudo apt-mark hold kubelet`

**NOTE:** You will need to install the new versions of kubeadm, kubelet, and kubectl on your master as well as your worker nodes. `kubeadm upgrade` will only need to be run on the master. 
