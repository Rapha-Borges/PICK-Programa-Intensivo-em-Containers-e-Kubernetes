# Prometheus + Kubernetes

### O que é Prometheus?

O Prometheus é um sistema de monitoramento de código aberto que permite monitorar serviços e aplicações em execução. Ele coleta métricas de alvos configurados e armazena as informações em um banco de dados de séries temporais, onde é possível executar consultas, criar alertas, entre outras coisas.

### Prometheus Operator

O Prometheus Operator é um projeto que facilita a configuração e o gerenciamento do Prometheus e de outros componentes relacionados. Ele utiliza o Kubernetes para automatizar tarefas como a criação de recursos, o provisionamento de armazenamento e a configuração de alertas.

### Kube-Prometheus

O Kube-Prometheus é um projeto que utiliza o Prometheus Operator para implantar o Prometheus e outros componentes relacionados no Kubernetes. Ele também fornece um conjunto de dashboards para visualizar as métricas coletadas.

### Instalando o Kube-Prometheus

Clone o repositório do Kube-Prometheus:

```bash
git clone https://github.com/prometheus-operator/kube-prometheus.git
```

Acesse o diretório do Kube-Prometheus:

```bash
cd kube-prometheus
```

Crie o namespace e o CRDs:

```bash
kubectl create -f manifests/setup
```

Espere até que os CRDs sejam criados:

```bash
until kubectl get servicemonitors --all-namespaces ; do date; sleep 1; echo ""; done
```

Aplique os manifestos:

```bash
kubectl create -f manifests/
```

### Acessando o Prometheus

```bash
kubectl --namespace monitoring port-forward svc/prometheus-k8s 9090
```

Vai estar disponível em: http://localhost:9090

### Acessando o Grafana

```bash
kubectl --namespace monitoring port-forward svc/grafana 3000
```

Vai estar disponível em: http://localhost:3000

### Acessando o Alertmanager

```bash
kubectl --namespace monitoring port-forward svc/alertmanager-main 9093
```

Vai estar disponível em: http://localhost:9093

### Desinstalando o Kube-Prometheus

```bash
kubectl delete --ignore-not-found=true -f manifests/ -f manifests/setup
```

### Corrigindo o erro "too many open files" qunado estiver executando com Kind

Edite o arquivo '/etc/sysctl.conf':

```bash
sudo vim /etc/sysctl.conf
```

Adicione as seguintes linhas:

```bash
fs.inotify.max_user_watches = 524288
fs.inotify.max_user_instances = 512
```

### O que é o ServiceMonitor?

O ServiceMonitor é um recurso do Prometheus Operator que permite monitorar serviços e aplicações em execução no Kubernetes. Ele utiliza o Kubernetes para descobrir os alvos que devem ser monitorados e cria automaticamente os recursos necessários para coletar as métricas.