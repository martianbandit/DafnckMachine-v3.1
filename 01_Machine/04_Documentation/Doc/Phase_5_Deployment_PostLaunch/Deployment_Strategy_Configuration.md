# Deployment Strategy Configuration

## Overview
This document provides comprehensive guidance for implementing deployment strategies for DafnckMachine v3.1, covering blue-green deployments, canary releases, rolling updates, and zero-downtime deployment patterns with automated rollback capabilities.

## Blue-Green Deployment Strategy

### Architecture Overview
```yaml
# Blue-Green deployment configuration
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: dafnckmachine-rollout
  namespace: production
spec:
  replicas: 5
  strategy:
    blueGreen:
      activeService: dafnckmachine-active
      previewService: dafnckmachine-preview
      autoPromotionEnabled: false
      scaleDownDelaySeconds: 30
      prePromotionAnalysis:
        templates:
        - templateName: success-rate
        args:
        - name: service-name
          value: dafnckmachine-preview
      postPromotionAnalysis:
        templates:
        - templateName: success-rate
        args:
        - name: service-name
          value: dafnckmachine-active
  selector:
    matchLabels:
      app: dafnckmachine
  template:
    metadata:
      labels:
        app: dafnckmachine
    spec:
      containers:
      - name: dafnckmachine
        image: dafnckmachine:latest
        ports:
        - containerPort: 3000
        env:
        - name: NODE_ENV
          value: "production"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
```

### Service Configuration
```yaml
# Active service
apiVersion: v1
kind: Service
metadata:
  name: dafnckmachine-active
  namespace: production
spec:
  selector:
    app: dafnckmachine
  ports:
  - port: 80
    targetPort: 3000
    protocol: TCP
  type: ClusterIP

---
# Preview service
apiVersion: v1
kind: Service
metadata:
  name: dafnckmachine-preview
  namespace: production
spec:
  selector:
    app: dafnckmachine
  ports:
  - port: 80
    targetPort: 3000
    protocol: TCP
  type: ClusterIP

---
# Load balancer service
apiVersion: v1
kind: Service
metadata:
  name: dafnckmachine-lb
  namespace: production
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
spec:
  selector:
    app: dafnckmachine
  ports:
  - port: 80
    targetPort: 3000
    protocol: TCP
  type: LoadBalancer
```

### Traffic Switching
```bash
#!/bin/bash
# scripts/blue-green-switch.sh

set -e

NAMESPACE=${1:-production}
NEW_VERSION=${2}

echo "Starting Blue-Green deployment switch to version: $NEW_VERSION"

# Update the rollout with new image
kubectl argo rollouts set image dafnckmachine-rollout \
  dafnckmachine=dafnckmachine:$NEW_VERSION \
  -n $NAMESPACE

# Wait for rollout to be ready
kubectl argo rollouts get rollout dafnckmachine-rollout \
  -n $NAMESPACE --watch

# Run health checks on preview environment
echo "Running health checks on preview environment..."
PREVIEW_URL=$(kubectl get svc dafnckmachine-preview -n $NAMESPACE -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')

# Health check loop
for i in {1..10}; do
  if curl -f http://$PREVIEW_URL/health; then
    echo "Health check $i passed"
  else
    echo "Health check $i failed"
    exit 1
  fi
  sleep 10
done

# Promote the rollout
echo "Promoting rollout to active..."
kubectl argo rollouts promote dafnckmachine-rollout -n $NAMESPACE

echo "Blue-Green deployment completed successfully"
```

## Canary Deployment Strategy

### Canary Configuration
```yaml
# Canary deployment configuration
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: dafnckmachine-canary
  namespace: production
spec:
  replicas: 10
  strategy:
    canary:
      steps:
      - setWeight: 10
      - pause: {duration: 2m}
      - setWeight: 20
      - pause: {duration: 2m}
      - setWeight: 40
      - pause: {duration: 2m}
      - setWeight: 60
      - pause: {duration: 2m}
      - setWeight: 80
      - pause: {duration: 2m}
      canaryService: dafnckmachine-canary
      stableService: dafnckmachine-stable
      trafficRouting:
        istio:
          virtualService:
            name: dafnckmachine-vs
            routes:
            - primary
      analysis:
        templates:
        - templateName: success-rate
        - templateName: latency
        args:
        - name: service-name
          value: dafnckmachine-canary
        startingStep: 2
        interval: 30s
  selector:
    matchLabels:
      app: dafnckmachine
  template:
    metadata:
      labels:
        app: dafnckmachine
    spec:
      containers:
      - name: dafnckmachine
        image: dafnckmachine:latest
        ports:
        - containerPort: 3000
```

### Istio Virtual Service
```yaml
# Istio traffic routing
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: dafnckmachine-vs
  namespace: production
spec:
  hosts:
  - dafnckmachine.com
  http:
  - name: primary
    match:
    - headers:
        canary:
          exact: "true"
    route:
    - destination:
        host: dafnckmachine-canary
        port:
          number: 80
      weight: 100
  - name: stable
    route:
    - destination:
        host: dafnckmachine-stable
        port:
          number: 80
      weight: 100
    - destination:
        host: dafnckmachine-canary
        port:
          number: 80
      weight: 0
```

### Analysis Templates
```yaml
# Success rate analysis
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata:
  name: success-rate
  namespace: production
spec:
  args:
  - name: service-name
  metrics:
  - name: success-rate
    interval: 30s
    count: 5
    successCondition: result[0] >= 0.95
    failureLimit: 3
    provider:
      prometheus:
        address: http://prometheus.monitoring.svc.cluster.local:9090
        query: |
          sum(rate(http_requests_total{service="{{args.service-name}}",status!~"5.."}[2m])) /
          sum(rate(http_requests_total{service="{{args.service-name}}"}[2m]))

---
# Latency analysis
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata:
  name: latency
  namespace: production
spec:
  args:
  - name: service-name
  metrics:
  - name: latency
    interval: 30s
    count: 5
    successCondition: result[0] <= 500
    failureLimit: 3
    provider:
      prometheus:
        address: http://prometheus.monitoring.svc.cluster.local:9090
        query: |
          histogram_quantile(0.95,
            sum(rate(http_request_duration_seconds_bucket{service="{{args.service-name}}"}[2m])) by (le)
          ) * 1000
```

## Rolling Update Strategy

### Rolling Update Configuration
```yaml
# Rolling update deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dafnckmachine-rolling
  namespace: production
spec:
  replicas: 6
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 2
  selector:
    matchLabels:
      app: dafnckmachine
      deployment: rolling
  template:
    metadata:
      labels:
        app: dafnckmachine
        deployment: rolling
    spec:
      containers:
      - name: dafnckmachine
        image: dafnckmachine:latest
        ports:
        - containerPort: 3000
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
      terminationGracePeriodSeconds: 30
```

### Pod Disruption Budget
```yaml
# Pod disruption budget
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: dafnckmachine-pdb
  namespace: production
spec:
  minAvailable: 4
  selector:
    matchLabels:
      app: dafnckmachine
```

## Automated Rollback Configuration

### Rollback Automation
```yaml
# Rollback automation script
apiVersion: v1
kind: ConfigMap
metadata:
  name: rollback-automation
  namespace: production
data:
  rollback.sh: |
    #!/bin/bash
    set -e
    
    DEPLOYMENT_NAME=${1:-dafnckmachine-rollout}
    NAMESPACE=${2:-production}
    THRESHOLD=${3:-0.95}
    
    echo "Monitoring deployment: $DEPLOYMENT_NAME"
    
    # Monitor success rate for 5 minutes
    for i in {1..10}; do
      SUCCESS_RATE=$(kubectl exec -n monitoring deployment/prometheus -- \
        promtool query instant \
        'sum(rate(http_requests_total{service="'$DEPLOYMENT_NAME'",status!~"5.."}[2m])) / sum(rate(http_requests_total{service="'$DEPLOYMENT_NAME'"}[2m]))' \
        | grep -oP '\d+\.\d+' | head -1)
      
      echo "Current success rate: $SUCCESS_RATE"
      
      if (( $(echo "$SUCCESS_RATE < $THRESHOLD" | bc -l) )); then
        echo "Success rate below threshold. Initiating rollback..."
        kubectl argo rollouts abort $DEPLOYMENT_NAME -n $NAMESPACE
        kubectl argo rollouts undo $DEPLOYMENT_NAME -n $NAMESPACE
        exit 1
      fi
      
      sleep 30
    done
    
    echo "Deployment monitoring completed successfully"

---
# Rollback CronJob
apiVersion: batch/v1
kind: CronJob
metadata:
  name: deployment-monitor
  namespace: production
spec:
  schedule: "*/5 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: monitor
            image: bitnami/kubectl:latest
            command:
            - /bin/bash
            - -c
            - |
              # Check if any rollouts are in progress
              ROLLOUTS=$(kubectl argo rollouts list -n production -o json | jq -r '.[] | select(.status.phase == "Progressing") | .metadata.name')
              
              for rollout in $ROLLOUTS; do
                echo "Monitoring rollout: $rollout"
                /scripts/rollback.sh $rollout production 0.95
              done
            volumeMounts:
            - name: scripts
              mountPath: /scripts
          volumes:
          - name: scripts
            configMap:
              name: rollback-automation
              defaultMode: 0755
          restartPolicy: OnFailure
```

## Feature Flags Integration

### Feature Flag Configuration
```yaml
# Feature flag service
apiVersion: v1
kind: ConfigMap
metadata:
  name: feature-flags
  namespace: production
data:
  flags.json: |
    {
      "newFeature": {
        "enabled": false,
        "rolloutPercentage": 0,
        "conditions": {
          "userSegment": "beta",
          "region": "us-west-2"
        }
      },
      "improvedAlgorithm": {
        "enabled": true,
        "rolloutPercentage": 25,
        "conditions": {}
      }
    }

---
# Feature flag deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: feature-flag-service
  namespace: production
spec:
  replicas: 2
  selector:
    matchLabels:
      app: feature-flag-service
  template:
    metadata:
      labels:
        app: feature-flag-service
    spec:
      containers:
      - name: feature-flags
        image: flagsmith/flagsmith:latest
        ports:
        - containerPort: 8000
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: feature-flags-secret
              key: database-url
        volumeMounts:
        - name: config
          mountPath: /app/config
      volumes:
      - name: config
        configMap:
          name: feature-flags
```

## Database Migration Strategy

### Migration Job Configuration
```yaml
# Database migration job
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migration-{{ .Values.version }}
  namespace: production
spec:
  template:
    spec:
      containers:
      - name: migration
        image: dafnckmachine:{{ .Values.version }}
        command:
        - npm
        - run
        - migrate
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: database-secret
              key: url
        - name: MIGRATION_MODE
          value: "safe"
      restartPolicy: Never
  backoffLimit: 3

---
# Migration validation
apiVersion: batch/v1
kind: Job
metadata:
  name: db-validation-{{ .Values.version }}
  namespace: production
spec:
  template:
    spec:
      containers:
      - name: validation
        image: dafnckmachine:{{ .Values.version }}
        command:
        - npm
        - run
        - validate-schema
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: database-secret
              key: url
      restartPolicy: Never
  backoffLimit: 1
```

### Backward Compatibility Check
```bash
#!/bin/bash
# scripts/compatibility-check.sh

set -e

OLD_VERSION=${1}
NEW_VERSION=${2}

echo "Checking backward compatibility between $OLD_VERSION and $NEW_VERSION"

# Run compatibility tests
docker run --rm \
  -e DATABASE_URL=$DATABASE_URL \
  -e OLD_VERSION=$OLD_VERSION \
  -e NEW_VERSION=$NEW_VERSION \
  dafnckmachine:$NEW_VERSION \
  npm run test:compatibility

# Check API compatibility
docker run --rm \
  -v $(pwd)/api-tests:/tests \
  postman/newman:latest \
  run /tests/compatibility-tests.json \
  --environment /tests/production.json

echo "Compatibility check completed successfully"
```

## Monitoring and Alerting

### Deployment Metrics
```yaml
# Deployment monitoring
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: deployment-metrics
  namespace: production
spec:
  selector:
    matchLabels:
      app: dafnckmachine
  endpoints:
  - port: metrics
    interval: 30s
    path: /metrics

---
# Deployment alerts
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: deployment-alerts
  namespace: production
spec:
  groups:
  - name: deployment
    rules:
    - alert: DeploymentFailed
      expr: kube_deployment_status_replicas_unavailable > 0
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "Deployment has unavailable replicas"
        description: "Deployment {{ $labels.deployment }} has {{ $value }} unavailable replicas"
    
    - alert: HighErrorRate
      expr: rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m]) > 0.05
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: "High error rate detected"
        description: "Error rate is {{ $value | humanizePercentage }}"
    
    - alert: HighLatency
      expr: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) > 0.5
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "High latency detected"
        description: "95th percentile latency is {{ $value }}s"
```

## Deployment Automation Scripts

### Automated Deployment Pipeline
```bash
#!/bin/bash
# scripts/automated-deployment.sh

set -e

VERSION=${1}
STRATEGY=${2:-blue-green}
ENVIRONMENT=${3:-production}

echo "Starting automated deployment of version $VERSION using $STRATEGY strategy"

# Pre-deployment checks
echo "Running pre-deployment checks..."
./scripts/pre-deployment-checks.sh $VERSION $ENVIRONMENT

# Database migrations
echo "Running database migrations..."
kubectl apply -f k8s/migration-job.yaml
kubectl wait --for=condition=complete job/db-migration-$VERSION -n $ENVIRONMENT --timeout=300s

# Deploy based on strategy
case $STRATEGY in
  "blue-green")
    echo "Executing Blue-Green deployment..."
    ./scripts/blue-green-deploy.sh $VERSION $ENVIRONMENT
    ;;
  "canary")
    echo "Executing Canary deployment..."
    ./scripts/canary-deploy.sh $VERSION $ENVIRONMENT
    ;;
  "rolling")
    echo "Executing Rolling deployment..."
    ./scripts/rolling-deploy.sh $VERSION $ENVIRONMENT
    ;;
  *)
    echo "Unknown deployment strategy: $STRATEGY"
    exit 1
    ;;
esac

# Post-deployment validation
echo "Running post-deployment validation..."
./scripts/post-deployment-validation.sh $VERSION $ENVIRONMENT

echo "Deployment completed successfully"
```

### Rollback Script
```bash
#!/bin/bash
# scripts/emergency-rollback.sh

set -e

DEPLOYMENT=${1:-dafnckmachine-rollout}
NAMESPACE=${2:-production}

echo "EMERGENCY ROLLBACK: Rolling back $DEPLOYMENT in $NAMESPACE"

# Immediate rollback
kubectl argo rollouts abort $DEPLOYMENT -n $NAMESPACE
kubectl argo rollouts undo $DEPLOYMENT -n $NAMESPACE

# Wait for rollback to complete
kubectl argo rollouts get rollout $DEPLOYMENT -n $NAMESPACE --watch

# Verify rollback
echo "Verifying rollback..."
for i in {1..10}; do
  if curl -f https://api.dafnckmachine.com/health; then
    echo "Health check passed after rollback"
    break
  fi
  sleep 10
done

# Send notifications
curl -X POST $SLACK_WEBHOOK_URL \
  -H 'Content-type: application/json' \
  --data '{"text":"🚨 Emergency rollback completed for '$DEPLOYMENT' in '$NAMESPACE'"}'

echo "Emergency rollback completed"
```

## Best Practices

### Deployment Guidelines
- Always run database migrations before application deployment
- Use feature flags for gradual feature rollouts
- Implement comprehensive health checks
- Monitor key metrics during deployments
- Have automated rollback triggers

### Testing Strategy
- Run compatibility tests between versions
- Validate database schema changes
- Test rollback procedures regularly
- Monitor performance impact
- Verify feature flag functionality

### Security Considerations
- Scan container images for vulnerabilities
- Validate secrets and configurations
- Implement network policies
- Monitor for security anomalies
- Audit deployment activities

## Troubleshooting

### Common Issues
1. **Deployment Stuck**: Check resource limits and node capacity
2. **Health Check Failures**: Verify application startup time and dependencies
3. **Traffic Routing Issues**: Check service selectors and ingress configuration
4. **Rollback Failures**: Verify previous version availability and compatibility

### Debug Commands
```bash
# Check rollout status
kubectl argo rollouts get rollout dafnckmachine-rollout -n production

# View deployment events
kubectl get events -n production --sort-by='.lastTimestamp'

# Check pod logs
kubectl logs -f deployment/dafnckmachine -n production

# Verify service endpoints
kubectl get endpoints dafnckmachine-active -n production

# Test connectivity
kubectl run debug --image=busybox -it --rm -- wget -qO- http://dafnckmachine-active/health
```

---

**Last Updated**: 2025-01-27  
**Version**: 1.0  
**Related Documents**: 
- [CI/CD Pipeline Implementation](mdc:01_Machine/04_Documentation/Doc/Phase_5_Deployment_PostLaunch/CI_CD_Pipeline_Implementation.md)
- [Infrastructure as Code Framework](mdc:01_Machine/04_Documentation/Doc/Phase_5_Deployment_PostLaunch/Infrastructure_as_Code_Framework.md)
- [Monitoring Alerting Setup](mdc:01_Machine/04_Documentation/Doc/Phase_5_Deployment_PostLaunch/Monitoring_Alerting_Setup.md) 