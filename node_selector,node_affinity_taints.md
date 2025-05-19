# TOPICS : Node Selector, Node Affinity, Taints and Tolerations



# NodeSelector:

'Node Selector' is a simple way to assign a Pod to specific nodes in a Kubernetes cluster based on key-value labels.
It allows you to control which nodes are eligible to run specific Pods by matching the node's labels with the selector defined in the Pod's specification.


**Example : **

```bash
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  nodeSelector:
    disktype: ssd
  containers:
    - name: nginx
      image: nginx

```

🔄 Explanation:

**nodeSelector**: Defines the key-value pair the Pod looks for.

**disktype**: ssd: The Pod only runs on nodes with the label disktype=ssd.

If there is no matching node, the Pod remains **Pending**.


# 🚀 Real-World Examples & Use Cases:

**High-Performance Workloads:**

Run machine learning jobs or high I/O operations only on nodes with SSD storage.

```bash
spec:
  nodeSelector:
    disktype: ssd
```

**Environment Segmentation:**
Separate dev, staging, and prod workloads by node labels.

```bash
spec:
  nodeSelector:
    environment: production

```

**Geographical Separation:**

Deploy Pods to nodes in a specific region for latency optimization.

```bash
spec:
  nodeSelector:
    region: us-west

```



# Assigning a Label to Node:
You can assign a label to a node using the kubectl label command.

```bash
kubectl label nodes <node-name> <key>=<value>
kubectl label nodes worker-node-1 disktype=ssd
```

# Check Labels for a node:

```bash
kubectl get nodes --show-labels
kubectl get node worker-node-1 --show-labels

```


# Remove a Label:

```bash
kubectl label nodes worker-node-1 disktype-
```
 **NOTE:**    `The  - at the end removes the label.`




**Troubleshooting Error **: Failed Scheduling ?



# Question ! AKS Worker-Nodes :  On azure AKS worker node can remove and add due to auto scaling so how label will persist?

if you are using Cluster Autoscaler and nodes are dynamically added or removed, manually applied labels will not persist when a node is scaled down and replaced.

Solution : Add label for nodepool

When you create an AKS Node Pool, you can specify labels directly. These labels are applied automatically to every node in that pool.
These labels will persist even when nodes are auto-scaled.




