- [Application Lifecyle Management](#application-lifecyle-management)
  - [Deployment, Rolling Updates, and Rollbacks](#deployment-rolling-updates-and-rollbacks)
    - [Updating the App](#updating-the-app)
    - [Rolling Back Updates](#rolling-back-updates)
  - [Configuring Apps for High Availability](#configuring-apps-for-high-availability)
    - [Passing Configuration Options to a Pod](#passing-configuration-options-to-a-pod)
      - [ConfigMaps](#configmaps)
      - [Secrets](#secrets)
  - [Creating a Self-Healing Application](#creating-a-self-healing-application)
    - [ReplicaSets](#replicasets)
    - [StatefulSets](#statefulsets)

# Application Lifecyle Management
Notes taken via Linux Academy's Certified Kubernetes Admin course.

## Deployment, Rolling Updates, and Rollbacks
To deploy your applications, you should use a **Deployment** object. Deployments are declarative and provide an easy way to manage your application through its lifecycle. 

When you create a Deployment, it automatically creates a **ReplicaSet**. Pods created with this ReplicaSet will have a hash value appended to them.

Example Deployment manifest:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kubeserve
spec:
  replicas: 3
  selector:
    matchLabels:
      app: kubeserve
  template:
    metadata:
      name: kubeserve
      labels:
        app: kubeserve
    spec:
      containers:
      - image: linuxacademycontent/kubeserve:v1
        name: app
```

To create the Deployment: `kubectl apply -f <deployment.yaml> --record`  
To check the rollout status: `kubectl rollout status deployments kubeserve`  
To list the new ReplicaSet: `kubectl get replicasets`  
To scale your Deployment: `kubectl scale deployment kubeserve --replicas=5`  

### Updating the App
K8s allows you to update you application without having to take it down for end users. There are a couple ways to do this.

* Update the YAML for the Deployment to pull a new image version. Use `kubectl apply -f <YAML>` or `kubectl replace -f <YAML>` to deploy the changes.
  * How these differ: `apply` will create the object if it doesn't already exist, but will otherwise replace the current object. `replace` expects the object to already exist.
* Preferred method: perform a **rolling update**. This requires no downtime and is the quickest way to update your app.
  * Perform a rolling update by updating the Deployment's image: `kubectl set image deployments/kubeserve app=linuxacademycontent/kubeserve:v2 --v 6`

Rolling updates create a new ReplicaSet behind the scenes. As new Pods are created under the new ReplicaSet, the Pods from the previous ReplicaSet are terminated.

### Rolling Back Updates
Assume that we introduce a bug in version three of the kubeserve application. K8s offers an undo feature to allow you to quickly roll back to the previous version. 

* Roll out the buggy app: `kubectl set image deployment kubeserve app=linuxacademycontent/kubeserve:v3`
* Undo the rollout: `kubectl rollout undo deployments kubeserve`

Rollout undo is possible because the Deployment keeps a revision history. The history is stored in the underlying ReplicaSets. 

* To see rollout history: `kubectl rollout history` if you use `--record` when applying your YAML file, the CHANGE-CAUSE column will show the reason for the revision. 
* You can roll back to a specific revision in your revision history using the following: `kubectl rollout undo deployment kubeserve --to-revision=2`

Sometimes you may want to pause a full rollout of a new app version. This can be used as a canary release to test your new app version while also having your previous release available to end users. 
* `kubectl rollout pause deployment kubeserve`
* To resume normal rollout: `kubectl rollout resume deployment kubeserve`

## Configuring Apps for High Availability
What happens when you release an update with bad code? Using the `minReadySeconds` property in a Deployment manifest, you can prevent applications with bad code from being released to the public. This property is used to detail how long the Pod should be "Ready" before it is considered available. 

The rollout will not continue until the pod is available. This combined with a **readiness probe** will ensure that bad code you release will not become available. The readiness probe determines if the Pod is ready to receive customer traffic. Once the readiness probe returns a success, the Pod is marked available. 

Example manifest:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kubeserve
spec:
  replicas: 3
  selector:
    matchLabels:
      app: kubeserve
  minReadySeconds: 10 # How many seconds a Pod needs to be in a healthy state before another replica is deployed
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
    type: RollingUpdate
  template:
    metadata:
      name: kubeserve
      labels:
        app: kubeserve
    spec:
      containers:
      - image: linuxacademycontent/kubeserve:v3
        name: app
        readinessProbe: # Defines the type of readiness probe (in this case an HTTP GET request)
          periodSeconds: 1
          httpGet:
            path: /
            port: 80
```

### Passing Configuration Options to a Pod
Sometimes you may want your application to run using certain environment variables. In k8s, you can set up a **ConifgMap**, which allows you to set certain Key-Value pairs to pass into your containers as environment variables. If you need to instead pass sensitive data, k8s allows you to create a **Secret** which can also be passed into containers as environment variables.

NOTE: You can reference the same ConfigMap or Secret for multiple containers. 

#### ConfigMaps 
* Create a simple ConfigMap: `kubectl create configmap appconfig --from-literal=key1=value1 --from-literal=key2=value2`
* Get the ConfigMap in YAML format: `kubectl get configmap appconfig -o yaml`
* Example Pod manifest using ConfigMap:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-pod
spec:
  containers:
  - name: app-container
    image: busybox:1.28
    command: ['sh', '-c', "echo $(MY_VAR) && sleep 3600"]
    env:
    - name: MY_VAR
      valueFrom:
        configMapKeyRef:
          name: appconfig
          key: key1
```
* To see your variables: `kubectl get logs <Pod Name>`

Another potential way to pass environment variables into a container is via a mounted volume:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-volume-pod
spec:
  containers:
  - name: app-container
    image: busybox
    command: ['sh', '-c', "echo $(MY_VAR) && sleep 3600"]
    volumeMounts:
      - name: configmapvolume
        mountPath: /etc/config
  volumes:
    - name: configmapvolume
      configMap:
        name: appconfig
```
* To get the keys from the volume on the container: `kubectl exec configmap-volume-pod -- ls /etc/config`
* To get the values from the volume on the Pod: `kubectl exec configmap-volume-pod -- cat /etc/config/key1`

#### Secrets
Example manifest:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: appsecret
stringData:
  cert: value
  key: value
```

To associate a secret with a Pod:
```yaml 
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
spec:
  containers:
  - name: app-container
    image: busybox
    command: ['sh', '-c', "echo Hello, Kubernetes! && sleep 3600"]
    env:
    - name: MY_CERT
      valueFrom:
        secretKeyRef:
          name: appsecret
          key: cert
```

To associate a secret with a volume and mount it to a Pod:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-volume-pod
spec:
  containers:
  - name: app-container
    image: busybox
    command: ['sh', '-c', "echo $(MY_VAR) && sleep 3600"]
    volumeMounts:
      - name: secretvolume
        mountPath: /etc/certs
  volumes:
    - name: secretvolume
      secret:
        secretName: appsecret
```

## Creating a Self-Healing Application
K8s makes things such as service discovery, scaling, load balancing, and self healing much easier for developers. We have seen several examples of this already in this course. You no longer have to babysit servers and monitor for errors; Pods can automatically replace apps that have crashed or are faulty. 

### ReplicaSets
**ReplicaSets** (and Deployments by proxy) ensure that there are many copies of your application running throughout the nodes. So even if a node is lost,  you should still have replicas of your application running on healthy nodes.

Example manifest:
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myreplicaset
  labels:
    app: app
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: main
        image: linuxacademycontent/kubeserve
```

To create a Pod that will be picked up by this ReplicaSet:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  labels:
    tier: frontend
spec:
  containers:
  - name: main
    image: linuxacademycontent/kubeserve
```
Since our ReplicaSet has already created three Pods, this new Pod would be terminated right away. Similarly, you can remove a Pod from a ReplicaSet by changing it label.

NOTE: You should always use a Deployment to manage your ReplicaSets instead of defining them manually in this fashion. 

### StatefulSets
A **StatefulSet** is needed when you care about the state of each of your Pods and you need them to be unique. Additionally, Services in StatefulSets must be headless since each of your Pods are unique. 

Example manifest:
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

Each Pod within the StatefulSet requires its own storage since it's unique. 

* To view your StatefulSet: `kubectl get statefulsets`
* To describe StatefulSets: `kubectl describe statefulsets`