# AKS Networking - CNI Overlay


### Brownfield environment: # Deploy AKS on Existing infra/environment.
`Classic Hub-Spoke architecture`: 

A brownfield network environment in networking is a previously developed network or infrastructure that is modified, upgraded, or integrated with new technologies and systems, rather than being built from scratch.


**1**- Kubenet \
**2**- CNI Overlay (POD Subnet)

## 1- Kubenet  -deprecated

`kubenet` Provides Basic networking - **it's not recommended**


In `kubenet1 pods receive IP addresses from a logically separate address space, and traffic is NATed to the node's IP address when leaving the node.


**Three networks/CIDR**:

1- **NODE Network** (Subnet of VNET where AKS Cluster deployed)  **172.16.239.0/24** \
2- **Cluster Network** (within the cluster) - used with 'Services' to communicate with pods  **10.101.0.0/16** \
3- **POD Network** **10.244.0.0/16** (within the cluster)



### in Kubenet:

- NAT is performed.
- An additional hop is required in the design of kubenet, which adds minor latency to pod communication.
- Route tables and user-defined routes are required for using kubenet, which adds complexity to operations

<img width="780" height="349" alt="image" src="https://github.com/user-attachments/assets/0384632f-9b70-4776-b5b7-ac35e7b1097c" />


**Use kubenet when**:

- You have **limited IP** address space.
- Most of the pod communication is **within the cluster**.
- You don't need **advanced AKS features**, such as virtual nodes or Azure Network Policy.

<img width="704" height="197" alt="image" src="https://github.com/user-attachments/assets/21ba158b-8c92-4be6-95ef-424b5aaf7092" />



## 2- Azure CNI Overlay 

### IP Address Planning:

`'Similarly, Azure CNI Overlay networking simplifies IP management by assigning pod IPs from a separate, private CIDR range, not the virtual network (VNet) subnet'`

This means your VNet subnet can be smaller, as it only needs to accommodate node IPs. 

However, you must carefully plan the private CIDR range to ensure sufficient IP addresses for your pods, considering future scaling. Each node gets a /24 subnet for pods, so the overall overlay network subnet must accommodate the total number of nodes and their associated pod IPs.




# in Overlay:

`Pod IPs are not NATed at all inside the cluster.`

When Pod A talks to Pod B (even across nodes), the original Pod IP is preserved end-to-end.

`The only “trick” is that the Pod IP ranges are not part of the VNET → so Azure CNI overlay uses VXLAN encapsulation between nodes to carry that Pod-to-Pod traffic.`


### Communication directions/flow:

🔹 **1- East-West (inside the cluster):**

**Pod ↔ Pod on same node** → They just use the overlay bridge, no NAT.

**Pod ↔ Pod across nodes** → Overlay (VXLAN) tunnels the traffic between nodes. The original **Pod IPs remain intact**.

👉 So **within the cluster, Pods always talk to each other directly with Pod IPs (no NAT involved).**



🔹 **2-  North-South (outside the cluster / VNET / internet):**

When a Pod sends traffic outside the cluster overlay (e.g., internet, Azure SQL, Storage Account, on-prem):

The Pod IP **is not routable outside the overlay.**  -> so to reach the internet (from pod)  the pod's ip should be NAted to node's ip. 

The node’s host network does **SNAT **the Pod IP → node IP (or NAT Gateway/SLB outbound IP).

👉 So the **outside world never sees the Pod IP** — it only sees the node/NAT IP.



<img width="1099" height="746" alt="image" src="https://github.com/user-attachments/assets/9fa431c2-b698-4aa1-b657-ce20283d3c3c" />


<img width="1446" height="674" alt="image" src="https://github.com/user-attachments/assets/f6d9ecc2-6458-414d-9221-39d36b44b731" />

<img width="883" height="226" alt="image" src="https://github.com/user-attachments/assets/380c4ac8-9051-456d-9fbe-bf8fb4a275b1" />




### Outbound connectivity Options for PODs:

1- Standard Public LoadBalancer **(Default)**. 

2- NAT Gateway.

3- Azure Firewall.




### Architecture Diagram - Communication Flows

**1**- internet to pod.\
**2**- pod to the internet.\
**3**- pod to on-premises.\


<img width="1309" height="608" alt="image" src="https://github.com/user-attachments/assets/75bded35-1e02-480c-9d57-35da6dcc1ad6" />



# LAB:

deployment.yaml

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: aks-helloworld  
  namespace: hello-web-app-routing
spec:
  replicas: 1
  selector:
    matchLabels:
      app: aks-helloworld
  template:
    metadata:
      labels:
        app: aks-helloworld
    spec:
      containers:
      - name: aks-helloworld
        image: nginxdemos/hello
        ports:
        - containerPort: 80
```

**Validate POD IP:**

```bash
# kubectl get deployment,pod -o wide
```

So, till now, ^ nodes deployed, pod also deployed using deployment.\
 Now create an external service - expose pods from the internet/outside world

service-external-lb.yaml

```bash
apiVersion: v1
kind: Service
metadata:
  name: nginx-service-external
  namespace: demo-ns
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 80
```


If you want access a service running on pods from on-premise or from with the same VNET where AKS cluster deployed or any other peered VNET then you have to deploy one more loadbalancer service using internal/private load balancer.

service-internal-lb.yaml

```bash
apiVersion: v1
kind: Service
metadata:
  name: nginx-service-internal
  namespace: demo-ns
  annotations:
    service.beta.kubernetes.io/azure-load-balancer-internal: "true"    # -> this make load balancer as internal/private
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 80
```


### now validate service endpoints:
it should point to pods (that have IPs from the overlay network)


```bash
kubectl describe service <external service>
kubectl describe service <internal service>

```

# Troubleshooting
If there is any issue while provisioning/configuring Azure load balancer, get 'events'


```bash
kubectl get events -n <namespace>
``

If any issues related pod:

```bash
kubectl logs <pod name> -f
```

<img width="1165" height="193" alt="image" src="https://github.com/user-attachments/assets/fd990870-d390-41cb-a5e6-56abe46c8423" />


### Implementation:

# Verify inbound and outbound flows
#
# Outbound
k exec -it <nginx-pod-name> -n demo-ns
curl ifconfig.me
# inbound
curl <external-ip>:8080



Validate POD CIDR's for each subnet.
Validate POD IP.
Validate Service IP.


