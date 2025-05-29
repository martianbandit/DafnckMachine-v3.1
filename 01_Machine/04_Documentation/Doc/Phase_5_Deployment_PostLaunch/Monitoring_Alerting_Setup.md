# Monitoring & Alerting Setup

## Overview
This document provides comprehensive guidance for implementing monitoring and alerting systems for DafnckMachine v3.1, covering Prometheus, Grafana, alerting rules, log aggregation, and observability best practices.

## Prometheus Configuration

### Core Prometheus Setup
```yaml
# prometheus-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: monitoring
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
      evaluation_interval: 15s
      external_labels:
        cluster: 'dafnckmachine-production'
        environment: 'production'
    
    rule_files:
      - "/etc/prometheus/rules/*.yml"
    
    alerting:
      alertmanagers:
        - static_configs:
            - targets:
              - alertmanager:9093
    
    scrape_configs:
      - job_name: 'prometheus'
        static_configs:
          - targets: ['localhost:9090']
      
      - job_name: 'kubernetes-apiservers'
        kubernetes_sd_configs:
          - role: endpoints
        scheme: https
        tls_config:
          ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
        relabel_configs:
          - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_service_name, __meta_kubernetes_endpoint_port_name]
            action: keep
            regex: default;kubernetes;https
      
      - job_name: 'kubernetes-nodes'
        kubernetes_sd_configs:
          - role: node
        scheme: https
        tls_config:
          ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
        relabel_configs:
          - action: labelmap
            regex: __meta_kubernetes_node_label_(.+)
          - target_label: __address__
            replacement: kubernetes.default.svc:443
          - source_labels: [__meta_kubernetes_node_name]
            regex: (.+)
            target_label: __metrics_path__
            replacement: /api/v1/nodes/${1}/proxy/metrics
      
      - job_name: 'kubernetes-pods'
        kubernetes_sd_configs:
          - role: pod
        relabel_configs:
          - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
            action: keep
            regex: true
          - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
            action: replace
            target_label: __metrics_path__
            regex: (.+)
          - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
            action: replace
            regex: ([^:]+)(?::\d+)?;(\d+)
            replacement: $1:$2
            target_label: __address__
          - action: labelmap
            regex: __meta_kubernetes_pod_label_(.+)
          - source_labels: [__meta_kubernetes_namespace]
            action: replace
            target_label: kubernetes_namespace
          - source_labels: [__meta_kubernetes_pod_name]
            action: replace
            target_label: kubernetes_pod_name
      
      - job_name: 'dafnckmachine-app'
        kubernetes_sd_configs:
          - role: endpoints
            namespaces:
              names:
                - production
                - staging
        relabel_configs:
          - source_labels: [__meta_kubernetes_service_name]
            action: keep
            regex: dafnckmachine.*
          - source_labels: [__meta_kubernetes_endpoint_port_name]
            action: keep
            regex: metrics
```

### Prometheus Deployment
```yaml
# prometheus-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus
  template:
    metadata:
      labels:
        app: prometheus
    spec:
      serviceAccountName: prometheus
      containers:
      - name: prometheus
        image: prom/prometheus:v2.45.0
        args:
          - '--config.file=/etc/prometheus/prometheus.yml'
          - '--storage.tsdb.path=/prometheus/'
          - '--web.console.libraries=/etc/prometheus/console_libraries'
          - '--web.console.templates=/etc/prometheus/consoles'
          - '--storage.tsdb.retention.time=30d'
          - '--web.enable-lifecycle'
          - '--web.enable-admin-api'
        ports:
        - containerPort: 9090
        resources:
          requests:
            memory: "2Gi"
            cpu: "1000m"
          limits:
            memory: "4Gi"
            cpu: "2000m"
        volumeMounts:
        - name: prometheus-config-volume
          mountPath: /etc/prometheus/
        - name: prometheus-storage-volume
          mountPath: /prometheus/
        - name: prometheus-rules-volume
          mountPath: /etc/prometheus/rules/
      volumes:
      - name: prometheus-config-volume
        configMap:
          defaultMode: 420
          name: prometheus-config
      - name: prometheus-storage-volume
        persistentVolumeClaim:
          claimName: prometheus-pvc
      - name: prometheus-rules-volume
        configMap:
          name: prometheus-rules

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: prometheus-pvc
  namespace: monitoring
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 100Gi
  storageClassName: gp3
```

## Alerting Rules Configuration

### Application Alerts
```yaml
# prometheus-rules.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-rules
  namespace: monitoring
data:
  application.yml: |
    groups:
    - name: dafnckmachine.rules
      rules:
      - alert: HighErrorRate
        expr: |
          (
            sum(rate(http_requests_total{job="dafnckmachine-app",status=~"5.."}[5m])) /
            sum(rate(http_requests_total{job="dafnckmachine-app"}[5m]))
          ) > 0.05
        for: 2m
        labels:
          severity: critical
          service: dafnckmachine
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value | humanizePercentage }} for the last 5 minutes"
          runbook_url: "https://docs.dafnckmachine.com/runbooks/high-error-rate"
      
      - alert: HighLatency
        expr: |
          histogram_quantile(0.95,
            sum(rate(http_request_duration_seconds_bucket{job="dafnckmachine-app"}[5m])) by (le)
          ) > 0.5
        for: 5m
        labels:
          severity: warning
          service: dafnckmachine
        annotations:
          summary: "High latency detected"
          description: "95th percentile latency is {{ $value }}s"
          runbook_url: "https://docs.dafnckmachine.com/runbooks/high-latency"
      
      - alert: ServiceDown
        expr: up{job="dafnckmachine-app"} == 0
        for: 1m
        labels:
          severity: critical
          service: dafnckmachine
        annotations:
          summary: "Service is down"
          description: "DafnckMachine service has been down for more than 1 minute"
          runbook_url: "https://docs.dafnckmachine.com/runbooks/service-down"
      
      - alert: HighMemoryUsage
        expr: |
          (
            container_memory_working_set_bytes{pod=~"dafnckmachine-.*"} /
            container_spec_memory_limit_bytes{pod=~"dafnckmachine-.*"}
          ) > 0.8
        for: 5m
        labels:
          severity: warning
          service: dafnckmachine
        annotations:
          summary: "High memory usage"
          description: "Memory usage is {{ $value | humanizePercentage }} for pod {{ $labels.pod }}"
      
      - alert: HighCPUUsage
        expr: |
          (
            rate(container_cpu_usage_seconds_total{pod=~"dafnckmachine-.*"}[5m]) /
            container_spec_cpu_quota{pod=~"dafnckmachine-.*"} * container_spec_cpu_period{pod=~"dafnckmachine-.*"}
          ) > 0.8
        for: 5m
        labels:
          severity: warning
          service: dafnckmachine
        annotations:
          summary: "High CPU usage"
          description: "CPU usage is {{ $value | humanizePercentage }} for pod {{ $labels.pod }}"

  infrastructure.yml: |
    groups:
    - name: infrastructure.rules
      rules:
      - alert: NodeDown
        expr: up{job="kubernetes-nodes"} == 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Node is down"
          description: "Node {{ $labels.instance }} has been down for more than 5 minutes"
      
      - alert: NodeHighCPU
        expr: |
          (
            100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
          ) > 80
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage on node"
          description: "CPU usage is {{ $value }}% on node {{ $labels.instance }}"
      
      - alert: NodeHighMemory
        expr: |
          (
            (node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) /
            node_memory_MemTotal_bytes
          ) > 0.8
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High memory usage on node"
          description: "Memory usage is {{ $value | humanizePercentage }} on node {{ $labels.instance }}"
      
      - alert: NodeDiskSpaceLow
        expr: |
          (
            (node_filesystem_size_bytes{fstype!="tmpfs"} - node_filesystem_free_bytes{fstype!="tmpfs"}) /
            node_filesystem_size_bytes{fstype!="tmpfs"}
          ) > 0.8
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Low disk space on node"
          description: "Disk usage is {{ $value | humanizePercentage }} on {{ $labels.device }} of node {{ $labels.instance }}"

  database.yml: |
    groups:
    - name: database.rules
      rules:
      - alert: PostgreSQLDown
        expr: pg_up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "PostgreSQL is down"
          description: "PostgreSQL instance {{ $labels.instance }} is down"
      
      - alert: PostgreSQLHighConnections
        expr: |
          (
            pg_stat_database_numbackends /
            pg_settings_max_connections
          ) > 0.8
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High number of PostgreSQL connections"
          description: "Connection usage is {{ $value | humanizePercentage }}"
      
      - alert: PostgreSQLSlowQueries
        expr: |
          rate(pg_stat_database_tup_returned[5m]) /
          rate(pg_stat_database_tup_fetched[5m]) < 0.1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "PostgreSQL slow queries detected"
          description: "Query efficiency is {{ $value | humanizePercentage }}"
```

## Alertmanager Configuration

### Alertmanager Setup
```yaml
# alertmanager-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: alertmanager-config
  namespace: monitoring
data:
  alertmanager.yml: |
    global:
      smtp_smarthost: 'smtp.gmail.com:587'
      smtp_from: 'alerts@dafnckmachine.com'
      smtp_auth_username: 'alerts@dafnckmachine.com'
      smtp_auth_password: '${SMTP_PASSWORD}'
    
    route:
      group_by: ['alertname', 'cluster', 'service']
      group_wait: 10s
      group_interval: 10s
      repeat_interval: 1h
      receiver: 'default'
      routes:
      - match:
          severity: critical
        receiver: 'critical-alerts'
        group_wait: 5s
        repeat_interval: 5m
      - match:
          severity: warning
        receiver: 'warning-alerts'
        repeat_interval: 30m
    
    receivers:
    - name: 'default'
      slack_configs:
      - api_url: '${SLACK_WEBHOOK_URL}'
        channel: '#alerts'
        title: 'DafnckMachine Alert'
        text: '{{ range .Alerts }}{{ .Annotations.summary }}{{ end }}'
    
    - name: 'critical-alerts'
      slack_configs:
      - api_url: '${SLACK_WEBHOOK_URL}'
        channel: '#critical-alerts'
        title: '🚨 CRITICAL: DafnckMachine Alert'
        text: |
          {{ range .Alerts }}
          *Alert:* {{ .Annotations.summary }}
          *Description:* {{ .Annotations.description }}
          *Severity:* {{ .Labels.severity }}
          *Service:* {{ .Labels.service }}
          {{ if .Annotations.runbook_url }}*Runbook:* {{ .Annotations.runbook_url }}{{ end }}
          {{ end }}
        send_resolved: true
      email_configs:
      - to: 'oncall@dafnckmachine.com'
        subject: '🚨 CRITICAL Alert: {{ .GroupLabels.alertname }}'
        body: |
          {{ range .Alerts }}
          Alert: {{ .Annotations.summary }}
          Description: {{ .Annotations.description }}
          Severity: {{ .Labels.severity }}
          Service: {{ .Labels.service }}
          {{ if .Annotations.runbook_url }}Runbook: {{ .Annotations.runbook_url }}{{ end }}
          {{ end }}
    
    - name: 'warning-alerts'
      slack_configs:
      - api_url: '${SLACK_WEBHOOK_URL}'
        channel: '#warnings'
        title: '⚠️ WARNING: DafnckMachine Alert'
        text: |
          {{ range .Alerts }}
          *Alert:* {{ .Annotations.summary }}
          *Description:* {{ .Annotations.description }}
          *Severity:* {{ .Labels.severity }}
          {{ end }}
        send_resolved: true
    
    inhibit_rules:
    - source_match:
        severity: 'critical'
      target_match:
        severity: 'warning'
      equal: ['alertname', 'cluster', 'service']
```

### Alertmanager Deployment
```yaml
# alertmanager-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: alertmanager
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: alertmanager
  template:
    metadata:
      labels:
        app: alertmanager
    spec:
      containers:
      - name: alertmanager
        image: prom/alertmanager:v0.25.0
        args:
          - '--config.file=/etc/alertmanager/alertmanager.yml'
          - '--storage.path=/alertmanager'
          - '--web.external-url=https://alertmanager.dafnckmachine.com'
        ports:
        - containerPort: 9093
        resources:
          requests:
            memory: "256Mi"
            cpu: "100m"
          limits:
            memory: "512Mi"
            cpu: "200m"
        volumeMounts:
        - name: alertmanager-config-volume
          mountPath: /etc/alertmanager
        - name: alertmanager-storage-volume
          mountPath: /alertmanager
        env:
        - name: SLACK_WEBHOOK_URL
          valueFrom:
            secretKeyRef:
              name: alertmanager-secrets
              key: slack-webhook-url
        - name: SMTP_PASSWORD
          valueFrom:
            secretKeyRef:
              name: alertmanager-secrets
              key: smtp-password
      volumes:
      - name: alertmanager-config-volume
        configMap:
          name: alertmanager-config
      - name: alertmanager-storage-volume
        persistentVolumeClaim:
          claimName: alertmanager-pvc
```

## Grafana Configuration

### Grafana Deployment
```yaml
# grafana-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      containers:
      - name: grafana
        image: grafana/grafana:10.0.0
        ports:
        - containerPort: 3000
        env:
        - name: GF_SECURITY_ADMIN_PASSWORD
          valueFrom:
            secretKeyRef:
              name: grafana-secrets
              key: admin-password
        - name: GF_INSTALL_PLUGINS
          value: "grafana-piechart-panel,grafana-worldmap-panel"
        resources:
          requests:
            memory: "256Mi"
            cpu: "100m"
          limits:
            memory: "512Mi"
            cpu: "200m"
        volumeMounts:
        - name: grafana-storage
          mountPath: /var/lib/grafana
        - name: grafana-config
          mountPath: /etc/grafana/grafana.ini
          subPath: grafana.ini
        - name: grafana-datasources
          mountPath: /etc/grafana/provisioning/datasources
        - name: grafana-dashboards-config
          mountPath: /etc/grafana/provisioning/dashboards
        - name: grafana-dashboards
          mountPath: /var/lib/grafana/dashboards
      volumes:
      - name: grafana-storage
        persistentVolumeClaim:
          claimName: grafana-pvc
      - name: grafana-config
        configMap:
          name: grafana-config
      - name: grafana-datasources
        configMap:
          name: grafana-datasources
      - name: grafana-dashboards-config
        configMap:
          name: grafana-dashboards-config
      - name: grafana-dashboards
        configMap:
          name: grafana-dashboards
```

### Grafana Datasources
```yaml
# grafana-datasources.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: grafana-datasources
  namespace: monitoring
data:
  datasources.yaml: |
    apiVersion: 1
    datasources:
    - name: Prometheus
      type: prometheus
      access: proxy
      url: http://prometheus:9090
      isDefault: true
      editable: true
    - name: Loki
      type: loki
      access: proxy
      url: http://loki:3100
      editable: true
    - name: Jaeger
      type: jaeger
      access: proxy
      url: http://jaeger-query:16686
      editable: true
```

### Application Dashboard
```json
{
  "dashboard": {
    "id": null,
    "title": "DafnckMachine Application Metrics",
    "tags": ["dafnckmachine", "application"],
    "timezone": "browser",
    "panels": [
      {
        "id": 1,
        "title": "Request Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "sum(rate(http_requests_total{job=\"dafnckmachine-app\"}[5m])) by (status)",
            "legendFormat": "{{status}}"
          }
        ],
        "yAxes": [
          {
            "label": "Requests/sec"
          }
        ]
      },
      {
        "id": 2,
        "title": "Response Time",
        "type": "graph",
        "targets": [
          {
            "expr": "histogram_quantile(0.50, sum(rate(http_request_duration_seconds_bucket{job=\"dafnckmachine-app\"}[5m])) by (le))",
            "legendFormat": "50th percentile"
          },
          {
            "expr": "histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket{job=\"dafnckmachine-app\"}[5m])) by (le))",
            "legendFormat": "95th percentile"
          },
          {
            "expr": "histogram_quantile(0.99, sum(rate(http_request_duration_seconds_bucket{job=\"dafnckmachine-app\"}[5m])) by (le))",
            "legendFormat": "99th percentile"
          }
        ],
        "yAxes": [
          {
            "label": "Seconds"
          }
        ]
      },
      {
        "id": 3,
        "title": "Error Rate",
        "type": "singlestat",
        "targets": [
          {
            "expr": "sum(rate(http_requests_total{job=\"dafnckmachine-app\",status=~\"5..\"}[5m])) / sum(rate(http_requests_total{job=\"dafnckmachine-app\"}[5m]))"
          }
        ],
        "format": "percentunit",
        "thresholds": "0.01,0.05"
      },
      {
        "id": 4,
        "title": "Memory Usage",
        "type": "graph",
        "targets": [
          {
            "expr": "container_memory_working_set_bytes{pod=~\"dafnckmachine-.*\"} / 1024 / 1024",
            "legendFormat": "{{pod}}"
          }
        ],
        "yAxes": [
          {
            "label": "MB"
          }
        ]
      },
      {
        "id": 5,
        "title": "CPU Usage",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(container_cpu_usage_seconds_total{pod=~\"dafnckmachine-.*\"}[5m]) * 100",
            "legendFormat": "{{pod}}"
          }
        ],
        "yAxes": [
          {
            "label": "Percent"
          }
        ]
      }
    ],
    "time": {
      "from": "now-1h",
      "to": "now"
    },
    "refresh": "30s"
  }
}
```

## Log Aggregation with Loki

### Loki Configuration
```yaml
# loki-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: loki-config
  namespace: monitoring
data:
  loki.yaml: |
    auth_enabled: false
    
    server:
      http_listen_port: 3100
      grpc_listen_port: 9096
    
    common:
      path_prefix: /loki
      storage:
        filesystem:
          chunks_directory: /loki/chunks
          rules_directory: /loki/rules
      replication_factor: 1
      ring:
        instance_addr: 127.0.0.1
        kvstore:
          store: inmemory
    
    query_range:
      results_cache:
        cache:
          embedded_cache:
            enabled: true
            max_size_mb: 100
    
    schema_config:
      configs:
        - from: 2020-10-24
          store: boltdb-shipper
          object_store: filesystem
          schema: v11
          index:
            prefix: index_
            period: 24h
    
    ruler:
      alertmanager_url: http://alertmanager:9093
    
    limits_config:
      enforce_metric_name: false
      reject_old_samples: true
      reject_old_samples_max_age: 168h
      ingestion_rate_mb: 16
      ingestion_burst_size_mb: 32

---
# Loki deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: loki
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: loki
  template:
    metadata:
      labels:
        app: loki
    spec:
      containers:
      - name: loki
        image: grafana/loki:2.8.0
        args:
          - -config.file=/etc/loki/loki.yaml
        ports:
        - containerPort: 3100
        resources:
          requests:
            memory: "512Mi"
            cpu: "200m"
          limits:
            memory: "1Gi"
            cpu: "500m"
        volumeMounts:
        - name: loki-config
          mountPath: /etc/loki
        - name: loki-storage
          mountPath: /loki
      volumes:
      - name: loki-config
        configMap:
          name: loki-config
      - name: loki-storage
        persistentVolumeClaim:
          claimName: loki-pvc
```

### Promtail Configuration
```yaml
# promtail-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: promtail-config
  namespace: monitoring
data:
  promtail.yaml: |
    server:
      http_listen_port: 9080
      grpc_listen_port: 0
    
    positions:
      filename: /tmp/positions.yaml
    
    clients:
      - url: http://loki:3100/loki/api/v1/push
    
    scrape_configs:
      - job_name: kubernetes-pods
        kubernetes_sd_configs:
          - role: pod
        relabel_configs:
          - source_labels:
              - __meta_kubernetes_pod_controller_name
            regex: ([0-9a-z-.]+?)(-[0-9a-f]{8,10})?
            action: replace
            target_label: __tmp_controller_name
          - source_labels:
              - __meta_kubernetes_pod_label_app_kubernetes_io_name
              - __meta_kubernetes_pod_label_app
              - __tmp_controller_name
              - __meta_kubernetes_pod_name
            regex: ^;*([^;]+)(;.*)?$
            action: replace
            target_label: app
          - source_labels:
              - __meta_kubernetes_pod_label_app_kubernetes_io_instance
              - __meta_kubernetes_pod_label_release
            regex: ^;*([^;]+)(;.*)?$
            action: replace
            target_label: instance
          - source_labels:
              - __meta_kubernetes_pod_label_app_kubernetes_io_component
              - __meta_kubernetes_pod_label_component
            regex: ^;*([^;]+)(;.*)?$
            action: replace
            target_label: component
          - action: replace
            source_labels:
            - __meta_kubernetes_pod_node_name
            target_label: node_name
          - action: replace
            source_labels:
            - __meta_kubernetes_namespace
            target_label: namespace
          - action: replace
            replacement: /var/log/pods/*$1/*.log
            separator: /
            source_labels:
            - __meta_kubernetes_pod_uid
            - __meta_kubernetes_pod_container_name
            target_label: __path__
        pipeline_stages:
          - cri: {}
          - json:
              expressions:
                level: level
                timestamp: timestamp
                message: message
          - timestamp:
              source: timestamp
              format: RFC3339Nano
          - labels:
              level:

---
# Promtail DaemonSet
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: promtail
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: promtail
  template:
    metadata:
      labels:
        app: promtail
    spec:
      serviceAccountName: promtail
      containers:
      - name: promtail
        image: grafana/promtail:2.8.0
        args:
          - -config.file=/etc/promtail/promtail.yaml
        ports:
        - containerPort: 9080
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
        volumeMounts:
        - name: promtail-config
          mountPath: /etc/promtail
        - name: varlog
          mountPath: /var/log
          readOnly: true
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      volumes:
      - name: promtail-config
        configMap:
          name: promtail-config
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
      tolerations:
      - effect: NoSchedule
        operator: Exists
```

## Application Metrics Integration

### Node.js Metrics Setup
```javascript
// metrics/prometheus.js
const promClient = require('prom-client');
const express = require('express');

// Create a Registry to register the metrics
const register = new promClient.Registry();

// Add default metrics
promClient.collectDefaultMetrics({
  register,
  prefix: 'dafnckmachine_',
});

// Custom metrics
const httpRequestDuration = new promClient.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status'],
  buckets: [0.1, 0.3, 0.5, 0.7, 1, 3, 5, 7, 10],
});

const httpRequestTotal = new promClient.Counter({
  name: 'http_requests_total',
  help: 'Total number of HTTP requests',
  labelNames: ['method', 'route', 'status'],
});

const activeConnections = new promClient.Gauge({
  name: 'active_connections',
  help: 'Number of active connections',
});

const databaseQueryDuration = new promClient.Histogram({
  name: 'database_query_duration_seconds',
  help: 'Duration of database queries in seconds',
  labelNames: ['query_type', 'table'],
  buckets: [0.01, 0.05, 0.1, 0.3, 0.5, 1, 3, 5],
});

// Register custom metrics
register.registerMetric(httpRequestDuration);
register.registerMetric(httpRequestTotal);
register.registerMetric(activeConnections);
register.registerMetric(databaseQueryDuration);

// Middleware to collect HTTP metrics
const metricsMiddleware = (req, res, next) => {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;
    const route = req.route ? req.route.path : req.path;
    
    httpRequestDuration
      .labels(req.method, route, res.statusCode)
      .observe(duration);
    
    httpRequestTotal
      .labels(req.method, route, res.statusCode)
      .inc();
  });
  
  next();
};

// Metrics endpoint
const metricsApp = express();
metricsApp.get('/metrics', async (req, res) => {
  res.set('Content-Type', register.contentType);
  res.end(await register.metrics());
});

module.exports = {
  register,
  metricsMiddleware,
  metricsApp,
  metrics: {
    httpRequestDuration,
    httpRequestTotal,
    activeConnections,
    databaseQueryDuration,
  },
};
```

### Database Metrics
```javascript
// metrics/database.js
const { metrics } = require('./prometheus');

class DatabaseMetrics {
  static trackQuery(queryType, table, duration) {
    metrics.databaseQueryDuration
      .labels(queryType, table)
      .observe(duration);
  }
  
  static async withMetrics(queryType, table, queryFn) {
    const start = Date.now();
    try {
      const result = await queryFn();
      const duration = (Date.now() - start) / 1000;
      this.trackQuery(queryType, table, duration);
      return result;
    } catch (error) {
      const duration = (Date.now() - start) / 1000;
      this.trackQuery(`${queryType}_error`, table, duration);
      throw error;
    }
  }
}

module.exports = DatabaseMetrics;
```

## Health Checks and SLIs

### Health Check Endpoint
```javascript
// health/healthcheck.js
const express = require('express');
const { Pool } = require('pg');
const redis = require('redis');

const router = express.Router();

// Database connection check
async function checkDatabase() {
  const pool = new Pool({
    connectionString: process.env.DATABASE_URL,
  });
  
  try {
    const client = await pool.connect();
    await client.query('SELECT 1');
    client.release();
    return { status: 'healthy', latency: Date.now() };
  } catch (error) {
    return { status: 'unhealthy', error: error.message };
  }
}

// Redis connection check
async function checkRedis() {
  const client = redis.createClient({
    url: process.env.REDIS_URL,
  });
  
  try {
    await client.connect();
    await client.ping();
    await client.disconnect();
    return { status: 'healthy' };
  } catch (error) {
    return { status: 'unhealthy', error: error.message };
  }
}

// External service check
async function checkExternalServices() {
  const checks = [];
  
  // Check external API
  try {
    const response = await fetch('https://api.external-service.com/health');
    checks.push({
      service: 'external-api',
      status: response.ok ? 'healthy' : 'unhealthy',
      statusCode: response.status,
    });
  } catch (error) {
    checks.push({
      service: 'external-api',
      status: 'unhealthy',
      error: error.message,
    });
  }
  
  return checks;
}

// Health endpoint
router.get('/health', async (req, res) => {
  const health = {
    status: 'healthy',
    timestamp: new Date().toISOString(),
    version: process.env.APP_VERSION || 'unknown',
    checks: {},
  };
  
  try {
    // Run all health checks
    const [database, redis, external] = await Promise.all([
      checkDatabase(),
      checkRedis(),
      checkExternalServices(),
    ]);
    
    health.checks.database = database;
    health.checks.redis = redis;
    health.checks.external = external;
    
    // Determine overall health
    const isHealthy = 
      database.status === 'healthy' &&
      redis.status === 'healthy' &&
      external.every(check => check.status === 'healthy');
    
    health.status = isHealthy ? 'healthy' : 'unhealthy';
    
    res.status(isHealthy ? 200 : 503).json(health);
  } catch (error) {
    health.status = 'unhealthy';
    health.error = error.message;
    res.status(503).json(health);
  }
});

// Readiness endpoint
router.get('/ready', async (req, res) => {
  try {
    // Check if application is ready to serve traffic
    const database = await checkDatabase();
    
    if (database.status === 'healthy') {
      res.status(200).json({ status: 'ready' });
    } else {
      res.status(503).json({ status: 'not ready', reason: 'database unavailable' });
    }
  } catch (error) {
    res.status(503).json({ status: 'not ready', error: error.message });
  }
});

// Liveness endpoint
router.get('/live', (req, res) => {
  res.status(200).json({ status: 'alive' });
});

module.exports = router;
```

## SLI/SLO Configuration

### Service Level Indicators
```yaml
# sli-slo-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: sli-slo-config
  namespace: monitoring
data:
  sli-rules.yml: |
    groups:
    - name: sli.rules
      interval: 30s
      rules:
      # Availability SLI
      - record: sli:availability:rate5m
        expr: |
          sum(rate(http_requests_total{job="dafnckmachine-app",status!~"5.."}[5m])) /
          sum(rate(http_requests_total{job="dafnckmachine-app"}[5m]))
      
      # Latency SLI (95th percentile)
      - record: sli:latency:p95:5m
        expr: |
          histogram_quantile(0.95,
            sum(rate(http_request_duration_seconds_bucket{job="dafnckmachine-app"}[5m])) by (le)
          )
      
      # Error rate SLI
      - record: sli:error_rate:rate5m
        expr: |
          sum(rate(http_requests_total{job="dafnckmachine-app",status=~"5.."}[5m])) /
          sum(rate(http_requests_total{job="dafnckmachine-app"}[5m]))
      
      # Throughput SLI
      - record: sli:throughput:rate5m
        expr: |
          sum(rate(http_requests_total{job="dafnckmachine-app"}[5m]))

  slo-alerts.yml: |
    groups:
    - name: slo.alerts
      rules:
      # SLO: 99.9% availability
      - alert: SLOAvailabilityBreach
        expr: sli:availability:rate5m < 0.999
        for: 2m
        labels:
          severity: critical
          slo: availability
        annotations:
          summary: "Availability SLO breach"
          description: "Availability is {{ $value | humanizePercentage }}, below 99.9% SLO"
      
      # SLO: 95th percentile latency < 500ms
      - alert: SLOLatencyBreach
        expr: sli:latency:p95:5m > 0.5
        for: 5m
        labels:
          severity: warning
          slo: latency
        annotations:
          summary: "Latency SLO breach"
          description: "95th percentile latency is {{ $value }}s, above 500ms SLO"
      
      # SLO: Error rate < 0.1%
      - alert: SLOErrorRateBreach
        expr: sli:error_rate:rate5m > 0.001
        for: 2m
        labels:
          severity: warning
          slo: error_rate
        annotations:
          summary: "Error rate SLO breach"
          description: "Error rate is {{ $value | humanizePercentage }}, above 0.1% SLO"
```

## Troubleshooting and Runbooks

### Common Issues and Solutions
```markdown
# Monitoring Troubleshooting Guide

## High Error Rate Alert

### Symptoms
- Error rate above 5%
- Increased 5xx responses
- User complaints about service unavailability

### Investigation Steps
1. Check application logs for error patterns
2. Verify database connectivity
3. Check external service dependencies
4. Review recent deployments

### Resolution
1. If database issue: Check connection pool, restart if needed
2. If external service issue: Enable circuit breaker
3. If application issue: Rollback recent deployment
4. Scale up resources if needed

## High Latency Alert

### Symptoms
- 95th percentile latency > 500ms
- Slow page load times
- Timeout errors

### Investigation Steps
1. Check database query performance
2. Review application performance metrics
3. Check resource utilization (CPU, memory)
4. Verify network connectivity

### Resolution
1. Optimize slow database queries
2. Scale up application instances
3. Enable caching for frequently accessed data
4. Review and optimize application code

## Service Down Alert

### Symptoms
- Health check failures
- No response from application
- Load balancer marking instances as unhealthy

### Investigation Steps
1. Check pod status in Kubernetes
2. Review application logs
3. Verify resource availability
4. Check network connectivity

### Resolution
1. Restart failed pods
2. Scale up if resource constraints
3. Check and fix configuration issues
4. Verify external dependencies
```

---

**Last Updated**: 2025-01-27  
**Version**: 1.0  
**Related Documents**: 
- [CI/CD Pipeline Implementation](mdc:01_Machine/04_Documentation/Doc/Phase_5_Deployment_PostLaunch/CI_CD_Pipeline_Implementation.md)
- [Infrastructure as Code Framework](mdc:01_Machine/04_Documentation/Doc/Phase_5_Deployment_PostLaunch/Infrastructure_as_Code_Framework.md)
- [Deployment Strategy Configuration](mdc:01_Machine/04_Documentation/Doc/Phase_5_Deployment_PostLaunch/Deployment_Strategy_Configuration.md) 