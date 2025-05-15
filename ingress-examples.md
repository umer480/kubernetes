# ingress controller:

- Why we need ingress
- benefits of using ingress over nodeport
- implementing ingress rules
- implementing ingress controller to process ingress rules
- understanding yml files for ingress


Use a Single IP Address to access your all different kubernetes services that are resides inside cluster -reduce the  provisioning of  additional external public ip address for each service to manage additional cost on cloud.


When you want to access service over hostname (instead of IP Address) and with HTTPS the use ingress. Which forwards the request to service based on hostname/path.


![image](https://github.com/user-attachments/assets/26c3815d-0da7-4034-9b2d-a69004c27b2c)



![image](https://github.com/user-attachments/assets/1474182a-e412-41bf-b10a-df609067cef6)


![image](https://github.com/user-attachments/assets/616df5f4-f3ac-4fb2-82c8-92645515e339)


![image](https://github.com/user-attachments/assets/dd8384d7-dba9-4f80-b1dd-9a6aa5024eca)


**Basic Auth:**
![image](https://github.com/user-attachments/assets/9e5dfb17-6bd2-4dd6-958f-4086dc9aa42c)


# External Service Vs ingress:
Create External Service (nodeport) and access it using node’s ip address.


![image](https://github.com/user-attachments/assets/abf72247-a2d4-45ce-a776-05fe58aab337)

But best approach is to access uing https and hostname (means hide actual pod where service is running)
Create  internal service ( instead of external)

**Traffic Flow:**
User/browser -->  my-all.com --> ingress  internal service --> pod

![image](https://github.com/user-attachments/assets/e65a9058-048a-499e-b67b-3fea33d417b2)



![image](https://github.com/user-attachments/assets/a7a003d1-2d1f-4da2-96a7-efcd91383d9b)

**Example File for External Service:**

![image](https://github.com/user-attachments/assets/b1e317d3-f229-4147-9077-bed984d9b8e0)

Example YMAL File for ingress:
![image](https://github.com/user-attachments/assets/23bb731f-f112-4126-b441-59d9fe2b7433)

![image](https://github.com/user-attachments/assets/f9684969-7b25-42c1-a7ea-ae2e6c811f71)


**Inter conecntion between ingress and internal service:**

![image](https://github.com/user-attachments/assets/2464b052-e1bf-4f9d-b3dc-96f234e5ec66)

In ingress yml  service name should be exact that you define in internal service yml

Service port shoud be same in ingress yml that you define exactly in internal service yml


![image](https://github.com/user-attachments/assets/34da46df-2f5f-4081-844a-233017ced8dd)

**DNS name (my-app.com) should point to the ip address of node;**

![image](https://github.com/user-attachments/assets/c268abcf-ec10-4e97-807e-f90088303c34)

If you define the server outside kubernets cluster then domain name/hostname should point to it



![image](https://github.com/user-attachments/assets/75b9dda8-c37b-46ed-8c7f-cdaf3d2468be)



### Ingress controller:
My-app ingress is just yaml ingress compunent but to make it wortk you need ingress controller – which is another pod/set of pod runs in your kubernetes cluster

![image](https://github.com/user-attachments/assets/c4cd0526-aa1a-414c-931d-0b9a23fb4733)


![image](https://github.com/user-attachments/assets/45ff6686-7525-41ed-a8e4-95ed2cd4ef42)


**What is “ingress controller”:**
It evalue all the rules of ingress, and route traffic accordingly – act as a entrypoint for all ingress objects
You may have 50 ingress rules (ingress yml component) and it evaluate all the rules and decide which rule to follow based on request.

![image](https://github.com/user-attachments/assets/1ae66946-f3a5-49e6-a2fe-1a2ca084edf7)

**Ingress controller Options:**
-	K8S Nginx ingress Controller  - Builtin controller

-	![image](https://github.com/user-attachments/assets/408eb448-121b-423d-afa2-7c9e90848a19)

**-	Ingress Controller Options:**
https://kubernetes.github.io/ingress-nginx/deploy/baremetal/


**Complete Traffic Flow:**
LoadBalancer (AWA/AZURE provided – outside K8s Cluster) -- > ingress controller Nginx deployed inside k8s cluster > ingress componentInternal Service Pod

# **SSL/TLS configuration with ingress Constroller:**
![image](https://github.com/user-attachments/assets/0e1885b5-a4d3-4510-b494-4d61cefbb6e6)


![image](https://github.com/user-attachments/assets/377ed26b-93ad-4824-b289-725fcf4ff9e4)




![image](https://github.com/user-attachments/assets/cc5a5159-5193-46c2-b9fe-16edf83f7b65)


























