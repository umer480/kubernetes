# CronJob in kubernetes

If you’ve worked with scheduled tasks or cron jobs in linux-based systems, you’ll find that CronJobs in Kubernetes provide similar functionality.

`Kubernetes CronJobs are resources used for scheduling recurring tasks based on a cron expression. it automate repetitive tasks.`


## What is a CronJob in Kubernetes?
In simple terms, a `CronJob` is a `Kubernetes resource` used to run tasks on a scheduled basis, much like a cron job in traditional Linux environments.
Instead of running tasks at fixed intervals manually, Kubernetes CronJobs allow you to automate tasks such as backups, report generation, or database maintenance within your cluster.


## Use Cases for CronJobs

**Periodic Maintenance**: Automate routine maintenance tasks such as log rotation, database cleanup, or temporary file deletion to optimize system performance and resource utilization.

**Database Backup**s: Schedule periodic backups of databases to ensure data is not lost.

**Batch Processing**: Run data processing tasks such as generating reports or cleaning up old data.

**Scheduled Deployments**: Automate deployment tasks like clearing caches or updating resources at regular intervals.





## How Does It Work?
CronJobs in Kubernetes run as` Pods based` on a defined schedule. The schedule follows a format similar to the cron expression used in Unix systems:

**Minute**: 0–59  \
**Hour**: 0–23  \
**Day of Month**: 1–31  \
**Month**: 1–12  \
**Day of Week**: 0–6 (0=Sunday)  \
**For example**, a cron expression `0 0 * * *` will run a task at midnight every day.

## Creating a CronJob in Kubernetes

```bash
apiVersion: batch/v1
kind: CronJob
metadata:
  name: example-cronjob
spec:
  schedule: "0 * * * *"  # Run every hour
  jobTemplate:   # Contains the Job specification that defines the pod template and the task to run.
    spec:
      template:
        spec:
          containers:
            - name: hello
              image: busybox
              command:
                - "echo"
                - "Hello, Kubernetes CronJob!"
          restartPolicy: OnFailure
```

**In this example:**

The **CronJob** is scheduled to run every hour (0 * * * *).   \
The **jobTemplate** specifies the container (hello) that runs the echo command every time the CronJob is triggered.\
The **restartPolicy** is set to OnFailure, meaning the container will only restart if it fails.\

**Pod Restart Policy**: If a pod fails, the Job controller can create new pods to replace the failed ones until the task is completed.


## Run Cron Job after every 1 min
```bash
apiVersion: batch/v1
kind: CronJob
metadata:
  name: basic-cronjob
spec:
  schedule: "*/1 * * * *"  # Run every 1 minutes
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: basic-container
            image: busybox
            command: ["echo", "Hello from the basic CronJob"]
          restartPolicy: OnFailure
```

Note: 'Cronjob at every run, new job will be created.'

A CronJob creates Jobs on a schedule.

![image](https://github.com/user-attachments/assets/04d25406-766f-4ec8-9ee4-957cc7b19b8c)

## Commands
```bash
kubectl get cronjobs
kubectl get pod,job,cronjob
kubectl logs <job-name>
kubectl suspend cronjob <cronjob-name>   # You can pause a CronJob from creating new jobs
```



## Time Zone Support:
For CronJobs with no time zone specified, the kube-controller-manager interprets schedules relative to its local time zone.

Set Cutom TimeZone:
The following CronJob definition will run every day at `23:00 in Istanbul, Turkey` , regardless of the time configuration of the Kubernetes cluster.

```bash
apiVersion: batch/v1
kind: CronJob
metadata:
  name: timezone-test
spec:
  timeZone: 'Asia/Istanbul' #You can change this line with any timezone
  schedule: "0 23 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: timezone-test
            image: busybox:1.28
            imagePullPolicy: IfNotPresent
            command:
            - /bin/sh
            - -c
            - date; echo Selam!
          restartPolicy: OnFailure
```

You can find the list of time zones you can use from the link below.

**Valid TZ’s**: 

```bash
https://en.wikipedia.org/wiki/List_of_tz_database_time_zones
```




## Concurrency Policy
You can control how CronJobs handle multiple concurrent executions.

**Allow (default)**: CronJob allows multiple jobs to run at the same time.

**Forbid**: Ensures that only one job runs at a time. The CronJob does not allow concurrent runs; if it is time for a new Job run and the previous Job run hasn't finished yet, the CronJob skips the new Job run.

**Replace**: Replaces the currently running job with a new one if the CronJob triggers while another is still running. (If it is time for a new Job run and the previous Job run hasn't finished yet, the CronJob replaces the currently running Job run with a new Job run)


**Example**:
```bash
apiVersion: batch/v1
kind: CronJob
metadata:
  name: concurrency-cronjob
spec:
  schedule: "*/1 * * * *"  # Run every 1 minutes
  concurrencyPolicy: Forbid  # Do not allow concurrent executions
  # Allowed Values are
  # : Allow (default)
  # : Forbid
  # : Replace
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: concurrency-container
            image: busybox
            command: ["echo", "Hello from the concurrency CronJob"]
          restartPolicy: Never
```




## Job History Limit
CronJobs create jobs when triggered, and Kubernetes keeps track of the last few completed jobs. You can limit how many jobs are retained using the `successfulJobsHistoryLimit` and `failedJobsHistoryLimit` options.


**Example**:


```bash
apiVersion: batch/v1
kind: CronJob
metadata:
  name: history-limit-cronjob
spec:
  schedule: "*/1 * * * *"  # Run every 1 minutes
  successfulJobsHistoryLimit: 2  # Retain up to 2 successful Job completions
  failedJobsHistoryLimit: 1  # Retain only the latest failed Job completion
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: history-limit-container
            image: busybox
            command: ["echo", "Hello from the history-limit CronJob"]
          restartPolicy: Never
```


This CronJob configuration schedules a job to run every minute. It retains the history of successful job completions for up to 2 instances and the history of failed job completions for only the latest instance.


```bash

$ kubectl get pod,job,cronjob
NAME                                       READY   STATUS      RESTARTS   AGE
pod/history-limit-cronjob-28558295-qtdz6   0/1     Completed   0          71s
pod/history-limit-cronjob-28558296-774lw   0/1     Completed   0          11s

NAME                                       COMPLETIONS   DURATION   AGE
job.batch/history-limit-cronjob-28558295   1/1           4s         71s
job.batch/history-limit-cronjob-28558296   1/1           4s         11s

NAME                                  SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
cronjob.batch/history-limit-cronjob   */1 * * * *   False     0        11s             3m9s

```



## backoffLimit

Limits the number of retries before considering the Job as failed.

🔍 **Details**:
- A CronJob creates Jobs on a schedule.

- Each Job may fail and be retried.

- backoffLimit is the number of retries allowed for a Job before it's marked as failed.

- Once this limit is hit, no more retries are attempted, and the Job status is set to Failed.


**In this example**, if the Job fails, Kubernetes will retry it up to 3 more times. If all fail, the Job is marked as Failed.

```bash
apiVersion: batch/v1
kind: CronJob
metadata:
  name: example-cronjob
spec:
  schedule: "*/5 * * * *"
  jobTemplate:
    spec:
      backoffLimit: 3   # Retry up to 3 times on failure
      template:
        spec:
          containers:
          - name: example
            image: busybox
            args: ["sh", "-c", "exit 1"]  # Simulate failure
          restartPolicy: Never

```


## 🔁 CronJob vs 🧭 Job in Kubernetes

| Feature         | **Job**                              | **CronJob**                                    |
| --------------- | ------------------------------------ | ---------------------------------------------- |
| **Purpose**     | Run a one-time task                  | Run a task on a schedule (recurring)           |
| **Trigger**     | Manual (imperative or manifest)      | Automatic, based on a cron schedule            |
| **Scheduling**  | No built-in schedule                 | Uses cron syntax (`*/5 * * * *`)               |
| **Creates**     | Pods                                 | Jobs (which in turn create Pods)               |
| **Use Case**    | One-off batch jobs (e.g., backup DB) | Recurring jobs (e.g., daily report generation) |



# Real Example - : Automating Database Backup

To illustrate the use of Jobs and CronJobs in a real-life scenario, let’s consider automating the backup of a MongoDB database running in a Kubernetes cluster.

## Deploy MongoDB in Kubernetes:

### Step1:

We need a MongoDB instance running in our cluster. We can use the following YAML file to deploy MongoDB using a StatefulSet and a Service.



```bash

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongodb-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mongodb
spec:
  serviceName: "mongodb"
  replicas: 1
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
      - name: mongodb
        image: mongo:latest
        ports:
        - containerPort: 27017
        volumeMounts:
        - name: mongodb-data
          mountPath: /data/db
  volumeClaimTemplates:
  - metadata:
      name: mongodb-data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
---
apiVersion: v1
kind: Service
metadata:
  name: mongodb
spec:
  ports:
  - port: 27017
    targetPort: 27017
  selector:
    app: MongoDB
```

### Step2:

Next, we’ll create a Kubernetes Job that runs a backup script. This script will connect to the MongoDB instance and perform a backup, storing the backup in a persistent volume.


```bash
apiVersion: batch/v1
kind: Job
metadata:
  name: mongodb-backup-job
spec:
  template:
    spec:
      containers:
      - name: backup
        image: mongo:latest
        command: ["sh", "-c", "mongodump --uri=mongodb://mongodb:27017/admin --out=/backup && echo Backup completed!"]
        volumeMounts:
        - name: backup-storage
          mountPath: /backup
      restartPolicy: OnFailure
      volumes:
      - name: backup-storage
        persistentVolumeClaim:
          claimName: backup-pvc
  backoffLimit: 4
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: backup-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi

```
Check the status of the Job and the logs to ensure the backup was completed successfully:

```bash
kubectl get jobs
kubectl get pods
kubectl logs <pod-name>
```

### Step3:

Create a Kubernetes CronJob for Scheduled Backups.

To automate daily backups, we will create a CronJob that schedules the backup Job to run every day at midnight.

```bash
apiVersion: batch/v1
kind: CronJob
metadata:
  name: mongodb-backup-cronjob
spec:
  schedule: "0 0 * * *"  # Runs at midnight every day       ##### "*/2 * * * *"  # Run every 2 minutes
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: mongo:latest
            command: ["sh", "-c", "mongodump --uri=mongodb://mongodb:27017/admin --out=/backup && echo Backup completed!"]
            volumeMounts:
            - name: backup-storage
              mountPath: /backup
          restartPolicy: OnFailure
          volumes:
          - name: backup-storage
            persistentVolumeClaim:
              claimName: backup-pvc
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
```

Monitor the CronJob to ensure it is creating Jobs as scheduled:

```bash
kubectl get cronjobs
kubectl get jobs
kubectl get pods
```

## Backup Validaiton by inspecting persistent volume: (Optional)

Create a pod and mount volume with it so you can inspect it;

```bash
apiVersion: v1
kind: Pod
metadata:
  name: inspect-backup-pvc
spec:
  containers:
  - name: shell
    image: ubuntu
    command: [ "sleep", "3600" ]
    volumeMounts:
    - mountPath: /backup
      name: backup-volume
  volumes:
  - name: backup-volume
    persistentVolumeClaim:
      claimName: backup-pvc
  restartPolicy: Never

```
### List the content of the mounted volume:

```bash

kubectl exec -it <pod name> -- ls -l /backup
```


### 🔍 **How to Check Backup Files on AKS**:



**🔧 Step 1: Determine the Backing Storage Type:**

### Setup Recap:

**PVC name**: backup-pvc

**Mounted at**: /backup

**Storage**: Likely backed by Azure Disk or Azure Files, depending on your StorageClass.


**run**;

```bash
kubectl get pvc backup-pvc -o jsonpath="{.spec.storageClassName}"
```
**then**;

```bash
kubectl get sc <your-storage-class-name> -o yaml | grep provisioner

```

**You’ll see something like**:

disk.csi.azure.com → Azure Disk

file.csi.azure.com → Azure Files


