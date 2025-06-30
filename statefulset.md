# StateFulSet

'A StatefulSet is a Kubernetes is  used to manage stateful applications'

### 📘 Example Use Cases - Stateful Application Examples:

**Databases**: MySQL, PostgreSQL, MongoDB

**Distributed Systems**: Kafka, Zookeeper, Cassandra

**Clustered Applications**: Redis Cluster, Elasticsearch


### Properties of StateFulSet:

- Each pod has a stable identity ( name, and storage).

- Pods are created in order and deleted in reverse order.

- Scaling and rolling updates happen one pod at a time to preserve identity and state.


![image](https://github.com/user-attachments/assets/27626f82-35ec-47dc-8961-841766acdc7b)

![image](https://github.com/user-attachments/assets/a5dbff59-9b70-40b8-b5e7-5ede209a0e12)



## ❓ Does a Pod Have a Built-in DNS Name Without Any Service?

No, a Kubernetes pod does not have a stable or built-in DNS name unless you expose it through a Service, especially a Headless Service for per-pod DNS resolution.


🔍 Without a Service
Pods have IPs, but:

Those IPs are ephemeral (can change if the pod is recreated).

There's no DNS name assigned to a pod by default.

You cannot reliably communicate with pods using DNS without a service.

| Type                                     | DNS Behavior                                                                                         |
| ---------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| **ClusterIP Service (default)**          | Creates a **single DNS name** that **load balances** across pods.                                    |
| **Headless Service (`clusterIP: None`)** | Creates **individual DNS records per pod**, such as:<br>`pod-0.svc-name.namespace.svc.cluster.local` |


Note: `without a headless service, there's no way to resolve mysql-0 by name.`

### 🧠 Why use StatefulSet?


| Feature                         | Why it's Needed                                                                                |
| ------------------------------- | ---------------------------------------------------------------------------------------------- |
| **Stable Network Identity**     | Each pod gets a predictable DNS like `mysql-0`, `mysql-1`, etc.                                |
| **Stable Storage**              | Each pod gets its own persistent volume that **sticks** with the pod even if it's rescheduled. |
| **Ordered Startup/Termination** | Useful for databases or clusters (e.g. MySQL master-slave, Kafka, Elasticsearch).              |
| **Scaling with Identity**       | Useful for systems where each instance has a specific role (e.g. master, replica).             |



# LAB 
Combine concepts of : 
1- Stateful Set
2- Headless Service
3- PV/PVC

![image](https://github.com/user-attachments/assets/2e33fdf1-1459-4ee2-88a9-4d78d19c3b66)


Headless Service is required to get dns names that directly points to pod ips;

![image](https://github.com/user-attachments/assets/8391d828-f011-4d08-9224-179d31ca2bb6)


PV connectivity:

![image](https://github.com/user-attachments/assets/edd97429-8343-42f9-b03d-9059a057d689)



Stateful Set:

Syntax: <headless service-name>.<namespace>.svc.cluster.local

Example:
busybox.default.svc.cluster.local
will return ips of all pods.

now it depends on client to which pod it will make direct conncetion , try 1st ip and so on in case of failure.



When You specify HeadLess in statefullset specs then it generates dns names for each pod seperately.

```bash
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: busybox
spec:
  serviceName: "busybox"   --headless service name
  replicas: 2
```



## 1- Stateful Set When all pods share same PV???
## 2 Stateful Set When all pods has it own separate PV PVC binding ??? in this case you need to manage daya sync between master/slave on app level

### 1- Stateful Set When all pods share same PV???


✅ What You Should Do for Per-Pod Storage:

```bash
volumeClaimTemplates:
- metadata:
    name: data
  spec:
    accessModes: ["ReadWriteOnce"]
    resources:
      requests:
        storage: 1Gi

```
In this case, Kubernetes will create one PVC per pod, named:

data-mysql-0

data-mysql-1

...

Each pod will then get its own independent PV, preventing conflicts.





### 2 Stateful Set When all pods has it own separate PV PVC binding ??? in this case you need to manage daya sync between master/slave on app level

🧬 In StatefulSet:
If you do not use volumeClaimTemplates, and instead just reference a pre-created PVC like this:

```bash

volumeMounts:
- name: shared-data
  mountPath: /var/lib/mysql
volumes:
- name: shared-data
  persistentVolumeClaim:
    claimName: shared-mysql-pvc
```

`Then all pods in the StatefulSet will mount the same PVC (shared-mysql-pvc) — and that PVC is backed by a single PV. This can lead to`:

Data corruption

File locking issues

Unexpected behaviors (especially for databases)





Syntax:
<pod-name>.<service-name>.<namespace>.svc.cluster.local

Example:
nslookup busybox-1.busybox.default.svc.cluster.local
ping busybox-1.busybox.default.svc.cluster.local



<pod-name>.<service-name>.<namespace>.svc.cluster.local



| Scenario                      | Behavior                                           |
| ----------------------------- | -------------------------------------------------- |
| `volumeClaimTemplates` used   | ✅ Each pod gets **its own PV**                     |
| Common PVC name used manually | ❌ All pods share **one PV**                        |
| `emptyDir` used               | ✅ Each pod gets a **new empty volume** (ephemeral) |
| No volume defined             | ✅ Container runs without persistent storage        |




![image](https://github.com/user-attachments/assets/9e54e53c-f9a2-4fbe-bf70-59d53ece9abd)

![image](https://github.com/user-attachments/assets/eff3b9aa-c45b-4617-a164-622da1250fd6)

![image](https://github.com/user-attachments/assets/7ec97256-8674-47c9-9b3a-53641c1efdda)


![image](https://github.com/user-attachments/assets/06fbd6e1-4e0f-41d4-bcd9-f0b0be1e6d05)


![image](https://github.com/user-attachments/assets/5c719ab8-a4ae-46fd-b71f-45dc18cb7255)



![image](https://github.com/user-attachments/assets/14154b90-2861-4cef-93b9-9c6b6d29b413)

