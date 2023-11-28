# StatefulSets e Services

### O que é um StatefulSet?

Os StatefulSets são uma funcionalidade do Kubernetes que gerencia o deployment e o scaling de um conjunto de Pods, fornecendo garantias sobre a ordem de deployment e a singularidade desses Pods.
Diferente dos Deployments e Replicasets que são considerados stateless (sem estado), os StatefulSets são utilizados quando você precisa de mais garantias sobre o deployment e scaling. Eles garantem que os nomes e endereços dos Pods sejam consistentes e estáveis ao longo do tempo.
Os StatefulSets funcionam criando uma série de Pods replicados. Cada réplica é uma instância da mesma aplicação que é criada a partir do mesmo spec, mas pode ser diferenciada por seu índice e hostname.

### O StatefulSet e os Volumes Persistentes

Um aspecto chave dos StatefulSets é a integração com Volumes Persistentes. Quando um Pod é recriado, ele se reconecta ao mesmo Volume Persistente, garantindo a persistência dos dados entre as recriações dos Pods.

### O que são os Headless Services?

Os Headless Services são um tipo de serviço que não possui um IP Cluster. Eles são utilizados para descoberta de Pods através de DNS. Quando um Headless Service é criado, o Kubernetes cria um DNS A Record para cada Pod que faz parte do Service. Isso permite que os Pods sejam acessados diretamente através de seu nome.

### O StatefulSet e os Headless Services

Os StatefulSets e os Headless Services são utilizados em conjunto para garantir que os Pods tenham nomes e endereços consistentes e estáveis ao longo do tempo. Quando um StatefulSet é criado, o Kubernetes cria um Headless Service para o StatefulSet. Esse Headless Service é utilizado para descoberta de Pods através de DNS. 

### Criando um StatefulSet

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nginx
spec:
  serviceName: "nginx"
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
          name: http
        volumeMounts:
        - name: nginx-persistent-storage
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: nginx-persistent-storage
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
          requests:
            storage: 1Gi
```

### Criando um Headless Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: http
  clusterIP: None
  selector:
    app: nginx
```

### O que é um Service?

Um Service é um objeto do Kubernetes que define um conjunto de Pods e uma política de acesso a esses Pods. Os Services permitem que um conjunto de Pods seja acessado através de um único IP e DNS name. Eles também permitem que os Pods sejam escalados e movidos sem que o cliente perceba.

### Tipos de Services

Existem 4 tipos de Services no Kubernetes:

- **ClusterIP**: Expõe o Service em um IP interno no cluster. Este tipo torna o Service acessível apenas dentro do cluster.
- **NodePort**: Expõe o Service na mesma porta de cada Node selecionado no cluster usando NAT. Torna o Service acessível de fora do cluster usando {NodeIP}:{NodePort}.
- **LoadBalancer**: Cria um balanceador de carga externo no ambiente de nuvem atual (se suportado) e atribui um IP fixo, externo ao cluster, ao Service. Tornando o Service acessível de fora do cluster.
- **ExternalName**: O Service é acessível através de um DNS name externo. Esse tipo de Service é utilizado para acessar recursos externos ao cluster.

### Criando um Service do tipo ClusterIP, NodePort, LoadBalancer

Para criarmos um service precisamos de um Deployment ou um StatefulSet rodando. Vamos usar no exemplo o Nginx.

```bash
kubectl create deployment nginx --image=nginx --port=80
kubectl expose deployment nginx --port=80 --type=ClusterIP
kubectl expose deployment nginx --port=80 --type=NodePort
kubectl expose deployment nginx --port=80 --type=LoadBalancer
```

Para termos um serviço do tipo LoadBalancer funcionando, precisamos de um provedor de nuvem que suporte esse tipo de serviço ou podemos usar um Ingress Controller, como por exemplo o MetalLB. Caso contrário, o serviço será criado, mas não terá um IP externo atribuído. 

### Criando um Service do tipo ExternalName

```bash
kubectl create service externalname giropops-db --external-name db.giropops.com.br
```

### Criando um Service expondo outro Service

Podemos criar um Service que exponha outro Service. Isso é útil quando queremos expor um Service que não é acessível de fora do cluster, como por exemplo um Service do tipo ClusterIP. 

```bash
kubectl create deployment nginx --image=nginx --port=80
kubectl expose deployment nginx --port=80 --type=ClusterIP
kubectl expose service nginx --name opa --type=NodePort
```

### O que são os Endpoints?

Os Endpoints são objetos do Kubernetes que armazenam informações sobre os Pods que fazem parte de um Service. Eles são criados automaticamente pelo Kubernetes quando um Service é criado. Os Endpoints são utilizados para descoberta de Pods através de DNS.