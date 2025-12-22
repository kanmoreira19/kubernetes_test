# Ingress + Storage no MicroK8s

## LAB 4

Este laboratório demonstra como expor uma aplicação via **hostname** usando Ingress e persistir dados com **PVC + PV** utilizando o storage nativo do MicroK8s.

## ✅ Entregas esperadas
- Ingress expondo aplicação via hostname (`myapp.local`)
- PVC + PV usando storage do MicroK8s (`hostpath-storage`)
- Arquitetura local documentada em formato tree

---

## 🏗️ Arquitetura Local

local-machine  
└── /etc/hosts  
└── 192.168.1.3 myapp.local # Hostname → IP do nó MicroK8s  

microk8s-cluster  
├── addons  
│ ├── ingress # NGINX Ingress Controller  
│ └── hostpath-storage # StorageClass: microk8s-hostpath  
└── namespaces  
└── dev  
├── ingress  
│ └── app-ingress  
│ └── host: myapp.local → app-service:80  
├── services  
│ └── app-service (ClusterIP:80 → Pod:80)  
├── deployments  
│ └── app-deployment  
│ └── pod  
│ └── volumeMount: /usr/share/nginx/html ← app-pvc  
└── storage  
├── pvc: app-pvc (1Gi, RWO, microk8s-hostpath)  
└── pv: pvc-xxx (hostPath: /var/snap/microk8s/common/ default-storage/...)  

host-filesystem  
└── /var/snap/microk8s/common/default-storage/  
└── pvc-xxx/ # Dados persistentes da aplicação  

---

## 🚀 Pré-requisitos

### 1. Habilitar Addons do MicroK8s

microk8s enable ingress hostpath-storage dns

### 2. Verificar Addons

```bash
microk8s kubectl get pods -n ingress # Ingress controller rodando
microk8s kubectl get storageclass # microk8s-hostpath (default)
```

### 3. Configurar Hostname Local

IP do nó MicroK8s
```bash
NODE_IP=$(hostname -I | awk '{print $1}')
echo "$NODE_IP myapp.local" | sudo tee -a /etc/hosts
```

---

## 📁 Manifests Kubernetes

### 1. Namespace `dev`

```yaml
namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
name: dev
```

### 2. PVC (Storage MicroK8s)

```yaml
pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
name: app-pvc
namespace: dev
spec:
accessModes:
- ReadWriteOnce
resources:
requests:
storage: 1Gi
storageClassName: microk8s-hostpath
```

### 3. Deployment + Service

```yaml
deployment-service.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
name: app-deployment
namespace: dev
spec:
replicas: 1
selector:
matchLabels:
app: myapp
template:
metadata:
labels:
app: myapp
spec:
containers:
- name: nginx
image: nginx:1.25-alpine
ports:
- containerPort: 80
volumeMounts:
- name: app-storage
mountPath: /usr/share/nginx/html
volumes:
- name: app-storage
persistentVolumeClaim:
claimName: app-pvc

apiVersion: v1
kind: Service
metadata:
name: app-service
namespace: dev
spec:
selector:
app: myapp
ports:

port: 80
targetPort: 80
type: ClusterIP
```

### 4. Ingress (Hostname)

```yaml
ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
name: app-ingress
namespace: dev
annotations:
kubernetes.io/ingress.class: public
spec:
rules:

host: myapp.local
http:
paths:

path: /
pathType: Prefix
backend:
service:
name: app-service
port:
number: 80
```

---

## ⚙️ Deploy Sequencial

1. Namespace
```bash
microk8s kubectl apply -f namespace.yaml
```

2. PVC (Storage)
```bash
microk8s kubectl apply -f pvc.yaml
```

3. Deployment + Service
```bash
microk8s kubectl apply -f deployment-service.yaml
```

4. Ingress
```bash
microk8s kubectl apply -f ingress.yaml
```

---

## 🔍 Verificações

Status geral
```bash
microk8s kubectl -n dev get all
``

PVC e PV
```bash
microk8s kubectl -n dev get pvc
microk8s kubectl get pv
``

Ingress
```bash
microk8s kubectl -n dev get ingress
microk8s kubectl -n dev describe ingress app-ingress
```

Logs
```bash
microk8s kubectl -n dev logs deployment/app-deployment
```
