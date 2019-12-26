

# Security 
Notes taken via Linux Academy's Certified Kubernetes Admin course.

## Security Primitives
The API server needs to evaluate and authenticate requests before they can be actioned. It first determines whether requests are coming from a **Service Account** or a **User**.

### Users
Normal users are authenticated via private keys, user stores, or even files containing usernames and passwords. K8s does not have an object that defines users, and therefore users cannot be added to the cluster with API calls. 

#### Security Context
You may want to administer k8s from systems other than the master server. In order to do this, you must create a security context for the remote server. 

Basic discovery:
* `kubectl config view` -- issue this command to see your current security context
* `kubectl config set-credentials chad --username=chad --password=password` - set new credentials for the cluster
* `kubectl create clusterrolebinding cluster-system-anonymous --cluserrole=cluster-admin --user=system:anonymous` -- Creates a cluster role binding for anonymous users (NOT RECOMMENDED)
* `scp ca.crt cloud_user@<publicIp>:~` Copy the CA cert to the remote server

On the remote workstation:
* `kubectl config set-cluster kubernetes --server=https://<privateIp>:6443 --certificate-authority=ca.crt --embed-certs=true` -- sets the new cluster info
* `kubectl config set-credentials chad --username=chad --password=password`
* `kubectl config set-context kubernetes --cluster=kubernetes --user=chad --namespace=default` -- set security and cluster context
* `kubectl config use-context kubernetes` -- use the context you've created

### Service Accounts
Manage the identity of requests coming in from Pods. A default Service Account is created along with the cluster. A secret is created along with SAs that contain the public certificate of the API server along with signed JSON of a web token. The default SA is used for Pods if one is not explicitly defined. 

Basic discovery:
* `kubectl get serviceaccounts`
* `kubectl create serviceaccount jenkins` -- create a Service Account and Secret for your Jenkins server
* `kubectl get sa jenkins -o yaml` -- see the YAML of an SA (includes secret name)
* `kubectl get secret <jenkins secret name>` -- see the Secret

Add an SA to Pod YAML:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox
  namespace: default
spec:
  serviceAccountName: jenkins # add the SA under the Pod spec
  containers:
  - image: busybox:1.28.4
    command:
      - sleep
      - "3600"
    imagePullPolicy: IfNotPresent
    name: busybox
  restartPolicy: Always
```