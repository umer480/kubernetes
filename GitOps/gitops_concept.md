# GitOps

<img width="451" height="162" alt="image" src="https://github.com/user-attachments/assets/279a6273-f435-4d52-a8c0-3df6b789c8c4" />


🔹 **What is GitOps?**

**GitOps** = Managing infrastructure and applications using Git as the single source of truth.

Instead of manually running kubectl apply or editing configs directly in Kubernetes,
👉 you put all your YAML/Helm charts/Terraform/etc. in **Git**.


**A tool like Argo CD or Flux then:**

- Watches the Git repo

- Applies changes automatically to your Kubernetes cluster

- Ensures your cluster state always matches what’s in Git

  <img width="859" height="287" alt="image" src="https://github.com/user-attachments/assets/896a5c33-ff93-432b-848f-0c6203f8a8b6" />



**Without GitOps:**

- Dev updates deployment YAML on laptop

- Runs kubectl apply -f deployment.yaml

 - Deployment updated, but no one else knows exactly what changed

**⚠️ Risk → No audit trail, manual errors, inconsistency**


<img width="708" height="498" alt="image" src="https://github.com/user-attachments/assets/9751d62a-f70b-47c7-9b65-5e788bd7753d" />



**🔹 Traditional CI/CD vs GitOps**

| Feature                    | Traditional CI/CD                                                                                              | GitOps                                                                 |
| -------------------------- | -------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| **Source of Truth**        | CI/CD pipeline config + cluster itself                                                                         | **Git repo (only Git is truth)**                                       |
| **Who pushes changes?**    | Pipeline/tool pushes changes into the cluster (e.g., `kubectl apply` in Azure DevOps, GitHub Actions, Jenkins) | GitOps tool (Argo CD/Flux) **pulls** changes from Git into the cluster |
| **Deployment Trigger**     | After build, pipeline deploys (push model)                                                                     | Change in Git repo triggers reconciliation (pull model)                |
| **Cluster Drift Handling** | If someone edits the cluster manually, the pipeline doesn’t know → drift stays                                 | GitOps detects drift → can auto-correct to match Git                   |
| **Audit Trail**            | Partial (some logs in CI/CD + manual edits are hidden)                                                         | Full → every change is a Git commit                                    |
| **Rollback**               | Needs pipeline rollback or manual reapply                                                                      | Just revert Git commit → Argo CD syncs                                 |
| **Environments**           | Pipelines often maintain separate configs for dev/stage/prod                                                   | Environments are just different Git branches or folders                |
| **Security**               | Pipeline needs direct credentials to the cluster (risk if leaked)                                              | GitOps tools run inside cluster → no external pipeline creds needed    |


<img width="792" height="483" alt="image" src="https://github.com/user-attachments/assets/fc9eb92d-6958-4fc8-bea1-4d9ca9adc527" />


**✅ Summary :**
“Traditional CI/CD pipelines push changes into Kubernetes, but they don’t guarantee the cluster matches the desired state later. GitOps, on the other hand, uses Git as the single source of truth and continuously reconciles the cluster to match what’s in Git. This makes deployments more reliable, auditable, and self-healing.”

