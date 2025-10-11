# External Secret  - Key Vault integration in AKS


Reference:

- https://dev.to/learn4ops/the-perfect-combination-for-azure-key-vault-integration-with-aks-external-secret-operator--4fgi
- https://learn.microsoft.com/en-us/azure/aks/csi-secrets-store-driver


### Normal secret:

In Kubernetes, a Secret is an object used to store sensitive data (like passwords, API keys, certificates). Normally, you create and manage these directly inside the cluster.


### Downside of normal Kubernetes secret:

There are a few downsides to using this built-in secrets management mechanism. Specifically, Kubernetes Secrets have several downsides such as:

- Stored as Base64 encoded objects so anyone with Cluster access can decode the secrets.
- Created either by kubectl CLI or in YAML manifests, making them insecure to integrate with version control systems. Avoid hardcoding secrets into Kubernetes YAML or GitHub repos.
- Difficult to manage and synchronize when managing multiple environments.
- No default mechanism to rotate and update the secrets -
An External Secret means that instead of storing secret data directly in Kubernetes, you sync secrets from an external secret manager (like Azure Key Vault, AWS Secrets Manager, HashiCorp Vault, etc.) into Kubernetes Secret objects.


**External Secrets covers above mentioned downsides:**


- Provides Centralized Secret Management.
- No need to manually update Kubernetes Secrets when passwords or keys rotate in Key Vault.) -ESO or CSI drivers handle syncing automatically.
- Enables automatic secret rotation inside pods when Key Vault secrets change.
- Separation of Concerns - `Developers` only consume Kubernetes Secrets and `Security/DevSecOps teams` manage real secrets in Azure Key Vault.

### External Secret:

`This is typically done using tools such as`:

1 - **External Secrets Operator (ESO)** - (most popular, open-source).

2 - **Cloud provider operators** (e.g., Azure Key Vault CSI driver, Secrets Store CSI driver).


### External Secrets Operator (ESO):

To use Kubernetes External Secrets, you must configure an external secrets backend and create a Kubernetes Secret object that points to the external backend. Kubernetes will then interact with the secret backend to read and write the secrets.

The External Secrets Operator makes using external secret management systems easier with Kubernetes. The External Secrets Operator will read the required information from the external API and inject it into a Kubernetes Secret for you. With this operator, you can easily incorporate secrets from providers like AWS Secrets Manager, HashiCorp Vault, and many more.


<img width="1216" height="519" alt="image" src="https://github.com/user-attachments/assets/f39e1688-8e9f-4dca-9402-42eb42e17099" />


<img width="1123" height="373" alt="image" src="https://github.com/user-attachments/assets/e7ce3552-502d-455e-b68d-a1e2268111b7" />



# is there any way to keep secret's values outside yaml file - and pass through using ADO / CICD pipelines at runtime - it will eliminate the use of external secrets.   -VNET learning -

