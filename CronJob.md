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
          restartPolicy: Never
```

Note: 'Cronjob at every run, new job will be created.'


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

```bash
apiVersion: batch/v1
kind: CronJob
metadata:
  name: db-backup-cronjob
spec:
  schedule: "0 0 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: db-backup
            image: busybox
            command: ["sh", "-c", "echo Backing up database...; sleep 10; echo Database backed up!"]
          restartPolicy: OnFailure
  backoffLimit: 3
```

# Real Example - Taking up database backup:

CronJob vs Job

https://medium.com/@ravipatel.it/understanding-kubernetes-jobs-and-cronjobs-with-example-772f416b7e69
