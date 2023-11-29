# ServiceMonitor, PodMonitor e PrometheusRule

### O que é o ServiceMonitor?

O ServiceMonitor é um recurso do Prometheus Operator que permite que você configure o Prometheus para monitorar um serviço. Ele é um Custom Resource Definition (CRD) que pode ser criado no Kubernetes. 

### Criando o Deployment e o Service para serem utilizados com o ServiceMonitor
    
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-server
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "9113"
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
          name: http
        volumeMounts:
        - name: nginx-config
          mountPath: /etc/nginx/conf.d/default.conf
          subPath: nginx.conf
      - name: nginx-exporter
        image: "nginx/nginx-prometheus-exporter:0.11.0"
        args:
        - "-nginx.scrape-uri=http://localhost/metrics"
        resources:
          requests:
            cpu: 0.1
            memory: 64Mi
          limits:
            cpu: 0.3
            memory: 128Mi
        ports:
        - containerPort: 9113
          name: metrics
      volumes:
        - configMap:
            defaultMode: 420
            name: nginx-config
          name: nginx-config
```

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  nginx.conf: |
    server {
      listen 80;
      location / {
        root /usr/share/nginx/html;
        index index.html index.htm;
      }
      location /metrics {
        stub_status on;
        access_log off;
      }
    }
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-server
  labels:
    app: nginx
spec:
  selector:
    app: nginx
  ports:
  - name: http
    port: 80
    targetPort: 80
  - name: metrics
    port: 9113
    targetPort: 9113
```

### Criando o ServiceMonitor

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: nginx-service-monitor
  labels:
    app: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  endpoints:
  - port: metrics
    interval: 30s
    path: /metrics
    targetPort: 9113
```

### O que é o PodMonitor?

O PodMonitor é um recurso do Prometheus Operator que permite que você configure o Prometheus para monitorar um pod. Ele é um Custom Resource Definition (CRD) que pode ser criado no Kubernetes.

### Criando nosso Pod e o PodMonitor

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx-pod
spec:
  containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 80
          name: http
      volumeMounts:
        - name: nginx-config
          mountPath: /etc/nginx/conf.d/default.conf
          subPath: nginx.conf
    - name: nginx-exporter
      image: "nginx/nginx-prometheus-exporter:0.11.0"
      args:
        - "-nginx.scrape-uri=http://localhost/metrics"
      resources:
        requests:
          cpu: 0.1
          memory: 64Mi
        limits:
          cpu: 0.3
          memory: 128Mi
      ports:
        - containerPort: 9113
          name: metrics
  volumes:
    - configMap:
        defaultMode: 420
        name: nginx-config
      name: nginx-config
```

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PodMonitor
metadata:
  name: nginx-pod-monitor
  labels:
    app: nginx
spec:
  namespaceSelector:
    matchNames:
      - default
  selector:
    matchLabels:
      app: nginx-pod
  podMetricsEndpoints:
  - port: metrics
    interval: 10s
    path: /metrics
    targetPort: 9113
```

### O que é o PrometheusRule?

O PrometheusRule é um recurso do Prometheus Operator que permite que você configure o Prometheus para monitorar um pod. Ele é um Custom Resource Definition (CRD) que pode ser criado no Kubernetes.

### Criando o PrometheusRule

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: nginx-prometheus-rule
  namespace: monitoring
  labels:
    prometheus: k8s
    role: alert-rules
    app.kubernetes.io/name: kube-prometheus
    app.kubernetes.io/part-of: kube-prometheus
spec:
  groups:
  - name: nginx-prometheus-rule
    rules:
    - alert: NginxDown
      expr: nginx_up == 0
      for: 1m
      labels:
        severity: critical
      annotations:
        summary: "Nginx server down"
        description: "Nginx server is down for more than 1 minute {{ $labels.pod }}"
    - alert: NginxHighRequestRate
      expr: rate(nginx_http_requests_total[5m]) > 10
      for: 1m
      labels:
        severity: warning
      annotations:
        summary: "Nginx server high request"
        description: "Nginx server is receiving more than 10 requests per second with status code 5xx {{ $labels.pod }}"      
```