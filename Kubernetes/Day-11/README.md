# Ingress e Contexts no Kubernetes

### O que é Ingress?

O Ingress é um recurso do Kubernetes que gerencia o acesso externo aos serviços dentro de um cluster. Ele funciona como uma camada de roteamento HTTP/HTTPS, permitindo a definição de regras para direcionar o tráfego externo para diferentes serviços back-end.

O Ingress pode fornecer balanceamento de carga, terminação SSL e hospedagem virtual baseada em nome.

### Configurando o Kind para usar o Ingress

Por padrão, o Kind não vem com o Ingress habilitado. Para habilitá-lo, precisamos criar um arquivo de configuração do Kind com o seguinte conteúdo:

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
```

O arquivo de configuração acima habilita o Ingress e mapeia as portas 80 e 443 do host para as portas 80 e 443 do container.

Para criar o cluster com o arquivo de configuração acima, execute o seguinte comando:

```bash
kind create cluster --config kind-ingress.yaml
```

### Instalando o [Ingress NGINX Controller no Kind](https://kind.sigs.k8s.io/docs/user/ingress/#ingress-nginx)

Para instalar o Ingress NGINX Controller, execute o seguinte comando:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

Aguarde a instalação do Ingress NGINX Controller:

```bash
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=90s
```

Para instalar o Ingress NGINX Controller em um cluster que não seja o Kind, você pode seguir as instruções ds [documentação oficial](https://kubernetes.github.io/ingress-nginx/deploy/).

### Criando um Ingress

Para criar um Ingress, precisamos ter um serviço rodando no cluster. Para isso, vamos criar um deployment e um serviço do Giropops Senha + Redis com os seguintes comandos (execute os comandos no diretório `kubernetes/Day-11/manisfests` pois os arquivos de configuração estão nesse diretório):

```bash
kubectl apply -f app-deployment.yaml
kubectl apply -f app-service.yaml
kubectl apply -f redis-deployment.yaml
kubectl apply -f redis-service.yaml
```

Agora, vamos criar o Ingress propriamente dito. Para isso, vamos utilizar o arquivo `ingress.yaml`:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: giropops-senhas-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: giropops-senhas
            port:
              number: 5000
```

```bash
kubectl apply -f ingress.yaml
```

### Criando multiplos Ingress no mesmo Ingress Controller

Para criar multiplos Ingress no mesmo Ingress Controller, precisamos utilizar o recurso `host` do Ingress.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: giropops.nginx.io
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx
            port:
              number: 80
```

```bash
kubectl apply -f ingress-nginx.yaml
```

### Contexts no Kubernetes

Contexts são uma forma de organizar e acessar múltiplos clusters Kubernetes. Eles são úteis quando você precisa acessar mais de um cluster Kubernetes, seja localmente ou em um ambiente de produção.

Para listar os contextos disponíveis, execute o seguinte comando:

```bash
kubectl config get-contexts
```

Para mudar de contexto, execute o seguinte comando:

```bash
kubectl config use-context <context-name>
```

### Instalando o Ingress NGINX Controller no EKS

Para instalar o Ingress NGINX Controller no EKS, precisamos executar os seguintes comandos. Não esqueça de mudar o contexto para o cluster EKS antes de executar os comandos:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.44.0/deploy/static/provider/aws/deploy.yaml
```

### Criando um Ingress no EKS

Para criar um Ingress no EKS, precisamos criar um arquivo de configuração do Ingress ingress-eks.yaml com o seguinte conteúdo:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: giropops-senhas-eks-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: {DOMAIN_NAME}
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: giropops-senhas
            port:
              number: 5000
```

Para que o Ingress funcione corretamente, precisamos substituir o valor `{DOMAIN_NAME}` por um domínio válido. Inserindo uma entrada do tipo 'CNAME' no seu provedor de DNS, apontando para o endereço do Load Balancer do Ingress. 