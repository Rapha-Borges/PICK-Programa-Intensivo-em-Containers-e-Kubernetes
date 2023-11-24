# Pods

### O que é um Pod?

Um Pod é a menor unidade que pode ser criada e gerenciada pelo Kubernetes. Ele é composto por um ou mais containers, que compartilham recursos como rede e volumes.

### Kubectl get pods e kubectl describe pods

O comando `kubectl get pods` é utilizado para listar os pods de um cluster. Já o comando `kubectl describe pods` é utilizado para obter informações detalhadas de um pod.

```bash
kubectl get pods
```

```bash
kubectl describe pods
```

### Kubectl Attach e Kubectl Exec

O comando `kubectl attach` é utilizado para conectar o terminal do usuário a um pod em execução. Já o comando `kubectl exec` é utilizado para executar um comando em um pod em execução.

```bash
kubectl attach <pod-name> -c <container-name>
```

```bash
kubectl exec <pod-name> -- <command>
```

### Criando no primeiro pod Multi-Container

```bash
k run girus --image alpine --dry-run=client -o yaml > pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: girus
    service: webserver
  name: girus
spec:
    containers:
    - image: nginx
      name: nginx
      resources: {}
    - image: busybox
      name: busybox
      args:
      - sleep
      - "600"
      resources: {}
    dnsPolicy: ClusterFirst
    restartPolicy: Always
```

```bash
k apply -f pod.yaml
```

```bash
k logs girus -c nginx
```

### Limitando recursos de um Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: giropops
  name: giropops
spec:
    containers:
    - image: ubuntu
      name: ubuntu
      args:
      - sleep
      - "1800"
      resources:
        limits:
          cpu: "0.5"
          memory: "128Mi"
        requests:
          cpu: "0.3"
          memory: "64Mi"
    dnsPolicy: ClusterFirst
    restartPolicy: Always
```

### Configurando volume EmptyDir

O volume EmptyDir é um volume temporário que é criado quando um pod é criado e é deletado quando o pod é deletado. Ele é utilizado para compartilhar arquivos entre os containers de um pod.

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: giropops
  name: giropops
spec:
    containers:
    - image: nginx
      name: webserver
      volumeMounts:
      - mountPath: /giropops
        name: primeiro-emptydir
      resources:
        limits:
          cpu: "1"
          memory: "128Mi"
        requests:
          cpu: "0.5"
          memory: "64Mi"
    volumes:
    - name: primeiro-emptydir
      emptyDir:
        sizeLimit: "256Mi"
    dnsPolicy: ClusterFirst
    restartPolicy: Always
```