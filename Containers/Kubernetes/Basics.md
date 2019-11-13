## Kubernetes Basics - Getting Started
Notes taken from Linux Academy's Kubernetes the Hard Way course.

### Kubernetes Architecture
In this course, the following architecture will be used:
* Kube API load balancer (nginx in this example)
    * This load balancer will proxy kubectl API commands between the two controller nodes' kube-apiserver services
    * Worker node kubelet and kube-proxy services also access the kube-apiserver via the API load balancer
* Controllers - act as k8s control plane
    * etcd
    * kube-apiserver
    * nginx exposing healthz endpoint via HTTP :80 > :6443
    * kube-controller-manager
    * kube-scheduler
* Workers - pods/applications run on these nodes
    * containerd (container runtime)
    * kubelet (service that manages containers on the worker node)
    * kube-proxy (helps deal with networking between worker nodes)

### Client Tools
Tools needed on the client side:
* cfssl - manage certificates we'll create for the k8s cluster
* kubectl - interact with the k8s control plane from a client machine

### CA and TLS Certificates
* **Certificates** are used to authenticate identity. They prove you are who you say you are. 
* A **Certificate Authority (CA)** provides the ability to confirm a certificate is valid. The CA can be used to validate any certifiate issued by that same CA.
* K8s uses these certificates for a variety of functions. Different parts of the clsuter will validate certs using the CA.

#### What certificates do we need?
* **Client Certificates** - provide client authentication for various users: admin, kube-controller-manager, kube-proxy, kube-scheduler, and the kubelet client on each worker node
* **Kubernetes API Server certificate** - TLS cert for the k8s API
* **Service Account Key Pair** - k8s uses a cert to sign service account tokens (allows users to authenticate to the service account)

#### Provisioning a CA
* Make a new directory to store your certs, i.e. `mkdir ~/kube`
* Generate a basic CA config and CSR using the following script and JSON:
```bash
{

cat > ca-config.json << EOF
{
  "signing": {
    "default": {
      "expiry": "8760h"
    },
    "profiles": {
      "kubernetes": {
        "usages": ["signing", "key encipherment", "server auth", "client auth"],
        "expiry": "8760h"
      }
    }
  }
}
EOF

cat > ca-csr.json << EOF
{
  "CN": "Kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "CA",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert -initca ca-csr.json | cfssljson -bare ca

}
```
* This provisions the self-signed CA that will be used to generate certificates to be used by the k8s cluster

#### Generating Client Certs
* Several client certificates will need to be generated. These client certs are used by various k8s components. 
* The following code can be used to generate the **admin** client cert:
```bash
{

cat > admin-csr.json << EOF
{
  "CN": "admin",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:masters",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  admin-csr.json | cfssljson -bare admin

}
```
* The following script can be used to generate the certs for each of your **kubelet** clients:
```bash
WORKER0_HOST=<Public hostname of your first worker node cloud server>
WORKER0_IP=<Private IP of your first worker node cloud server>
WORKER1_HOST=<Public hostname of your second worker node cloud server>
WORKER1_IP=<Private IP of your second worker node cloud server>

{
cat > ${WORKER0_HOST}-csr.json << EOF
{
  "CN": "system:node:${WORKER0_HOST}",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:nodes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=${WORKER0_IP},${WORKER0_HOST} \
  -profile=kubernetes \
  ${WORKER0_HOST}-csr.json | cfssljson -bare ${WORKER0_HOST}

cat > ${WORKER1_HOST}-csr.json << EOF
{
  "CN": "system:node:${WORKER1_HOST}",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:nodes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=${WORKER1_IP},${WORKER1_HOST} \
  -profile=kubernetes \
  ${WORKER1_HOST}-csr.json | cfssljson -bare ${WORKER1_HOST}

}
```
* The following code can be used to generate the **kube-controller-manager** client cert:
```bash
{

cat > kube-controller-manager-csr.json << EOF
{
  "CN": "system:kube-controller-manager",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:kube-controller-manager",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-controller-manager-csr.json | cfssljson -bare kube-controller-manager

}
```
* The following code can be used to generate the **kube-proxy** client cert:
```bash
{

cat > kube-proxy-csr.json << EOF
{
  "CN": "system:kube-proxy",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:node-proxier",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-proxy-csr.json | cfssljson -bare kube-proxy

}
```
* Finally, this code can be used to generate the **kube-scheduler** client cert:
```bash
{

cat > kube-scheduler-csr.json << EOF
{
  "CN": "system:kube-scheduler",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:kube-scheduler",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-scheduler-csr.json | cfssljson -bare kube-scheduler

}
```

#### Generating the API server certificate
* We still require a server certificate for our k8s API server. You can use this code to generate that:
```bash
CERT_HOSTNAME=10.32.0.1,<controller node 1 Private IP>,<controller node 1 hostname>,<controller node 2 Private IP>,<controller node 2 hostname>,<API load balancer Private IP>,<API load balancer hostname>,127.0.0.1,localhost,kubernetes.default

{

cat > kubernetes-csr.json << EOF
{
  "CN": "kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=${CERT_HOSTNAME} \
  -profile=kubernetes \
  kubernetes-csr.json | cfssljson -bare kubernetes

}
```
* **NOTE**: some clients communicating with the API server may use the internal IP addresses/DNS names 10.32.0.1, 127.0.0.1 (localhost), or kubernetes.default to communicate with the API server. That's why these values are included in the hostnames variable, which is used when signing the certificate.

#### Generating the Service Account Key Pair
* The following code can be used to generate the service account key pair. This cert is used to sign tokens that can be created for service accounts:
```bash
{

cat > service-account-csr.json << EOF
{
  "CN": "service-accounts",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  service-account-csr.json | cfssljson -bare service-account

}
```

#### Distributing Certificates to Servers
* You can use `scp` to copy necessary files to your controllers and worker nodes. For example:
```bash
# Move certificates to worker nodes
scp ca.pem <worker 1 hostname>-key.pem <worker 1 hostname>.pem user@<worker 1 public IP>:~/
scp ca.pem <worker 2 hostname>-key.pem <worker 2 hostname>.pem user@<worker 2 public IP>:~/

# Move certificates to controller nodes
scp ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem \
    service-account-key.pem service-account.pem user@<controller 1 public IP>:~/
scp ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem \
    service-account-key.pem service-account.pem user@<controller 2 public IP>:~/
```