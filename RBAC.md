# LAB 3 — Namespaces + RBAC Básico (Namespace `dev`)

Este laboratório demonstra a criação de um namespace `dev` com RBAC restrito, permitindo que um ServiceAccount liste apenas pods.

## ✅ Entregas esperadas
- Namespace `dev` criado
- ServiceAccount `pod-reader` com permissão restrita
- Role + RoleBinding para listar pods
- Teste validando RBAC funcionando

---

## 🚀 Pré-requisitos

Criar diretório do lab
mkdir -p lab3-namespaces-rbac && cd lab3-namespaces-rbac

text

---

## 📄 Manifests Kubernetes

### 1. Namespace `dev`

namespace.yaml

```yaml
apiVersion: v1
kind: Namespace
metadata:
name: dev
labels:
name: dev
environment: development
```

### 2. ServiceAccount Restrito

serviceaccount.yaml

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
name: pod-reader
namespace: dev
```

### 3. Role (Permissão Listar Pods)

role.yaml

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
namespace: dev
name: pod-list-role
rules:

apiGroups: [""]
resources: ["pods"]
verbs: ["get", "list", "watch"]
```

### 4. RoleBinding

rolebinding.yaml

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
name: pod-reader-binding
namespace: dev
subjects:

kind: ServiceAccount
name: pod-reader
namespace: dev
roleRef:
kind: Role
name: pod-list-role
apiGroup: rbac.authorization.k8s.io
``