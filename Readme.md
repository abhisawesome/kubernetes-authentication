# Kubernetes

## Authentication using Service account

Below configs to create for github auto deployment using config file.

Reference: [https://kubernetes.io/docs/reference/access-authn-authz/rbac](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

## 1. Create cluster role like the below file

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: continuous-deployment
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "update", "list"]
  - apiGroups: ["apps"]
    resources: ["deployments"]
    verbs: ["get", "update", "list", "create"]
```

## 2. Bind the cluster role

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: continuous-deployment
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: continuous-deployment
subjects:
  - kind: ServiceAccount
    name: githubactions
    namespace: default
```

## 3. Create service account

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: githubactions
  namespace: kube-system
secrets:
  - name: github-action-secret
```

## 4. Create secret for the service account

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: github-action-secretsssss
  annotations:
    kubernetes.io/service-account.name: githubactions
type: kubernetes.io/service-account-token
```

## 5.

Create a config file as like the below

```yaml
apiVersion: v1
current-context: { context-name }
kind: Config
clusters:
  - cluster:
      certificate-authority-data: { cluster-ca }
      server: { server-dns }
    name: { cluster-name }
contexts:
  - context:
      cluster: { cluster-name }
      user: { user-name }
    name: { context-name }

users:
  - name: { user-name }
    user:
      token: { secret-token }
```

|   Variable   |                  Description                  |          Command to get value          |
| :----------: | :-------------------------------------------: | :------------------------------------: |
| context-name |  Name of the current context for the config   |                any name                |
|  cluster-ca  |           From your current context           | kubectl config view --flatten --minify |
|  server-dns  |           From your current context           | kubectl config view --flatten --minify |
| cluster-name |                 cluster name                  |                any name                |
|  user-name   |                   User name                   |                any name                |
| secret-token | secret created from above yaml file on step 4 | kubectl describe secret {secret_name}  |

## 6. use the command

```bash
kubectl apply -f dpeploy.yml --kubeconfig config
```
