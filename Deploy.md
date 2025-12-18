Um deploy completo pode ser feito com um app Nginx simples, usando Deployment, dois Services (ClusterIP e NodePort), um ConfigMap injetado via env e probes HTTP. Abaixo vão os manifests em YAML que atendem exatamente ao que foi pedido.​

1. ConfigMap (variáveis de ambiente)