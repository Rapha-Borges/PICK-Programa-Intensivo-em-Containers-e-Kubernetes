# Volumes - Kubernetes

### O que são Volumes?

Volumes são diretórios que são montados dentro do Pod. Eles podem ser usados para persistir e/ou compartilhar dados. Quando falamos de Kubernetes, existem dois tipos de volumes: volumes efêmeros e volumes persistentes.

### O que são Volumes Efêmeros?

Volumes efêmeros são volumes que são criados e destruídos junto com o Pod. Por exemplo, se você tem um Pod com dois containers, um container pode escrever dados em um volume efêmero e o outro container pode ler esses dados.

### O que são Volumes Persistentes?

Volumes persistentes são volumes que são criados e destruídos independentemente do Pod. Por exemplo, se você tem um Pod com dois containers, um container pode escrever dados em um volume persistente e o outro container pode ler esses dados. Se o Pod for destruído e recriado, o volume persistente ainda existirá e os dados ainda estarão disponíveis.

### O que é o StorageClass?

O StorageClass é um objeto do Kubernetes que permite que você defina diferentes tipos de armazenamento para diferentes tipos de necessidades. Por exemplo, você pode ter um StorageClass para armazenamento SSD e outro para armazenamento HDD.
O StorageClass também permite que você defina diferentes tipos de provisionadores de armazenamento. Por exemplo, você pode ter um provisionador de armazenamento para armazenamento local e outro para armazenamento em nuvem.

### Criando um StorageClass

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: giropops
provisioner: kubernetes.io/no-provisioner
reclaimPolicy: Retain
volumeBindingMode: WaitForFirstConsumer
```

```bash
kubectl apply -f storageclass.yaml
```

### O que é o PersistentVolume?

O PersistentVolume é um objeto do Kubernetes que representa um volume físico, como um disco rígido, que está disponível para uso. O PersistentVolume é criado pelo administrador do cluster e pode ser usado por qualquer usuário do cluster.

### Criando um PersistentVolume

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  labels:
    storage: lento
  name: meu-pv
spec:
  storageClassName: giropops
    capacity:
      storage: 1Gi
    accessModes:
      - ReadWriteOnce
    persistentVolumeReclaimPolicy: Retain
    hostPath:
      path: /mnt/data
```

```bash
kubectl apply -f persistentvolume.yaml
```

### AccessModes

O AccessModes é um campo do PersistentVolume que define como o volume pode ser montado. Existem três tipos de AccessModes:

- **ReadWriteOnce**: O volume pode ser montado como leitura e escrita por um único nó.
- **ReadOnlyMany**: O volume pode ser montado como somente leitura por muitos nós.
- **ReadWriteMany**: O volume pode ser montado como leitura e escrita por muitos nós.

### persistentVolumeReclaimPolicy

O persistentVolumeReclaimPolicy é um campo do PersistentVolume que define o que acontece com o volume quando o PersistentVolumeClaim é excluído. Existem três tipos de persistentVolumeReclaimPolicy:

- **Retain**: O volume não é excluído e deve ser excluído manualmente.
- **Delete**: O volume é excluído.
- **Recycle**: O volume é limpo e pode ser usado novamente.

### Path

O [Path](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#types-of-persistent-volumes) é um campo do PersistentVolume que define o caminho do volume. Alguns exemplos de caminhos são:

- **csi**: O volume é um volume CSI (Container Storage Interface).
- **fc**: O volume é um volume Fibre Channel.
- **hostPath**: O volume é um volume local (for single node testing only; WILL NOT WORK in a multi-node cluster; consider using local volume instead).
- **iscsi**: O volume é um volume iSCSI (SCSI over IP).
- **local**: O volume é um volume local.
- **nfs**: O volume é um volume NFS (Network File System).

### PersistentVolume com NFS

Compartilhando um diretório do host com NFS:

```bash
sudo apt install nfs-kernel-server nfs-common
sudo mkdir /mnt/nfs
sudo chown raphael.raphael /mnt/nfs
```
    
```bash
echo "/mnt/nfs           *(rw,sync,no_root_squash,no_subtree_check)" | sudo tee -a /etc/exports
sudo exportfs -ar
```

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  labels:
    storage: nfs
  name: meu-pv-nfs
spec:
  storageClassName: giropops
    capacity:
      storage: 1Gi
    accessModes:
      - ReadWriteOnce
    persistentVolumeReclaimPolicy: Retain
    nfs:
      server: {IP_DO_NFS_SERVER}
      path: /mnt/nfs
```

```bash
kubectl apply -f pv-nfs.yaml
```

### O que é o PersistentVolumeClaim?

O PersistentVolumeClaim é um objeto do Kubernetes que representa uma solicitação de armazenamento, é usado para solicitar um PersistentVolume. O Kubernetes tentará encontrar um PersistentVolume que corresponda ao PersistentVolumeClaim, se não conseguir encontrar um PersistentVolume que corresponda ao PersistentVolumeClaim, ele criará um PersistentVolume.

### Criando um PersistentVolumeClaim

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  labels:
    pvc: meu-primeiro-pvc
  name: meu-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: giropops
  selector:
    matchLabels:
      storage: nfs
```

```bash
kubectl apply -f pvc.yaml
``` 

### Criando um Deployment com um PersistentVolumeClaim

```yaml
apiVersion: v1
kind: Deployment
metadata:
  labels:
    app: redis
  name: redis
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - image: redis
        name: redis
        ports:
        - containerPort: 6379
        volumeMounts:
        - mountPath: /data
          name: redis-data
      volumes:
      - name: redis-data
        persistentVolumeClaim:
          claimName: meu-pvc
```

```bash
kubectl apply -f redis-deployment.yaml
```


## A sua lição de casa

A sua lição de casa é criar um deployment do Nginx, que possua um volume montado no /usr/share/nginx/html. Fique a vontade em utilizar diferentes tipos de provisionadores e/ou diferentes tipos de PV. Deixe a sua imaginação te guiar e aproveite para estudar as diferentes aplicabilidades. :)

### Criando o deployment do Nginx

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - mountPath: /usr/share/nginx/html
          name: nginx-data
      volumes:
      - name: nginx-data
        persistentVolumeClaim:
          claimName: meu-pvc
```