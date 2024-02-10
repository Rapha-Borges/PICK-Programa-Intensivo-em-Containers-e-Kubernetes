# Horizontal Pod Autoscaler(HPA), Metrics Server e Locust

O HPA é um controlador que permite que o número de pods em um deployment, replicaset ou statefulset seja aumentado ou diminuído automaticamente com base em métricas de uso de CPU ou personalizadas fornecidas pelo usuário.

### Metrics Server

O [Metrics Server](https://kubernetes.io/docs/tasks/debug/debug-cluster/resource-metrics-pipeline/#metrics-server) é um agregador de métricas que coleta métricas de uso de CPU e memória de cada nó e pod do cluster Kubernetes, e disponibiliza essas métricas para o HPA.

### Instalando o Metrics Server

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

### Utilizando o TOP para visualizar as métricas

```bash
kubectl top nodes
kubectl top pods
```

### Criando o HPA

```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: locust-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

### O que é o Locust?

O [Locust](https://locust.io/) é uma ferramenta de teste de carga de código aberto. Ele permite que você escreva cenários de teste em Python para simular o comportamento de usuários reais e medir o desempenho do sistema sob carga.

### Instalando o Locust via Deploy

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: locust-giropops
  name: locust-giropops
spec:
  replicas: 1
  selector:
    matchLabels:
      app: locust-giropops
  template:
    metadata:
      labels:
        app: locust-giropops
    spec:
      containers:
      - image: linuxtips/locust-giropops:1.0
        name: locust-giropops
        env:
          - name:  LOCUST_LOCUSTFILE
            value: "/usr/src/app/scripts/locustfile.py"
        ports:
        - containerPort: 8089
        imagePullPolicy: Always
        volumeMounts:
        - name: locust-scripts
          mountPath: /usr/src/app/scripts
      volumes:
      - name: locust-scripts
        configMap:
          name: locust-scripts
          optional: true
```

### Criando o Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: locust-giropops
spec:
  selector:
    app: locust-giropops
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8089
  type: ClusterIP
```

### Criando o Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: locust-giropops
spec:
  rules:
  - host: locust.r11s.com.br
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: locust-giropops
            port:
              number: 80
```

### Criando o ConfigMap

```yaml
apiVersion: v1
data:
  locustfile.py: |-
    from locust import HttpUser, task, between

    class Giropops(HttpUser):
        wait_time = between(1, 2)

        @task(1)
        def listar_senha(self):
            self.client.get("/")
kind: ConfigMap
metadata:
  name: locust-scripts
```

Agora, vamos criar os recursos no Kubernetes:

```bash
kubectl apply -f manifests/
```

Acesse o endereço `http://locust.r11s.com.br` e faça o teste de carga.

### Autoscaling com base na utilização de Memória

Além de utilizar a métrica de CPU, podemos utilizar a métrica de memória para realizar o autoscaling.

```yaml
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: giropops-hpa
  namespace: giropops-senhas
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: giropops-deployment
  minReplicas: 1
  maxReplicas: 3
  metrics:
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 50
```

### Autoscaling com base na utilização de de recursos dentro do container

Também é possível realizar o autoscaling com base na utilização de recursos do container, e não apenas do pod.

```yaml
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: giropops-hpa
  namespace: giropops-senhas
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: giropops-deployment
  minReplicas: 1
  maxReplicas: 3
  metrics:
    - type: ContainerResource
      containerResource:
        name: cpu
        container: giropops-senhas
        target:
          type: Utilization
          averageUtilization: 50
```

### Configurações avançadas de ScaleUp e ScaleDown

O HPA possui configurações padrão para o tempo de espera para realizar o ScaleUp e ScaleDown. No entanto, é possível alterar essas configurações.

```yaml
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: giropops-hpa
  namespace: giropops-senhas
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: giropops-deployment
  minReplicas: 1
  maxReplicas: 3
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
        - type: Pods
          value: 100
          periodSeconds: 15
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
        - type: Pods
          value: 100
          periodSeconds: 15
```

No exemplo acima, estamos configurando o tempo de espera para realizar o ScaleUp imediatamente, e o ScaleDown após 300 segundos.