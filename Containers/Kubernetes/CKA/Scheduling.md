- [Scheduling in Kubernetes](#scheduling-in-kubernetes)
  - [Configuring the Scheduler](#configuring-the-scheduler)
    - [Default Scheduler Steps](#default-scheduler-steps)
    - [Affinity](#affinity)
      - [Labeling Nodes and Setting Affinity](#labeling-nodes-and-setting-affinity)
  - [Running Multiple Schedulers](#running-multiple-schedulers)
    - [Create Roles and Role Bindings](#create-roles-and-role-bindings)
    - [Applying Your Custom Scheduler](#applying-your-custom-scheduler)
    - [Create Pods Using Multiple Schedulers](#create-pods-using-multiple-schedulers)
  - [Scheduling Pods with Resource Limits and Label Selectors](#scheduling-pods-with-resource-limits-and-label-selectors)
    - [Scheduling Pods on Specific Nodes](#scheduling-pods-on-specific-nodes)
    - [Checking for Resource Constraints](#checking-for-resource-constraints)
  - [DaemonSets and Manually Scheduled Pods](#daemonsets-and-manually-scheduled-pods)
  - [Displaying Scheduler Events](#displaying-scheduler-events)

# Scheduling in Kubernetes
Notes taken via Linux Academy's Certified Kubernetes Admin course.

## Configuring the Scheduler
The k8s **Scheduler's** job is to assign your Pod to a node based on the resource requirements of the Pod. These rules are in place by default, but it is possible to modify them depending on your use case. 

Why customize the Scheduler?
* You may want to save costs and have all of your pods run on a single node
* You may have requirements that some applications run on a node with an SSD instead of HDD.
* Many more possibilities.

### Default Scheduler Steps
1. Identify if the node has adequate hardware resources
2. Is the node running out of resources (memory or disk pressure conditions)
3. Does the Pod request a specific node?
4. Does the node have a label that matches the nodeSelector in the Pod spec
5. If the Pod is requesting to be bound to a specific host port, is it available?
6. If the Pod is requesting a volume, can it be mounted? 
7. Does the Pod tolerate the taints of the node? (e.x. environment=prod)
8. Does the Pod specify node or Pod affinity rules?

After these rules have been evaluated, the Scheduler determines the best node to schedule Pod creation. If multiple nodes have the same priority, it will schedule in a round-robin fashion.

### Affinity
**Affinity** is a means by which you can set priority for the Scheduler. It tells the Scheduler that you'd prefer certain nodes over others. However, if resource constraints exist, you're okay with your Pods being scheduled elsewhere. 

Assume that you have multiple sets of servers in multiple AZs across the globe. One AZ you have exclusive access to, the other is shared by other teams. You want to set affinity for your local AZ. 

#### Labeling Nodes and Setting Affinity
* Indicate one node as being in AZ1: `kubectl label node <dedicated node> availability-zone=zone1`
* Label the node as being dedicated infrastructure: `kubectl label node <dedicated node> share-type=dedicated`
* Indicate the second node as being in AZ2: `kubectl label node <shared node> availability-zone=zone2`
* Label the node as being shared infrastructure: `kubectl label node <shared node> share-type=shared`
* Setting node affinity:
```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: pref
spec:
  replicas: 5
  template:
    metadata:
      labels:
        app: pref
    spec:
      affinity:
        nodeAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 80
            preference:
              matchExpressions:
              - key: availability-zone
                operator: In
                values:
                - zone1
          - weight: 20
            preference:
              matchExpressions:
              - key: share-type
                operator: In
                values:
                - dedicated
      containers:
      - args:
        - sleep
        - "99999"
        image: busybox
        name: main
```

## Running Multiple Schedulers
If there are cases where you would like to schedule certain Pods under certain conditions, it is possible to create custom schedulers or even run multiple schedulers simultaneously. For example, assume that you want to set different rules for the scheduler to run all of your Pods on one node.

In order to create a custom scheduler, you must follow the direction in the k8s docs to [create a Docker image](https://kubernetes.io/docs/tasks/administer-cluster/configure-multiple-schedulers/).

### Create Roles and Role Bindings
You must then create a cluster role and cluster role binding for the scheduler:
* `ClusterRole.yaml`:
```yaml
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: csinodes-admin
rules:
- apiGroups: ["storage.k8s.io"]
  resources: ["csinodes"]
  verbs: ["get", "watch", "list"]
```

* `ClusterRoleBinding.yaml`:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-csinodes-global
subjects:
- kind: ServiceAccount
  name: my-scheduler
  namespace: kube-system
roleRef:
  kind: ClusterRole
  name: csinodes-admin
  apiGroup: rbac.authorization.k8s.io
```

A role and role binding for your scheduler to speak with your Pods will also be necessary:
* `Role.yaml`:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: system:serviceaccount:kube-system:my-scheduler
  namespace: kube-system
rules:
- apiGroups:
  - storage.k8s.io
  resources:
  - csinodes
  verbs:
  - get
  - list
  - watch
```

* `RoleBinding.yaml`:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-csinodes
  namespace: kube-system
subjects:
- kind: User
  name: kubernetes-admin
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role 
  name: system:serviceaccount:kube-system:my-scheduler
  apiGroup: rbac.authorization.k8s.io
```

### Applying Your Custom Scheduler
Edit the existing kube-scheduler role to include your custom scheduler with `kubectl edit clusterrole system:kube-scheduler`:
```yaml
- apiGroups:
  - ""
  resourceNames:
  - kube-scheduler
  - my-scheduler
  resources:
  - endpoints
  verbs:
  - delete
  - get
  - patch
  - update
- apiGroups:
  - storage.k8s.io
  resources:
  - storageclasses
  verbs:
  - watch
  - list
  - get
```

Then, apply your custom scheduler YAML:
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-scheduler
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: my-scheduler-as-kube-scheduler
subjects:
- kind: ServiceAccount
  name: my-scheduler
  namespace: kube-system
roleRef:
  kind: ClusterRole
  name: system:kube-scheduler
  apiGroup: rbac.authorization.k8s.io
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    component: scheduler
    tier: control-plane
  name: my-scheduler
  namespace: kube-system
spec:
  selector:
    matchLabels:
      component: scheduler
      tier: control-plane
  replicas: 1
  template:
    metadata:
      labels:
        component: scheduler
        tier: control-plane
        version: second
    spec:
      serviceAccountName: my-scheduler
      containers:
      - command:
        - /usr/local/bin/kube-scheduler
        - --address=0.0.0.0
        - --leader-elect=false
        - --scheduler-name=my-scheduler
        image: chadmcrowell/custom-scheduler # Change this value depending on where your scheduler image is stored
        livenessProbe:
          httpGet:
            path: /healthz
            port: 10251
          initialDelaySeconds: 15
        name: kube-second-scheduler
        readinessProbe:
          httpGet:
            path: /healthz
            port: 10251
        resources:
          requests:
            cpu: '0.1'
        securityContext:
          privileged: false
        volumeMounts: []
      hostNetwork: false
      hostPID: false
      volumes: []
```

* Apply the scheduler: `kubectl create -f my-scheduler.yaml`
* Verify it is running: `kubectl get pods -n kube-system`

### Create Pods Using Multiple Schedulers
* `Pod1.yaml`:
```yaml
# If no scheduler is defined in the Pod spec, the default will be used
apiVersion: v1
kind: Pod
metadata:
  name: no-annotation
  labels:
    name: multischeduler-example
spec:
  containers:
  - name: pod-with-no-annotation-container
    image: k8s.gcr.io/pause:2.0
```

* `Pod2.yaml`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: annotation-default-scheduler
  labels:
    name: multischeduler-example
spec:
  schedulerName: default-scheduler
  containers:
  - name: pod-with-default-annotation-container
    image: k8s.gcr.io/pause:2.0
```

* `Pod3.yaml`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: annotation-second-scheduler
  labels:
    name: multischeduler-example
spec:
  schedulerName: my-scheduler
  containers:
  - name: pod-with-second-annotation-container
    image: k8s.gcr.io/pause:2.0
```

## Scheduling Pods with Resource Limits and Label Selectors
Nodes can be **tainted** in order to offload work elsewhere. The simplest way of seeing this in action is by describing the master node: `kubectl describe node <master node>`. Under the details, you will see `Taints: node-role.kubernetes.io/master:NoSchedule`. This ensures that Pods will never be scheduled on the master.

Pod specs can include **tolerations** which can ignore taints; this means that you could potentially schedule a Pod on the master node if you declare a toleration. 

The scheduler schedules Pods to nodes based on the sum of the resources requested on that node. When the scheduler goes through it selection algorithm, there is a **least requested priority** and **most requested priority**. These are mostly self-explanatory and one or the other can be configured on the scheduler. You may want to use the most requested function if you're trying to optimize cost usage on a cloud provider. 

In order to the see capacity on a node:
* Run `kubectl describe node <node name>`
* Under the details, check `Capacity:` and `Allocatable:`
  * Capacity is overall capacity of the node, Allocatable is how much can be allocated to Pods

### Scheduling Pods on Specific Nodes
Example manifest:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-pod1
spec:
  nodeSelector:
    kubernetes.io/hostname: "chadcrowell3c.mylabserver.com" # Schedules to specific node
  containers:
  - image: busybox
    command: ["dd", "if=/dev/zero", "of=/dev/null"]
    name: pod1
    resources: # Resources requested 
      requests:
        cpu: 800m
        memory: 20Mi
```

### Checking for Resource Constraints
If you try to overcommit your node, your Pod will not be scheduled. You can use `kubectl describe pods <pod name>` if you see a Pod stuck in Pending. You can then describe your node to see how it got overcommitted.

Instead of using resource requests, you can use **resource limits** within your Pod spec. Resource limits limit the amount of CPU/memory that can be used on a Pod at a given time. 

**NOTE:** Pods with limits can go beyond the total amount of resources available on a node. If this occurs, k8s automatically will kill Pods. 

Example of resource limits:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: limited-pod
spec:
  containers:
  - image: busybox
    command: ["dd", "if=/dev/zero", "of=/dev/null"]
    name: main
    resources:
      limits:
        cpu: 1
        memory: 20Mi
```

## DaemonSets and Manually Scheduled Pods
**DaemonSets** do not use a scheduler to deploy Pods. DaemonSets are good for Pods that need exactly one replica running on each node. Some DaemonSets Pods run by default in the kube-system namespace (ex. kube-proxy and kube-flannel)

Why not schedule these Pods on each node? DaemonSet Pods have a special set of instructions to run on each node and also run immediately when a new node joins the cluster. Additionally, when you delete a DaemonSet Pod, it will immediately be recreated. They also ignore taints on the nodes.

Let's assume that we want to create a new DaemonSet that monitors the status of our SSDs on each node:
* Create a node label: `kubectl label node <node name> disk=ssd`
* Example DaemonSet manifest:
```yaml
apiVersion: apps/v1beta2
kind: DaemonSet
metadata:
  name: ssd-monitor
spec:
  selector:
    matchLabels:
      app: ssd-monitor
  template:
    metadata:
      labels:
        app: ssd-monitor
    spec:
      nodeSelector:
        disk: ssd
      containers:
      - name: main
        image: linuxacademycontent/ssd-monitor
```
* After doing this, you can then label other nodes with `disk=ssd` and the DaemonSet Pod will automatically be created on it. Similarly, Pods will automatically terminate if you modify or delete the label

## Displaying Scheduler Events
There are several different ways to view scheduler events. Problems with the scheduler can be identified in the following ways:

* Pod-level:
  * Run `kubectl describe pods <pod name>` and view the events at the bottom of the output
* Events-level:
  * `kubectl get events` -- Events in the default namespace
  * `kubectl get events -n kube-system` -- Events in the kube-system namespace
  * `kubectl get events -w` -- Watch events occur in real time
* Logs-level:
  * `kubectl logs <kube-scheduler pod name> -n kube-system` -- Shows kube-scheduler logs