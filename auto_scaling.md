# AutoScaling in Kubernetes

## Scaling Types:



 ###  Horizontal AutoScaling:

 - PODS (workloads)    HPA ( Horizontal POD Autoscaling) \ 
 - Infra (nodes) -Cluster AutoScaler


### Vertical POD Autoscaling):
 - PODS (workloads)  ( Vertical POD Autoscaling):   \
 - Infra (nodes) -NodeAutoProvisioning 



==========================================
Event Based AutoScaling :  KEDA        \
Cron/Schedule Based AutoScaling
============================================


![image](https://github.com/user-attachments/assets/d5004c3c-2c01-42d7-bfc5-b742778188a8)


![image](https://github.com/user-attachments/assets/3c916663-c7fc-4917-946d-5ed9574b5371)





# HPA ( Horizontal POD Autoscaling)


![image](https://github.com/user-attachments/assets/59ca0208-6a4f-4af3-9fa8-0f2510a920bc)



HPA POD --- > Build in kubernetes - collect metrics from metric server ( cpu/memory)
VPA POD - Build in kubernetes - collect metrics from metric server ( cpu/memory)

 - AutoScaler and NodeAutoProvisioning  both are related to CloudProvider Managed Kubernetes Cluster AKS,EKS,GKE 



:EMOJICODE📓

**Reference**: https://thinksys.com/devops/kubernetes-autoscaling/
