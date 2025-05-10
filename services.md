**#Why we need Service !!!! ?**


## PODs are Empheral !!!
PODs are empheral , means they can die and recreate ,and move on another node.  POD IP addresses is dynamic and can change on pod crash/re-creation whenever there is some issue with pod's health kubernetes tries to recover it by re-creating it.

So IP Address of POD does not provide reliable stable connectivity this is where **SERVICE** comes to provide stable network connectivity using static ip address with DNS name/FQDN.

![image](https://github.com/user-attachments/assets/f711a949-50fb-415d-a386-b4eee754bd67)


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


![image](https://github.com/user-attachments/assets/109b6fa9-40c7-4ff4-9c91-ea99c592322d)




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
it will resolve to static ip address of a service.

When you create a service, Kubernetes sets up a virtual IP address (ClusterIP) and a DNS entry that resolves to this IP address. This DNS entry is the **hostname** you can use to access the service from other pods within the same Kubernetes cluster.

## DNS Provides: Access Service via its 'name' instead of its ip-address
**1.	Abstraction**: It abstracts away the complexity of managing IP addresses and allows services to be referenced by a meaningful name.
**2.	Resilience:** If a pod is rescheduled or replaced, the DNS name will still resolve to the correct IP address of the new pod.

When configuring your applications to communicate with services within a Kubernetes cluster, it's recommended to use the DNS name (hostname) of the service rather than its IP address. This makes your application more flexible and resilient to changes within the cluster.

**Example:**
In the case of the MongoDB example, the Apache application deployed in the same cluster can connect to the MongoDB service using the DNS name **mongodb-service**.
This DNS name resolves to the IP address of the MongoDB service, enabling communication between the Apache application and the MongoDB database.


# Create a ClusterIP using a YAML file

```bash

apiVersion: v1
kind: Service
metadata:
  name: example-service
spec:
  selector:
    app: example-app
  ports:
  - name: http
    port: 80
    targetPort: 8080
  type: ClusterIP
```

In this example, the Service is named example-service and is of type ClusterIP.

The selector **app: example-app** is used to select all Pods that app: example-app will be part of this service.

The Service will be available on **port 80**. **Target port 8080** is used to forward traffic to the container.

Once you have created the definition file, you can create the Service by running the following command:

```bash
kubectl create -f cluster-ip-service.yaml
```
This will create a new Service in the cluster, which can be reached from within the cluster using the Service’s IP address.

![image](https://github.com/user-attachments/assets/5648d2aa-de34-4746-9c5a-b4bd3081295a)



# More Explanation about ClusterIP Service:

When a client within the cluster makes a request to the ClusterIP, it is automatically load balanced to one of the available pods. This way, you can access your service using the ClusterIP, and Kubernetes will automatically route the traffic to the appropriate pods.

In summary, ClusterIP services are designed for internal communication within a cluster and provide a simple way to access a set of pods behind a stable IP address.

The ClusterIP exposes the Service on a cluster-internal IP, which makes it reachable from within the cluster only.

The ClusterIP provides a load-balanced IP address. One or more pods that match a label selector can forward traffic to the IP address. The ClusterIP service must define one or more ports to listen on with target ports to forward TCP/UDP traffic to containers.



#  How does a Kubernetes Service Work?

**Based on Labels:**

![image](https://github.com/user-attachments/assets/0287dc0c-06ec-4031-bd95-4038dfef66c0)


Services make use of the **labels** that are assigned to the pod or deployment to select the correct pod. A service object can even be configured to target a deployment, so all the pods created by a deployment will be exposed by the service object. A single service can also be used to target a group of different pods. You just need to ensure that the correct labels are being selected.

Let’s say that you have three different pods, with different labels. We want to create a single service that will expose all the different pods. You can create this service, by adding a common label to the pods. Let’s say that the common **label is app=demo**. The service will select and expose all the pods that have the **label app=demo.**

![image](https://github.com/user-attachments/assets/cffbb735-8106-4369-aa50-b2fd1487d045)



**Reference:** https://devtron.ai/blog/understanding-kubernetes-services/




## Commands
#kubectl get svc
#kubectl get service
#kubectl describe service
#kubectl get services -o wide   --> wide view


# How Service know to which deployment route a traffic : or which deployment is a part of service
service must know which pods should register with it -- **so HOW ???**
Define label on deployment and call it from service by defining selector ( service selector uses deployment's 'lables')

selector (of a service) basically creates a connection between the deployment ( or can say its pods)


![image](https://github.com/user-attachments/assets/1574d3f1-dfc8-4fda-acb5-940fe0c282cc)


## External Service:

**NodePort:** 
Exposes the service on a static port on each Node’s IP, so it’s accessible from outside the cluster.

Example: A simple web application that you want to expose externally or want to access from internet or even from LAN.

![image](https://github.com/user-attachments/assets/bd70f81c-b6c1-4825-a2ae-7ecc1231e5ff)



![image](https://github.com/user-attachments/assets/1e9fca04-2024-4f60-9578-02f52b20887d)

## Combination of ClusterIP and NodePort Sevice:
**APP :** External/NodePort

**DB :** Internal/ClusterIP/Private

Connect app with db via internal service

![image](https://github.com/user-attachments/assets/576501bc-dab7-4bb8-90e3-756ba403e89c)




## Create NodePort Service :

# When you define container pod:

```bash
kubectl expose deployment nginx --type NodePort --target-port 80
```
# When you dont't define container pod: (--target-port)

```bash
kubectl expose deployment nginx --type NodePort
```


**See screenshot reference below;**

![image](https://github.com/user-attachments/assets/f6310307-3122-4c57-a509-13efd1727608)

In this picture – you haven’t explicitly specified  container's port ( because while creating pods you already defined its ports  (80,443) so service get ports details from there – and as the type is NodePort so this command automatically expose/bind 2 host/node ports to container's both ports separately.

**PORTs Mapping:**
30817 --> 443
31989 --> 80 



 # Create NodePort via yaml:
```bash

apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  selector:
    app: nginx  # Ensure this matches your deployment's label
  type: NodePort
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP

```
**Notes:**
selector. **app: nginx **must match the label of the nginx deployment.
If your deployment uses a different label (e.g., app: web), adjust the selector accordingly.


you can create a deployment using below file and the above created service will route traffic to this deployment (based on label and selector)
```bash

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:latest
          ports:
            - containerPort: 80


```


# NodePort also creates ClusterIP: :)  Real Concept !!!!

when you create a NodePort Service, Kubernetes also creates a ClusterIP Service automatically.

 ============ **A NodePort service is built on top of a ClusterIP.** ============

**When you define a NodePort service, it:**

1- **Automatically creates an internal ClusterIP service for internal communication.**

2- **Exposes that service to a specific port on each Node in your cluster.**

**You can access your application in two ways:**

1- Internally via the ClusterIP within the Kubernetes cluster.

2- Externally via the Node's IP and the specified NodePort.


![image](https://github.com/user-attachments/assets/5be44d1e-d177-467e-9949-25caae5ff3fc)

# **Real Scenario : Use Case**

![image](https://github.com/user-attachments/assets/52146238-3c42-44f7-8cfe-0d2d465914da)



 # NodePort Range:

![image](https://github.com/user-attachments/assets/24abd953-f9d8-42a2-bae2-5127df5a1baa)


Override nodeport:

**By default,** Kubernetes assigns a random port from the range 30000–32767, but you can override this by specifying your desired nodePort value within that range.

```bash
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80            # Service (cluster-internal) port
      targetPort: 80      # Container port
      nodePort: 31080     # External port exposed on all nodes ⚠️

```
## **NOTE: **      ⚠️⚠️⚠️⚠️⚠️⚠️⚠️
nodePort: 31080     # External port exposed on all nodes

In this example, port 31080 on any worker node will forward traffic to port 80 inside the pod/container.

⚠️ The nodePort must be within the Kubernetes NodePort range (30000–32767) unless you change the default config on the kube-apiserver.



#### Very Very important   - on which worker node port actually  open ?????
 
When you use nodeport then it open on all Kubernetes worker nodes – you can validate using netstat – all worker nodes will listen to that port.


![image](https://github.com/user-attachments/assets/3b46810c-4c6f-45b0-990b-36683b34e5d6)




**LoadBalancer:**

A LoadBalancer is also a type of service that exposes the pod to external traffic. As the name implies, a LoadBalancer service distributes the traffic between the pods that are targeted by the service.
So for example, if you had 5 pods of the same application which were targetted by the load balancer service, the traffic would get distributed equally among the five pods. When you use a NodePort service, there are no such load-balancing capabilities. The traffic must be manually distributed. 


Automatically provisions an external load balancer (usually in a cloud environment like AWS, Azure, etc.) to expose the service outside the cluster.

Example: A production-ready API that needs to be accessed over the internet.


# Why we need LoadBalancer Type Service:???
**1- Security Concern**-   NodePort services do not provide the same level of network security as LoadBalancer services, as they expose the service directly to the node.

**2- No Advance Level LoadBalancing:** NodePort services are useful in situations where you need to expose a service to the external network but do not require the advanced features of a LoadBalancer service.

**3- No Node-Level Redundancy/LoadBalancing:** Kubernetes doesn’t automatically load balance incoming traffic across nodes.You must handle that externally if you want high availability or balanced traffic at the node level.

**Key-Concept:**
- kube-proxy ensures the NodePort on any node can route to any pod in the cluster, not just local pods.
- This is cluster-level load balancing to pods, not node-level load balancing from outside the cluster.


# Create LoadBalancer Service via yml
```bash
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app
  ports:
  - name: http
    port: 80
    targetPort: 8080
  type: LoadBalancer
```

**In this example,** the LoadBalancer service named my-service will expose a service running on port 80 and target port 8080. The service will balance the load between pods labeled with app: my-app.

Once you create this service, Kubernetes will automatically create a load balancer in your cloud provider and configure it to forward traffic to the pods in your service. The service can then be accessed from the external network using the IP address of the load balancer.

![image](https://github.com/user-attachments/assets/3956d8c8-d686-4f8d-9cb4-7a4fdee34b20)


You have a limited budget: LoadBalancer is often more expensive than NodePort, as it requires a load balancer in the cloud provider. If you have a limited budget, NodePort is a more cost-effective solution.

**ExternalName**:
it does not create a cluster IP or route traffic within the cluster. Instead, it acts as an alias for an external DNS name by returning a CNAME record for the provided external name.

Maps the service to an external DNS name (e.g., an external service outside the cluster).

Example: When you want to access a service running outside of Kubernetes (like a database hosted externally).


Apart from the three services that we’ve learned about above, Kubernetes also has a special type of service called ExternalName. ExternalName is unique as it does not use labels and selectors like the other types of services. Instead, the service maps to a DNS name using a CNAME record.

Within the manifest file of the ExternalName service, we will have to define a DNS endpoint in the field called as externalName. Whenever a request is made to this service, the traffic gets routed to the DNS record that is defined in the manifest file.

The ExternalName service type is particularly useful when you wish to access a service that is hosted outside the Kubernetes cluster. It’s also helpful when you are trying to migrate your applications to Kubernetes. The ExternalName service helps maintain dependencies on external applications while you are migrating the applications to Kubernetes.

Let’s take a look at an example of how to create an ExternalName service. In the below YAML manifest, we have defined the service named as external-demo, and the service type is defined as ExternalName. We have also configured the ExternalName field so that it routes the requests to api.google.com. 


# Create ExternalName via Yaml

```bash
apiVersion: v1
kind: Service
metadata:
  name: my-external-service
spec:
  type: ExternalName
  externalName: api.external-service.com
```
# Real Use-cases where we can implement or utilize ExternalName Service


**DNS-based redirection:**

You can access an external service using a Kubernetes-native DNS name.

This is especially helpful when you want applications inside the cluster to use a consistent service name, regardless of whether it's internal or external.

**No need to hardcode external URLs:**

Abstracts the external address so your apps don't need to know or manage the real hostname.

If the external endpoint changes, you only need to update the ExternalName—not every app using it.

**Simplifies configuration and testing:**

Ideal when integrating with third-party APIs or services like databases, SaaS platforms, or APIs that live outside your cluster.


# Limitation:
**1-** Only works at DNS level: It doesn't proxy or load balance — it just returns a CNAME.

**2-** No built-in health checking: Kubernetes doesn’t validate if the target host is reachable or healthy.



# Summary - Take Aways - Key Points

**Top 4 Kubernetes Service Types in one diagram.**

**The diagram below shows 4 ways to expose a Service.**

![image](https://github.com/user-attachments/assets/45b07e3a-7493-4a8c-b4f1-8a1ef0ca3d0d)




In Kubernetes, a Service is a method for exposing a network application in the cluster. We use a Service to make that set of Pods available on the network so that users can interact with it.

There are **4 types **of Kubernetes services: **ClusterIP, NodePort, LoadBalancer and ExternalName**. The “type” property in the Service's specification determines how the service is exposed to the network.

**🔹 ClusterIP**
ClusterIP is the default and most common service type. Kubernetes will assign a cluster-internal IP address to ClusterIP service. This makes the service only reachable within the cluster.

**🔹 NodePort**
This exposes the service outside of the cluster by adding a cluster-wide port on top of ClusterIP. We can request the service by NodeIP:NodePort.

**🔹 LoadBalancer**
This exposes the Service externally using a cloud provider’s load balancer.

**🔹 ExternalName**
This maps a Service to a domain name. This is commonly used to create a service within Kubernetes to represent an external database.


