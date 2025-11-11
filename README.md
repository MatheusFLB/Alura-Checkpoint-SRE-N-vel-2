# <span style="color:red;">Alura Checkpoint SRE Nível 2</span>
#### Instalação do Docker e Minikube;
#### Configuração do kubectl para gerenciar cluster Kubernetes;
#### Instalação do Helm para gerenciar pacotes no Kubernetes;
#### Configuração do Istio para gerenciar a segurança e o tráfego de rede;
#### Utilização do Visual Studio Code por SSH para editar arquivos YAML e scripts;
#### Utilização do Postman para testar APIs e endpoints da aplicação;
#### Atividade executada em uma VM Ubuntu Server com 4 CPU's e 4096Gb de memória RAM.

# <span style="color:red;">Instalação do Docker e Minikube no Kubernetes</span>
## INSTALAR DOCKER
https://docs.docker.com/engine/install/ubuntu/
### Set up Docker's apt repository
```
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```
### Install the Docker packages
```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```
### adiciona o usuário ao grupo docker
```
sudo usermod -aG docker $USER
```
### reinicia sessão docker
```
newgrp docker
```
### verificação do docker
```
docker system info
```
## INSTALAR MINIKUBE
https://minikube.sigs.k8s.io/docs/start/?arch=%2Flinux%2Fx86-64%2Fstable%2Fbinary+download
### instalar minikube
```
curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64
```
### verificação do minikube
```
minikube version
```
### iniciar minikube com driver docker e memória e cpu ajustados
```
minikube start --driver=docker --memory=2048 --cpus=2
```
### verificação do status do minikube
```
minikube status
```
### verificação de informações do cluster
```
minikube profile list
```
# <span style="color:red;">Configuração do kubectl para gerenciar cluster Kubernetes</span>
https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/
### instalar kubectl
```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

```
### verificação do kubectl
```
kubectl version --client
kubectl cluster-info
```
### o minikube configura automaticamente o kubectl, mas se necessário, pode configurar manualmente
```
minikube kubectl -- config view
kubectl config use-context minikube
```
### script para verificação do ambiente minikube
## <span style="color:yellow;">verify-cluster.sh</span>
```
#!/bin/bash
echo "======================================="
echo "     Diagnóstico do Ambiente Minikube"
echo "=======================================

"

# --- 1. STATUS BÁSICO DO MINIKUBE E CLUSTER ---
echo "=== 1. Verificando Status do Minikube ==="
minikube status

echo -e "\n=== 2. Verificando Status do kubectl e Cluster ==="
kubectl cluster-info
kubectl version --client=true

# --- 2. INFORMAÇÕES DE NÓS (INFRAESTRUTURA) ---
echo -e "\n=== 3. Listando Nodes do Cluster ==="
kubectl get nodes -o wide

echo -e "\n=== 4. Detalhes do Node principal (minikube) ==="
kubectl describe node minikube

# --- 3. NAMESPACES E COMPONENTES DO SISTEMA ---
echo -e "\n=== 5. Listando todos os Namespaces ==="
kubectl get namespaces

echo -e "\n=== 6. Verificando Pods do Sistema (kube-system) ==="
kubectl get pods -n kube-system -o wide

echo -e "\n=== 7. Verificando Serviços do Sistema (kube-system) ==="
kubectl get svc -n kube-system

# --- 4. MÉTRICAS E RECURSOS ---
echo -e "\n=== 8. Verificando Capacidade e Uso de Recursos ==="
kubectl top nodes || echo "⚠️  Métricas indisponíveis (Metrics Server pode não estar instalado)."

echo -e "\n✅ Diagnóstico concluído"
echo "======================================="
```
## <span style="color:green;">torna executável e executa</span>
```
chmod +x verify-cluster.sh
./verify-cluster.sh
```
# <span style="color:red;">Instalação do Helm para gerenciar pacotes no Kubernetes</span>
https://helm.sh/docs/intro/install/
### instalação do helm
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```
### verificação do helm
```
helm version
```
### adicionar repositórios estáveis
```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```
### listar repositórios
```
helm repo list
```
### criar diretório para nosso projeto
```
mkdir techsafe-helm-demo
cd techsafe-helm-demo
```
### criar um novo helm chart chamado 'techsafe-app'
```
helm create techsafe-app
```
### verificar a estrutura criada
```
ls -la techsafe-app/
ls -la techsafe-app/templates/
```
## customizar arquivo "techsafe-app/values.yaml"
```
# techsafe-app/values.yaml

# Configuração global
global:
  appName: "techsafe-demo"
  environment: "development"

# Frontend Service
frontend:
  replicaCount: 2
  image:
    repository: nginx
    tag: "1.25"
    pullPolicy: IfNotPresent
  service:
    type: ClusterIP
    port: 80
    targetPort: 80
  resources:
    requests:
      memory: "128Mi"
      cpu: "100m"
    limits:
      memory: "256Mi"
      cpu: "200m"

# Backend API Service
backend:
  replicaCount: 2
  image:
    repository: redis
    tag: "7.2"
    pullPolicy: IfNotPresent
  service:
    type: ClusterIP
    port: 6379
    targetPort: 6379
  resources:
    requests:
      memory: "128Mi"
      cpu: "100m"
    limits:
      memory: "256Mi"
      cpu: "200m"

# Database Service
database:
  enabled: true
  image:
    repository: redis
    tag: "7.2"
    pullPolicy: IfNotPresent
  service:
    type: ClusterIP
    port: 6379
    targetPort: 6379
  resources:
    requests:
      memory: "256Mi"
      cpu: "200m"
    limits:
      memory: "512Mi"
      cpu: "500m"
```
### remover templates padrão e criar nossos próprios
```
cd techsafe-app/templates
rm -rf *
```
## template para techsafe-app/templates/deployment-frontend.yaml
```
# techsafe-app/templates/deployment-frontend.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.global.appName }}-frontend
  labels:
    app: {{ .Values.global.appName }}-frontend
    environment: {{ .Values.global.environment }}
spec:
  replicas: {{ .Values.frontend.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Values.global.appName }}-frontend
  template:
    metadata:
      labels:
        app: {{ .Values.global.appName }}-frontend
        environment: {{ .Values.global.environment }}
    spec:
      containers:
      - name: frontend
        image: "{{ .Values.frontend.image.repository }}:{{ .Values.frontend.image.tag }}"
        imagePullPolicy: {{ .Values.frontend.image.pullPolicy }}
        ports:
        - containerPort: 80
        resources:
          {{- toYaml .Values.frontend.resources | nindent 12 }}
---
apiVersion: v1
kind: Service
metadata:
  name: {{ .Values.global.appName }}-frontend-service
spec:
  type: {{ .Values.frontend.service.type }}
  ports:
  - port: {{ .Values.frontend.service.port }}
    targetPort: {{ .Values.frontend.service.targetPort }}
  selector:
    app: {{ .Values.global.appName }}-frontend
```
## template para techsafe-app/templates/deployment-backend.yaml
```
# techsafe-app/templates/deployment-backend.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.global.appName }}-backend
  labels:
    app: {{ .Values.global.appName }}-backend
    environment: {{ .Values.global.environment }}
spec:
  replicas: {{ .Values.backend.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Values.global.appName }}-backend
  template:
    metadata:
      labels:
        app: {{ .Values.global.appName }}-backend
        environment: {{ .Values.global.environment }}
    spec:
      containers:
      - name: backend
        image: "{{ .Values.backend.image.repository }}:{{ .Values.backend.image.tag }}"
        imagePullPolicy: {{ .Values.backend.image.pullPolicy }}
        ports:
        - containerPort: 6379
        resources:
          {{- toYaml .Values.backend.resources | nindent 12 }}
---
apiVersion: v1
kind: Service
metadata:
  name: {{ .Values.global.appName }}-backend-service
spec:
  type: {{ .Values.backend.service.type }}
  ports:
  - port: {{ .Values.backend.service.port }}
    targetPort: {{ .Values.backend.service.targetPort }}
  selector:
    app: {{ .Values.global.appName }}-backend
```
### implantar com helm
```
# Voltar ao diretório principal
cd ../..

# Instalar/atualizar o chart
helm upgrade --install techsafe-demo ./techsafe-app

# Verificar a implantação
helm list

# Verificar os recursos criados
kubectl get all
```
### script para verificação da implementação por helm
## <span style="color:yellow;">verify-helm-deployment.sh</span>
```
#!/bin/bash
# ============================================================
#  Script de Verificação - TechSafe Demo (Helm + Kubernetes)
# ============================================================
#  Objetivo:
#    Este script realiza uma verificação completa da implantação
#    Helm, incluindo releases, pods, serviços, deployments e logs.
# ============================================================

# Encerrar o script caso algum comando falhe
set -e

# ---------- Funções de estilo ----------
info()    { echo -e "\033[1;34m[INFO]\033[0m $1"; }
success() { echo -e "\033[1;32m[SUCCESS]\033[0m $1"; }
warn()    { echo -e "\033[1;33m[WARN]\033[0m $1"; }
error()   { echo -e "\033[1;31m[ERROR]\033[0m $1"; }

echo "==========================================="
echo "🔍 Verificação da Implantação - TechSafe Demo"
echo "==========================================="

# ---------- Passo 1: Verificar Helm ----------
echo -e "\n=== [1] Verificando Releases Helm ==="
info "Listando releases instaladas..."
helm list || error "Falha ao listar releases Helm."

info "Verificando status detalhado da release 'techsafe-demo'..."
helm status techsafe-demo || warn "Release 'techsafe-demo' não encontrada."

info "Verificando valores configurados..."
helm get values techsafe-demo || warn "Não foi possível obter valores da release."

# ---------- Passo 2: Verificar Recursos do Kubernetes ----------
echo -e "\n=== [2] Verificando Recursos Criados ==="
info "Listando todos os recursos do namespace padrão..."
kubectl get all || error "Falha ao listar recursos Kubernetes."

# ---------- Passo 3: Verificar Pods ----------
echo -e "\n=== [3] Verificando Pods ==="
kubectl get pods -o wide || warn "Não foi possível listar pods."

info "Filtrando apenas pods em execução..."
kubectl get pods --field-selector=status.phase=Running || warn "Nenhum pod em execução encontrado."

# ---------- Passo 4: Verificar Services ----------
echo -e "\n=== [4] Verificando Services ==="
kubectl get services || warn "Falha ao listar serviços."

info "Detalhes do service 'techsafe-demo-frontend-service'..."
kubectl describe service techsafe-demo-frontend-service || warn "Serviço frontend não encontrado."

# ---------- Passo 5: Verificar Deployments ----------
echo -e "\n=== [5] Verificando Deployments ==="
kubectl get deployments || warn "Não foi possível listar deployments."

# ---------- Passo 6: Verificar Logs ----------
echo -e "\n=== [6] Verificando Logs dos Pods do Frontend ==="
info "Mostrando logs dos pods com label 'app=techsafe-demo-frontend'..."
kubectl logs -l app=techsafe-demo-frontend --tail=30 || warn "Nenhum log encontrado para o frontend."

# ---------- Passo 7: Teste de Acesso ----------
echo -e "\n=== [7] Teste de Acesso (Manual) ==="
warn "Para testar o frontend localmente, execute em outro terminal:"
echo "kubectl port-forward service/techsafe-demo-frontend-service 8080:80"
echo "curl http://localhost:8080"
echo "⚠️  Pressione CTRL+C para encerrar o port-forward após o teste."

# ---------- Passo 8: Histórico e Rollback ----------
echo -e "\n=== [8] Histórico e Gerenciamento Helm ==="
info "Verificando histórico de revisões da release..."
helm history techsafe-demo || warn "Histórico de revisões não encontrado."

warn "Para reverter a release, use:"
echo "helm rollback techsafe-demo <número-da-revisão>"

# ---------- Passo 9: Resumo Final ----------
echo -e "\n=== [9] Resumo Final ==="
info "Resumo do estado atual dos recursos principais:"

echo -e "\n→ Pods:"
kubectl get pods --no-headers | awk '{print "• " $1 "\t" $3 "\t(" $2 ")"}'

echo -e "\n→ Services:"
kubectl get svc --no-headers | awk '{print "• " $1 "\t" $3 "\t" $4 ":" $5}'

echo -e "\n→ Deployments:"
kubectl get deploy --no-headers | awk '{print "• " $1 "\t" $2 " disponíveis de " $3}'

success "\n✅ Cluster TechSafe Demo verificado com sucesso"
echo "==========================================="

# ---------- Conclusão ----------
echo -e "\n==========================================="
success "✅ Verificação completa concluída com sucesso"
echo "==========================================="

```
## <span style="color:green;">torna executável e executa</span>
```
chmod +x verify-helm-deployment.sh
./verify-helm-deployment.sh
```
# <span style="color:red;">Configuração do Istio para gerenciar a segurança e o tráfego de rede</span>
### adicionar o repositório oficial do Istio
```
helm repo add istio https://istio-release.storage.googleapis.com/charts
helm repo update
```
### criar namespaces para o istio
```
kubectl create namespace istio-system
kubectl create namespace istio-ingress
```
### instalar o chart base do istio (CRDs)
```
helm install istio-base istio/base -n istio-system
```
### instalar o istiod (control plane)
```
helm install istiod istio/istiod -n istio-system --wait
```
### VERIFICA istio-system
```
helm ls -n istio-system
kubectl get pods -n istio-system -w
```
### instalar o gateway istio-ingress
```
# comando para subir sem loadbalancer
helm install istio-ingress istio/gateway -n istio-ingress --wait --set service.type=NodePort --debug

# comando para subir com loadbalancer
helm install istio-ingress istio/gateway -n istio-ingress --wait --debug
```
### VERIFICA istio-ingress
```
helm ls -n istio-ingress
```
### LIMPA O ISTIO PARA TENTAR NOVAMENTE
```
helm uninstall istio-ingress -n istio-ingress
helm uninstall istiod -n istio-system
helm uninstall istio-base -n istio-system

kubectl get crds | grep 'istio.io' | awk '{print $1}' | xargs kubectl delete crd

kubectl delete namespace istio-system
kubectl delete namespace istio-ingress

kubectl get pods --all-namespaces | grep istio
kubectl get svc --all-namespaces | grep istio
kubectl get crds | grep istio.io
```
### script para verificação do istio
## <span style="color:yellow;">verify-istio-installation.sh</span>
```
#!/bin/bash

echo "=== ISTIO INSTALLATION VERIFICATION ==="

echo -e "\n1. Helm Releases:"
helm ls -A | grep istio

echo -e "\n2. Istio System Pods:"
kubectl get pods -n istio-system

echo -e "\n3. Istio Ingress Pods:"
kubectl get pods -n istio-ingress

echo -e "\n4. Application Pods with Sidecars:"
kubectl get pods -o wide | grep techsafe-demo

echo -e "\n5. Istio Services:"
kubectl get svc -n istio-system
kubectl get svc -n istio-ingress

echo -e "\n6. Istio Injection Status:"
kubectl get namespace -L istio-injection

echo -e "\n7. Istio CRDs:"
kubectl get crd | grep istio | wc -l

echo -e "\n=== VERIFICATION COMPLETE ==="
```
## <span style="color:green;">torna executável e executa</span>
```
chmod +x verify-istio-installation.sh
./verify-istio-installation.sh
```
# <span style="color:red;">Garantir disponibilidade e backups automatizados</span>
## Configurar Liveness Probes nos Deployments
## Atualize o techsafe-app/templates/deployment-frontend.yaml
```
# techsafe-app/templates/deployment-frontend.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.global.appName }}-frontend
  labels:
    app: {{ .Values.global.appName }}-frontend
    environment: {{ .Values.global.environment }}
spec:
  replicas: {{ .Values.frontend.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Values.global.appName }}-frontend
  template:
    metadata:
      labels:
        app: {{ .Values.global.appName }}-frontend
        environment: {{ .Values.global.environment }}
    spec:
      containers:
      - name: frontend
        image: "{{ .Values.frontend.image.repository }}:{{ .Values.frontend.image.tag }}"
        imagePullPolicy: {{ .Values.frontend.image.pullPolicy }}
        ports:
        - containerPort: 80
        resources:
          {{- toYaml .Values.frontend.resources | nindent 12 }}
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
---
apiVersion: v1
kind: Service
metadata:
  name: {{ .Values.global.appName }}-frontend-service
spec:
  type: {{ .Values.frontend.service.type }}
  ports:
  - port: {{ .Values.frontend.service.port }}
    targetPort: {{ .Values.frontend.service.targetPort }}
  selector:
    app: {{ .Values.global.appName }}-frontend
```
## Atualize o techsafe-app/templates/deployment-backend.yaml
```
# techsafe-app/templates/deployment-backend.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.global.appName }}-backend
  labels:
    app: {{ .Values.global.appName }}-backend
    environment: {{ .Values.global.environment }}
spec:
  replicas: {{ .Values.backend.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Values.global.appName }}-backend
  template:
    metadata:
      labels:
        app: {{ .Values.global.appName }}-backend
        environment: {{ .Values.global.environment }}
    spec:
      containers:
      - name: backend
        image: "{{ .Values.backend.image.repository }}:{{ .Values.backend.image.tag }}"
        imagePullPolicy: {{ .Values.backend.image.pullPolicy }}
        ports:
        - containerPort: 6379
        command: ["redis-server"]
        args: ["--save", "60", "1", "--loglevel", "warning"]
        resources:
          {{- toYaml .Values.backend.resources | nindent 12 }}
        livenessProbe:
          exec:
            command:
            - redis-cli
            - ping
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        readinessProbe:
          exec:
            command:
            - redis-cli
            - ping
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
        volumeMounts:
        - name: redis-data
          mountPath: /data
      volumes:
      - name: redis-data
        emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  name: {{ .Values.global.appName }}-backend-service
spec:
  type: {{ .Values.backend.service.type }}
  ports:
  - port: {{ .Values.backend.service.port }}
    targetPort: {{ .Values.backend.service.targetPort }}
  selector:
    app: {{ .Values.global.appName }}-backend
```
## Crie um novo template techsafe-app/templates/cronjob-backup.yaml
```
# techsafe-app/templates/cronjob-backup.yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: {{ .Values.global.appName }}-redis-backup
  labels:
    app: {{ .Values.global.appName }}-backup
    environment: {{ .Values.global.environment }}
spec:
  schedule: "{{ .Values.backup.schedule }}"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: redis:7.2
            command:
            - /bin/sh
            - -c
            - |
              set -e
              echo "Starting Redis backup at $(date)"
              
              # Criar backup usando redis-cli
              redis-cli -h {{ .Values.global.appName }}-backend-service SAVE
              
              # Copiar dump.rdb para local
              redis-cli -h {{ .Values.global.appName }}-backend-service --rdb /tmp/dump-$(date +%Y%m%d-%H%M%S).rdb
              
              # Verificar se o backup foi criado
              ls -la /tmp/dump-*.rdb
              echo "Backup completed successfully at $(date)"
            volumeMounts:
            - name: backup-storage
              mountPath: /tmp
          restartPolicy: OnFailure
          volumes:
          - name: backup-storage
            emptyDir: {}
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 3
```
## Atualize o techsafe-app/values.yaml para incluir configurações de backup
```
# Adicione esta seção ao values.yaml existente
backup:
  schedule: "*/5 * * * *"  # A cada 5 minutos para teste
  # Para produção use: "0 2 * * *" (diariamente às 2AM)
```
### Aplique as mudanças no cluster
```
# navegar para o diretório do Helm chart
cd ~/techsafe-helm-demo

# atualizar a release com as novas configurações
helm upgrade techsafe-demo ./techsafe-app

# verificar a implantação
kubectl get pods
```
### script para verificação da aplicação
## <span style="color:yellow;">verify-sre-setup.sh</span>
```
#!/bin/bash

echo "=== VERIFICAÇÃO SRE - DISPONIBILIDADE E BACKUPS ==="

echo -e "\n1. Status dos Pods e Probes:"
kubectl get pods -o wide

echo -e "\n2. Verificar Liveness Probes:"
kubectl describe pods -l app=techsafe-demo-frontend | grep -A 10 "Liveness"
kubectl describe pods -l app=techsafe-demo-backend | grep -A 10 "Liveness"

echo -e "\n3. Verificar CronJobs:"
kubectl get cronjobs

echo -e "\n4. Verificar Jobs de Backup:"
kubectl get jobs

echo -e "\n5. Logs dos Últimos Backups:"
JOB_POD=$(kubectl get pods --sort-by=.metadata.creationTimestamp | grep techsafe-demo-redis-backup | tail -1 | awk '{print $1}')
if [ -n "$JOB_POD" ]; then
  echo "Logs do último backup ($JOB_POD):"
  kubectl logs $JOB_POD
else
  echo "Nenhum job de backup encontrado ainda."
fi

echo -e "\n6. Eventos do Cluster:"
kubectl get events --sort-by=.metadata.creationTimestamp | tail -10

echo -e "\n7. Testar Disponibilidade dos Serviços:"
kubectl port-forward service/techsafe-demo-frontend-service 8080:80 &
sleep 2
curl -s http://localhost:8080 > /dev/null && echo "✅ Frontend respondendo" || echo "❌ Frontend offline"
pkill -f "port-forward"

echo -e "\n8. Testar Conexão com Backend:"
kubectl exec -it $(kubectl get pods -l app=techsafe-demo-backend -o name | head -1) -- redis-cli ping

echo -e "\n=== VERIFICAÇÃO COMPLETA ==="
```
## <span style="color:green;">torna executável e executa</span>
```
chmod +x verify-sre-setup.sh
./verify-sre-setup.sh
```
# <span style="color:red;">PARA TESTES DE CONECTIVIDADE EM APICAÇÕES COMO O POSTMAN</span>
```
# expor o frontend temporariamente para teste
kubectl port-forward service/techsafe-demo-frontend-service 8080:80

# testar o acesso
curl http://localhost:8080
```
# para testar o acesso por palicações como postman, usar os seguintes comandos
```
# expor o frontend temporariamente para teste
kubectl port-forward service/techsafe-demo-frontend-service 8080:80 --address <ip do host>

# testar o acesso
get http://<ip do host>:8080
```
