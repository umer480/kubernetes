![image](https://github.com/user-attachments/assets/1ff84946-3d6c-43e0-9ba6-d73459527fcc)# Persistent Volume in Kubernetes

## Why we need persistent volume ?

**Problem**:

`data will be loss on pod restart`

![image](https://github.com/user-attachments/assets/ded3cf09-52e1-4aa3-ad8c-2b013421b367)

**Solution**:

`so you need that storage that does not depend on pod lifecycle`

![image](https://github.com/user-attachments/assets/84f391cf-32db-49c5-b767-9d74b514e2b6)



If you have multiple pods across different nodes then storage (with up to date data) must be available/accessible from all the nodes.

![image](https://github.com/user-attachments/assets/27a11094-3614-4a1b-ac23-6001b69e7e65)



Storage outside kubernetes cluster:

![image](https://github.com/user-attachments/assets/c8f471d3-4f33-48ea-b839-98ebacc5ba7e)


another use case where you should have persitent volume:
its not only specific to databases;

![image](https://github.com/user-attachments/assets/f51467f8-f3e9-4b87-89cc-8bca7903c848)


## Local Storage Vs Remote Storage:

![image](https://github.com/user-attachments/assets/074fe635-ff26-469c-9ab8-e7fa5bd13565)




## How to create/define persistent volume:

its a kubernetes resource object defined via YAML. but it is an abstract component - means it need some physical storage. like

local hard driver from the cluster nodes.
External NFS Servers outside kubernetes cluster.
Cloud Storage , AWS, Azure etc

![image](https://github.com/user-attachments/assets/fc25db46-e563-43ae-9b93-0a1ccde611f0)


![image](https://github.com/user-attachments/assets/cb731dd7-6054-4118-8efc-f8ade1974d6e)


## NameSpace - Relation

PV are not namespace-specific - it can be used across all namespaces.
BUT pvc and pod should be exist in the same namespace

![image](https://github.com/user-attachments/assets/4d96920d-e48d-45ea-9797-e3ff84766f3c)


Types of Volumes that kubernetes support:

![image](https://github.com/user-attachments/assets/4624f5bc-89f5-48e2-b0c9-010ce324fded)



Storage Administrator = provision PV
Application Administrator = provisiong PVC
application has to claim the persitent volume

**Flow**:

![image](https://github.com/user-attachments/assets/f4b3dd08-d201-42a9-bb6e-975bbaf84ee3)

### Persistent Volume Claim (PVC:
When you define PVC you define required specs of volume ; like desired storage space,access mode.

Whatever PV matches criteria or satisfies the claim will be used for the application.

### How ot works !
`A single PersistentVolume (PV) can only be bound to one PersistentVolumeClaim (PVC) at a time`.

- When a PVC is created, Kubernetes searches for a matching PV.

- Once it binds the PVC to a PV, that PV is marked as “Bound” and cannot be claimed again.

![image](https://github.com/user-attachments/assets/0fa5eb5e-d6ee-48bb-a951-c912badb5ca5)

### How to Bound PVC with POD:

- POD requests the Volume through PV Claim.


![image](https://github.com/user-attachments/assets/aee65e30-b476-4aea-93f5-e28e5c5fee54)


### Levels of Volume abstractions;

- POD requests the Volume through PV Claim.
- Claim tries to find a volume in the cluster
- which PV satisfies the claim - volume has the actual backend storage.

![image](https://github.com/user-attachments/assets/726fdf87-a4f4-4f50-827a-477083178e58)



![image](https://github.com/user-attachments/assets/921e29eb-a1dd-4ab0-aa6c-ef5c8bd32591)


**Easier for developers;**

![image](https://github.com/user-attachments/assets/c3d923fc-14bb-4e9a-a26e-3ff31de4de82)



### mounting with multiple containers of same POD

If you have multiple containers within the same POD, you will need to mount volume explicitly with all containers according to your requirement.


### Different Volume Types in a single POD:

![image](https://github.com/user-attachments/assets/689d1e60-f094-4466-8792-e7e197905de4)

![image](https://github.com/user-attachments/assets/46f365de-3421-4d90-972e-305deff8a11f)


# Storage Class:

⚙️ **Dynamic provisioning** : `SC provisions persistent volumes dynamically.` \
     --------------------------------------------------------------------->  `When PersistentVolumeClaim claim it`

A StorageClass is a blueprint or template that tells Kubernetes how to dynamically provision PersistentVolumes (PVs) when a PersistentVolumeClaim (PVC) is created.

Instead of pre-creating PVs manually, StorageClasses automate the creation of storage using a specific backend (like Azure Disk, AWS EBS, NFS, etc.).


![image](https://github.com/user-attachments/assets/26dc4692-8e0c-4bd5-9a4f-e1319ab5cf57)


![image](https://github.com/user-attachments/assets/5f68c031-a372-47ad-ac17-20167dc8b546)


### How to call or refer a Storage Class:

![image](https://github.com/user-attachments/assets/988679da-ebfb-4b21-b94b-eb1d472a4f85)


### Complete Flow - How POD get Volume via SC when request through PVC:

![image](https://github.com/user-attachments/assets/d1857816-d956-4fe3-becd-43e5abfacde2)



## Access Modes:
| Access Mode     | Description                         | Multiple Pods    | Multiple PVCs |
| --------------- | ----------------------------------- | ---------------- | ------------- |
| `ReadWriteOnce` | Mounted as R/W by **one node only** | ✅ (on same node) | ❌             |
| `ReadOnlyMany`  | Mounted as ReadOnly by many nodes   | ✅                | ❌             |
| `ReadWriteMany` | Mounted as R/W by many nodes        | ✅                | ❌             |


## 📐 Matching Conditions Between PVC and PV (for static provisioning)

Kubernetes uses the following fields to match a PVC to an existing PV:
| PVC Field                    | Must match with PV Field                            |
| ---------------------------- | --------------------------------------------------- |
| `storageClassName`           | `storageClassName`                                  |
| `accessModes`                | Must be **subset** of PV's                          |
| `resources.requests.storage` | PV’s `capacity.storage` must be **equal or larger** |
| `volumeMode` (optional)      | Must match if specified                             |
| `selector` (optional)        | Matches labels on PV                                |


🧪 Example: PVC requesting a PV

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: ""


PV (manually created):


```bash
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-volume
  labels:
    type: local
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  storageClassName: ""
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: "/mnt/data"
```

✅ What happens:
accessModes: match ✅

storage: 10Gi ≥ 5Gi ✅

storageClassName: both are empty (no dynamic provisioning) ✅

→ Kubernetes binds the PVC to the PV.


💡 What if StorageClassName is defined?
Then it uses dynamic provisioning:

```bash
storageClassName: standard
```
- If no matching PV exists, Kubernetes uses the StorageClass definition to provision a new PV dynamically.

- The new PV is then bound to the PVC.

🔄 **One-Time Binding**

Once a PV is bound to a PVC, it cannot be reused by another PVC unless:

The PVC is deleted,

And the PV’s reclaimPolicy is set to Retain or manually reset to Available.


## 🔍 Matching Diagram (Text-Based)

```bash
PVC
 ├── storage: 5Gi
 ├── accessModes: RWO
 └── storageClassName: "fast"

     ⬇

Kubernetes:
  - Finds existing PV with ≥5Gi, RWO, "fast"
  - OR dynamically provisions using StorageClass "fast"
     ⬇

PV is Bound to PVC

```

**Summary**:


| Component           | Condition                  |
| ------------------- | -------------------------- |
| Access Modes        | PVC must be subset of PV   |
| Storage Size        | PV must be equal or larger |
| Storage Class       | Must match exactly         |
| Selector (optional) | Matches labels on PV       |

