# Desafio - Parte 1

Vá para o diretório /root/manifests:

```bash
cd /root/manifests
```

Crie um certificado e uma chave privada:

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout nginx.key -out nginx.crt
```

Crie o secret no kubernetes:

```bash
kubectl create secret tls nginx-secret --cert=nginx.crt --key=nginx.key
```

Crie o configmap no kubernetes:

```bash
vim nginx.conf
```
  
```bash
kubectl create configmap nginx-config --from-file=nginx.conf
```

Crie o pod no kubernetes:

```bash
vim nginx-https-pod.yaml
```

```bash
kubectl apply -f nginx-https-pod.yaml
```

Crie o service no kubernetes:

```bash
vim nginx-service.yaml
```

```bash
kubectl apply -f nginx-service.yaml
```