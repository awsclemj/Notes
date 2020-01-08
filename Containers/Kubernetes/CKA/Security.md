- [Security](#security)
  - [Security Primitives](#security-primitives)
    - [Users](#users)
      - [Security Context](#security-context)
    - [Service Accounts](#service-accounts)
  - [Cluster Authentication and Authorization](#cluster-authentication-and-authorization)
  - [Network Policies](#network-policies)
    - [Creating Ingress Rules](#creating-ingress-rules)
      - [Creating an Ingress Rule for Namespaces](#creating-an-ingress-rule-for-namespaces)
      - [Creating an Ingress Rule for CIDR Blocks](#creating-an-ingress-rule-for-cidr-blocks)
    - [Creating Egress Rules](#creating-egress-rules)
  - [Managing TLS Certificates](#managing-tls-certificates)
  - [Secure Images](#secure-images)
  - [Defining Security Contexts](#defining-security-contexts)
    - [Definiing Non-root Users](#definiing-non-root-users)
    - [Privileged Pods](#privileged-pods)
    - [Kernel-level Privileges](#kernel-level-privileges)
    - [Read-only File System](#read-only-file-system)
    - [Pod-level Security Context](#pod-level-security-context)
  - [Securing Persistent Key-Value Store](#securing-persistent-key-value-store)

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

## Cluster Authentication and Authorization
Steps for Authentication and Autorization:
1. Authentication is the first step in receiving a request. The API server evaluates who the request is coming from (Pod or user). 
2. Authorization occurs after authentication, and it is used to determine what the user or Pod can do. These authorization rules are configured in RBAC (role-based access control)
   1. Roles and role bindings: roles and role bindings operate at the namespace level
   2. Cluster roles and cluster role bindings operate at the cluster level. Some objects are not namespaced, which is where you would use the ClusterRole. 
   3. Roles/ClusterRoles define what a user can do, but not who can do it. That's where the bindings come into play

Creating a role:
* First, create a new namespace: `kubectl create ns web`
* Create the role:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: web
  name: service-reader
rules:
- apiGroups: [""]
  verbs: ["get", "list"]
  resources: ["services"]
```
* Create a RoleBinding: `kubectl create rolebinding test --role=service-reader --serviceaccount=web:default -n web`
* Create a ClusterRole for listing PVs: `kubectl create clusterrole pv-reader --verb=get,list --resource=persistentvolumes`
* Create a ClusterRoleBinding: `kubectl create clusterrolebinding pv-test --clusterrole=pv-reader --serviceaccount=web:default`
* To test the ClusterRoleBinding, create a Pod that can use curl and also a kubectl proxy:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: curlpod
  namespace: web
spec:
  containers:
  - image: tutum/curl
    command: ["sleep", "9999999"]
    name: main
  - image: linuxacademycontent/kubectl-proxy
    name: proxy
  restartPolicy: Always
```
* curl the PVs to test your ClusterRole: `curl localhost:8001/api/v1/persistentvolumes`

## Network Policies
Network Policies govern how Pods can communicate with one another. By default, they are open and can be accessed by anyone. Network Policies apply to Pods that match a Pod or namespace label selector. You can set ingress/egress rules and CIDR block ranges for your Pods. 

* Creating network policies:
* Grab the plugin: `wget -O canal.yaml https://docs.projectcalico.org/v3.5/getting-started/kubernetes/installation/hosted/canal/canal.yaml`
* Apply it: `kubectl apply -f canal.yaml`
* Example deny all policy:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {} # Leaving this blank causes all Pods in the namespace to inherit this policy
  policyTypes:
  - Ingress
```
* You can test this Network Policy by creating a simple deployment and trying to access it:
* `kubectl run nginx --image=nginx --replicas=2`
* `kubectl expose deployment nginx --port=80`
* `kubectl run busybox --rm -it --image=busybox /bin/sh`
* `wget --spider --timeout=1 nginx` -- This will fail due to the deny all policy

### Creating Ingress Rules
Assume you want to create an ingress rule to allow your web servers access to a backend database:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-netpolicy
spec:
  podSelector:
    matchLabels:
      app: db
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: web
    ports:
    - port: 5432
```
* Ensure that your Pods have the appropriate label to match this policy: `kubectl label pods [pod_name] app=db`

#### Creating an Ingress Rule for Namespaces
You can also create policies that apply to an entire Namespace:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: ns-netpolicy
spec:
  podSelector:
    matchLabels:
      app: db
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          tenant: web
    ports:
    - port: 5432
```

#### Creating an Ingress Rule for CIDR Blocks
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: ipblock-netpolicy
spec:
  podSelector:
    matchLabels:
      app: db
  ingress:
  - from:
    - ipBlock:
        cidr: 192.168.1.0/24
```

### Creating Egress Rules
Creating egress rules is quite similar:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: egress-netpol
spec:
  podSelector:
    matchLabels:
      app: web
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: db
    ports:
    - port: 5432
```

## Managing TLS Certificates
A **Certificate Authority (CA)** is used to generate TLS certs to authenticate to your API server. The CA certificate is by default mounted at `/var/run/secrets/kuberbetes.io/serviceaccount`

* To find your CA certificate: `kubectl exec busybox -- ls /var/run/secrets/kubernetes.io/serviceaccount`
* You can use a tool such as `cfssl` to generate new certs
* Creating a certificate signing request (CSR):
```bash
cat <<EOF | cfssl genkey - | cfssljson -bare server
{
  "hosts": [
    "my-svc.my-namespace.svc.cluster.local",
    "my-pod.my-namespace.pod.cluster.local",
    "172.168.0.24",
    "10.0.34.2"
  ],
  "CN": "my-pod.my-namespace.pod.cluster.local",
  "key": {
    "algo": "ecdsa",
    "size": 256
  }
}
EOF
```
* Create a CSR API object:
```yaml
apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
  name: pod-csr.web
spec:
  groups:
  - system:authenticated
  request: $(cat server.csr | base64 | tr -d '\n')
  usages:
  - digital signature
  - key encipherment
  - server auth
EOF
```
* View CSRs in the cluster: `kubectl get csr`
* Describe CSRs: `kubectl describe csr pod-csr.web`
* Approve the CSR: `kubectl certificate approve pod-csr.web`
* View the certificate: `kubectl get csr pod-csr.web -o yaml`
* Extract the certificate to use in a file: `kubectl get csr pod-csr.web -o jsonpath='{.status.certificate}' | base64 --decode > server.crt`

## Secure Images
When you input a container image name in Kubernetes, the image is pulled from a container registry (Docker Hub if using Docker daemon). It's imperative that the images you work with a secure and devoid of vulnerabilities. You can ensure your images all pass vulnerability scans, etc. and then store them in a private registry.

* Docker stores creds here: `/home/<username>/.docker/config.json`
* You can log in to a private registry using the following command: `sudo docker login -u podofminerva -p 'otj701c9OucKZOCx5qrRblofcNRf3W+e' podofminerva.azurecr.io`
* Tag an image to push it to a private registry: `sudo docker tag busybox:1.28.4 podofminerva.azurecr.io/busybox:latest`
* Create a Secret to store your Docker credentials: `kubectl create secret docker-registry acr --docker-server=https://podofminerva.azurecr.io --docker-username=podofminerva --docker-password='otj701c9OucKZOCx5qrRblofcNRf3W+e' --docker-email=user@example.com`
* Modify the default ServiceAccount to use the new Secret: `kubectl patch sa default -p '{"imagePullSecrets": [{"name": "acr"}]}'`
* Create a Pod using the image you pushed to your private repo:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: acr-pod
  labels:
    app: busybox
spec:
  containers:
    - name: busybox
      image: podofminerva.azurecr.io/busybox:latest
      command: ['sh', '-c', 'echo Hello Kubernetes! && sleep 3600']
      imagePullPolicy: Always
```

## Defining Security Contexts
Security contexts define access controls for Pods and containers. When defined, only certain processes can do certain things. It allows you to give or take away control from your containers.

### Definiing Non-root Users
Alpine Linux by default will run as root. Assume you don't want this. Example YAML for a container to run as a user:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: alpine-user-context
spec:
  containers:
  - name: main
    image: alpine
    command: ["/bin/sleep", "999999"]
    securityContext:
      runAsUser: 405
```
* To validate the IDs of the new container using the user permissions: `kubectl exec alpine-user-context id`
* If you simply want to specify the Pod not run as root instead of a specific user:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: alpine-nonroot
spec:
  containers:
  - name: main
    image: alpine
    command: ["/bin/sleep", "999999"]
    securityContext:
      runAsNonRoot: true
```
* Note that the above will fail because the Alpine image is configured to run as root. You would either have to modify the Dockerfile or specify a user to run as. 

### Privileged Pods
* To create a Pod with privileged permissions:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: privileged-pod
spec:
  containers:
  - name: main
    image: alpine
    command: ["/bin/sleep", "999999"]
    securityContext:
      privileged: true
```
* This allows your Pod to not only see container-level resources, but node-level resources as well. 

### Kernel-level Privileges
If container kernel-level privileges are locked down by default, you can specify with functions you want to allow in the Pod manifest:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kernelchange-pod
spec:
  containers:
  - name: main
    image: alpine
    command: ["/bin/sleep", "999999"]
    securityContext:
      capabilities:
        add:
        - SYS_TIME # This allows you to modify the system time value
```

* Similarly, you can take away privileges:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: remove-capabilities
spec:
  containers:
  - name: main
    image: alpine
    command: ["/bin/sleep", "999999"]
    securityContext:
      capabilities:
        drop:
        - CHOWN
```

### Read-only File System
You can ensure the container's local file system isn't written to by specifying the following context. You may want to do this to ensure that nothing is written to the container's ephemeral storage:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: readonly-pod
spec:
  containers:
  - name: main
    image: alpine
    command: ["/bin/sleep", "999999"]
    securityContext:
      readOnlyRootFilesystem: true
    volumeMounts:
    - name: my-volume
      mountPath: /volume
      readOnly: false
  volumes:
  - name: my-volume
    emptyDir:
```

### Pod-level Security Context
The following is an example of a manifest that will grant one group permission to a Pod, and a different group access to another Pod:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: group-context
spec:
  securityContext:
    fsGroup: 555
    supplementalGroups: [666, 777]
  containers:
  - name: first
    image: alpine
    command: ["/bin/sleep", "999999"]
    securityContext:
      runAsUser: 1111
    volumeMounts:
    - name: shared-volume
      mountPath: /volume
      readOnly: false
  - name: second
    image: alpine
    command: ["/bin/sleep", "999999"]
    securityContext:
      runAsUser: 2222
    volumeMounts:
    - name: shared-volume
      mountPath: /volume
      readOnly: false
  volumes:
  - name: shared-volume
    emptyDir:
```

## Securing Persistent Key-Value Store
Sensitive data may need to be passed to your container. Your sensitive data may also need to live beyond the life of your container. This is where persistent key-value stores come into play.

**Secrets** are used to secure sensitive data that you may access from your Pod. The data is never written to a disk; it's stored in-memory (tmpfs). 

You can expose secrets as environment variables or mount them to the filesystem of a Pod. Note: it is not best practice to expose secrets as environment variables, as many containers will dump their environment variables in logs.

Basic discovery:
* `kubectl get secrets`
* `kubectl describe pods pod-with-defaults` -- Describe pods with the default secret mounted
* `kubectl describe secrets` -- View the token, cert, and namespace for the secret

Example use case:
* Assume you want to generate an SSL certificate for your website and want to keep the certificates secret
* Generate an RSA key for your server: `openssl genrsa -out https.key 2048`
* Create a certificate: `openssl req -new -x509 -key https.key -out https.cert -days 3650 -subj /CN=www.example.com`
* Create an empty file for the secret: `touch file`
* Generate the secret: `kubectl create secret generic example-https --from-file=https.key --from-file=https.cert --from-file=file`
* View the YAML for your secret: `kubectl get secrets example-https -o yaml`
* Create a ConfigMap that will mount to your Pod:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: config
data:
  my-nginx-config.conf: |
    server {
        listen              80;
        listen              443 ssl;
        server_name         www.example.com;
        ssl_certificate     certs/https.cert;
        ssl_certificate_key certs/https.key;
        ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers         HIGH:!aNULL:!MD5;

        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
        }

    }
  sleep-interval: |
    25
```
* The YAML for your Pod:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-https
spec:
  containers:
  - image: linuxacademycontent/fortune
    name: html-web
    env:
    - name: INTERVAL
      valueFrom:
        configMapKeyRef:
          name: config
          key: sleep-interval
    volumeMounts:
    - name: html
      mountPath: /var/htdocs
  - image: nginx:alpine
    name: web-server
    volumeMounts:
    - name: html
      mountPath: /usr/share/nginx/html
      readOnly: true
    - name: config
      mountPath: /etc/nginx/conf.d
      readOnly: true
    - name: certs
      mountPath: /etc/nginx/certs/
      readOnly: true
    ports:
    - containerPort: 80
    - containerPort: 443
  volumes:
  - name: html
    emptyDir: {}
  - name: config
    configMap:
      name: config
      items:
      - key: my-nginx-config.conf
        path: https.conf
  - name: certs
    secret:
      secretName: example-https
```