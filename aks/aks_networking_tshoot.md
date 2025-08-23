# Networking Troubleshooting - AKS



```bash
"Many people are not fully clear about which tool to use and how to use it correctly when troubleshooting network issues. Fresh engineers, or even experienced ones, often struggle to think in multiple directions due to limited exposure. Below are some key points you should keep in mind while troubleshooting network-related problems — always approach the issue from multiple angles."
```



- Basic Network Communication Rule
- Communication direction is very important - ingress/egress - inbound/outbound - incoming/outgoing
- Routing vs NAT difference - Dual NAT
- Routing vs Application Layer
- Ping/Echo, Traceroute is not enough - its limitations and right usage.
- Firewall filtering throughout the path of communication
- Communication Protocols and Custom Ports.
- Trace TCP ports utilities
- DNS (Domain Name System) is a crucial part
- Involvement of SSL/TLS  Certificate for data transmission. Client <> Server  Communication.



## 🔹🔹🔹 Back to Basics:🔹🔹🔹 

================================================================================

### 🔹 Understand Traffic Directions:
 

```bash

Egress traffic → traffic leaving a Pod/node/cluster (outbound).

Ingress traffic → traffic arriving into a Pod/node/cluster (inbound).
```



### 🔹 Basic Network Communication Rule:


=======================================================================================

<img width="961" height="411" alt="image" src="https://github.com/user-attachments/assets/69bbc4d8-58af-4255-b101-91644b9cf267" />

===================================================================================
<img width="538" height="280" alt="image" src="https://github.com/user-attachments/assets/9516d117-13c1-4aff-920c-f9030185ed19" />






<img width="982" height="307" alt="image" src="https://github.com/user-attachments/assets/0a312308-9131-41fd-b98f-b05d36ca280a" />



### 🔹 Routing should be proper in both directions ( reverse/response path too):


<img width="1140" height="577" alt="image" src="https://github.com/user-attachments/assets/4f3c7b30-ead8-462b-9e7f-1e77550e137e" />



### 🔹 You should be aware of the Communication Type:

1- Traffic Route????   `If IPs are preserved → it’s routing.` 

2- Traffic Nat ???    `If IPs are changed → it’s NAT.`



### 🔹 Ping/ICHO test is not enough:

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
`Some cloud services (e.g., Azure LB, NAT Gateway) don’t respond to ping, even though the application behind them works fine.`



**Trace Route**: It just  shows you the path packets take from your machine to the destination. - its even not enough to track where exactly traffic dropped. Mean does not trace TCP traffic.






### 🔹 You should be aware of the protocols TCP/UDP:

TCP / UDP / PORT number ???   

HTTP 80  (custom 8080)  \
HHTPS 443   (custom 440)  \
SSH 22 (custom 2200)  \
RDP 3389 ( custom 3398)  \
DNS 53 TCP+UDP 


### Examples: foolish things !!! by devops / sysadmin / cloudadmin guys that have a weak concept in networking -- even no common sense!!!! :) 

```bash

'Some people use https://admin.cloudlyncs.com   while the actual port of service is 440 - so the correct URL is https://admin.cloudlyncs.com:440'

same https://admin.cloudlyncs.com:8080

Some applications listen on port 80 (http) but don't redirect automatically to 443 (https/ssl) - so you have to manually type https:// in browser or on client side
```



### 🔹 Trace TCP Port: Routing validation:

**Basic** : telnet client. \
**Advance**: TraceTCP /TCPPing  Check if the Specific TCP Port is accessible throughout the path:  like utilities :  'TraceTCP /TCPPing' ,Test-NetConnection (Powershell)  , Netcat



### 🔹 Firewall throughout the Path:



If Routing is fine in both directions, then you must think about firewall filtering while troubleshooting  \

`Make Sure the Communication Protocol is also allowed over the Network`


<img width="309" height="212" alt="image" src="https://github.com/user-attachments/assets/89e7b959-c8d4-4488-8ac8-7041eea021dc" />



# 🔹 DNS:



`If you don’t understand DNS well, you’ll always get stuck when fixing network problems.`


```bash

“Poor DNS knowledge means network troubleshooting will keep tripping you up.”
```


<img width="189" height="170" alt="image" src="https://github.com/user-attachments/assets/9f1cbb83-ba5d-43f7-9c40-153ece993ab3" />


```bash

“DNS is foundational to networking; lacking strong DNS knowledge will result in recurring troubleshooting roadblocks.”

“Inadequate understanding of DNS leads to persistent difficulties in diagnosing and resolving network issues.”
```





### 🔹 SSL/TLS Certs are involved in communication:

- SSL Expiry \
- Certificate Chain/Trust issues. \ 
- SSL Version mismatch (TLS1.0 , 1.2, 1.3 )  -Client-Server Model
- is mTLS implemented? is client certificate  required?

