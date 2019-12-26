- [Storage](#storage)
  - [Persistent Volumes](#persistent-volumes)
    - [Access Modes](#access-modes)
  - [Persistent Volume Claims](#persistent-volume-claims)
    - [Reclaim Policies](#reclaim-policies)
  - [Storage Objects](#storage-objects)
    - [Storage Object in use Protection](#storage-object-in-use-protection)
    - [Storage Classes](#storage-classes)
  - [Applications with Persistent Storage](#applications-with-persistent-storage)

# Storage
Notes taken via Linux Academy's Certified Kubernetes Admin course.

## Persistent Volumes
Pods are ephemeral. They are meant to be moved around, duplicated, and recreated. Due to this, the storage of the container must be independent. However, if a container is moved to another Pod, you want the storage to go with it.

**Persistent volumes** are an object in k8s that allows for the above use case. They are designed to live beyond the life of a Pod and are native to k8s. 

Example manifest:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mongodb 
spec:
  volumes:
  - name: mongodb-data
    gcePersistentDisk: # This object is specific to Google Cloud
      pdName: mongodb
      fsType: ext4
  containers:
  - image: mongo
    name: mongodb
    volumeMounts:
    - name: mongodb-data
      mountPath: /data/db
    ports:
    - containerPort: 27017
      protocol: TCP
```

* You can test the persistence by creating a document on the Mongo container:
  * `kubectl exec -it <mongo Pod name> mongo`
  * `use mystore`
  * `db.foo.insert({name:'foo'})`
  * `db.foo.find()` to view the document that you created
* If you delete the Pod after performing these actions, you should see that the data still exists if you create a new Pod using that volume.

* Instead of declaring the volume within the Pod spec as we see above, it is best practice to declare a PV resource separately as an abstraction. This makes things easier for the application developer, as they don't need to worry about the underlying infrastructure.

Example manifest:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mongodb-pv
spec:
  capacity: 
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
    - ReadOnlyMany
  persistentVolumeReclaimPolicy: Retain # This pertains to Persistent Volume Claims, which will be discussed later
  gcePersistentDisk:
    pdName: mongodb
    fsType: ext4
```

###  Access Modes
When creating a PV, you can specify the PV's **access modes**. This specifies if you allow the PV to be mounted to one or many nodes, as well as if it can be read by one or many nodes.

* RWO (ReadWriteOnce) - Only one node can mount the volume for read/write access.
* ROX (ReadOnlyMany) - Many nodes can mount the volume for read access. 
* RWX (ReadWriteMany) - Many nodes can mount the volume for read/write access.

Note the following: these are the access modes for the *Nodes* not the *Pods*. The volume can also only be mounted using one access mode at a time. 

## Persistent Volume Claims
When referencing a PV in a manifest, you must use a Persistent Volume Claim (PVC). This is a way to reserve already provisioned storage for use by a Pod. A PVC remains with a PV even if a Pod is destroyed or recreated. 

Example PVC manifest:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongodb-pvc 
spec:
  resources:
    requests:
      storage: 1Gi
  accessModes:
  - ReadWriteOnce
  storageClassName: ""
```

Basic discovery:
* `kubectl get pvc` - View volume claims
* `kubectl get pv` - View volumes and which claims are utilizing them

Pod using a PVC:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mongodb 
spec:
  containers:
  - image: mongo
    name: mongodb
    volumeMounts:
    - name: mongodb-data
      mountPath: /data/db
    ports:
    - containerPort: 27017
      protocol: TCP
  volumes:
  - name: mongodb-data
    persistentVolumeClaim:
      claimName: mongodb-pvc
```

### Reclaim Policies

In the event that a PVC is deleted, the PV will either be retained or deleted based on the **reclaim policy** specified for the PV. 
* **Retain** - the PV and its data will persist beyond the life of a PVC
* **Recycle** - the data stored on the volume will be deleted, but the PV will be recycled for use with another PVC
* **Delete** - the underlying storage will be deleted

## Storage Objects
### Storage Object in use Protection
Storage Object in use Protection ensures that PVs/PVCs are not removed prematurely. This means that if a Pod that is currently using the volume exists, the PV/PVC will not be deleted until that Pod is terminated. 

### Storage Classes
Storage Classes are an even easier way to provision storage in k8s. This allows you to tell k8s what the storage provisioner is, and the control plane creates the underlying storage for you.

Example manifest:
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: kubernetes.io/gce-pd # Specific to Google Cloud
parameters:
  type: pd-ssd
```

Basic discovery:
* `kubectl get sc` - see provisioned Storage Classes

To use the new SC in your PVC:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongodb-pvc 
spec:
  resources:
    requests:
      storage: 1Gi
  accessModes:
  - ReadWriteOnce
  storageClassName: "fast" # Place the name of the SC here
```

Creating a local disk SC:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-hostpath
spec:
  storageClassName: local-storage
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```

To create a Pod with an empty directory volume (data that's deleted with the Pod):
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: emptydir-pod
spec:
  containers:
  - image: busybox
    name: busybox
    command: ["/bin/sh", "-c", "while true; do sleep 3600; done"]
    volumeMounts:
    - mountPath: /tmp/storage
      name: vol
  volumes:
  - name: vol
    emptyDir: {}
```

## Applications with Persistent Storage
Let's wrap this up with a practical example. We will build upon our kubeserve Deployment from the last section to use a PVC. 

First step - create the Storage Class:
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
```

Next, create the PVC, which will automatically provision a volume:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: kubeserve-pvc 
spec:
  storageClassName: fast
  resources:
    requests:
      storage: 100Mi
  accessModes:
    - ReadWriteOnce
```

Next, you can ensure that your PVC and PV are **bound**:
* `kubectl get pvc`
* `kubectl get pv` -- to ensure the PV was automatically provisioned for your PVC

Finally, we create our Deployment that will make sure of the PVC:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kubeserve
spec:
  replicas: 1
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
      - env:
        - name: app
          value: "1"
        image: linuxacademycontent/kubeserve:v1
        name: app
        volumeMounts:
        - mountPath: /data
          name: volume-data
      volumes:
      - name: volume-data
        persistentVolumeClaim:
          claimName: kubeserve-pvc
```