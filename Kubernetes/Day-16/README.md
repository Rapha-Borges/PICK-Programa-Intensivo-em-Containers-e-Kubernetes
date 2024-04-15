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
```

### Instalando e Habilitando o Addon do AWS VPC CNI

Após criar o Cluster, é necessário instalar o Addon do AWS VPC CNI. Para isso, execute o comando:

```bash
eksctl create addon --name vpc-cni --version v1.16.0-eksbuild.1 --cluster eks-cluster --force
```

Abra a página do Cluster no Console do EKS, na aba ['Add-ons'](https://console.aws.amazon.com/eks/home#/clusters/eks-cluster/addons) e acesso o Addon do AWS VPC CNI. Clique em 'Edit', expanda a seção 'Optional Configuration settings', em 'Configuration Values', adicione "enableNetworkPolicy": "true" e clique em 'Save'.

### Instalando o Nginx Ingress Controller, Deployments e Services e o Ingress 

Para instalar o Nginx Ingress Controller, precisamos utilizar a versão compatível com o Kubernetes e o Cloud Provider que estamos utilizando. Para o EKS, vamos utilizar a versão `1.9.5`.

Para instalar o Nginx Ingress Controller, execute o comando:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.9.5/deploy/static/provider/cloud/deploy.yaml
```

Aguarde a instalação do Nginx Ingress Controller.

```bash
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=90s
```

Para criar a aplicação e o Redis, aplique o manifesto [deployment.yaml](./deployment.yaml):

```bash
kubectl create ns giropops
kubectl apply -f deployment.yaml -n giropops
```

Crie o Service conforme o manifesto [service.yaml](./service.yaml):

```bash
kubectl apply -f service.yaml -n giropops
```

Crie o Ingress conforme o manifesto [ingress.yaml](./ingress.yaml), mas não esqueça de incluir um domínio válido:

```bash
kubectl apply -f ingress.yaml -n giropops
```

### Criando Network Policies

Para criar Network Policies, é necessário criar um arquivo YAML com as regras de comunicação entre os Pods. Por exemplo, para permitir que apenas os Pods do Namespace `giropops` possam acessar o Redis, crie o arquivo [permitir-redis-somente-mesmo-ns.yaml](./permitir-redis-somente-mesmo-ns.yaml):

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: redis-allow-same-namespace
  namespace: giropops
spec:
  podSelector: # Seleciona os Pods que serão afetados pela Network Policy
    matchLabels: # Seleciona os Pods que possuem a label app=redis
      app: redis 
  ingress: # Define as regras de ingress
  - from: # Define de onde o tráfego pode vir
    - podSelector: {} # Permite que o tráfego venha de qualquer Pod no Namespace giropops
```

Sempre que usamos o `podSelector: {}`, estamos selecionando todos os Pods que atendem as condições definidas. No exemplo acima, estamos permitindo que qualquer Pod no Namespace `giropops` possa acessar o Redis.

Para aplicar a Network Policy, execute o comando:

```bash
kubectl apply -f permitir-redis-somente-mesmo-ns.yaml
```

Para testar a Network Policy, crie um Pod no Namespace `giropops` e tente acessar o Redis.

```bash
kubectl run -it --rm -n giropops --image redis redis-client -- redis-cli -h redis-service.giropops.svc.cluster.local ping
```

Se a Network Policy estiver funcionando corretamente, o acesso deve ser permitido e você deve receber a resposta `PONG`.

Agora, crie um Pod em outro Namespace e tente acessar o Redis:

```bash
kubectl run -it --rm --image redis redis-client -- redis-cli -h redis-service.giropops.svc.cluster.local ping
```

Se a Network Policy estiver funcionando corretamente, o acesso deve ser bloqueado. 

Podemos também criar uma Network Policy para permitir que apenas os Pods dentro do mesmo Namespace (giropops) possam acessar os Pods no mesmo Namespace. Para isso, crie o arquivo [nao-permitir-nada-externo.yaml](./nao-permitir-nada-externo.yaml):

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-same-ns
  namespace: giropops
spec:
  podSelector: {}
  policyTypes: # Define os tipos de políticas que serão aplicadas
  - Ingress # Define que a política será aplicada no tráfego de entrada
  ingress:
  - from: # Define de onde o tráfego pode vir
    - namespaceSelector: # Seleciona o Namespace por meio de labels
        matchLabels: # Seleciona os Namespaces que possuem a label abaixo
          kubernetes.io/metadata.name: giropops # Seleciona o Namespace giropops
```

Para aplicar a Network Policy, execute o comando:

```bash
kubectl apply -f nao-permitir-nada-externo.yaml
```

Vamos testar a Network Policy rodando o comando `curl` na nossa aplicação:

```bash
kubectl run -it --rm --image curlimages/curl curl-client -- curl giropops-senhas.giropops.svc
```

Mas com isso criamos um "problema", pois o Nginx Ingress Controller não consegue acessar a aplicação. Para resolver isso, precisamos criar uma Network Policy que permita o acesso do Nginx Ingress Controller ao Namespace `giropops`. Para isso, crie o arquivo [permitir-acesso-nginx-ingress.yaml](./permitir-acesso-nginx-ingress.yaml):

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-same-ns-and-ingress-controller
  namespace: giropops
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector: # Seleciona o Namespace que terá acesso por meio de labels
        matchLabels: # Seleciona o Namespace que possui a label abaixo
          kubernetes.io/metadata.name: ingress-nginx # Seleciona o Namespace ingress-nginx
    - namespaceSelector: # Seleciona o Namespace que terá acesso por meio de labels
        matchLabels: # Seleciona o Namespace que possui a label abaixo
          kubernetes.io/metadata.name: giropops # Seleciona o Namespace giropops
```

Para aplicar a Network Policy, execute o comando:

```bash
kubectl apply -f permitir-acesso-nginx-ingress.yaml
```

Agora, o Nginx Ingress Controller consegue acessar a aplicação.

## Bloqueando todo o tráfego de entrada e saída

Para bloquear todo o tráfego de entrada e saída de um Pod, é necessário criar uma Network Policy que bloqueie todo o tráfego de entrada e saída. Para isso, crie o arquivo [bloquear-todo-trafego.yaml](./bloquear-todo-trafego.yaml):

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-traffic
  namespace: giropops
spec:
    podSelector: {}
    policyTypes:
    - Ingress
    - Egress
```


### Tipos de Network Policies