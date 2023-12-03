# Cert-Manager, Annotations e Labels

### O que são Annotations?

Annotations são metadados que podem ser adicionados a um objeto Kubernetes. Eles podem ser usados ​​por controladores e ferramentas Kubernetes para armazenar informações arbitrariamente estruturadas. 

#### Comandos úteis para trabalhar com Annotations:

```bash
kubectl annotate <tipo de objeto> <nome do objeto> <chave>=<valor> # Adiciona uma annotation
kubectl annotate <tipo de objeto> <nome do objeto> <chave>- # Remove uma annotation
kubectl annotate <tipo de objeto> <nome do objeto> <chave>- --overwrite # Remove uma annotation e sobrescreve o valor
kubectl annotate <tipo de objeto> <nome do objeto> <chave> # Exibe o valor de uma annotation
kubectl annotate <tipo de objeto> <nome do objeto> <chave> --list # Exibe todas as annotations de um objeto
kubectl annotate <tipo de objeto> <nome do objeto> <chave>- --all # Remove todas as annotations de um objeto
kubectl annotate <tipo de objeto> <nome do objeto> <chave>- --all --overwrite # Remove todas as annotations de um objeto e sobrescreve o valor
```

### O que são Labels?

Labels são pares de chave/valor que podem ser adicionados a um objeto Kubernetes. Eles podem ser usados ​​para selecionar e filtrar objetos Kubernetes.

#### Comandos úteis para trabalhar com Labels:

```bash
kubectl label <tipo de objeto> <nome do objeto> <chave>=<valor> # Adiciona uma label
kubectl label <tipo de objeto> <nome do objeto> <chave>- # Remove uma label
kubectl label <tipo de objeto> <nome do objeto> <chave>- --overwrite # Remove uma label e sobrescreve o valor
kubectl label <tipo de objeto> <nome do objeto> <chave> # Exibe o valor de uma label
kubectl label <tipo de objeto> <nome do objeto> <chave> --list # Exibe todas as labels de um objeto
kubectl label <tipo de objeto> <nome do objeto> <chave>- --all # Remove todas as labels de um objeto
kubectl label <tipo de objeto> <nome do objeto> <chave>- --all --overwrite # Remove todas as labels de um objeto e sobrescreve o valor
kubectl get <tipo de objeto> -L <chave>=<valor> # Lista todos os objetos que possuem uma label
```


### O que é o [Cert-Manager](https://cert-manager.io/)?

O Cert-Manager é um controlador do Kubernetes que automatiza a emissão e renovação de certificados TLS (Transport Layer Security) com base em recursos do Kubernetes.

### Instalando o Cert-Manager

```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.2/cert-manager.yaml
```

### Criando um Issuer

```bash
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    email: user@example.com
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: letsencrypt-staging
    solvers:
    - http01:
        ingress:
          ingressClassName: nginx
```

### Criando o ClusterIssuer

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    email: user@example.com
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          ingressClassName: nginx
```

### Configurando o Ingress para utilizar o Cert-Manager e ter HTTPS

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: giropops-senhas-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  ingressClassName: nginx
  tls:
  - hosts:
      - giropops.containers.expert
    secretName: giropops-containers-expert-tls
  rules:
  - host: giropops.containers.expert
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
### Adicionando Autenticação no Ingress

Para adicionar autenticação no ingress, vamos começar adicionando 2 annotations no ingress:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: giropops-senhas-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/auth-type: basic
    nginx.ingress.kubernetes.io/auth-secret: basic-auth
    nginx.ingress.kubernetes.io/auth-realm: 'Authentication Required'
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  ingressClassName: nginx
  tls:
  - hosts:
      - giropops.containers.expert
    secretName: giropops-containers-expert-tls
  rules:
  - host: giropops.containers.expert
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

Vamos utilizar o [htpasswd](https://httpd.apache.org/docs/2.4/programs/htpasswd.html) para criar o arquivo de autenticação:

```bash
sudo apt-get update && sudo apt-get install apache2-utils -y
```

```bash
htpasswd -c auth raphael
```

```bash
kubectl create secret generic basic-auth --from-file=auth
```

### O que são Affinity Cookies?

Affinity Cookies são cookies que são adicionados ao navegador do usuário para que ele seja redirecionado sempre para o mesmo pod. Isso é muito útil para aplicações que não são stateless e precisam manter o estado de uma sessão.

### Configurando Affinity Cookies no Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: giropops-senhas-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/auth-type: basic
    nginx.ingress.kubernetes.io/auth-secret: basic-auth
    nginx.ingress.kubernetes.io/auth-realm: 'Authentication Required'
    nginx.ingress.kubernetes.io/affinity: "cookie"
    nginx.ingress.kubernetes.io/session-cookie-name: "giropops-cookie"
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  ingressClassName: nginx
  tls:
  - hosts:
      - giropops.containers.expert
    secretName: giropops-containers-expert-tls
  rules:
  - host: giropops.containers.expert
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

### O que é o Upstream Hash?

O Upstream Hash é um método de balanceamento de carga que utiliza o hash do IP do cliente para determinar para qual pod o tráfego será enviado. Isso é muito útil para aplicações que não são stateless e precisam manter o estado de uma sessão.

### Configurando o Upstream Hash no Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: giropops-senhas-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/auth-type: basic
    nginx.ingress.kubernetes.io/auth-secret: basic-auth
    nginx.ingress.kubernetes.io/auth-realm: 'Authentication Required'
    nginx.ingress.kubernetes.io/upstream-hash-by: "$request_uri"
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  ingressClassName: nginx
  tls:
  - hosts:
      - giropops.containers.expert
    secretName: giropops-containers-expert-tls
  rules:
  - host: giropops.containers.expert
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

### O que é Canary Deployment com o Ingress?

Canary Deployment é uma técnica de deploy que consiste em enviar uma pequena porcentagem do tráfego para uma nova versão da aplicação para testar se ela está funcionando corretamente antes de enviar todo o tráfego para a nova versão.

A regra do Canary Deployment deve ser adicionada no Ingress do novo serviço que está sendo testado.

É importante lembrar que o Canary Deployment deve ser utilizado em conjunto com o Affinity Cookie para que o usuário seja redirecionado sempre para o mesmo pod.

### Configurando Canary Deployment com o Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: giropops-senhas-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-weight: "10"  # 10% do tráfego será enviado para o novo serviço
    nginx.ingress.kubernetes.io/affinity: "cookie"
    nginx.ingress.kubernetes.io/session-cookie-name: "giropops-cookie"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
      - giropops.containers.expert
    secretName: giropops-containers-expert-tls
  rules:
  - host: giropops.containers.expert
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

### Limitando o número de requisições com o Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: giropops-senhas-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/auth-type: basic
    nginx.ingress.kubernetes.io/auth-secret: basic-auth
    nginx.ingress.kubernetes.io/auth-realm: 'Authentication Required'
    nginx.ingress.kubernetes.io/upstream-hash-by: "$request_uri"
    nginx.ingress.kubernetes.io/limit-rps: "10" # 10 requisições por segundo
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  ingressClassName: nginx
  tls:
  - hosts:
      - giropops.containers.expert
    secretName: giropops-containers-expert-tls
  rules:
  - host: giropops.containers.expert
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