# Resource Quotas in Kubernetes


`Resource quotas in Kubernetes are used to allocate and limit compute resources for individual namespaces within a cluster. They help ensure fair and efficient resource utilization among multiple applications or teams running within the same Kubernetes cluster.`



## 🔹 What Happens by Default? When you dont apply any resource quotas/limits ?

**1. No Limits = No Cap**:

- The container can use as much CPU and memory as it wants (or as much as the node has available).

- There's no upper bound on usage, so a single container can consume all the resources on the node if it's allowed to.

🔥 This is risky, especially for memory: if the container uses too much memory, it can cause OOMKill (Out of Memory Kill) — the kernel forcefully kills the container.


**2. No Requests = Best Effort QoS**:

2. No Requests = Best Effort QoS

- The scheduler doesn't reserve any CPU/memory for the pod.

- The pod is assigned a BestEffort QoS (Quality of Service) class.

- It gets resources only if others aren't using them.


**🧪 Example:**

```bash
resources: {}
```

**This means:**

- **No requests**: Kubernetes won't reserve any guaranteed resources.

- **No limits**: Pod can use as much as it wants (up to node capacity), unless quota restricts it.



1-Resource Quotas.
2- Limit Ranges.


## 🔹 1. ResourceQuota

To apply a Resource Quota, you need to define a ResourceQuota object and apply it to the desired namespace


**Purpose:**

`Sets overall resource usage limits for the entire namespace.`

🔧 **Example Use Case:**
Limit the total number of pods, services, or the sum of CPU/memory all pods can use in a namespace.

🧩 **Example:**

```bash
apiVersion: v1
kind: ResourceQuota
metadata:
  name: team-quota
spec:
  hard:
    pods: "10"                # max 10 pods in namespace
    requests.cpu: "4"         # total requested CPU can't exceed 4 cores
    limits.memory: "10Gi"     # total memory limit can't exceed 10Gi

```


- Tracks and enforces namespace-wide limits
- Prevents a single team from using all cluster resources

**You can verify the applied Resource Quota using the following command**:

```bash
kubectl get resourcequota resource-quota-example -n example-namespace
```
`This command displays the current usage and the defined limits, helping you monitor resource consumption within the namespace.`

![image](https://github.com/user-attachments/assets/476723d1-2b97-4987-85d1-252d0822b1e9)


## 🔹 2. LimitRange:
**Purpose:**
`Sets default and max/min limits per Pod or Container within a namespace.`

**🔧 Example Use Case:**


Automatically apply default CPU/memory limits if a developer forgets to set them

Prevent a single container from using too much memory

```bash
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
spec:
  limits:
  - default:
      cpu: 500m
      memory: 512Mi
    defaultRequest:
      cpu: 250m
      memory: 256Mi
    type: Container

```


- Applies per-container defaults and boundaries
- Ensures developers follow consistent resource usage patterns



![image](https://github.com/user-attachments/assets/c948b30c-c959-4f15-8646-fcb197761bb9)



✅ **Use Them Together**

**LimitRange** ensures individual containers behave well.

**ResourceQuota** ensures the whole namespace doesn’t exceed its fair share.




## EXAMPLE:

🔸 **Visual Representation**:

```bash
Namespace: team-a
├── ResourceQuota (Namespace-wide)
│   ├── Max 4 CPU
│   ├── Max 8Gi Memory
│   └── Max 10 Pods
│
└── LimitRange (Per-container)
    ├── Default Request: 250m CPU, 256Mi Memory
    ├── Default Limit: 500m CPU, 512Mi Memory
    └── Max Limit: 1 CPU, 1Gi Memory

```

**Resource Quota YAML:**
```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: team-a-quota
  namespace: team-a
spec:
  hard:
    pods: "10"                 # Max number of pods in the namespace
    requests.cpu: "4"          # Total CPU requests allowed in the namespace
    limits.memory: "8Gi"       # Total memory limits allowed in the namespace

```

```bash

Limit Range Quota :
apiVersion: v1
kind: LimitRange
metadata:
  name: team-a-default-limits
  namespace: team-a
spec:
  limits:
  - type: Container
    defaultRequest:
      cpu: 250m                # Default CPU requested if not specified
      memory: 256Mi            # Default memory requested if not specified
    default:
      cpu: 500m                # Default CPU limit if not specified
      memory: 512Mi            # Default memory limit if not specified
    max:
      cpu: "1"                 # Max CPU limit a container can set
      memory: 1Gi              # Max memory limit a container can set
```


✅ **How They Work Together**

**If a developer forgets to define CPU/memory in their Pod**:

     - The LimitRange sets default requests and limits.

**Kubernetes sums all the containers' requests and limit**s.

     - The ResourceQuota makes sure the total stays within the defined budget.

**If a Pod causes the quota to exceed**:

    - The Pod won’t be scheduled until more resources become available.





## Define Resource Limits directly on POD/Contianer level: (without applying directly at NS level)


# Question : what if a pod specifiy resource limit in its own object then resource limits will override?

✅ Yes — Pod/Container resource limits override the defaults from LimitRange.


**Example**: 

**Limit Range**:

```bash

apiVersion: v1
kind: LimitRange
metadata:
  name: default-limit
  namespace: dev
spec:
  limits:
  - type: Container
    default:
      cpu: "500m"
      memory: "512Mi"
    defaultRequest:
      cpu: "250m"
      memory: "256Mi"
```


🧩 **Pod with custom limits**:

```bash
apiVersion: v1
kind: Pod
metadata:
  name: custom-limits-pod
  namespace: dev
spec:
  containers:
  - name: my-app
    image: nginx
    resources:
      requests:
        cpu: "1"        # Overrides LimitRange defaultRequest
        memory: "1Gi"   # Overrides LimitRange defaultRequest
      limits:
        cpu: "2"        # Overrides LimitRange default
        memory: "2Gi"   # Overrides LimitRange default

```


🟢 **In this case**:

- The pod uses its own values for requests and limits.

- The LimitRange is ignored for this pod.

**Summary**

- Pod/Container resource specs take precedence.

- LimitRange is just a fallback (default) for when limits/requests are not specified.


![image](https://github.com/user-attachments/assets/4d27acde-c8a0-4e29-9abd-e1b0015ddc3a)



![image](https://github.com/user-attachments/assets/63cdd258-27b3-40d3-bf72-31dae124aeff)



![image](https://github.com/user-attachments/assets/fee88aa5-b986-45d0-915c-ae996a06b82f)

Requests for the creation, deletion, and update of system resources go through the Kubernetes API server. There are different admission controllers that can view and filter the requests. The quota operates until the resource limit is reached or violated.

Once the resource quota object is defined on a namespace by the Kubernetes cluster administrator, the Kubernetes quota admission controller watches for the new objects created in that namespace. Then it will keep monitoring and tracking resource usage.
Reference : https://medium.com/@muppedaanvesh/a-hand-on-guide-to-kubernetes-resource-quotas-limit-ranges-%EF%B8%8F-8b9f8cc770c5

