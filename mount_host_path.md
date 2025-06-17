# Mounting a host path (a directory on the worker node) inside a Kubernetes pod is done using a hostPath volume.

⚠️ Important:
This approach is not portable across nodes.

Use it only for scenarios like logs, tools, or temporary access to the host’s filesystem.

🧱 **Use Cases**
Access node-specific logs

Access Docker or containerd socket: /var/run/docker.sock

GPU drivers (e.g., /usr/local/nvidia)

Dev/debug/testing scenarios


Let’s say you want to mount /data/logs from the AKS node into a pod at /mnt/logs.


```bash

apiVersion: v1
kind: Pod
metadata:
  name: hostpath-demo
spec:
  containers:
  - name: app
    image: ubuntu
    command: [ "sleep", "3600" ]
    volumeMounts:
    - name: host-logs
      mountPath: /mnt/logs
  volumes:
  - name: host-logs
    hostPath:
      path: /data/logs  # This is the path on the AKS node
      type: DirectoryOrCreate

```

| Field           | Description                                                                                |
| --------------- | ------------------------------------------------------------------------------------------ |
| `hostPath.path` | The actual path on the AKS worker node                                                     |
| `mountPath`     | Where the path will appear inside the container                                            |
| `type`          | Creates directory if it doesn't exist. Use `Directory`, `File`, etc., based on requirement |


⚠️ **Warnings**
Not ideal for production workloads.

If the pod moves to another node, the volume content changes or might not exist.


**Alternantive - Best approach:**

 ---> `Use DaemonSet + hostPath`

If you need this on all AKS nodes, deploy it using a DaemonSet with hostPath, so one pod runs per node and accesses the local path.

### Daemon Set:

'A DaemonSet ensures that exactly one pod runs on every node (or selected nodes) in the Kubernetes cluster'

**Key Behaviors**:
When a new node joins the cluster ➜ A pod is automatically scheduled on it.

When a node is removed ➜ The corresponding pod is deleted.

You can also restrict it to certain nodes using nodeSelector, affinity, or tolerations.

💡 Use Case: Access Node Local Path
When you combine DaemonSet + hostPath volume, you can:

✅ Run one pod per node
✅ Mount a local directory from that node into the pod
✅ Access node-local logs, binaries, drivers, etc.

### 🧪 Example: DaemonSet That Mounts /data/logs From Each Node
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: log-agent
  namespace: kube-system
spec:
  selector:
    matchLabels:
      name: log-agent
  template:
    metadata:
      labels:
        name: log-agent
    spec:
      containers:
      - name: agent
        image: ubuntu
        command: ["/bin/sh", "-c", "while true; do ls /mnt/logs; sleep 30; done"]
        volumeMounts:
        - name: host-logs
          mountPath: /mnt/logs
      volumes:
      - name: host-logs
        hostPath:
          path: /data/logs   # Path on each AKS node
          type: DirectoryOrCreate
      tolerations:
      - operator: "Exists"   # To allow running on all nodes, including tainted ones

| Feature            | DaemonSet + hostPath               |
| ------------------ | ---------------------------------- |
| One pod per node   | ✅ Yes                              |
| Access local path  | ✅ Yes                              |
| Works on new nodes | ✅ Pods are automatically scheduled |




