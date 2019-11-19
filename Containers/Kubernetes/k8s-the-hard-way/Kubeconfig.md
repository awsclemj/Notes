## Kubeconfig
Notes taken from Linux Academy's Kubernetes the Hard Way course.

### What is a kubeconfig and why do we need it?
* **kubeconfig** - a k8s configuration file that stores information about clusters, users, namespaces, and authentication mechanisms. It contains configuration data need to connect and interact with one or more k8s clusters
* More information: [https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/]
* Kubeconfigs contain:
    * The location of the cluster you're connecting to
    * What user you want to authenticate as
    * Data needed in order to authenticate, such as tokens or client certs
* You can define multiple contexts in a kubeconfig file, which allows you to easily switch between clusters

* Why do we need this?
    * We use kubeconfigs to store configuration data that allows the many components of k8s to connect and interact with one another
    * Ex: how does the kubelet service know how to locate and authenticate with the k8s API? Using a kubeconfig!

### Generating kubeconfigs
Kubeconfigs are generated using `kubectl`
* Use `kubectl config set-cluster` to set up the configuration for the location of the cluster
* Use `kubectl config set-credentials` to set the username and client cert that will be used to authenticate
* Use `kubectl config set-context default` to set up the default context
* Use `kubectl config use-context default` to set the current context to the configuration we provided

We need several Kubeconfigs for the cluster:
* Kubelet (one for each worker)
* Kube-proxy
* Kube-controller-manager
* Kube-scheduler
* Admin

You can use the following script to generate Kubeconfigs for each of your workerss **kubelet** services:
```bash
KUBERNETES_ADDRESS=<load balancer private ip>

for instance in <worker 1 hostname> <worker 2 hostname>; do
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBERNETES_ADDRESS}:6443 \
    --kubeconfig=${instance}.kubeconfig

  kubectl config set-credentials system:node:${instance} \
    --client-certificate=${instance}.pem \
    --client-key=${instance}-key.pem \
    --embed-certs=true \
    --kubeconfig=${instance}.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:node:${instance} \
    --kubeconfig=${instance}.kubeconfig

  kubectl config use-context default --kubeconfig=${instance}.kubeconfig
done
```

The following script can be used to generate the **kube-proxy** config:
```bash
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBERNETES_ADDRESS}:6443 \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-credentials system:kube-proxy \
    --client-certificate=kube-proxy.pem \
    --client-key=kube-proxy-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-proxy \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
}
```

The following script can be used to generate the **kube-controller-manager** kubeconfig:
```bash
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-credentials system:kube-controller-manager \
    --client-certificate=kube-controller-manager.pem \
    --client-key=kube-controller-manager-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-controller-manager \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config use-context default --kubeconfig=kube-controller-manager.kubeconfig
}
```

For the **kube-scheduler** config:
```bash
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-credentials system:kube-scheduler \
    --client-certificate=kube-scheduler.pem \
    --client-key=kube-scheduler-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-scheduler \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config use-context default --kubeconfig=kube-scheduler.kubeconfig
}
```

And finally, the **admin** config:
```bash
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=admin.kubeconfig

  kubectl config set-credentials admin \
    --client-certificate=admin.pem \
    --client-key=admin-key.pem \
    --embed-certs=true \
    --kubeconfig=admin.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=admin \
    --kubeconfig=admin.kubeconfig

  kubectl config use-context default --kubeconfig=admin.kubeconfig
}
```

### Distributing the Kubeconfigs
We can distrbute the kubeconfig files using `scp` just as we did for the certificates:
```bash
# For the worker nodes
scp <worker 1 hostname>.kubeconfig kube-proxy.kubeconfig user@<worker 1 public IP>:~/
scp <worker 2 hostname>.kubeconfig kube-proxy.kubeconfig user@<worker 2 public IP>:~/

# For the controller nodes
scp admin.kubeconfig kube-controller-manager.kubeconfig kube-scheduler.kubeconfig user@<controller 1 public IP>:~/
scp admin.kubeconfig kube-controller-manager.kubeconfig kube-scheduler.kubeconfig user@<controller 2 public IP>:~/
```