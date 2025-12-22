# LAB 5 — Troubleshooting Kubernetes

Este laboratório cobre os **3 cenários de troubleshooting mais comuns** em Kubernetes com prints reais, comandos e soluções práticas.

## ✅ Entregas esperadas
- **CrashLoopBackOff** - Pod reiniciando infinitamente
- **Ingress não sobe** - 404/503 mesmo com pods rodando
- **Node NotReady** - Nó fora do ar no cluster

---

## 🚨 1. CrashLoopBackOff - "Por que meu Pod está reiniciando?"

### Sintoma

```bash
kubectl get pods

NAME              READY STATUS           RESTARTS AGE
broken-app-abc123  0/1  CrashLoopBackOff     5    10m
```

