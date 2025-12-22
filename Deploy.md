## LAB 2

Um deploy completo pode ser feito com um app Nginx simples, usando Deployment, dois Services (ClusterIP e NodePort), um ConfigMap injetado via env e probes HTTP. Abaixo vão os manifests em YAML que atendem exatamente ao que foi pedido.​

1. ConfigMap (variáveis de ambiente)

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  APP_NAME: "nginx-k8s-lab"
  APP_ENV: "production"
  APP_MESSAGE: "Hello from ConfigMap in Kubernetes!"
```

Aplicar:

```bash
kubectl apply -f configmap.yaml
```

O ConfigMap guarda pares chave-valor não sensíveis e será consumido como variáveis de ambiente no Pod.
​
2. Deployment com probes e uso do ConfigMap

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx-lab
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-lab
  template:
    metadata:
      labels:
        app: nginx-lab
    spec:
      containers:
        - name: nginx
          image: nginx:1.27-alpine
          ports:
            - containerPort: 80
          envFrom:
            - configMapRef:
                name: nginx-config
          livenessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 15
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 5
            periodSeconds: 5
```

Aplicar:

```bash
kubectl apply -f deployment.yaml
```

As probes HTTP verificam / na porta 80 para saber se o container está vivo e pronto para receber tráfego.
​

3. Service ClusterIP (uso interno)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-clusterip
  labels:
    app: nginx-lab
spec:
  type: ClusterIP
  selector:
    app: nginx-lab
  ports:
    - name: http
      port: 80
      targetPort: 80
```

Aplicar:

```bash
kubectl apply -f service-clusterip.yaml
```

Esse Service expõe o Deployment apenas dentro do cluster, acessível via nginx-clusterip:80 por outros Pods.
​
4. Service NodePort (acesso externo)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
  labels:
    app: nginx-lab
spec:
  type: NodePort
  selector:
    app: nginx-lab
  ports:
    - name: http
      port: 80        # porta do Service
      targetPort: 80  # porta do container
      nodePort: 30080 # porta exposta no Node (30000–32767)
```

Aplicar:

```bash
kubectl apply -f service-nodeport.yaml
```

O NodePort abre a porta 30080 em todos os nodes; acesso via http://IP_DO_NODE:30080.
​
5. Testes e Screenshot
Verificar recursos:

```bash
kubectl get deploy,pods,svc -l app=nginx-lab
```

Obter IP do node (minikube/microk8s):

```bash
kubectl get nodes -o wide
```

Acessar no browser:

```text
http://IP_DO_NODE:30080
```

Tirar screenshot da página padrão do Nginx acessada via NodePort.

Esse conjunto (configmap.yaml, deployment.yaml, service-clusterip.yaml, service-nodeport.yaml) cobre todos os itens: Deployment, Services ClusterIP/NodePort, ConfigMap via env, readiness/liveness probes e acesso externo para o print da tela.
