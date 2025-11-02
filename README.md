### Alura Checkpoint SRE Nível 2.
### Instalação do Docker e Minikube.
### Configuração do kubectl para gerenciar cluster Kubernetes.
### Instalação do Helm para gerenciar pacotes no Kubernetes.
### Configuração do Istio para gerenciar a segurança e o tráfego de rede.
### Utilização do Visual Studio Code para editar arquivos YAML e scripts.
### Utilização do Postman para testar APIs e endpoints da aplicação.
### Atividade executada em uma VM Ubuntu Server com 4 CPU's e 4096Gb de memória RAM.

# Instalação do Docker e Minikube
# INSTALAR DOCKER
https://docs.docker.com/engine/install/ubuntu/
## Set up Docker's apt repository
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
## Install the Docker packages
```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```
## adiciona o usuário ao grupo docker
```
sudo usermod -aG docker $USER
```
## reinicia sessão docker
```
newgrp docker
```
## verificação do docker
```
docker system info
```
# INSTALAR MINIKUBE
https://minikube.sigs.k8s.io/docs/start/?arch=%2Flinux%2Fx86-64%2Fstable%2Fbinary+download
## instalar minikube
```
curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64
```
## iniciar minikube com driver docker e memória e cpu ajustados
```
minikube start --driver=docker --memory=2048 --cpus=2
```
## verificação do minikube
```
minikube status
```
# Configuração do kubectl para gerenciar cluster Kubernetes
https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/
## instalar kubectl
```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

```
## verificação do kubectl
```
kubectl version --client
kubectl cluster-info
```
# Instalação do Helm para gerenciar pacotes no Kubernetes
https://helm.sh/docs/intro/install/
## instalação do helm
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```
## verificação do helm
```
helm version
```
