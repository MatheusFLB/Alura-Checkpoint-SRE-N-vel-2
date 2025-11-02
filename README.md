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
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
## permissões
```
sudo usermod -aG docker $USER
```
## reiniciar sessão docker
```
newgrp docker
```
## verificar o docker
```
docker system info
```
# INSTALAR MINIKUBE
## instalar minikube
```
curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64
```
## iniciar minikube com driver docker e memória e cpu ajustados
```
minikube start --driver=docker --memory=2048 --cpus=2
```
