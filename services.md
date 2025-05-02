**#Why we need Service !!!! ?**


## PODs are Empheral !!!
PODs are empheral , means they can die and recreate ,and move on another node.  POD IP addresses is dynamic and can change on pod crash/re-creation whenever there is some issue with pod's health kubernetes tries to recover it by re-creating it.

So IP Address of POD does not provide reliable stable connectivity this is where **SERVICE** comes to provide stable network connectivity using static ip address with DNS name/FQDN.

**Static IP-Address using a Service:**
Service provides you a persistent static/stable IP address with the DNS Name (service name resolves to its own ip-address).
SO ,Instead of communication over POD’s ip address always use “**service**”

- 
# Service provides:
```bash
- To establish a Stable network Communication/connectivity between pods or different microservices.
- Access pods externally or from internet reliably
- LoadBalancing/Failover/Redundancy - Route traffic to multiple pods ( healthy pods only)
```


**Service Connectivity Flow:**   USER Request --->> Service IP > Registered Endpoints/PODs



**## Key Points that Service Provides:**
A **Service** in Kubernetes is like a network load balancer or a traffic manager for your application.
It helps to expose your pods (the running units of your application) so that they can be accessed by other services or external users, even if the underlying pods are dynamic (i.e., they might start, stop, or get replaced over time).

PODS automatically get registered as a **backend** endpoint for a service in case of re-creation.

**Key Points:**
- Service = Stable Network Access for your Pods.
- It provides a consistent IP address or DNS name for accessing a set of Pods.
- Kubernetes automatically handles the routing of requests to the available pods.


# **In-short :**
**A Service in Kubernetes is a stable endpoint for accessing a dynamic set of pods.** means maps a fixed ip address to a logical group of pods.
It acts as a bridge between users or other services and your pods, ensuring that requests reach the right pod, even as pods come and go.



**Types of Services in Kubernetes:**

## internal Service:

**ClusterIP (default):** 
Exposes the service only inside the cluster (you can’t access it externally).

Example: You have a set of microservices inside the cluster that need to talk to each other.

By default (when you dont specify service type while creating it), a Kubernetes service is private to the cluster. This means only applications inside the cluster can access them. so we use internal service when we want one pod to connect with other ( communication between pods) - and dont want to access it externally or outside cluster or from internet.

![image](https://github.com/user-attachments/assets/2167d2a9-1810-4240-b8d0-3347611fe601)


# Create a Service using command: (Expose a deployment using command)
```bash
#kubectl expose deployment nginx –type ClusterIP
```
**NOTE:**  this command wouldn't work if you havent specified  **port** while creating a deployment -  as service don’t know to which port send the traffic or pod listening on which port to receive the traffic. So you have to explicitly define the port as below;

```bash
#kubectl expose deployment nginx –type ClusterIP  --port 80
```
By default, type will be **ClusterIP** if you don’t specify it.

```bash
kubectl expose deployment nginx   --port 80
 ```

**Service DNS name resolution:**
```bash
dig nginx.default.svc.cluster.local 
```
it will resolve to static ip address of a service


## Commands
#kubectl get svc
#kubectl get service
#kubectl describe service
#kubectl get services -o wide   --> wide view


## External Service:

**NodePort:** 
Exposes the service on a static port on each Node’s IP, so it’s accessible from outside the cluster.

Example: A simple web application that you want to expose externally for testing.

**LoadBalancer:**
Automatically provisions an external load balancer (usually in a cloud environment like AWS, Azure, etc.) to expose the service outside the cluster.

Example: A production-ready API that needs to be accessed over the internet.

**ExternalName**:
Maps the service to an external DNS name (e.g., an external service outside the cluster).

Example: When you want to access a service running outside of Kubernetes (like a database hosted externally).



**# DNS Name Resolution in Service:**
