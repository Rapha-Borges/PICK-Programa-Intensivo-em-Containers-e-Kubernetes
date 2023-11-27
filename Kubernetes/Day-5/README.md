# Cluster Kubernetes e o Kubeadm

### O que é um Cluster Kubernetes?

Um cluster Kubernetes é um conjunto de nós que executam aplicações Kubernetes. Um cluster Kubernetes é composto por pelo menos um nó control plane e vários nós workers. O nó control plane é responsável por executar os componentes de controle do Kubernetes e é o ponto de entrada para todas as interações com o cluster. Os nós workers executam os componentes de trabalho do Kubernetes e executam as cargas de trabalho reais.

### O que é o Kubeadm?

O Kubeadm é uma ferramenta de linha de comando que facilita a instalação e a configuração do Kubernetes. Ele é projetado para configurar um cluster Kubernetes de maneira consistente em várias plataformas.

### O que é o CNI?

O Container Network Interface (CNI) é uma especificação e um conjunto de bibliotecas que definem uma interface entre o contêiner e a rede. O CNI fornece uma maneira de configurar a rede de um contêiner Linux, sem precisar interagir com o sistema operacional do host diretamente.

### Recomendações para criar um cluster Kubernetes

- **Sistema Operacional:** Linux (Ubuntu, Debian, CentOS, Red Hat, etc)
- **Hardware:** 2 CPUs, 2GB RAM, 20GB HD
- **Rede:** 1 IP fixo e 1 DNS
- **Portas:** 6443, 2379-2380, 10250-10255 e 30000-32767 (se for utilizar o serviço NodePort)
- **Usuário:** root
- **Swap:** desabilitado

### Criando as maquinas na AWS

- **Nome:** k8s
- **Nº de instâncias:** 3
- **OS:** Ubuntu Server 22.04 LTS
- **Tipo de instância:** t2.medium
- **Key pair:** k8s
- **Security group:** k8s
- **Network:** k8s

#### Configurações do Security Group

- **Nome:** k8s
- **Descrição:** k8s
- **Inbound rules:**
  - **SSH:** 22
  - **Custom TCP:** 6443 (Somente para as maquinas do mesmo security group)
  - **Custom TCP:** 2379-2380
  - **Custom TCP:** 10250-10255 (Somente para as maquinas do mesmo security group)
  - **Custom TCP:** 30000-32767 (se for utilizar o serviço NodePort)
  - **Custom TCP:** 6783 (Somente para as maquinas do mesmo security group)
  - **Custom UDP:** 6783-6784 (Somente para as maquinas do mesmo security group)

### Configurando as maquinas

Configurando o hostname

```bash
sudo hostnamectl set-hostname k8s-control-plane
sudo hostnamectl set-hostname k8s-worker-1
sudo hostnamectl set-hostname k8s-worker-2
```

Desabilitando o swap

```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

Habilitando os módulos do kernel

```bash
sudo vim /etc/modules-load.d/k8s.conf
```

```bash
overlay
br_netfilter
```

```bash
sudo modprobe overlay
sudo modprobe br_netfilter
```

Configurando o sysctl

```bash
sudo vim /etc/sysctl.d/k8s.conf
```

```bash
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
```

```bash
sudo sysctl --system
```

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gnupg lsb-release
```

Adicionando a chave do repositório

```bash
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
```

Adicionando o repositório

```bash
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

Instalando o kubeadm, kubelet e kubectl

```bash
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

### Instalando o Container Runtime (Containerd)

Adicionando a chave do repositório

```bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

Adicionando o repositório

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

```bash
sudo apt-get update && sudo apt-get install -y containerd.io
```

Criando o arquivo de configuração do containerd

```bash
sudo containerd config default | sudo tee /etc/containerd/config.toml
```

```bash
sudo sed -i 's/SystemdGroup = false/SystemdGroup = true/g' /etc/containerd/config.toml
```

```bash
sudo systemctl restart containerd
sudo systemctl enable containerd
sudo systemctl enable kubelet
```

Abrindo as portas no firewall

```bash
sudo ufw allow 6443/tcp
sudo ufw allow 6783/tcp
sudo ufw allow 10250:10255/tcp
sudo ufw reload
```

### Iniciando o cluster

```bash
sudo kubeadm init --pod-network-cidr=10.10.0.0/16 --apiserver-advertise-address={IP da maquina}

```

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

```bash
kubectl get nodes
```

### Iniciando os nós workers

```bash
kubeadm token create --print-join-command
```

```bash
sudo kubeadm join {IP da maquina}:6443 --token {token} --discovery-token-ca-cert-hash {hash}
```

### Instalando o Weave Net (CNI)

```bash
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
```

### Visualizando mais informações do cluster

```bash
kubectl get nodes -o wide
```

```bash
kubectl describe node {nome do node}
```