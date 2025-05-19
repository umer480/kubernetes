# TOPICS : Node Selector, Node Affinity, Taints and Tolerations



# 1- NodeSelector:  (Simple Scheduling)

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


Method 1: 

```bash
kubectl label nodes <node-name> <key>=<value>
kubectl label nodes worker-node-1 disktype=ssd
```


Method 2:

```bash
kubectl edit node <node-name>   -->  under metadata > labels > here you can add key-value 

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

❓ Limitations:

Hardmatch

It’s an exact match only; partial matches are not supported.

It’s static—you can't define logic like "prefer SSD, but fallback to HDD".


In Kubernetes, node affinity and node selector are mechanisms that control how pods are scheduled onto nodes based on labels. While both use node labels to influence scheduling decisions, they differ in flexibility and expressiveness.



# 2- Node Affinity: (Advanced Scheduling)


Node Affinity is a more expressive way to specify rules about the placement of pods relative to nodes' labels. It allows you to specify rules that apply only if certain conditions are met.


**Types of Node Affinity:**

1- equiredDuringSchedulingIgnoredDuringExecution: Hard requirement; the scheduler will only place the pod on nodes matching the criteria

2- PreferredDuringSchedulingIgnoredDuringExecution: Soft preference; the scheduler will try to place the pod on matching nodes but will schedule it elsewhere if necessary


Example:

```bash

spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd

```
This configuration ensures the pod is scheduled only on nodes where the disktype label is set to ssd

**Advantages:**

Supports complex expressions using operators like In, NotIn, Exists, and DoesNotExist.

Allows defining both hard requirements and soft preferences.

Enables more granular control over pod placement


# 🔍 Operators in Node Affinity:

Node affinity uses various operators to define matching rules:


**In**: The label's value must be in the specified list.

**NotIn**: The label's value must not be in the specified list.

**Exists**: The label must exist on the node, regardless of its value.

**DoesNotExist**: The label must not exist on the node.

**Gt**: The label's value (interpreted as a number) must be greater than a specified value.

**Lt**: The label's value (interpreted as a number) must be less than a specified value.


![image](https://github.com/user-attachments/assets/8db82ada-bba1-4646-9913-349af45019ef)







3. Taints
Taints are applied to nodes to repel certain pods. They allow nodes to refuse pods unless the pods have a matching toleration.
Usage: Use kubectl taint command to apply taints to nodes. Include tolerations field in the pod's YAML definition to tolerate specific taints.

4. Tolerations
Tolerations are applied to pods and allow them to schedule onto nodes with matching taints. They override the effect of taints.
Usage: Include tolerations field in the pod's YAML definition to specify which taints the pod tolerates.

