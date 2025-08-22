# AKS Networking

## Topics:

- AKS Networking Models/Plugins (Kubenet/CNI/CNI Overlay)
- Network Architecture - Hub-Spoke Architecture.
- Traffic Flow ingress and egress.
- Internal and External LoadBalancer.
- Application Gateway. AGIC Controller
- Managed and Unmanaged Nginx Controller.
- Nginx DNS and Key Vault integration. 
- Web Application Firewall / Layer7 Advanced/Smart Routing decisions.
- Front Door + AKS for global ingress
- Private Cluster (API Server only via private endpoint)






### Understand Communication Flows:

```bash
Pod Communication (pod-to-pod)
Pod-to-Service Networking
Internet-to-Service Networking
```

In other words:

```bash
- Communication from/to the internet - External Load Balancer (ELB)
- Communication from/to VNET
- Communication from/to On-Premise  -Internal Load Balancer (ILB)  -VNET Peering - S2S VPN Tunnel
- Communication with Azure Services that have Private EndPoints.
```


### UnderStand Traffic Directions: 🔹 Egress vs Ingress in AKS (and networking in general)


```bash

Egress traffic → traffic leaving a Pod/node/cluster (outbound).

Ingress traffic → traffic arriving into a Pod/node/cluster (inbound).
```

### Back to Basics:
`Basic Network Communication Rule`:

<img width="961" height="411" alt="image" src="https://github.com/user-attachments/assets/69bbc4d8-58af-4255-b101-91644b9cf267" />

<img width="538" height="280" alt="image" src="https://github.com/user-attachments/assets/9516d117-13c1-4aff-920c-f9030185ed19" />

### Make Sure the Communication Protocol is also allowed over the Network:

<img width="982" height="307" alt="image" src="https://github.com/user-attachments/assets/0a312308-9131-41fd-b98f-b05d36ca280a" />


# DNS

<img width="430" height="404" alt="image" src="https://github.com/user-attachments/assets/b34de026-9d3f-483f-9f54-760fd818d65d" />



“If you don’t understand DNS well, you’ll always get stuck when fixing network problems.” \ 
“Poor DNS knowledge means network troubleshooting will keep tripping you up.”

```bash

“DNS is foundational to networking; lacking strong DNS knowledge will result in recurring troubleshooting roadblocks.”

“Inadequate understanding of DNS leads to persistent difficulties in diagnosing and resolving network issues.”
```






# Networking Models / Network Modes / Network Plugins - in  AKS

## What is networking models in AKS:

The networking model in AKS (Azure Kubernetes Service) defines how Pods get their IP addresses and how they talk to each other, to the node, to the internet, and to your VNET.

`Think of it as the rules of the road for traffic inside and outside the cluster.`


1- kubenet. (basic, legacy) --> deprecated) ****  \
2- CNI Overlay  - CNI Pod Subnet \
3- CNI (non Overlay) - CNI Node Subnet \






Reference: 
```bash
https://medium.com/@h.stoychev87/azure-aks-network-components-part-1-2025-edition-ce5439f4c767
```










