# Kubernetes Architecture
Notes taken via Linux Academy's Certified Kubernetes Admin course.

## Kubernetes Cluster Architecture
Master node (control plane):
* **API Server** - communication hub for all cluster components; exposes the k8s API
* **Scheduler** - Assigns your app to a worker node. Auto-detects which pod to assign to which node based on resource requirements, hardware constraints, etc.
* **Controller manager** - Maintains the cluster state. Handles node failures, replication components, etc. 
* **etcd** - Data store for cluster configuration (recommended to back this up)
* **Note:** The master node will never run application components. It initiates and follows instructions to deploy pods and containers within.

Worker node components:
* Main role - Responsible for running application components and services
* **kublet** - runs and manages containers on the node and talks to API server and container runtime
* **kube-proxy** - Load balances traffic between application components
* **container runtime** - program that runs your containers (Docker, containerd, etc.)

Basic discovery commands:
* `kubectl get nodes` - shows all nodes and whether they are a master or worker
* `kubectl get pods --all-namespaces` - shows all pods in the cluster
* `kubectl get namespaces` - show all namespaces in the cluster
    * **Note:** a namespace can be thought of as a smaller, virtual cluster. These logically separate different cluster environments

Creating a basic Pod deployment:
```yaml
cat << EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
EOF
```
You can get detailed information about a specific pod by running `kubectl describe pod <pod name>`

## API Primitives
Every single k8s component communicates with the API server and the API server only. The API server is also the only component that communicates with the etcd data store.

You can get the health status of the k8s components by running `kubectl get componentstatus`

You interact with the API Server by writing manifest scripts in YAML format and passing them to `kubectl`. These scripts declare the state of the cluster that you'd like k8s to maintain. They define persistent objects (Pods, Services, etc.) that k8s will ensure remains in the desired state.

Example YAML manifest declaring a Deployment object with 2 Pod replicas:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

## Kubernetes Services and Network Primitives
When you create Pods either directly or through an abstraction such as a Deployment object, each pod is assigned an IP address. Communication between Pods is simple; it is a flat NAT-less network layer. 

Since assignment of these IP addresses is dynamic, Service objects exist in order to create a static IP/port assignment. This allows your applications to dynamically adjust to any changes in the underlying Pod layer.

Example Service:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort # This type reserves a port on the node for the service
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30080
  selector: # The selector tells which pod(s) to forward inbound traffic to based on the pod's label
    app: nginx
```

**kube-proxy** handles the traffic associated with Services by creating iptables rules. Will perform DNAT on inter-Pod communication when Service IP:port combinations are used. 