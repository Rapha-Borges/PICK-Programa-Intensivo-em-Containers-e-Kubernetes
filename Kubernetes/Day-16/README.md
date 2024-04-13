# Network Policies e EKS

## Network Policies

Network Policies são recursos do Kubernetes que permitem controlar o tráfego de rede entre os Pods. Com as Network Policies, é possível definir regras de comunicação entre os Pods, permitindo ou bloqueando o tráfego de acordo com as regras definidas.

Importante lembrar que nem todos os plugins de rede (CNI) suportam Network Policies. No EKS, por exemplo, é necessário utilizar o AWS VPC CNI para que as Network Policies funcionem.

### Criando um EKS Cluster com Network Policies

Para criar um EKS Cluster com suporte a Network Policies, é necessário utilizar o AWS VPC CNI. Para isso, ao criar o Cluster, é necessário adicionar a configuração `--pod-network-cidr` com o CIDR que será utilizado para os Pods.

Antes de criar o Cluster, é necessário instalar e configurar o `[eksctl](https://eksctl.io/)` e o `[AWS CLI](https://aws.amazon.com/pt/cli/)`. 

Por exemplo, para criar um Cluster EKS com o AWS VPC CNI e suporte a Network Policies, execute o comando:

```bash
eksctl create cluster --name eks-cluster --version 1.28 --region us-east-1 --nodegroup-name eks-cluster-nodegroup --node-type t3.medium --nodes 2 --nodes-min 1 --nodes-max 3 --managed