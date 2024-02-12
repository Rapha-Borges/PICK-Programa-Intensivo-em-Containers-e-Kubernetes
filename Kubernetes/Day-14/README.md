# Policies no Kubernetes com Kyverno

O Kyverno é uma ferramenta que permite criar políticas de validação e mutação de recursos no Kubernetes. Com ele, é possível definir regras para garantir que os recursos criados estejam de acordo com as políticas de segurança e conformidade da organização. Além disso, o Kyverno pode ser utilizado para criar recursos de forma automática, como por exemplo, uma NetworkPolicy para cada Namespace criado.

É possível configurar o Kyverno de duas formas: Enforcing e Audit. No modo Enforcing, as políticas são aplicadas de forma obrigatória, ou seja, os recursos que não estiverem de acordo com as políticas definidas serão bloqueados. Já no modo Audit, as políticas são aplicadas de forma passiva, ou seja, os recursos que não estiverem de acordo com as políticas definidas serão apenas registrados, sem bloqueio.

### Instalando o Kyverno

Vamos utilizar o Helm para instalar o Kyverno no nosso cluster. Primeiro, adicione o repositório do Kyverno:

```bash
helm repo add kyverno https://kyverno.github.io/kyverno/
helm repo update
```

Agora, instale o Kyverno:

```bash
helm install kyverno kyverno/kyverno --namespace kyverno --create-namespace
```

### Criando uma ClusterPolicy Validate

Uma Validate Policy é utilizada para validar se os recursos criados no cluster estão de acordo com as políticas definidas.

Vamos criar uma ClusterPolicy que valida se os recursos criados possuem a definição de limites de recursos. Crie o arquivo `require-resource-limits.yaml`:

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-resources-limits
spec:
  validationFailureAction: Enforce
  rules:
  - name: validate-limits
    match:
      resources:
        kinds:
        - Pod
    validate:
      message: "Define resource limits"
      patern:
        spec:
          containers:
          - name: "*"
            resources:
              limits:
                memory: "?*"
                cpu: "?*"
```

Agora, aplique a ClusterPolicy no cluster:

```bash
kubectl apply -f require-resource-limits.yaml
```

Se tentarmos criar um Pod sem definir limites de recursos, o Kyverno irá bloquear a criação do recursos.

### Criando uma Policy Mutate 

Com uma Mutate Policy, é possível alterar os recursos criados no cluster de acordo com as políticas definidas. 

Vamos criar uma Policy que adiciona o label `projeto: pick` em todos os Namespaces criados. Crie o arquivo `add-label-namespaces.yaml`:

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: add-label-namespaces
spec:
  rules:
  - name: add-label-ns
    match:
      resources:
        kinds:
        - Namespace
    mutate:
      pathStrategicMerge:
        metadata:
          labels:
            projeto: "pick"
```

Agora, aplique a ClusterPolicy no cluster:

```bash
kubectl apply -f add-label-namespaces.yaml
```

Ao criar um Namespace, o Kyverno irá adicionar o label `projeto: pick` automaticamente.

### Criando uma Policy Generate

Com uma Generate Policy, é possível criar recursos de forma automática no cluster de acordo com as políticas definidas. Vamos fazer um exemplo de uma Generate Policy que cria um `ConfigMap` para cada `Namespace` criado.

Crie o arquivo `create-configmap-ns.yaml`:

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: generate-cm-add-ns
spec:
  rules:
  - name: generate-cm-add-ns
    match:
      resources:
        kinds:
        - Namespace
    generate:
      apiVersion: v1
      kind: ConfigMap
      name: default-configmap
      namespace: "{{request.object.metadata.name}}"
      data:
        key1: "Giropops"
        key2: "Strigus"
```

### Criando uma Policy de Proibição

Com uma Policy de Proibição, é possível bloquear a criação de recursos no cluster de acordo com as políticas definidas. Vamos fazer um exemplo de uma Policy de Proibição que bloqueia a criação de Pods que estejam rodando como root.

Crie o arquivo `block-root-pods.yaml`:

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: block-root-pods
spec:
  validationFailureAction: Enforce
  rules:
  - name: check-runAsNonRoot
    match:
      resources:
        kinds:
        - Pod
    validate:
      message: "Root user is not allowed"
      pattern:
        spec:
          containers:
          - securityContext:
              runAsNonRoot: true
```

### Criando uma Policy para permitir apenas imagens de um registry específico

Podemos criar uma Policy para permitir apenas imagens de um registry específico. Vamos fazer um exemplo de uma Policy que permite apenas imagens do registry `cgr.dev/chainguard`.

Crie o arquivo `allow-only-chainguard-images.yaml`:

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: allow-only-chainguard-images
spec:
  validationFailureAction: enforce
  rules:
  - name: trusted-registry
    match:
      resources:
        kinds:
        - Pod
    validate:
      message: "Image is not from trusted registry"
      pattern:
        spec:
          containers:
          - image: "cgr.dev/chainguard/*"
```

### Utilizando o Exclude

Podemos utilizar o `exclude` para excluir recursos que não devem ser validados pela Policy. Vamos fazer um exemplo de uma Policy que permite apenas imagens do registry `cgr.dev/chainguard`, mas exclui o Namespace `no-chainguard`.

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: allow-trusted-registry-except-no-chainguard
spec:
  validationFailureAction: enforce
  rules:
  - name: trusted-registry-except-no-chainguard
    match:
      resources:
        kinds:
        - Pod
    exclude:
      resources:
        namespaces:
        - no-chainguard
    validate:
      message: "Image is not from trusted registry"
      pattern:
        spec:
          containers:
          - image: "cgr.dev/chainguard/*"
```