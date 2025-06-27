# DaemonSet in Kubernetes

DaemonSets in Kubernetes are API objects.
DaemonSet is a type of controller that ensures a copy of a specific Pod runs on all (or some) nodes in a cluster.
It automatically adds a Pod to new nodes as they join the cluster and removes the Pod when nodes are removed.

DaemonSets are commonly used for services that need to run continuously in the background, such as systems monitoring the Nodes or agents collecting logs. It’s important for these applications to have a Pod running on every Node in your cluster to ensure they work effectively.

DaemonSets are built to reliably run a Pod on every Node. They have built-in settings called ‘tolerations’ which let them schedule new Pods in situations where it might normally be blocked. For instance, even if a Node is low on resources or is not currently accepting new Pods, the DaemonSet Pods will still be scheduled on that

Scaling: Unlike Deployments, DaemonSets do not use the `replicas` field because they are concerned with ensuring that each node (or a subset of nodes based on labels) runs a pod.

Controlled Expansion: When you update a DaemonSet, new nodes will get the updated version of the pod, and existing nodes will eventually transition to the updated version.


### 🔧 Use Cases of DaemonSet
**Node-level Monitoring**

Tools like Prometheus Node Exporter, Datadog Agent, or Fluentd for collecting metrics/logs from each node.

**Log Collection**

Deploy log collectors (e.g., Fluent Bit, Filebeat, Logstash) to forward logs from /var/log/ on each node to a centralized logging system (e.g., Elasticsearch, Splunk).

**Security and Compliance**

Run security agents like Falco, OSSEC, or antivirus/malware scanners on every node.

**Storage Daemons**

Use DaemonSets for running distributed storage daemons like Ceph, GlusterFS, or OpenEBS that need to run on each storage node.

**Network Plugins / CNI Components**

Deploy CNI components (e.g., Calico, Weave, Cilium) on each node to manage networking and enforce policies.

**Custom Node Configuration**

Run Pods that mount node-level paths (e.g., /etc, /var, or device paths) to tweak configurations or perform health checks.



## 📦 Realistic Examples of DaemonSet

### Examples:
This YAML file would define a DaemonSet named fluentd-elasticsearch, which ensures that a container (configured with the specified image) runs on each node in the cluster, typically used for logging purposes.


`commands`:

To manage the DaemonSet, you can use commands like kubectl get daemonsets, 

```bash
kubectl get daemonset
kubectl describe daemonset <name>
kubectl edit daemonset <name>.
```


```bash
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
spec:
  selector:
    matchLabels:
      k8s-app: fluentd-logging
  template:
    metadata:
      labels:
        k8s-app: fluentd-logging
    spec:
      containers:
      - name: fluentd-elasticsearch
        image: docker.io/... # appropriate Docker image here
        resources: {}
        volumeMounts:
        - name: ...
          mountPath: ... # paths to mount volumes
      tolerations:
      - key: "..."
        operator: "Equal"
        value: "..."
        effect: "NoSchedule"
      volumes:
      - name: ...
        hostPath:
          path: ... # paths to the host file system
```

### 📝 1. Log Collector DaemonSet (e.g., Fluent Bit)

```bash
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluent-bit
  namespace: kube-system
spec:
  selector:
    matchLabels:
      name: fluent-bit
  template:
    metadata:
      labels:
        name: fluent-bit
    spec:
      containers:
      - name: fluent-bit
        image: fluent/fluent-bit:latest
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers

```
### 🛡️ 2. Security Agent DaemonSet (e.g., Falco)

```bash
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: falco
  namespace: falco
spec:
  selector:
    matchLabels:
      app: falco
  template:
    metadata:
      labels:
        app: falco
    spec:
      containers:
      - name: falco
        image: falcosecurity/falco:latest
        securityContext:
          privileged: true
        volumeMounts:
        - name: dev
          mountPath: /host/dev
        - name: proc
          mountPath: /host/proc
        - name: boot
          mountPath: /boot
      volumes:
      - name: dev
        hostPath:
          path: /dev
      - name: proc
        hostPath:
          path: /proc
      - name: boot
        hostPath:
          path: /boot
```




### 🔁 Behavior
When a new node is added: The DaemonSet controller ensures a Pod is automatically scheduled on it.

When a node is removed or tainted, the DaemonSet controller removes the associated Pod.

Can be limited to a subset of nodes using nodeSelector, affinity, or tolerations.
