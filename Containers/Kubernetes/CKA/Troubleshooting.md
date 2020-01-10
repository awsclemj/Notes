- [Troubleshooting](#troubleshooting)
  - [Applications Failures](#applications-failures)
  - [Control Plane Failures](#control-plane-failures)
  - [Worker Node Failures](#worker-node-failures)
  - [Network Failure](#network-failure)

# Troubleshooting
Notes taken via Linux Academy's Certified Kubernetes Admin course.

## Applications Failures
When troubleshooting application errors, you must ensure that the application developer puts in relevant debug details in the log files. k8s also makes app failures easier to debug by writing termination reasons in a log file.

Example:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod2
spec:
  containers:
  - image: busybox
    name: main
    command:
    - sh
    - -c
    - 'echo "I''ve had enough" > /var/termination-reason ; exit 1'
    terminationMessagePath: /var/termination-reason
```
* If you run `kubectl describe po pod2`, you will see this termination message in the state details. 

Another means to check if a Pod is healthy is by using the built-in health checking mechanism (healthz endpoint). **Note:** Not all images have the healthz endpoint configured. 

healthz example:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness
spec:
  containers:
  - image: linuxacademycontent/candy-service:2
    name: kubeserve
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8081
```

* View logs for details about your Pods/containers: `kubectl logs <pod>`
* Export Pod YAML if you can't edit it directly: `kubectl get po pod-with-defaults -o yaml --export > defaults-pod.yaml`
* Edit a Pod directly: `kubectl edit po nginx`

## Control Plane Failures
There could be a number of things that could go wrong in the k8s control plane. For example, the API server shuts down, the etcd data store could shut down/crash, software faults, etc. There are some preventative measures you can take to ensure that you don't have a single point of failure. 

Preventative measures:
* Choose infrastructure that has SLA guarantees and is set up in a highly-available manner
* Take periodic snapshots of the attached storage
* Use Deployments and Services to ensure your load gets balanced between Pods/nodes
* Federate multiple clusters together

Examples of potential issues:
* If you notice your Deployments aren't getting scheduled properly between nodes, ensure your kube-scheduler Pod is up and running.
  * `kubectl get events -n kube-system` and `kubectl logs <kube-scheduler Pod name> -n kube-system`
* Ensure Docker is running/enabled: `sudo systemctl status docker` and/or `sudo systemctl enable docker && systemctl start docker`
* Check the status of kubelet: `sudo systemctl status kubelet`
  * **NOTE:** kubelet will not run if swap is enabled. to disable swap: `sudo su - \ swapoff -a && sed -i '/ swap / s/^/#/' /etc/fstab`
* Check the status of your system's firewall

## Worker Node Failures
This is a very similar approach to troubleshooting a non-responsive server.

* We can get the basic node status by simply running `kubectl get nodes`
* Get more detailed info with `kubectl describe`
* Try to SSH into the node

Assume that someone accidentally deleted your cloud server and this is why the worker stopped responding. Follow these steps to add the new node into the cluster:
* Generate a new token on the master: `sudo kubeadm token generate`
* Create a k8s join command for the new worker: `sudo kubeadm token create [token_name] --ttl 2h --print-join-command`

Now assume that the problem is a bit more simple, for example the kubelet service dies. 
* View the journalctl logs: `sudo journalctl -u kubelet`
* You could also check the syslog for kubelet messages: `sudo more syslog | tail -120 | grep kubelet`

## Network Failure
Network issues in k8s usually have to do with inter-cluster communication and with services. 

* Verify your services: `kubectl get svc`
* Create a service by exposing a port: `kubectl expose deployment hostnames --port=80 --target-port=9376`
* From a busybox, check if DNS is resolving hostnames: `nslookup hostnames`
* cat the `resolv.conf` file from within the container
* If there is a problem with DNS, you can troubleshoot if there's something wrong with the default `kubernetes` service: `nslookup kubernetes.default`
* Check to ensure your service is configured with the correct port(s) and protocol(s): `kubectl get svc hostnames -o json`
* View the endpoints for your service: `kubectl get ep`
* Bypass the service and connect directly to a Pod's IP/port combo: `kubectl get po <pod name> -o wide` and `wget -qO- <Pod IP:port>`
* Check if kube-proxy is running on the node: `ps auxw | grep kube-proxy`
* Ensure your kube-proxy is configuring iptables rules on the node: `iptables-save | grep hostnames`
* Ensure you have a CNI plugin installed