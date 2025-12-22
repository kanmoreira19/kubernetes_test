# Instalação MicroK8s do Zero - Cluster Completo

## LAB - 1

Documentação completa para instalação MicroK8s em Ubuntu Server 22.04/24.04 LTS, criação de cluster HA, ativação de addons essenciais (DNS, Storage, Ingress) e validações.

## 📋 Pré-requisitos

Atualizar sistema

```bash
sudo apt update && sudo apt upgrade -y && sudo reboot
```

Instalar snapd (se necessário)

```bash
sudo apt install snapd -y
```

Após instalação MicroK8s, adicionar usuário ao grupo

```bash
sudo usermod -aG microk8s $USER
newgrp microk8s
```

## 🚀 Instalação MicroK8s Single Node

Instalar MicroK8s versão estável 1.32

```bash
sudo snap install microk8s --classic --channel=1.32
```

Aguardar readiness (2-5 minutos)

```bash
microk8s status --wait-ready
```

Verificar primeiro node

```bash
microk8s kubectl get nodes
```

**✅ Saída esperada:**

```bash
NAME     STATUS ROLES                AGE VERSION
microk8s Ready  control-plane,master 5m  v1.32.x
```

## 🔧 Ativar Addons Essenciais

DNS com resolvers Google/Cloudflare
microk8s enable dns:8.8.8.8,1.1.1.1

Storage provisioner (hostpath)
microk8s enable storage

Ingress NGINX controller
microk8s enable ingress

Helm 3 (opcional)
microk8s enable helm3

Registry (opcional)
microk8s enable registry

**Verificar pods:**

microk8s kubectl get pods -n kube-system


## 🧠 Criar Cluster HA (3+ Nodes)

### No PRIMEIRO NODE (Master):

microk8s add-node

**Copie o comando gerado (exemplo):**

microk8s join 192.168.1.100:25000/abc123-token-aqui

### Nos DEMAIS NODES (Workers):

Instalar MicroK8s
sudo snap install microk8s --classic --channel=1.32

Executar comando de join do master
microk8s join 192.168.1.100:25000/abc123-token-aqui

Adicionar usuário ao grupo
sudo usermod -aG microk8s $USER
newgrp microk8s

## ✅ Validações Finais

Status completo do cluster
microk8s status

Nodes do cluster
microk8s kubectl get nodes -o wide

Todos os recursos kube-system
microk8s kubectl get all -n kube-system

Diagnóstico completo
microk8s inspect

## 🔍 Comandos Úteis Pós-Instalação

Alias kubectl global
alias kubectl='microk8s kubectl'

Configurar kubectl externo
microk8s config > ~/.kube/config

Verificar ingress
microk8s kubectl get svc -n ingress

IP do Ingress
microk8s kubectl get nodes -o wide | awk 'NR>1{print $6}'