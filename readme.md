# Kubernetes Learning Hub 🚀

A comprehensive, hands-on collection of Kubernetes topics, practical examples, configuration files, deployment manifests, troubleshooting guides, and best practices for cloud engineers, DevOps engineers, and platform engineers.

This repository is designed to help you learn Kubernetes from fundamentals to advanced concepts through practical implementations.

## 📚 Table of Contents

- [Introduction to Kubernetes](#-introduction-to-kubernetes)
- [Kubernetes Architecture](#-kubernetes-architecture)
- [Core Kubernetes Objects](#-core-kubernetes-objects)
- [Workloads and Deployments](#-workloads-and-deployments)
- [Networking](#-networking)
- [Storage](#-storage)
- [Configuration and Secrets](#-configuration-and-secrets)
- [Security and RBAC](#-security-and-rbac)
- [Helm and Package Management](#-helm-and-package-management)
- [Autoscaling and Resource Management](#-autoscaling-and-resource-management)
- [Monitoring and Logging](#-monitoring-and-logging)
- [CI/CD and GitOps](#-cicd-and-gitops)
- [Cloud Kubernetes Services](#-cloud-kubernetes-services)
- [Troubleshooting](#-troubleshooting)
- [Best Practices](#-best-practices)
- [Repository Structure](#-repository-structure)

## ☸️ Introduction to Kubernetes

Learn the fundamentals of Kubernetes, an open-source container orchestration platform used to deploy, scale, manage, and operate containerized applications.

Topics include:

- What is Kubernetes and why it is used
- Containers vs. virtual machines
- Kubernetes vs. Docker Swarm
- Kubernetes distributions and managed services
- Installing Kubernetes locally using Minikube or Kind
- Setting up a cluster using kubeadm
- Introduction to `kubectl`
- Kubernetes YAML manifests

## 🏗️ Kubernetes Architecture

Understand Kubernetes components and how they work together.

- Control plane architecture
- API Server
- etcd
- Scheduler
- Controller Manager
- Cloud Controller Manager
- Worker nodes
- kubelet
- kube-proxy
- Container Runtime Interface (CRI)
- Container Network Interface (CNI)
- Cluster communication and reconciliation loops

## 📦 Core Kubernetes Objects

Practical examples and YAML manifests for essential Kubernetes resources.

- Pods
- ReplicaSets
- Deployments
- StatefulSets
- DaemonSets
- Jobs and CronJobs
- Namespaces
- Labels and selectors
- Annotations
- Services
- Endpoints and EndpointSlices

## 🚀 Workloads and Deployments

Learn how to deploy and manage applications reliably.

- Creating and managing deployments
- Scaling applications manually
- Rolling updates and rollbacks
- Blue-green deployments
- Canary deployments
- Init containers
- Sidecar containers
- Multi-container Pods
- Liveness, readiness, and startup probes
- Pod affinity and anti-affinity
- Node selectors and node affinity
- Taints and tolerations
- Pod disruption budgets

## 🌐 Networking

Explore Kubernetes networking from basic service discovery to advanced traffic routing.

- Kubernetes networking fundamentals
- ClusterIP, NodePort, and LoadBalancer
- Headless Services
- CoreDNS and service discovery
- Ingress and Ingress Controllers
- Gateway API
- Network Policies
- CNI plugins: Cilium, Calico, and others
- Internal and external traffic routing
- Service-to-service communication
- DNS troubleshooting
- Private cluster networking

## 💾 Storage

Learn how Kubernetes manages persistent application data.

- Volumes and volume types
- emptyDir and hostPath
- PersistentVolumes (PV)
- PersistentVolumeClaims (PVC)
- StorageClasses
- Dynamic volume provisioning
- StatefulSet storage
- CSI drivers
- Volume expansion
- Storage snapshots and backup strategies
- Cloud storage integration with Azure and AWS

## ⚙️ Configuration and Secrets

Manage application configuration securely and consistently.

- ConfigMaps
- Secrets
- Environment variables
- Configuration mounted as files
- Secret encryption at rest
- External Secrets Operator
- Azure Key Vault integration
- AWS Secrets Manager integration
- Workload identity and managed identities
- Configuration management across environments

## 🔐 Security and RBAC

Apply Kubernetes security best practices to production environments.

- Role-Based Access Control (RBAC)
- Roles and ClusterRoles
- RoleBindings and ClusterRoleBindings
- ServiceAccounts
- Pod Security Standards
- Security contexts
- Least-privilege access
- Network Policies
- Image scanning and signing
- Admission controllers
- Pod Security Admission
- OPA Gatekeeper and Kyverno
- Secrets management
- Container runtime security
- Kubernetes audit logging
- Supply chain security

## 📦 Helm and Package Management

Manage reusable application deployments with Helm.

- Helm architecture and installation
- Helm charts and templates
- `values.yaml` configuration
- Creating custom Helm charts
- Chart dependencies
- Environment-specific values
- Helm upgrades and rollbacks
- Chart repositories and OCI registries
- Helm release management
- Chart validation and testing

## 📈 Autoscaling and Resource Management

Optimize cluster performance, reliability, and infrastructure costs.

- CPU and memory requests and limits
- ResourceQuota and LimitRange
- Horizontal Pod Autoscaler (HPA)
- Vertical Pod Autoscaler (VPA)
- Cluster Autoscaler
- Kubernetes-native node autoscaling
- Metrics Server
- Pod scheduling and resource allocation
- QoS classes
- Capacity planning
- Cost optimization and resource rightsizing

## 📊 Monitoring and Logging

Build observability for Kubernetes workloads and infrastructure.

- Prometheus
- Grafana dashboards
- Alertmanager
- kube-state-metrics
- Node Exporter
- Container metrics using cAdvisor
- Centralized application logging
- Loki and Fluent Bit
- OpenTelemetry
- Distributed tracing
- Kubernetes events and audit logs
- Alerting for Pod failures, node health, and resource saturation

## 🔄 CI/CD and GitOps

Automate application delivery and cluster configuration management.

- GitHub Actions
- Azure DevOps Pipelines
- Docker image build and push
- Azure Container Registry (ACR)
- Amazon Elastic Container Registry (ECR)
- Kubernetes deployment automation
- Helm-based CI/CD
- GitOps principles
- Argo CD
- Flux CD
- Image promotion across environments
- Deployment approvals and rollback strategies
- Secure pipeline authentication using workload identity federation

## ☁️ Cloud Kubernetes Services

Explore managed Kubernetes platforms and their cloud integrations.

### Microsoft Azure

- Azure Kubernetes Service (AKS)
- Azure CNI and Azure CNI Overlay
- AKS networking and private clusters
- Azure Application Gateway and ingress integration
- Azure Key Vault and workload identity
- Azure Container Registry integration
- Azure Monitor and Managed Prometheus
- AKS autoscaling and cost optimization

### Amazon Web Services

- Amazon Elastic Kubernetes Service (EKS)
- EKS networking and VPC integration
- AWS Load Balancer Controller
- IAM Roles for Service Accounts (IRSA)
- EKS Pod Identity
- AWS Secrets Manager integration
- Amazon ECR integration
- CloudWatch Container Insights
- EKS autoscaling and cost optimization

## 🛠️ Troubleshooting

Common Kubernetes troubleshooting scenarios and their solutions.

- Pod stuck in Pending
- CrashLoopBackOff
- ImagePullBackOff and ErrImagePull
- OOMKilled containers
- Failed scheduling
- Service connectivity issues
- DNS resolution failures
- Ingress and load balancer issues
- PVC stuck in Pending
- Node NotReady
- Certificate and TLS issues
- RBAC permission errors
- High CPU and memory usage
- Cluster capacity and autoscaling problems
- Application logs and event analysis

Useful commands:

```bash
# View cluster nodes
kubectl get nodes -o wide

# List Pods across all namespaces
kubectl get pods -A -o wide

# Inspect a Pod
kubectl describe pod <pod-name> -n <namespace>

# View application logs
kubectl logs <pod-name> -n <namespace>

# Follow logs in real time
kubectl logs -f <pod-name> -n <namespace>

# View recent cluster events
kubectl get events -A --sort-by=.metadata.creationTimestamp

# Check resource utilization
kubectl top nodes
kubectl top pods -A

# Inspect a Service
kubectl describe svc <service-name> -n <namespace>

# Validate a manifest
kubectl apply --dry-run=client -f deployment.yaml

# Roll back a deployment
kubectl rollout undo deployment/<deployment-name> -n <namespace>
```

## ✅ Best Practices

- Use declarative YAML and version control.
- Define CPU and memory requests and limits.
- Configure appropriate health probes.
- Run containers as non-root whenever possible.
- Apply least-privilege RBAC.
- Keep secrets out of Git repositories.
- Use image scanning and trusted registries.
- Apply Network Policies where supported.
- Separate workloads using namespaces.
- Use Helm or GitOps for repeatable deployments.
- Configure monitoring, logging, and actionable alerts.
- Test backups and disaster recovery procedures.
- Regularly review Kubernetes versions and security updates.
- Track resource utilization and cloud infrastructure costs.

## 📁 Repository Structure

```text
kubernetes-learning-hub/
├── README.md
├── 01-fundamentals/
├── 02-architecture/
├── 03-pods-and-workloads/
├── 04-services-and-networking/
├── 05-storage/
├── 06-configmaps-and-secrets/
├── 07-security-and-rbac/
├── 08-helm/
├── 09-autoscaling-and-resources/
├── 10-monitoring-and-logging/
├── 11-cicd-and-gitops/
├── 12-aks/
├── 13-eks/
├── 14-troubleshooting/
├── 15-best-practices/
└── projects/
```

Each topic directory may contain:

- `README.md` — concept explanation and implementation guide
- `manifests/` — Kubernetes YAML files
- `scripts/` — automation scripts
- `examples/` — practical examples
- `troubleshooting.md` — common problems and solutions

## 🧪 Hands-on Projects

Recommended projects to build practical experience:

1. Deploy a web application using Kubernetes Deployments and Services.
2. Configure an Ingress Controller with TLS.
3. Deploy a highly available application with multiple replicas.
4. Deploy a stateful application using StatefulSets and persistent storage.
5. Implement RBAC and namespace isolation.
6. Build a CI/CD pipeline using GitHub Actions or Azure DevOps.
7. Deploy Prometheus and Grafana for cluster monitoring.
8. Implement autoscaling using HPA and cluster autoscaling.
9. Deploy an application to AKS or EKS using Helm.
10. Implement GitOps using Argo CD.
11. Integrate cloud secret management using workload identity.
12. Configure production-grade security, backup, and disaster recovery.

## 🎯 Learning Roadmap

| Level | Focus Areas |
|---|---|
| Beginner | Containers, Pods, Deployments, Services, `kubectl` |
| Intermediate | Networking, Storage, ConfigMaps, Secrets, Helm |
| Advanced | Security, RBAC, Autoscaling, Monitoring, GitOps |
| Production | AKS/EKS, High Availability, Disaster Recovery, FinOps |

## 📖 Official Documentation

- [Kubernetes
