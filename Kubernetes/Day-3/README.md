# Deployment

### O que é um Deployment?

Um Deployment é um objeto do Kubernetes que garante que um conjunto de Pods esteja sempre rodando, mesmo que um deles falhe ou seja deletado.

### Criando um Deployment através de um manifesto

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deployment
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
        app: nginx-deployment
  strategy: {}
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers:
      - image: nginx
        name: nginx
        resources:
          limits:
            cpu: 0.5
            memory: 256Mi
          requests:
            cpu: 0.3
            memory: 64Mi
```

```bash
kubectl apply -f nginx-deployment.yaml
Kubectl get deployments
kubectl describe deployment nginx-deployment
```

### Crianção de um Deployment utilizando o dry-run

```bash
kubectl create deployment  --image=nginx --replicas 3 nginx-deployment --dry-run=client -o yaml > nginx-deployment.yaml
kubectl apply -f nginx-deployment.yaml
```

### Como atualizar um Deployment utilizando Strategy

Para efetuar o update de um Deployment você pode editar o arquivo de manifesto e aplicar novamente. O parâmetro strategy é utilizado para definir como será feito o update.

RollingUpdate: É o padrão, ele atualiza os Pods um a um.
 maxSurge: Define o número máximo de Pods que podem ser criados acima do número de Pods desejados.
 maxUnavailable: Define o número máximo de Pods que podem ser indisponibilizados durante o update.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deployment
  name: nginx-deployment
  namespace: giropops
spec:
  replicas: 10
  selector:
    matchLabels:
      app: nginx-deployment
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers:
      - image: nginx:1.15.0
        name: nginx
        resources:
          limits:
            cpu: 0.5
            memory: 256Mi
          requests:
            cpu: 0.3
            memory: 64Mi
```

Recreate: Deleta todos os Pods e cria novos com a nova versão.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deployment
  name: nginx-deployment
  namespace: giropops
spec:
  replicas: 10
  selector:
    matchLabels:
      app: nginx-deployment
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers:
      - image: nginx:1.15.0
        name: nginx
        resources:
          limits:
            cpu: 0.5
            memory: 256Mi
          requests:
            cpu: 0.3
            memory: 64Mi
```

### Fazendo o rollback de um Deployment e conhecendo o comando rollout

Para exibir o histórico de alterações de um Deployment, utilize o comando rollout history.

```bash
kubectl rollout history deployment -n giropops nginx-deployment
```

Revisando uma versão específica

```bash
kubectl rollout history deployment -n giropops nginx-deployment --revision 3 
```

Para voltar para uma versão anterior, utilize o comando rollout undo.

```bash
kubectl rollout undo deployment -n giropops nginx-deployment --to-revision 3
```

Pause, Resume e Restart

```bash
kubectl rollout pause deployment -n giropops nginx-deployment
kubectl rollout resume deployment -n giropops nginx-deployment
kubectl rollout restart deployment -n giropops nginx-deployment
```

#### DICA:
Utilize `kubectl rollout pause` naquele deployment critico em produção que você não quer que seja atualizado sem supervisão. Assim quando for necessário atualizar, basta utilizar o comando `kubectl rollout resume` para voltar a atualizar o deployment.
