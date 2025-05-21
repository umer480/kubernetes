# Troubleshooting - Pods not schedulable:

1- Node Selector.

2- Node Affinity.

3- Taints.

4- Tolerations.




# Questions !!!!!  ❓❓❓

`What are the possible reasons - Why we need to schedule pods/specific applications  on a specific node?`


**There could be a lot of reasons . few of them are below;**

 **1** - You may have windows + linux applications ( Hybrid Cluster)
 
 **2** - Hardware dependent Licensing.
 
 **3**- Application is compatible with specific CPU architecture i.e ARM or AMD.
 
 **4**- Dedicated worker nodes to run only test/dev pods.
 
 **5**- Dedicated worker nodes to run a specific application that require high compute/memory on a server that has good performance and hardware specs.
 
 **6**  Particular node has some special network level access so pod/app can access required or depenent services.
 
 etc etc ....
 




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

**❓ Limitations:**

It’s an exact match (hard match)  only +  no fall back option  **-->** POD will not schedule 

It’s static—you can't define logic like "prefer SSD, but fallback to HDD".


In Kubernetes, node affinity and node selector are mechanisms that control how pods are scheduled onto nodes based on labels. While both use node labels to influence scheduling decisions, they differ in flexibility and expressiveness.



# 2- Node Affinity: (Advanced Scheduling)


Node Affinity is a more expressive way to specify rules about the placement of pods relative to nodes' labels. It allows you to specify rules that apply only if certain conditions are met.


**Types of Node Affinity:**

**1**- **RequiredDuringSchedulingIgnoredDuringExecution**:  Hard requirement; the scheduler will only place the pod on nodes matching the criteria.

**2**- **PreferredDuringSchedulingIgnoredDuringExecution**: Soft preference; the scheduler will try to place the pod on matching nodes but will schedule it elsewhere if necessary


**Example:**

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


```bash
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: disktype
                operator: In
                values:
                  - ssd        # OR bsaed selection from the provided list
                  - hdd
  containers:
    - name: nginx
      image: nginx

```




# Node Affinity with multiple expressions:  EXAMPLE 1

```bash
 
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
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
                  - hdd

          - matchExpressions:       # OR
              - key: region
                operator: In
                values:
                  - us-east1


  containers:
    - name: nginx
      image: nginx

```

# Node Affinity with multi expressions:  EXAMPLE 2

```bash
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - key: disktype
              operator: In
              values:
                - ssd
            - key: region
              operator: In
              values:
                - us-east1
        - matchExpressions:
            - key: disktype
              operator: In
              values:
                - hdd
            - key: region
              operator: In
              values:
                - us-west1

```
**In this configuration:**

`First` **nodeSelectorTerm**: Requires nodes to have both `disktype=ssd` and `region=us-east1.`

`Second` **nodeSelectorTerm**: Requires nodes to have both `disktype=hdd` and `region=us-west1.`

The pod will be scheduled on a node if it satisfies either of these terms.

This approach allows for flexible scheduling policies, enabling pods to be placed on nodes that meet any of the specified criteria combinations.


# 🔍 Operators in Node Affinity:

Node affinity uses various operators to define matching rules:


**In**: The label's value must be in the specified list.

**NotIn**: The label's value must not be in the specified list.

**Exists**: The label must exist on the node, regardless of its value.

**DoesNotExist**: The label must not exist on the node.



# Question - can i use a combination of NodeSelector and NodeAffinity
Yes, you can use both nodeSelector and nodeAffinity simultaneously in a Kubernetes Pod specification. When both are defined, both conditions must be satisfied for the Pod to be scheduled onto a node. This means the scheduler will only place the Pod on nodes that match all specified nodeSelector labels and fulfill the nodeAffinity rules.

**✅ Example: Combining nodeSelector and nodeAffinity** :

```bash
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  nodeSelector:
    disktype: ssd
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: region
                operator: In
                values:
                  - us-east1
  containers:
    - name: nginx
      image: nginx

```

**In this configuration:**

**nodeSelector** requires the node to have the label disktype=ssd.

**nodeAffinity** requires the node to have the label region=us-east1.

`Only nodes that satisfy both conditions will be eligible for scheduling this Pod.`




**Advantages  of Node Affinity:**

- Supports complex expressions using operators like In, NotIn, Exists, and DoesNotExist.

- Allows defining both hard requirements and soft preferences.

- Enables more granular control over pod placement



![image](https://github.com/user-attachments/assets/8db82ada-bba1-4646-9913-349af45019ef)





3. Taints
Taints are applied to nodes to repel certain pods. They allow nodes to refuse pods unless the pods have a matching toleration.
Usage: Use kubectl taint command to apply taints to nodes. Include tolerations field in the pod's YAML definition to tolerate specific taints.


-NoSchedule
-NoExecute
-PrefferedNoSchedule

# 🚫 Taint Effects Explained:

In Kubernetes, taints and tolerations work together to control pod placement on nodes. A taint is applied to a node to repel certain pods, while a toleration is applied to a pod to allow it to be scheduled on nodes with matching taints. This mechanism ensures that pods are only scheduled on appropriate nodes


- **NoSchedule**: Pods without matching tolerations will not be scheduled on the node. (use cases : upgrade cluster / worker nodem , patching OS, (drain before this activity, so pods move to other node, hardware maintenance, permanent removal from cluster, )
- **PreferNoSchedule**: Kubernetes will try to avoid scheduling pods without matching tolerations on the node, but it's not guaranteed.  ( node has some performance issues, or some unknown issue that you are currently troubleshooting... low cpu memory. bottleneck etc .obselete hardware that you are going to retire soon in youe next target phase)
- 
- **NoExecute**: Pods without matching tolerations will be evicted from the node if they're already running. (means all pods immediately stopped/removed on node)



4. Tolerations   -**Exceptions!!! for Taints**

Tolerations are applied to pods and allow them to schedule onto nodes with matching taints. They override the effect of taints.
Usage: Include tolerations field in the pod's YAML definition to specify which taints the pod tolerates.


add toleration on pod then it can still run that node where you configured NoSchedule   ( NoScheduled nodes could be a dedicated nodes to run a special application that require heavy workload   or run a temporary app on that server that you going to deprecate soon)


LAB:

Taint : 4 nodes -- 3 NoSchedule taint , 1 default  ( pod will be schedule on 4th node default/no taint configured)
Toleration : set 4th node as NoSchedule too ----- now add toleration in pod/deployment multiple replicas --> now pod would run on those nodes too that has NOSchedule taint


