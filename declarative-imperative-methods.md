### How to Deploy in Kubernetes: The Declarative and Imperative Approaches
### Kubernetes: Choosing Between Declarative and Imperative Command Styles
### Managing Kubernetes Resources: Declarative vs Imperative Approach



## 
1- **Declarative** = "Here’s the desired state. You figure it out."    ---> direct Command instruction

2- **Imperative** = "Here’s the exact command. **Do it now.**"         --->  (instructions via configuration/manifest/yml file







Imperative (create): Only runs the command once for creation, and fails if the resource already exists.
Declarative (apply): Can be run multiple times and will ensure the resource matches the desired state (creates or updates).



## Example - to understand different between both approaches

# **Running the Command Twice:**
**When you run the imperative command:**

```bash
kubectl create deployment my-app --image=nginx --replicas=3
```
It will create a deployment if one does not already exist. If you run it again (without any changes), Kubernetes will respond with an error like this:

<span style="color:red;">error: deployment.apps "my-app" already exists</span>

This is because the create command is meant to create new resources. <span style="color:red;">**It doesn't update or manage resources that already exist.**</span>

**Declarative Approach (What Would Happen There):**
Kubernetes would simply check the current state of the deployment and:

- Create it if it doesn't exist
- Update it if there are any changes in the configuration.


```bash
kubectl apply -f deployment.yaml
```


**So** with declarative, you can run the **command multiple times **and Kubernetes will ensure that the **desired state** is always maintained (it will either create or update as needed).


## **Declarative approach benefits:**
**Reproducible deployments:** Easily recreate the same environment from the YAML configuration files.
**Version control:** Manage changes to Kubernetes deployments in a version-controlled manner.

