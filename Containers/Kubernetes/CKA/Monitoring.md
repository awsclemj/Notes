- [Monitoring](#monitoring)
  - [Monitoring Cluster Components](#monitoring-cluster-components)
  - [Monitoring Applications](#monitoring-applications)
  - [Managing Cluster Component Logs](#managing-cluster-component-logs)
  - [Managing Application Logs](#managing-application-logs)

# Monitoring
Notes taken via Linux Academy's Certified Kubernetes Admin course.

## Monitoring Cluster Components
Kubernetes provides detailed logging and monitoring of your cluster components. You can capture this information by running a metrics server, which exposes the metrics server API. The metrics server discovers each node in the cluster and queries the kubelet for performance metrics. 

Installing the metrics server:
* `git clone https://github.com/linuxacademy/metrics-server`
* `kubectl apply -f ~/metrics-server/deploy/1.8+/`
* `kubectl get --raw /apis/metrics.k8s.io/` -- Get a response from the metrics API

Basic metrics:
* `kubectl top node` - Get CPU and memory usage for your nodes
* `kubectl top pods` - Get CPU/memory util for each of your pods
* `kubectl top pods --all-namespaces` - Utilization for all namespaces
* `kubectl top pods -l run=pod-with-defaults` Utilization for pods with label selector
* `kubectl top pods group-context --containers` - Utilization metrics for containers inside pods

## Monitoring Applications
There are some components within k8s that allow you to detect resource util. automatically. Namely, **liveness probes** and **readiness probes**. 

As the name suggests, **liveness probes** can be placed into your YAML manifests to ensure a container is alive. There are three types: HTTP GET, TCP socket, and exec (arbitrary command) probe. If any of these probes fail, k8s will attempt to restart the container. 

Example:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness
spec:
  containers:
  - image: linuxacademycontent/kubeserve
    name: kubeserve
    livenessProbe:
      httpGet:
        path: /
        port: 80
```

A **readiness probe** is a mechanism you can use to ensure your containers are ready to receive requests. Unlike the liveness probe, the readiness probe will not restart the container on failure. Instead, it gets removed from the endpoints object. 

Example:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: nginx
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
---
apiVersion: v1
kind: Pod
metadata:
  name: nginxpd
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:191 # This image tag is not valid, so the Pod will fail to run
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
```
* `kubectl get ep` -- show available endpoints

## Managing Cluster Component Logs
Application and system logs can help you understand what's happening inside your cluster. Cluster logs are very useful for debugging cluster activity. These logs can get cluttered when you have many microservices running inside your cluster. 

When containers write to `stdout`, the default directory for them to write to is `/var/log/containers`. In contrast, since the kubelet runs as a service on each node, you can find its logs in `/var/log`. 

Example of an application that writes to multiple log locations:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: counter
spec:
  containers:
  - name: count
    image: busybox
    args:
    - /bin/sh
    - -c
    - >
      i=0;
      while true;
      do
        echo "$i: $(date)" >> /var/log/1.log;
        echo "$(date) INFO $i" >> /var/log/2.log;
        i=$((i+1));
        sleep 1;
      done
    volumeMounts:
    - name: varlog
      mountPath: /var/log
  volumes:
  - name: varlog
    emptyDir: {}
```
* Check the directory the logs are being written to on the container: `kubectl exec counter -- ls /var/log`

You can instead sidecar the logs into separate containers, making it easier to separate the types of logs being written for your application:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: counter
spec:
  containers:
  - name: count
    image: busybox
    args:
    - /bin/sh
    - -c
    - >
      i=0;
      while true;
      do
        echo "$i: $(date)" >> /var/log/1.log;
        echo "$(date) INFO $i" >> /var/log/2.log;
        i=$((i+1));
        sleep 1;
      done
    volumeMounts:
    - name: varlog
      mountPath: /var/log
  - name: count-log-1
    image: busybox
    args: [/bin/sh, -c, 'tail -n+1 -f /var/log/1.log']
    volumeMounts:
    - name: varlog
      mountPath: /var/log
  - name: count-log-2
    image: busybox
    args: [/bin/sh, -c, 'tail -n+1 -f /var/log/2.log']
    volumeMounts:
    - name: varlog
      mountPath: /var/log
  volumes:
  - name: varlog
    emptyDir: {}
```
Then view logs for each type of log separately:
* `kubectl logs counter count-log-1`
* `kubectl logs counter count-log-2`

## Managing Application Logs
Containerized apps usually write their logs to `stdout` or `stderr`. Docker redirects those streams to files, and k8s exposes those files with `kubectl logs`. If there are multiple containers in a Pod, you must specify the container to get the logs from. 

Examples:
* `kubectl logs nginx`
* `kubectl logs counter -c count-log-1`
* `kubectl logs counter -c --all-containers=true` - Get logs for all containers in a Pod
* `kubectl logs -lapp=nginx` -- Get logs for containers with a certain label
* `kubectl logs -p -c nginx nginx` - Get logs for a previously terminated container within a Pod
* `kubectl logs -f -c count-log-1 counter` - Stream the logs from a container in a Pod
* `kubectl logs --tail=20 nginx` - Tail the logs to only view a certain number of lines
* `kubectl logs --since=1h nginx` - View the logs for a previous time duration 
* `kubectl logs deployment/nginx -c nginx` - View logs within a deployment
* `kubectl logs counter -c count-log-1 > count.log` - Redirect log output to a file