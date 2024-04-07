# CKA

## Adicione a taint NoSchedule em um dos nós e remova todos os pods em execução nesse nó:

1. Adicione a taint `NoSchedule` no Node `node01`:

```bash
k cordon node01
```

2. Remova todos os Pods em execução no Node `node01`:

```bash
k drain node01 --ignore-daemonsets=true
```

3. Verifique se os Pods foram removidos:

```bash
k get pods -o wide
```

4. Remova a taint `NoSchedule` do Node `node01`:

```bash
k uncordon node01
```

### Restaure o nó `node01` para o estado normal:

```bash
k uncordon node01
```

## Faça o backup e restaure o etcd:

1. Crie o Namespace `dr-test` e execute um Pod:

```bash
k create ns dr-test
k run webserver -n dr-test --image=nginx
```

2. Acesse o Pod do etcd:

```bash
k exec -it etcd-master-0 -n kube-system -- sh
```

3. Faça o backup do etcd:

```bash
ETCDCTL_API=3 etcdctl --endpoints=https://172.18.0.2:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/healthcheck-client.crt --key=/etc/kubernetes/pki/etcd/healthcheck-client.key snapshot save snapshot.db
```

4. Valide o backup:

```bash
ETCDCTL_API=3 etcdctl --write-out=table snapshot status snapshot.db
```

4. Saia do Pod do etcd e exclua o Namespace `dr-test`:

```bash
k delete ns dr-test
```

5. Restaure o etcd:

```bash
ETCDCTL_API=3 etcdctl --endpoints=https://172.18.0.2:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/healthcheck-client.crt --key=/etc/kubernetes/pki/etcd/healthcheck-client.key --data-dir /tmp/etcd-restore12 snapshot restore snapshot.db
```

6. Verifique se o Namespace `dr-test` foi restaurado e se o Pod `webserver` está em execução:

```bash
k get ns dr-test
k get pods -n dr-test
```

## Verifique os logs do Control Plane:

1. Verifique os Pods em execução no Namespace `kube-system` e os logs do Pod `kube-apiserver-control-plane`:

```bash
k get pods -n kube-system
k logs -f kube-apiserver-control-plane -n kube-system
```

2. Verifique os logs do Pod `kube-controller-manager-control-plane`:

```bash
k logs -f kube-controller-manager-control-plane -n kube-system
```

3. Verifique os logs do Pod `kube-scheduler-control-plane`:

```bash
k logs -f kube-scheduler-control-plane -n kube-system
```

## Crie um Pod `httpd` com a imagem `httpd:latest` e exponha a porta 80:

1. Crie o Pod `httpd`:

```bash
k run httpd --image=httpd:latest --port=80
```

## Crie um Deployment com `initContainers`:

1. Crie o manifesto do Deployment `ex5`:

```bash
k create deployment ex5 --image=nginx --dry-run=client -o yaml > ex5.yaml
```

2. Adicione o campo `initContainers`, `volumeMounts` e `volumes` no arquivo `ex5.yaml` conforme abaixo:

```yaml
spec:
      initContainers:
      - image: busybox
        name: busybox
        command: ['sh', '-c', 'echo Hello, World! > /volume/initfile.txt']
        volumeMounts:
        - name: volume
          mountPath: /volume
      containers:
      - image: nginx
        name: nginx
        resources: {}
        volumeMounts:
        - name: volume
          mountPath: /volume
      volumes:
      - name: volume
        emptyDir: {}
```

3. Aplique o arquivo `ex5.yaml`:

```bash
k apply -f ex5.yaml
```

4. Verifique se o Pod `ex5` foi criado e se o arquivo `initfile.txt` foi criado:

```bash
k get pods -o wide
k exec -it ex5 -- cat /volume/initfile.txt
```

## Crie um `cronjob`, com o schedule `*/1 * * * *" `, com a imagem `busybox`, o comando `echo VAII`, completation `5`, parallelism `5`, activeDeadlineSeconds `15`.

1. Crie o arquivo `ex6.yaml`:

```bash
k create cronjob cron --image=busybox --schedule="*/1 * * * *" --dry-run=client -o yaml > ex6.yaml
```

2. Adicione os campos `completations`, `parallelism` e `activeDeadlineSeconds` no arquivo `ex6.yaml` conforme abaixo:

```yaml
spec:
  completions: 5
  parallelism: 5
  activeDeadlineSeconds: 15
```

3. Aplique o arquivo `ex6.yaml`:

```bash
k apply -f ex6.yaml
```

4. Verifique se o `CronJob` foi criado e se os Pods estão sendo executados:

```bash
k get cronjob
k get pods
```

## Crie um Deployment chamado `rollback` com a imagem `nginx:1.23` e `1` réplica. Em seguida faça o upgrade para a versão `1.25` e faça o rollback para a versão `1.24`. Certifique-se de que a atualização da versão esteja registrada na anotação do recursos do Deployment.

1. Crie o Deployment `rollback` com a imagem `nginx:1.23`:

```bash
k create deployment rollback --image=nginx:1.23 --replicas=1
```

2. Edite o Deployment `rollback` e adicione a versão `1.24`:

```bash
k get deployment rollback -o yaml > ex7.yaml
```

3. Edite o arquivo `ex7.yaml` e adicione a versão `1.24` no campo `containers`:

```yaml
containers:
      - image: nginx:1.24
        imagePullPolicy: Always
        name: rollback
```

4. Aplique o arquivo `ex7.yaml`:

```bash
k apply -f ex7.yaml --record
```

5. Verifique se o Deployment `rollback` foi atualizado para a versão `1.24`:

```bash
k get deployment rollback
```

6. Faça o rollback para a versão `1.23`:

```bash
k rollout undo deployment rollback --to-revision=1
```

7. Verifique se o Deployment `rollback` foi atualizado para a versão `1.23`:

```bash
k get deployment rollback
```

# CKAD

## Crie dois Pods: O `nginx` com a imagem `nginx:latest`, na porta `80. E o `httpd` com a imagem `httpd:latest`, na porta `80`. Exponha os Pods com um `Service`. Em seguida, crie um `Ingress` para redirecionar o tráfego para o `nginx` e `httpd`.

1. Execute os dois pods:

```bash
k run nginx --image=nginx:latest --port=80
k run httpd --image=httpd:latest --port=80
```

2. Exponha os Pods com um `Service` na porta `8080`:

```bash
k expose pod nginx --port=8080 --target-port=80 --name=nginx-svc
k expose pod httpd --port=8080 --target-port=80 --name=httpd-svc
```

3. Crie o arquivo `ingress.yaml`:

```bash
cat <<EOF > ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
    ingressClassName: nginx
    rules:
    - http:
        paths:
        - path: /v1
          pathType: Prefix
          backend:
            service:
              name: nginx-svc
              port:
                number: 8080
        - path: /v2
          pathType: Prefix
          backend:
            service:
              name: httpd-svc
              port:
                number: 8080
EOF

4. Aplique o arquivo `ingress.yaml`:

```bash
k apply -f ingress.yaml
```

5. Verifique se o Ingress foi criado:

```bash
k get ingress
```

6. Teste o acesso ao `nginx` e `httpd`:

```bash
curl localhost/v1
curl localhost/v2
```

## Crie um `Deployment` chamado `autoscaling` com a imagem `nginx:latest` e `2` réplicas. Em seguida, crie um `Horizontal Pod Autoscaler` para o `Deployment` com o mínimo de `2` e o máximo de `5` réplicas, e a métrica `CPU` com o valor `50%`.

1. Crie o Deployment `autoscaling` com a imagem `nginx:latest` e `2` réplicas:

```bash
k create deployment autoscaling --image=nginx:latest --replicas=2
```

2. Crie o `Horizontal Pod Autoscaler` para o `Deployment`:

```bash
k autoscale deployment autoscaling --name hpa --cpu-percent=50 --min=2 --max=5
```

3. Verifique se o `Horizontal Pod Autoscaler` foi criado:

```bash
k get hpa
```

## TODO Crie um `Canary Deployment` com os `Deployments` `nginx` e `httpd` e faça o balanceamento de tráfego entre eles.

1. Crie dois deployments:

```bash
k create deployment v1 nginx --image=nginx --replicas=1
k create deployment v2 httpd --image=httpd --replicas=1
```

2. Exponha o deployment com um `Service`:

```bash
k expose deployment nginx --port=80 --target-port=80 --name=nginx-svc --type=NodePort 
k expose deployment httpd --port=80 --target-port=80 --name=httpd-svc --type=NodePort
```

3. Edite o `Deployment` do `nginx` adicionando a label `deploy: true`:

```bash
k label deployment nginx deploy=true
```

## Colete as métricas do Pod `v2`

1. Execute o comando:

```bash
k top pod v2
```

## Colete os logs do container `coredns`:

1. Execute o comando:

```bash
k logs -f coredns-xxxxx-xxxxx -n kube-system > /tmp/log.txt
```

## Liste os `CustomResourceDefinitions`:

1. Execute o comando:

```bash
k get crd
```

# CKS