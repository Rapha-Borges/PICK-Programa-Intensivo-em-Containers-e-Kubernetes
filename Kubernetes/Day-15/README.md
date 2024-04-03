# Taints, Tolerations e Node Affinity

## Taints (Manchas)

Taints são usados para repelir Pods de Nodes específicos. Eles são usados para garantir que Pods que não devem ser executados em um Node específico. O Control Plane do Kubernetes, por padrão, não agendará Pods em Nodes que possuem Taints, a menos que o Pod tenha uma tolerância correspondente.

### Tipos de Taints

- **NoSchedule**: Impede que novos Pods sejam agendados no Node.
- **PreferNoSchedule**: Indica que o Kubernetes deve tentar não agendar novos Pods no Node, mas não é uma garantia.
- **NoExecute**: Remove Pods que não toleram o Taint.

### Adicionando e validando Taints

Crie um Deployment conforme o arquivo `nginx-deployment.yaml`. Em seguida, faça o scale up para 20 réplicas, populando todos os Nodes disponíveis.

Com o comando `kubectl get pods -o wide`, verifique em quais Nodes os Pods estão sendo executados.

Agora vamos adicionar um Taint no Node `node01` indicando que este Node irá entrar em manutenção para que todos os Pods que não tolerarem este Taint sejam removidos. Para isso, execute o comando:

```bash
kubectl taint nodes node01 maintenance=true:NoExecute
```

Execute o comando `kubectl get pods -o wide` novamente e veja que os Pods que estavam no Node `node01` foram removidos. E com o comando `kubectl get nodes -o wide`, verifique que o Node `node01` está com o Taint `maintenance=true:NoExecute`.

Também podemos adicionar a Taint `NoSchedule` no Node `node02`, assim os Pods não serão agendados neste Node. Para isso, execute o comando:

```bash
kubectl taint nodes node02 maintenance=true:NoSchedule
```

Por fim, para remover o Taint do Node `node01`, execute o comando:

```bash
kubectl taint nodes node01 maintenance-
```

E para remover o Taint do Node `node02`, execute o comando:

```bash
kubectl taint nodes node02 maintenance-
```

Atenção: Quando um Taint é removido, os Pods que foram removidos não são recriados automaticamente. Para isso, você pode realizar o rollout do Deployment.

```bash
kubectl rollout restart deployment nginx-deployment
```

## Tolerations (Tolerâncias)

Tolerations são usadas para permitir que os Pods sejam agendados em Nodes que possuem Taints. Ou seja, se um Node possui um Taint, os Pods que possuem uma tolerância correspondente podem ser agendados neste Node.

### Adicionando Tolerations

Para adicionar uma tolerância a um Pod, vamos adicionar o campo `tolerations` no arquivo `nginx-deployment-gpu.yaml` conforme abaixo:

```yaml
tolerations:
    - key: "gpu"
      operator: "Equal"
      value: "true"
      effect: "NoSchedule"
```

Para testarmos, vamos adicionar um Taint no Node `node03` com a chave `gpu` e valor `true` e efeito `NoSchedule`. Para isso, execute o comando:

```bash
kubectl taint nodes node03 gpu=true:NoSchedule
```

Em seguida, aplique o arquivo `nginx-deployment-gpu.yaml`:

```bash
kubectl apply -f nginx-deployment-gpu.yaml
```

Com o comando `kubectl get pods -o wide`, verifique que os Pods estão sendo executados em todos os Nodes, inclusive no Node `node03`.

Por fim, para remover o Taint do Node `node03`, execute o comando:

```bash
kubectl taint nodes node03 gpu-
```

## Labels, Affinity e Anti-Affinity

Labels são usadas para identificar recursos no Kubernetes. Elas são pares chave-valor que são anexados a recursos como Pods e Nodes. Labels são usadas para identificar recursos e para selecionar recursos para operações como escalonamento e filtragem.

### Node Affinity

Node Affinity é uma maneira de especificar regras sobre quais Nodes um Pod pode ser agendado, utilizando os labels para isso.

Existem dois tipos de Node Affinity:

- **RequiredDuringSchedulingIgnoredDuringExecution**: O Pod só será agendado em Nodes que atendem aos requisitos de Node Affinity.
- **PreferredDuringSchedulingIgnoredDuringExecution**: O Pod será agendado em Nodes que atendem aos requisitos de Node Affinity, mas não é uma garantia.
