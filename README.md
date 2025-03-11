# **Documentação do Desafio DevOps**

## **1. Objetivo**

Provisionar um banco de dados PostgreSQL e realizar o build e deploy no Kubernetes de dois serviços (frontend e backend), configurando escalabilidade automática baseada no uso de memória.

## **2. Requisitos**

- Provisionar um banco de dados PostgreSQL no Kubernetes.
- Realizar o build e deploy dos serviços:
  - **Frontend** (porta 8000)
  - **Backend** (porta 8080) conectado ao banco PostgreSQL
- Configurar escalabilidade automática para os serviços quando o uso de memória ultrapassar 70%.
- Utilizar Minikube para execução local.
- O frontend deve se comunicar com o backend via `http://backend-service.app.svc.cluster.local:8080`.

## **3. Tecnologias Utilizadas**

- **Kubernetes** para orquestração de contêineres.
- **Minikube** para ambiente local.
- **Docker** para build das imagens.
- **Metrics Server** para suporte ao HPA.
- **k9s** para gerenciamento simplificado do cluster.
- **Ingress/Nginx** para gerenciamento de tráfego (sugestão de melhoria).
- **Grafana/Prometheus** para monitoramento de métricas (sugestão de melhoria).
- **Secrets do Kubernetes ou AWS Secret Manager** para segurança de credenciais (sugestão de melhoria).
- **CI/CD** para automatizar o build/deploy das aplicações. 

---

# **Guia de Instalação e Execução**

## **4. Instalação das Dependências**

```sh
# Instalar Minikube
curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64

# Iniciar o Minikube
minikube start

# Configurar Docker para usar Minikube
eval $(minikube docker-env)

# Instalar k9s (Opcional para monitoramento)
brew install k9s
```

---

## **5. Build das Imagens**

```sh
# Clonar o repositório
git clone https://github.com/*********/devops-microservices-demo.git
cd devops-microservices-demo

# Build da imagem do backend
docker build -t backend:latest -f backend/Docker/api/Dockerfile .

# Build da imagem do frontend
docker build -t frontend:latest -f frontend/Dockerfile .
```

---

## **6. Implantar os manifestos do Kubernetes**

```sh
# Habilitar o Metrics Server para HPA
minikube addons enable metrics-server

# Aplicar os manifestos
kubectl apply -f k8s.yaml
```

---

## **7. Verificar Status do Cluster(Utilizei k9s)**

```sh
# Listar todos os pods
kubectl get pods -A

# Ver logs do backend
kubectl logs -n app -l app=backend --tail=50

# Monitorar HPA
kubectl get hpa -n app
```

---

# **Explicação das Escolhas Técnicas**

## **8. Estrutura dos Manifestos Kubernetes**

### **Banco de Dados PostgreSQL**

- Utiliza **StatefulSet** para manter a persistência de dados.
- PVC (**Persistent Volume Claim**) garante armazenamento persistente para os dados do banco.
- Service **postgres-service** expõe o banco para o backend.

### **Backend**

- Deployment gerencia o backend com duas réplicas iniciais.
- Definição de variáveis de ambiente para conexão segura ao PostgreSQL.
- Liveness e Readiness Probes garantem disponibilidade.
- HorizontalPodAutoscaler (HPA) escalona de 2 até 10 réplicas baseado no uso de memória.

### **Frontend**

- Deployment com uma réplica inicial.
- Service **NodePort** para expor a aplicação localmente.
- Configuração de HPA para escalar até 10 réplicas.

---

# **Testes e Debug**

## **9. Testar a API Backend**

```sh
kubectl port-forward svc/backend-service -n app 8080:8080

# Testar a API
curl http://localhost:8080/
```

## **10. Testar o Frontend**

```sh
kubectl port-forward svc/frontend-service -n app 8000:8000

# Acessar no navegador
http://localhost:8000
```

---

# **Segurança e Boas Práticas**

## **11. Uso de Secrets para Credenciais**

Atualmente, as credenciais do PostgreSQL estão hardcoded no manifesto. Para melhorar a segurança, utilizar Secrets:

```sh
kubectl create secret generic postgres-secret -n db \
  --from-literal=POSTGRES_USER=admin \
  --from-literal=POSTGRES_PASSWORD=sAtImSOMPReDIaDacrEXPa
```

E referenciar no `StatefulSet`:

```yaml
env:
  - name: POSTGRES_USER
    valueFrom:
      secretKeyRef:
        name: postgres-secret
        key: POSTGRES_USER
  - name: POSTGRES_PASSWORD
    valueFrom:
      secretKeyRef:
        name: postgres-secret
        key: POSTGRES_PASSWORD
```

Alternativamente, serviços como **AWS Secrets Manager** ou **HashiCorp Vault** podem ser utilizados para armazenar credenciais de forma segura.

## **12. Melhorias Sugeridas**

- **Ingress/Nginx**: Implementar um controlador Ingress para gerenciar tráfego HTTP em vez de utilizar NodePort.
- **Grafana/Prometheus**: Adicionar monitoramento detalhado dos serviços e cluster Kubernetes.
- **Logging Centralizado**: Implementação de um stack ELK (Elasticsearch, Logstash e Kibana) ou Loki para análise centralizada de logs.

---

# **Entrega Final**

- Todo o código e manifestos estão no repositório GitHub.
- Documentação explica as escolhas técnicas e guias de execução.
- Testes de health check confirmam que a aplicação está rodando corretamente.

## **Conclusão**

O ambiente Kubernetes está configurado para alta disponibilidade, escalabilidade e performance, seguindo boas práticas de DevOps.

