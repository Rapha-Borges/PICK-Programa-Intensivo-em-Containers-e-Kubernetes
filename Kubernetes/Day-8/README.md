# Secrets e ConfigMaps

### O que são Secrets?

Secrets são objetos do Kubernetes que armazenam dados sensíveis, como senhas, tokens, chaves SSH, etc. Eles são armazenados no cluster Kubernetes e podem ser acessados pelos pods. Eles são armazenados em base64, mas não são criptografados.

### Tipos de Secrets

- **Opaque**: armazena dados codificados em base64. É o tipo padrão de secret.
- **Docker Registry (dockercfg)**: armazena credenciais para acessar um registro de imagens Docker.
- **Dockerconfigjson**: armazena credenciais para acessar um registro de imagens Docker. É uma versão mais atualizada do dockercfg.
- **BasicAuth**: armazena credenciais de autenticação básica.
- **SSHAuth**: armazena chaves SSH.
- **TLS**: armazena certificados TLS.
- **Bootstrap Token**: armazena tokens de inicialização de cluster.

### Criando um Secret do tipo Opaque

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: giropops-secret
type: Opaque
data:
  username: bnVkZXJ2YWw=
  password: Z2lyb3BvcHM=
```

```bash
kubectl apply -f secret.yaml
```

### Utilizando um Secret como variável de ambiente

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: giropops-pod
spec:
  containers:
    - name: giropops-container
      image: nginx
      env:
        - name: USERNAME
          valueFrom:
            secretKeyRef:
              name: giropops-secret
              key: username
        - name: PASSWORD
          valueFrom:
            secretKeyRef:
              name: giropops-secret
              key: password
```

```bash
kubectl apply -f pod.yaml
```

### Criando um Secret do tipo Docker Registry (dockercfg)

Faça o login no Docker Hub:

```bash
docker login
```

Pegue o conteúdo do arquivo ~/.docker/config.json em base64:

```bash
cat ~/.docker/config.json | base64
```

Crie o arquivo secret.yaml:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: dockerhub-secret
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: |
    {CONTEÚDO DO ARQUIVO ~/.docker/config.json EM BASE64}
```

```bash
kubectl apply -f secret.yaml
```

### Utilizando um Secret do tipo Docker Registry (dockercfg) como variável de ambiente

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: giropops-pod
spec:
  containers:
    - name: giropops-container
      image: {DOCKERHUB_USERNAME}/{IMAGE_NAME}
  imagePullSecrets:
  - name: dockerhub-secret
```

```bash
kubectl apply -f pod.yaml
```

### Criando um Secret do tipo TLS

Crie um certificado e uma chave privada:

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout chave-privada.key -out certificado.crt
```

Pegue o conteúdo dos arquivos certificado.crt e chave-privada.key em base64:

```bash
cat certificado.crt | base64
cat chave-privada.key | base64
```

Crie o arquivo secret.yaml:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: tls-secret
type: kubernetes.io/tls
data:
  tls.crt: |
    {CONTEÚDO DO ARQUIVO certificado.crt EM BASE64}
  tls.key: |
    {CONTEÚDO DO ARQUIVO chave-privada.key EM BASE64}
```

```bash
kubectl apply -f secret.yaml
```

O Secret também pode ser criado com o comando kubectl:

```bash
kubectl create secret tls meu-servico-web-tls-secret --cert=certificado.crt --key=chave-privada.key
```

### O que são ConfigMaps?

ConfigMaps são objetos do Kubernetes que armazenam dados não sensíveis, como variáveis de ambiente, arquivos de configuração, etc. 

### Tipos de ConfigMaps

- **Opaque**: armazena dados codificados em base64. É o tipo padrão de ConfigMap.
- **ConfigMap**: armazena dados não sensíveis.

### Criando um ConfigMap para utilizar com o Nginx

```yaml
apiVersion: v1
data:
  nginx.conf: |
    events { }
    http {
      server {
        listen 80;
        listen 443 ssl;
        
        ssl_certificate /etc/nginx/tls/certificado.crt;
        ssl_certificate_key /etc/nginx/tls/chave-privada.key;

        location / {
        return 200 'Hello, World!';
        add_header Content-Type text/plain;
        }
      }
    }
kind: ConfigMap
metadata:
  name: nginx-config
  namespace: default
```

```bash
kubectl apply -f configmap.yaml
```

O ConfigMap também pode ser criado com o comando kubectl, desde que exista o arquivo nginx.conf:

```bash
kubectl create configmap nginx-config --from-file=nginx.conf
```

### Utilizando um ConfigMap como volume

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: giropops-pod
  labels:  
    app: nginx
spec:
  containers:
    - name: giropops-container
      image: nginx
      ports:
        - containerPort: 80
        - containerPort: 443
      volumeMounts:
        - name: nginx-config-volume
          mountPath: /etc/nginx/nginx.conf
          subPath: nginx.conf
        - name: nginx-tls
          mountPath: /etc/nginx/tls
  volumes:
  - name: nginx-confip-volume
    configMap:
      name: nginx-config
  - name: nginx-tls
    secret:
      secretName: meu-servico-web-tls-secret
      items:
        - key: tls.crt
          path: certificado.crt
        - key: tls.key
          path: chave-privada.key
```

```bash
kubectl apply -f pod.yaml
```