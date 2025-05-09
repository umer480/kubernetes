
# How to get endpoints of a service: 

**Useful commands for troubleshooting in case of routing/traffic flow issues:
**

```bash
kubectl get ep
kubectl get endpoints <service-name> -o yaml    To view all endpoints of a Service in the default namespace:
kubectl get endpoints <service-name> -o jsonpath='{.subsets[*].addresses[*].ip}'    -->If you want it in a more readable format: 
```


```bash
kubectl describe endpoints <service-name>
```



![image](https://github.com/user-attachments/assets/0655e5e8-30a1-4909-bd2a-3a2fdf8ecfb2)


![image](https://github.com/user-attachments/assets/f5f27de6-58a4-4162-86f4-6e4ba939fb91)
