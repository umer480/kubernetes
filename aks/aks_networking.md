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


### UnderStand Traffic Directions:
 
 🔹 Egress vs Ingress in AKS (and networking in general)


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



### Routing should be proper in both directions ( reverse/response path too):


<img width="1140" height="577" alt="image" src="https://github.com/user-attachments/assets/4f3c7b30-ead8-462b-9e7f-1e77550e137e" />



### You should be aware of the Communication Type:

1- Traffic Route????   `If IPs are preserved → it’s routing.`  \

2- Traffic Nat ???    `If IPs are changed → it’s NAT.`



### Ping/ICHO test is not enough:

`Ping only tests ICMP, not real app traffic`

ping sends ICMP packets (echo request/reply).

Most apps use TCP (HTTP, SQL, SSH) or UDP (DNS, streaming).

Just because ICMP works doesn’t mean TCP/UDP will.



You can receive a response from ping (  if actual service is down or even the server is down ) -- if the ping response is up, then it certainly does not mean you sactual server/service/app is up - the desired app which you want to access.


**Firewalls and devices treat ICMP differently**:

`Many firewalls, load balancers, and cloud services block or ignore ping but allow TCP/HTTP`.

`Or the opposite: they may allow ping but block certain ports.`

**Ping doesn’t check ports**:

Example: You can ping a web server on 10.0.0.5, but if port 443 is blocked, your HTTPS app still won’t work.

**Routing vs Application Layer**

Ping only proves basic reachability (can I get a packet to that IP?).

It does not prove the service/application is available or responding correctly

**NOTE**: 
=====Some cloud services (e.g., Azure LB, NAT Gateway) don’t respond to ping, even though the application behind them works fine.=====


**Trace Route**: It just  Shows you the path packets take from your machine to the destination. - its even not enough to track where exactly traffic dropped. Mean does not trace TCP traffic.



### You should be aware of the protocols TCP/UDP:

TCP / UDP / PORT number ???   

HTTP 80  (custom 8080)  \
HHTPS 443   (custom 440)  \
SSH 22 (custom 2200)  \
RDP 3389 ( custom 3398)  \
DNS 53 TCP+UDP 


### Trace TCP Port: Routing validation:

**Basic** : telnet client. \
**Advance**: TraceTCP /TCPPing  Check if the Specific TCP Port is accessible throughout the path:  like utilities :  'TraceTCP /TCPPing' ,Test-NetConnection (Powershell)  , Netcat


### SSL/TLS Certs are involved in communication:

SSL Expiry \
Certificate Chain/Trust issues. \ 
SSL Version missmatch (TLS1.0 , 1.2, 1.3 )  -Client-Server Model
is mTLS implemented? is client certificate  required?


### Firewall throughout the Path:

If Routing is fine in both direction then you must think about firewall filtering while troubleshooting  \

<img width="309" height="212" alt="image" src="https://github.com/user-attachments/assets/89e7b959-c8d4-4488-8ac8-7041eea021dc" />



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










