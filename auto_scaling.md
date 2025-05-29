# AutoScaling in Kubernetes

## Scaling Types:



 ###  Horizontal AutoScaling:

 - PODS (workloads)  -  HPA -  Horizontal POD Autoscaling
 - Infra (nodes) - Cluster AutoScaler


### Vertical POD Autoscaling):
 - PODS (workloads) VPA  -  Vertical POD Autoscaling   
 - Infra (nodes) -NodeAutoProvisioning 



### Other Ways of AutoScaling:
- Event Based AutoScaling :  KEDA        
- Cron/Schedule Based AutoScaling



![image](https://github.com/user-attachments/assets/d5004c3c-2c01-42d7-bfc5-b742778188a8)


### HPA vs VPA

![image](https://github.com/user-attachments/assets/36e32c79-cced-48fb-89c9-4455fcef420d)




# HPA ( Horizontal POD Autoscaling)

Horizontal Pod Autoscaler deploys additional pods in the Kubernetes cluster automatically when the load increases. It modifies the entire workload resource and scales it automatically as per requirement. Whether scaling up or scaling down the pods, HPA can do it automatically to meet the load.

![image](https://github.com/user-attachments/assets/59ca0208-6a4f-4af3-9fa8-0f2510a920bc)





### The Working of HPA:

HPA follows a systematic approach while modifying the pods. It understands whether or not the pod replicas need to be increased or decreased by taking the mean of a per-pod metric value. Afterward, it analyzes whether raising or reducing the pod replicas will bring the mean value near to the desired number. This autoscaler type is managed by the controller manager, and it runs as a control loop.

For instance, if five pods are performing currently, your target utilization is fifty percent, and the current usage is nearly seventy-five percent. In that case, the HPA controller will add three pod replicas in the cluster to bring the mean number near the fifty percent target.



  `AutoScaler and NodeAutoProvisioning  both are related to CloudProvider Managed Kubernetes Cluster AKS,EKS,GKE`


 ### Pre-Requisites for HPA:

HPA needs 'Metric Server' - its mandatory.  it Collects CPU/memory usage from Kubelets
HPA monitor workload after every 15 sec ( by default) 


kubelet endpoint from where metric server collect data : /metrics/resource


# setting up metric-server:

Before testing Horizontal Pod Autoscaler (HPA) and Vertical Pod Autoscaler (VPA), it’s essential to have the Metrics Server installed in your Kubernetes cluster. The Metrics Server collects resource usage metrics from the cluster’s nodes and pods, which are necessary for autoscaling decisions.


'Download the Components Manifest: Use kubectl apply to download and apply the Components manifest directly from the latest release of the Metrics Server:'

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```


**Verify Installation:** After applying the manifest, verify that the Metrics Server pods are running successfully. You can check the pods in the kube-system namespace:

```bash
kubectl get pods -n kube-system | grep metrics-server
```

**Confirm Metrics Collection**: Once the Metrics Server is up and running, you can confirm that it’s collecting metrics by querying the API. For example, you can retrieve the CPU and memory usage metrics for nodes and pods:

```bash
kubectl top nodes kubectl top pods --all-namespaces
```

If the Metrics Server is properly installed and functioning, you should see CPU and memory usage metrics for nodes and pods in your cluster.

With the Metrics Server installed and collecting metrics, you can proceed to test the Horizontal Pod Autoscaler (HPA) and Vertical Pod Autoscaler (VPA) functionalities in your Kubernetes cluster.

### Commmands: Metric Server 

```bsah
kubectl top node
kubectl top pod

```

# # LAB :


```bash

apiVersion: apps/v1
kind: Deployment
metadata:
  name: php-apache
spec:
  selector:
    matchLabels:
      run: php-apache
  template:
    metadata:
      labels:
        run: php-apache
    spec:
      containers:
      - name: php-apache
        image: registry.k8s.io/hpa-example
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: 500m
          requests:
            cpu: 200m
---
apiVersion: v1
kind: Service
metadata:
  name: php-apache
  labels:
    run: php-apache
spec:
  ports:
  - port: 80
  selector:
    run: php-apache

```

HPA :

```bash
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: php-apache
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: php-apache
  minReplicas: 1
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```




# Testing : Load Testing

```bash
kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"
```



```


### Commmands:

```bash
kubectl autoscale deploy <deployment name> --cpu-percentage=50 --min=1 --max=10
kubectl get hpa
kubectl delete hpa <hpa name>
```





# VPA ( Vertical Pod AutoScaling)

![image](https://github.com/user-attachments/assets/e765b14d-8cb6-4b3a-bfa2-afe50e0d47d2)


![image](https://github.com/user-attachments/assets/9c9bcee9-c9bc-455b-a329-2ffaeb476e61)



**installing VPA :**
https://docs.aws.amazon.com/eks/latest/userguide/vertical-pod-autoscaler.html


:EMOJICODE📓

**Reference**: https://thinksys.com/devops/kubernetes-autoscaling/
https://medium.com/@muppedaanvesh/a-hands-on-guide-to-kubernetes-horizontal-vertical-pod-autoscalers-%EF%B8%8F-58903382ef71

