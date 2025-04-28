**#Why we need Service !!!! ?**

- Communication between pods
- Access pods externally or from internet
- 
PODs are empheral , means they can die and recreat,and move on another node.  POD Ips are dynamic and can change on pod crash/recreation.
So IP Address of POD does not provide reliable stable connectivity this is where **SERVICE** comes to provide stable network connectivity using static ip address.

**What Service actually does:**
Service provides you a persistent static/stable IP address with the DNS Name (service name resolve to its own ip address).
- Instead of communication over POD’s ip address use “service”
-Connect internal microservice/different pods via “service” (pods communicate with each other using a service)

Service also acts as a load balancer – it receives request and forward it to registered endpoints (pods actually)

**Service Connectivity Flow **:   USER Request --->> Service > Endpoints > Registered Pods



**## Key Points that Service Provides:**
A **Service** in Kubernetes is like a network load balancer or a traffic manager for your application.
It helps to expose your pods (the running units of your application) so that they can be accessed by other services or external users, even if the underlying pods are dynamic (i.e., they might start, stop, or get replaced over time).

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

By default (when you dont specify service type while creating it), a Kubernetes service is private to the cluster. This means only applications inside the cluster can access them. There are a number of ways around this, and one of the best is an ingress. In Kubernetes, an ingress lets us route traffic from outside the cluster to one or more services inside the cluster

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
