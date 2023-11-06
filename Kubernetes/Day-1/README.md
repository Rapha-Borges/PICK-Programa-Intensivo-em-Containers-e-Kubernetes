# Introdução ao Kubernetes

### O que é um Container?

Um container é uma unidade de software que empacota código e todas as suas dependências para que o aplicativo seja executado de forma rápida e confiável de um ambiente de computação para outro. Um programa de computador, seja um serviço, um processo ou uma ferramenta, é executado em um ou mais containers. Um container é executado diretamente no kernel do host e não requer um sistema operacional de inicialização completo. Isso significa que ele ocupa menos espaço do que uma máquina virtual (VM) e é mais rápido de inicializar. De uma forma objetiva, um container é sobre isolamento de recursos, sendo os recursos CPU, memória, I/O, rede, etc. Isso é feito usando namespaces do kernel e grupos de controle (cgroups).

### O que é um Container Engine?

Um container engine é um software responsável por executar os containers. Ele é responsável por fazer o download das imagens, executar os containers, gerenciar os volumes, redes, etc. O container engine mais conhecido é o Docker.

### O que é um Container Runtime?

Podem ser classificados como container runtime de alto nível e de baixo nível. O container runtime de alto nível é responsável por fazer a comunicação entre o container e o container engine, o mais conhecido é o containerd. Já o container runtime de baixo nível é responsável por fazer a comunicação entre o container e o kernel do host, o mais conhecido é o runc.

### O que é a OCI

A OCI é uma organização sem fins lucrativos que tem como objetivo criar padrões abertos para o ecossistema de containers. Ela é responsável por criar especificações para o container runtime, container image, container distribution, etc.

### O que é o Kubernetes?

O Kubernetes é um orquestrador de containers open source que automatiza a implantação, o dimensionamento e o gerenciamento de aplicativos em containers. Ele foi originalmente projetado pelo Google e agora é mantido pela Cloud Native Computing Foundation (CNCF). Ele funciona com uma variedade de ferramentas de contêineres, incluindo Docker. Muitas empresas usam o Kubernetes para automatizar o dimensionamento, o gerenciamento e a implantação de aplicativos em contêineres.

### O que são os Workers e Control Plane?

O Kubernetes é composto por dois tipos de máquinas (nodes), os Workers e o Control Plane. Os Workers são responsáveis por executar os containers, já o Control Plane é responsável por gerenciar os Workers. O Control Plane é composto por vários componentes, como o API Server, Controller Manager, Scheduler, etc e os Workers são compostos por um componente chamado Kubelet.

### Quais são os componentes do Control Plane?

- etcd: É o componente responsável por armazenar o estado do cluster. Ele é um "banco de dados" chave-valor distribuído e consistente. Ele é usado para armazenar todos os dados do Kubernetes, como configurações, estado, etc.

- API Server: É o componente responsável por fazer a comunicação entre o Kubernetes e o usuário e o único que se comunica diretamente com o etcd. Ele é responsável por fazer a validação e a execução das requisições feitas pelo usuário.

- Scheduler: É o componente responsável por agendar os pods nos Workers. É ele que escolhe o Worker que irá executar o pod e por manter a quantidade desejada de réplicas de um pod.

- Controller Manager: É o componente responsável por executar os controllers do Kubernetes. Os controllers são responsáveis por monitorar o estado do cluster e fazer as alterações necessárias para que o estado desejado seja alcançado. Alguns exemplos de controllers são o ReplicaSet, Deployment, StatefulSet, DaemonSet, etc. Temos uma variação dele que é o Cloud Controller Manager, que é responsável por fazer a comunicação entre o Kubernetes e o provedor de nuvem.

### Quais são os componentes do Worker?

- Kubelet: É o componente responsável por fazer a comunicação entre o Kubernetes e o Worker. Ele é responsável por executar os containers e por monitorar o estado dos pods.

- Kube-proxy: É o componente responsável por fazer a comunicação entre os pods e o mundo externo. Ele é responsável por fazer o balanceamento de carga entre os pods e por fazer o roteamento de rede.

### Portas utilizadas pelos componentes Kubernetes

- API Server: 6443

- etcd: 2379 - 2380

- Kubelet: 10250

- Scheduler: 10251

- Controller Manager: 10252

- NodePort: 30000 - 32767

- Kube-proxy: 10256

- Weave Net: 6783 (tcp) - 6784 (udp)

## Pods, replicaSet, Deployment e Service

### O que é um Pod?

É a menor unidade do Kubernetes. Composto por um ou mais containers, volumes, variáveis de ambiente, etc. Sendo um ambiente isolado, ou seja, cada pod tem seu próprio IP, hostname, etc. Um pod é efêmero, ou seja, ele pode ser criado, deletado, recriado, etc. 

### O que é um ReplicaSet?

É o responsável por monitorar o estado dos pods e por manter a quantidade desejada de réplicas. Ele é o responsável por criar, deletar, atualizar, etc, os pods.

### O que é um Deployment?

É o responsável por monitorar o estado dos ReplicaSets e por manter a quantidade desejada de réplicas. Ele é o responsável por criar, deletar, atualizar, etc, os ReplicaSets.

### O que é um Service?

É o responsável por fazer o balanceamento de carga entre os pods e por fazer o roteamento de rede. Ele é o responsável por fazer a comunicação entre os pods e o mundo externo.

## Entendendo e Instalando o Kubectl

### O que é o Kubectl?

O Kubectl é a ferramenta de linha de comando do Kubernetes. Ele é responsável por fazer a comunicação entre o usuário e o Kubernetes e por executar os comandos.

### Instalando o Kubectl

Para instalar o Kubectl, basta executar o seguinte comando:

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

```bash
kubectl version --client
```

Para adicionar o auto-complete você pode seguir as orientações do [site oficial](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#enable-shell-autocompletion).

## Criando um Cluster Kubernetes com Kind

### O que é o Kind?

O Kind (Kubernetes in Docker) é uma ferramenta que permite criar um cluster Kubernetes usando containers Docker como nodes. Ele é muito utilizado para testes e desenvolvimento.

### Instalando o Kind

Para instalar o Kind, basta executar o seguinte comando:

```bash
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

### Criando um Cluster Kubernetes com Kind

```bash
kind create cluster
```