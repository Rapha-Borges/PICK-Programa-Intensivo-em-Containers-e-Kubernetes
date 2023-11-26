# ReplicaSet, DaemonSet e Probes

### O que é um ReplicaSet?

O ReplicaSet é um controlador que garante que um determinado número de pods idênticos estejam em execução em um determinado momento. O ReplicaSet mantém um conjunto de pods idênticos em execução em todos os momentos. Se um pod falhar, o ReplicaSet o substituirá por um novo pod idêntico. O ReplicaSet também pode ser usado para aumentar ou diminuir o número de pods em execução para um aplicativo.

### Criando um ReplicaSet através de um manifesto

##### Não recomendado, pois o Deployment é mais completo e possui mais recursos.

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  labels:
    app: nginx-app
  name: nginx-replicaset
spec:
  replicas: 3
    selector:
      matchLabels:
        app: nginx-app
    template:
      metadata:
      labels:
          app: nginx-app
      spec:
      containers:
      - image: nginx:1.19.1
        name: nginx
        resources:
          limits:
              cpu: 0.5
              memory: 256Mi
          requests:
              cpu: 0.3
              memory: 128Mi
```
    
```bash
kubectl apply -f nginx-replicaset.yaml
```

### O que é um DaemonSet?

O DaemonSet é um objeto que garante que todos os nós do cluster executem uma réplica de um Pod, ou seja, ele garante que todos os nós do cluster executem uma cópia de um Pod. Se um novo nó for adicionado ao cluster, o DaemonSet criará um pod nesse nó. Se um nó for removido do cluster, o DaemonSet excluirá o pod desse nó.

### Criando um DaemonSet através de um manifesto

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  labels:
    app: node-exporter
  name: node-exporter
spec:
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      hostNetwork: true
      containers:
      - image: prom/node-exporter:lastest
        name: node-exporter
        ports:
        - containerPort: 9100
          hostPort: 9100
        volumeMounts:
        - name: proc
          mountPath: /host/proc
          readOnly: true
        - name: sys
          mountPath: /host/sys
          readOnly: true
      volumes:
      - name: proc
      hostPath:
        path: /proc
      - name: sys
      hostPath:
        path: /sys
```

```bash
kubectl apply -f nginx-daemonset.yaml
```

### O que são Probes?

Probes são mecanismos que permitem que o Kubernetes verifique a saúde de um Pod. Se um Pod falhar, o Kubernetes pode reiniciá-lo ou substituí-lo por um novo Pod. Existem três tipos de probes:

- **Liveness Probe**: verifica se o contêiner está em execução e reinicia o contêiner se ele não estiver em execução. 

- **Readiness Probe**: verifica se o contêiner está pronto para receber tráfego. Se o contêiner não estiver pronto, o Kubernetes não enviará tráfego para o contêiner.

- **Startup Probe**: verifica se o aplicativo dentro do contêiner foi iniciado. Se o aplicativo não for iniciado, o Kubernetes reiniciará o contêiner.