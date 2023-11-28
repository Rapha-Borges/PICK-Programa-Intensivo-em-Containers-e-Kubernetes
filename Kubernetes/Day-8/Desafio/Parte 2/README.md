# Desafio - Parte 2

Parece que temos um problema com o Secret. Para resolver, precisamos criar um novo Secret com o certificado e a chave privada do Nginx. E então, aplicar o Pod novamente.

```bash
k delete -f nginx-https-pod.yaml
k delete secret nginx-secret
```

```bash
kubectl create secret tls nginx-secret --cert=nginx.crt --key=nginx.key
k apply -f nginx-https-pod.yaml
```