# Persistent Volume in Kubernetes

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


## How to create/define persistent volume:

its a kubernetes resource object defined via YAML. but it is an abstract component - means it need some physical storage. like

local hard driver from the cluster nodes.
External NFS Servers outside kubernetes cluster.
Cloud Storage , AWS, Azure etc

![image](https://github.com/user-attachments/assets/fc25db46-e563-43ae-9b93-0a1ccde611f0)


![image](https://github.com/user-attachments/assets/cb731dd7-6054-4118-8efc-f8ade1974d6e)


Persistent Volume Claim (PVC):


Storage Class:
