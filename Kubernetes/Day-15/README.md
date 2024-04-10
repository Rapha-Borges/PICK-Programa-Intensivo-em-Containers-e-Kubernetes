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

## Labels

Labels são usadas para identificar recursos no Kubernetes. Elas são pares chave-valor que são anexados a recursos como Pods e Nodes. Labels são usadas para identificar recursos e para selecionar recursos para operações como escalonamento e filtragem.

### Adicionando Labels

Vamos criar um cenário onde temos 5 Nodes em alta disponibilidade, separados em 2 regiões e 3 zonas de disponibilidade. Com isso, podemos gereciar melhor a distribuição dos Pods levando em consideração questões de latência e disponibilidade.

Para isso, vamos adicionar os labels `region` e `zone` nos Nodes. Execute os comandos abaixo:

```bash
kubectl label nodes node01 region=us-east zone=us-east-1
kubectl label nodes node02 region=us-east zone=us-east-2
kubectl label nodes node03 region=us-east zone=us-east-3
kubectl label nodes node04 region=us-west zone=us-west-1
kubectl label nodes node05 region=us-west zone=us-west-2
```

Com essas labels, definimos que os Nodes `node01`, `node02` e `node03` estão na região `us-east`, porém em zonas diferentes. E os Nodes `node04` e `node05` estão na região `us-west`, também em zonas diferentes.

Para verificar os labels dos Nodes, execute o comando:

```bash
kubectl get nodes --show-labels
```

As labels também podem ser utilizadas para garantir que um determinado Pod que precisa de um recurso específico seja executado em um Node que possui este recurso. Como por exemplo, um Pod que precisa de uma GPU. Para isso, podemos adicionar a label `gpu=true` no Node que possui a GPU e utilizar o Node Affinity para garantir que o Pod seja executado neste Node.

## Affinity

### Node Affinity

Node Affinity é uma maneira de especificar regras sobre quais Nodes um Pod pode ser agendado, utilizando os labels para isso.

Existem dois tipos de Node Affinity:

- **RequiredDuringSchedulingIgnoredDuringExecution**: O Pod só será agendado em Nodes que atendem aos requisitos de Node Affinity.
- **PreferredDuringSchedulingIgnoredDuringExecution**: O Pod será agendado em Nodes que atendem aos requisitos de Node Affinity, mas não é uma garantia.

Para testar o Node Affinity, vamos criar um deployment que só pode ser executado em Nodes da região `us-east`. Para isso, adicione o campo `nodeAffinity` no arquivo `nginx-deployment-affinity.yaml` conforme abaixo.

```yaml
nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
            - matchExpressions:
                - key: "region"
                  operator: "In"
                  values:
                    - "us-east"

```

Em seguida, aplique o arquivo `nginx-deployment-affinity.yaml`:

```bash
kubectl apply -f nginx-deployment-affinity.yaml
```

Com o comando `kubectl get pods -o wide`, verifique que o Pod está sendo executado nos Nodes `node01`, `node02` e `node03`, que estão na região `us-east`.

## Anti-Affinity

O Anti-Affinity é o oposto do Affinity. Ele é usado para garantir que os Pods não sejam agendados em Nodes específicos. Com isso, aumentamos a disponibilidade e a tolerância a falhas dos nossos Pods distribuindo-os em diferentes Nodes que estão em diferentes regiões, zonas ou datacenters.

Existem dois tipos de Anti-Affinity:

- **RequiredDuringSchedulingIgnoredDuringExecution**: O Pod não será agendado em Nodes que atendem aos requisitos de Anti-Affinity.
- **PreferredDuringSchedulingIgnoredDuringExecution**: O Pod será agendado em Nodes que atendem aos requisitos de Anti-Affinity, mas não é uma garantia.

Para testar o Anti-Affinity, vamos utilizar o Pod Affinity para criar um deployment que irá restrigir a execução dos Pods em somente uma região, independente da quantidade de Nodes disponíveis. Para isso, adicione o campo `podAffinity` no arquivo `nginx-deployment-anti-affinity.yaml` conforme abaixo.

```yaml
podAntiAffinity:
  requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
      matchLabels:
        app: nginx
      topologyKey: "region"
```

Em seguida, aplique o arquivo `nginx-deployment-anti-affinity.yaml`:

```bash
kubectl apply -f nginx-deployment-anti-affinity.yaml
```

Podemos fazer diferente, utilizando o Pod Anti-Affinity para criar um deployment que irá restrigir a execução dos Pods em zonas de disponibilidade diferentes. Tendo um cenário de alta disponibilidade, onde cada Pod estará obrigatoriamente em uma zona de disponibilidade diferente. Para isso, adicione as seguintes linhas no arquivo `nginx-deployment-anti-affinity-zone.yaml`:

```yaml
podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
        matchLabels:
          app: nginx
        topologyKey: "az"
```

Em seguida, aplique o arquivo `nginx-deployment-anti-affinity-zone.yaml`:

```bash
kubectl apply -f nginx-deployment-anti-affinity-zone.yaml
```

Com o comando `kubectl get pods -o wide`, verifique que os Pods estão sendo executados em zonas de disponibilidade diferentes.

É importante ressaltar que para utilizar o Node Affinity e Anti-Affinity da melhor forma, é necessário planejar e entender a topologia da sua infraestrutura, para que não haja impactos negativos na execução dos seus Pods. Como por exemplo, a falta de recursos para execução dos Pods ou a distribuição inadequada dos Pods em Nodes que estão em regiões ou zonas de disponibilidade diferentes.

