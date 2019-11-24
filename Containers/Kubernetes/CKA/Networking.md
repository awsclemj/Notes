# Kubernetes Networking
Notes taken via Linux Academy's Certified Kubernetes Admin course.

## Pod and Node Networking
Networking in k8s is fairly simple. It uses Linux network namespaces.

For networking within a node: 
* Each Pod has an IP address associated with it. This is received from a virtual interface pair. One is for the node namespace and the other is for the Pod's network namespace.
* One of the virtual interfaces in the pair is given to the Pod and renamed eth0
* You can see the virtual interfaces on a worker node by using `ifconfig` on that node
* In order to see the interface on the container:
  * Run `docker ps` to see the running containers on the node
  * You will see a **/pause** container running, whose only purpose is to hold the Linux network namespace
  * To get the process ID of the container: `docker inspect --format '{{ .State.Pid }}' <container ID>`
  * You can then use the following to see the container's network info: `nsenter -t <container PID> -n ip addr`
    * You will see the IP address of the Pod as well as the number of the interface it is linked to on the node (for example, eth0@if6 for the 6th interface)
* For networking between Pods on the same node, a network bridge is used. This allows the veths to send out ARP requests to discover other Pods on the node. 

For networking between nodes:
* The source IP address of the Pod will be SNATed to the Node's IP address.
* This node-to-node communication is facilitated by a **container network interface (CNI)**

## Container Network Interface
A CNI is a network overlay that makes communication between Pods running on different nodes much simpler. It essentially builds a tunnel between the nodes. The CNI will encapsulate packets; it adds a new IP header for source and destination IP (from Pod1->Pod2 to Node1->Node2).

There is a mapping that exists within userspace that has the Pod IP to Node IP associations. When an encapsulated packet reaches the CNI on the destination node, it can then decapsulate the packet and send it to the node's network bridge.

**NOTE:** CNIs are not native to Kubernetes. They are plugins that make networking simpler. **Flannel** is used in our examples. There are many other CNIs in existence. 

In order to use the CNI plugin, you must notify kubelet. We did this in our previous lessons using: `kubeadm init --pod-network-cidr=10.244.0.0/16`

The container runtime calls the CNI plugin's executable to add or remove an instance to or from the container's networking namespace. The CNI plugin is responsible for creating and assigning IP addresses to Pods.
* The **/pause** containers we saw in the previous lesson allow the each of the containers in a Pod to have different IP addresses.

## Service Networking
Throughout the life of your cluster, Pods will come and go. Remember cattle vs. pets. It is much easier to replace a Pod than to babysit it. This, however, will likely cause the Pod to move/change IP addresses, etc. How do we manage this?

**Services** are used to manage the location of Pods even if they move, are replicated, etc. A Service creates **exactly one** virtual interface that is evenly distributed and assigned to Pods behind the virtual interface. This way, outside users and applications can reach Pods using this single virtual interface.

An example Service manifest:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30080 # Externally-reachable port
  selector:
    app: nginx # Any Pods with the label app=nginx will be exposed
```
* **NodePort** Service types expose a port on your Pod(s) container as a NodePort on the worker node.
* **ClusterIP** Service types - one is created for you by default with your Kubernetes cluster. This is an internal load balancer for the traffic in your cluster

**NOTE:** You will never be able to ping a Service, as the IP is not actually assigned to any network interface. The Service is simply a logical IP:port mapping for the nodes.

When a Service is created in Kubernetes, the API Server notifies all **kube-proxy** controllers of this state change. **kube-proxy** keeps track of Service endpoints and updates worker nodes' iptables entries accordingly.

Basic discovery:
* `kubectl get services`
* `kubectl get endpoints`
  * An Endpoint is created in the API Server automatically with any Service. It caches the IP addresses of the Pods in that Service.
* To see the `iptables` rules that are created by **kube-proxy** -- `sudo iptables-save | grep KUBE | grep nginx`

## Ingress and Load Balancers
These are required in order to expose services externally. A **LoadBalancer** is an extension of the NodePort Service type. An **Ingress** is a separate k8s object that defines an Ingress load balancer to expose Services. 

### Load Balancers
An extension of NodePort. This provisions a Load Balancer that can be accessed externally. Users/applications need only communicate with the load balancer's IP address. The load balancer then distributes traffic to NodePorts opened on the workers.

A basic example manifest:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-loadbalancer
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: nginx
```
Creating a replicated Deployment with a LoadBalancer Service:
* `kubectl run kubeserve2 --image=chadmcrowell/kubeserve2`
* `kubectl scale deployment/kubeserve2 --replicas=2`
* To see the replicated Pods along with the nodes they're running on: `kubectl get pods -o wide`
* Create a LoadBalancer Service for your Deployment: `kubectl expose deployment kubeserve2 --port 80 --target-port 8080 --type LoadBalancer`
* View your new Service: `kubectl get services`
* Watch as the external port is created: `kubectl get services -w`
* Look at the YAML for the service: `kubectl get services kubeserve2 -o yaml`
* `curl` the external IP of the LoadBalancer: `curl http://[external-ip]`
* View the annotation of the Service: `kubectl describe services kubeserve`
* Set the annotation to route load balancer traffic local to the node: `kubectl annotate service kubeserve2 externalTrafficPolicy=Local`
  * This helps in cases where you may have replicated Pods running on different nodes. The LoadBalancer will evenly balance traffic between nodes, but is not Pod-aware. Using the annotation will ensure that the traffic remains local to the node it's routed to so extra latency is not introduced. 

### Ingress
Similar to Load Balancers, but you can access multiple services with a single IP address/DNS name. LoadBalancer is a Service type, so it can only expose one Service. **Ingress** resources can expose multiple Services via HTTP(S) host and path-based routes to the outside world, as it operates in the application layer. 

We need two resources to facilitate Ingress: both an **Ingress Controller** and an **Ingress resource**.

Example Ingress manifest:
```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: service-ingress
spec:
  rules:
  - host: kubeserve.example.com # Host-based route to backend Service
    http:
      paths:
      - backend:
          serviceName: kubeserve2
          servicePort: 80
  - host: app.example.com # Second host-based route
    http:
      paths:
      - backend:
          serviceName: nginx
          servicePort: 80
  - http: # All other requests will be routed to the httpd Service
      paths:
      - backend:
          serviceName: httpd
          servicePort: 80
```
Basic manipulation/discovery:
* To edit the Ingress: `kubectl edit ingress`
* To view existing rules: `kubectl describe ingress`
* `curl` the Service: `curl http://kubeserve.example.com`

