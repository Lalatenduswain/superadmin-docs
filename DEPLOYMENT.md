# Deployment Guide

> Production deployment, scaling, and operations guide for the SuperAdmin SaaS platform.

---

## Table of Contents

1. [Docker Compose Production Configuration](#1-docker-compose-production-configuration)
2. [Kubernetes Deployment with Helm](#2-kubernetes-deployment-with-helm)
3. [Environment-Specific Configurations](#3-environment-specific-configurations)
4. [SSL/TLS with Certbot and Traefik](#4-ssltls-with-certbot-and-traefik)
5. [Database Migration Strategy](#5-database-migration-strategy)
6. [Zero-Downtime Deployments](#6-zero-downtime-deployments)
7. [Backup and Restore Procedures](#7-backup-and-restore-procedures)
8. [Scaling Guidelines](#8-scaling-guidelines)
9. [Health Check Endpoints](#9-health-check-endpoints)
10. [Monitoring Setup (Grafana and Prometheus)](#10-monitoring-setup-grafana-and-prometheus)
11. [Cloudflare Workers Edge Deployment](#11-cloudflare-workers-edge-deployment)
12. [CDN and Varnish Cache Configuration](#12-cdn-and-varnish-cache-configuration)

---

## 1. Docker Compose Production Configuration

### Production docker-compose.prod.yml

```yaml
services:
  app:
    image: ${REGISTRY}/superadmin-api:${VERSION:-latest}
    restart: always
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: "2.0"
          memory: 2G
        reservations:
          cpus: "0.5"
          memory: 512M
    environment:
      - NODE_ENV=production
      - DB_HOST=postgres
      - DB_PORT=5432
      - DB_NAME=superadmin_db
      - DB_USER=superadmin
      - DB_PASSWORD_FILE=/run/secrets/db_password
      - DB_SSL=true
      - DB_POOL_MIN=5
      - DB_POOL_MAX=30
      - REDIS_HOST=redis
      - REDIS_PORT=6379
      - REDIS_PASSWORD_FILE=/run/secrets/redis_password
      - REDIS_TLS=true
      - MINIO_ENDPOINT=minio
      - MINIO_PORT=9000
      - MINIO_USE_SSL=true
      - JWT_SECRET_FILE=/run/secrets/jwt_secret
      - ENCRYPTION_KEY_FILE=/run/secrets/encryption_key
    secrets:
      - db_password
      - redis_password
      - jwt_secret
      - encryption_key
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/api/v1/health"]
      interval: 15s
      timeout: 5s
      retries: 3
      start_period: 30s
    networks:
      - backend
      - frontend
    logging:
      driver: "json-file"
      options:
        max-size: "50m"
        max-file: "5"

  postgres:
    image: postgres:18-alpine
    restart: always
    deploy:
      resources:
        limits:
          cpus: "4.0"
          memory: 8G
        reservations:
          cpus: "1.0"
          memory: 2G
    environment:
      - POSTGRES_USER=superadmin
      - POSTGRES_PASSWORD_FILE=/run/secrets/db_password
      - POSTGRES_DB=superadmin_db
    secrets:
      - db_password
    volumes:
      - pgdata:/var/lib/postgresql/data
      - ./config/postgresql.conf:/etc/postgresql/postgresql.conf
      - ./config/pg_hba.conf:/etc/postgresql/pg_hba.conf
    command: postgres -c config_file=/etc/postgresql/postgresql.conf
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U superadmin -d superadmin_db"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - backend
    logging:
      driver: "json-file"
      options:
        max-size: "50m"
        max-file: "3"

  postgres-replica:
    image: postgres:18-alpine
    restart: always
    deploy:
      resources:
        limits:
          cpus: "2.0"
          memory: 4G
    environment:
      - POSTGRES_USER=superadmin
      - POSTGRES_PASSWORD_FILE=/run/secrets/db_password
    secrets:
      - db_password
    volumes:
      - pg-replica-data:/var/lib/postgresql/data
    depends_on:
      postgres:
        condition: service_healthy
    networks:
      - backend

  redis:
    image: redis:7-alpine
    restart: always
    deploy:
      resources:
        limits:
          cpus: "1.0"
          memory: 2G
        reservations:
          cpus: "0.25"
          memory: 256M
    command: >
      redis-server
        --requirepass_file /run/secrets/redis_password
        --appendonly yes
        --appendfsync everysec
        --maxmemory 1536mb
        --maxmemory-policy allkeys-lru
        --tcp-backlog 511
        --timeout 300
        --tcp-keepalive 60
    secrets:
      - redis_password
    volumes:
      - redis-data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "--no-auth-warning", "-a", "$$(cat /run/secrets/redis_password)", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - backend

  redis-sentinel:
    image: redis:7-alpine
    restart: always
    command: redis-sentinel /etc/redis/sentinel.conf
    volumes:
      - ./config/sentinel.conf:/etc/redis/sentinel.conf
    depends_on:
      redis:
        condition: service_healthy
    networks:
      - backend

  minio:
    image: minio/minio
    restart: always
    deploy:
      resources:
        limits:
          cpus: "2.0"
          memory: 4G
    command: server /data --console-address ":9001"
    environment:
      - MINIO_ROOT_USER_FILE=/run/secrets/minio_access_key
      - MINIO_ROOT_PASSWORD_FILE=/run/secrets/minio_secret_key
    secrets:
      - minio_access_key
      - minio_secret_key
    volumes:
      - minio-data:/data
    healthcheck:
      test: ["CMD", "mc", "ready", "local"]
      interval: 15s
      timeout: 5s
      retries: 3
    networks:
      - backend

  traefik:
    image: traefik:v3.2
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./config/traefik.yml:/etc/traefik/traefik.yml:ro
      - ./config/traefik-dynamic.yml:/etc/traefik/dynamic/config.yml:ro
      - traefik-certs:/letsencrypt
    networks:
      - frontend
    logging:
      driver: "json-file"
      options:
        max-size: "20m"
        max-file: "3"

secrets:
  db_password:
    file: ./secrets/db_password.txt
  redis_password:
    file: ./secrets/redis_password.txt
  jwt_secret:
    file: ./secrets/jwt_secret.txt
  encryption_key:
    file: ./secrets/encryption_key.txt
  minio_access_key:
    file: ./secrets/minio_access_key.txt
  minio_secret_key:
    file: ./secrets/minio_secret_key.txt

volumes:
  pgdata:
  pg-replica-data:
  redis-data:
  minio-data:
  traefik-certs:

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true
```

### Generate Secret Files

```bash
mkdir -p secrets
openssl rand -hex 32 > secrets/db_password.txt
openssl rand -hex 32 > secrets/redis_password.txt
openssl rand -hex 64 > secrets/jwt_secret.txt
openssl rand -hex 32 > secrets/encryption_key.txt
echo "minioadmin" > secrets/minio_access_key.txt
openssl rand -hex 32 > secrets/minio_secret_key.txt

# Restrict permissions
chmod 600 secrets/*.txt
```

### Deploy

```bash
docker compose -f docker-compose.prod.yml up -d

# View logs
docker compose -f docker-compose.prod.yml logs -f app

# Scale the API
docker compose -f docker-compose.prod.yml up -d --scale app=5
```

---

## 2. Kubernetes Deployment with Helm

### Helm Chart Structure

```
k8s/helm/superadmin/
├── Chart.yaml
├── values.yaml
├── values-staging.yaml
├── values-production.yaml
├── templates/
│   ├── _helpers.tpl
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── hpa.yaml
│   ├── pdb.yaml
│   ├── configmap.yaml
│   ├── secret.yaml
│   ├── serviceaccount.yaml
│   ├── migration-job.yaml
│   ├── cronjob-backup.yaml
│   └── networkpolicy.yaml
└── charts/
    ├── postgresql/
    ├── redis/
    └── minio/
```

### Deployment Manifest

```yaml
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "superadmin.fullname" . }}-api
  labels:
    {{- include "superadmin.labels" . | nindent 4 }}
    app.kubernetes.io/component: api
spec:
  replicas: {{ .Values.api.replicas }}
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1
  selector:
    matchLabels:
      {{- include "superadmin.selectorLabels" . | nindent 6 }}
      app.kubernetes.io/component: api
  template:
    metadata:
      labels:
        {{- include "superadmin.selectorLabels" . | nindent 8 }}
        app.kubernetes.io/component: api
      annotations:
        checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}
    spec:
      serviceAccountName: {{ include "superadmin.serviceAccountName" . }}
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        fsGroup: 1000
      containers:
        - name: api
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: http
              containerPort: 3000
              protocol: TCP
          env:
            - name: NODE_ENV
              value: production
            - name: DB_HOST
              valueFrom:
                configMapKeyRef:
                  name: {{ include "superadmin.fullname" . }}-config
                  key: db-host
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: {{ include "superadmin.fullname" . }}-db
                  key: password
            - name: REDIS_HOST
              valueFrom:
                configMapKeyRef:
                  name: {{ include "superadmin.fullname" . }}-config
                  key: redis-host
            - name: REDIS_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: {{ include "superadmin.fullname" . }}-redis
                  key: password
            - name: JWT_SECRET
              valueFrom:
                secretKeyRef:
                  name: {{ include "superadmin.fullname" . }}-jwt
                  key: secret
            - name: ENCRYPTION_KEY
              valueFrom:
                secretKeyRef:
                  name: {{ include "superadmin.fullname" . }}-encryption
                  key: key
          resources:
            requests:
              cpu: 250m
              memory: 512Mi
            limits:
              cpu: "2"
              memory: 2Gi
          livenessProbe:
            httpGet:
              path: /api/v1/health
              port: http
            initialDelaySeconds: 30
            periodSeconds: 15
            timeoutSeconds: 5
            failureThreshold: 3
          readinessProbe:
            httpGet:
              path: /api/v1/health
              port: http
            initialDelaySeconds: 10
            periodSeconds: 5
            timeoutSeconds: 3
            failureThreshold: 3
          startupProbe:
            httpGet:
              path: /api/v1/health
              port: http
            initialDelaySeconds: 5
            periodSeconds: 5
            failureThreshold: 12
      topologySpreadConstraints:
        - maxSkew: 1
          topologyKey: kubernetes.io/hostname
          whenUnsatisfiable: DoNotSchedule
          labelSelector:
            matchLabels:
              {{- include "superadmin.selectorLabels" . | nindent 14 }}
```

### Horizontal Pod Autoscaler

```yaml
# templates/hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: {{ include "superadmin.fullname" . }}-api
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: {{ include "superadmin.fullname" . }}-api
  minReplicas: {{ .Values.api.autoscaling.minReplicas }}
  maxReplicas: {{ .Values.api.autoscaling.maxReplicas }}
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
        - type: Pods
          value: 2
          periodSeconds: 60
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
        - type: Pods
          value: 1
          periodSeconds: 120
```

### Pod Disruption Budget

```yaml
# templates/pdb.yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: {{ include "superadmin.fullname" . }}-api
spec:
  minAvailable: 2
  selector:
    matchLabels:
      {{- include "superadmin.selectorLabels" . | nindent 6 }}
      app.kubernetes.io/component: api
```

### Network Policy

```yaml
# templates/networkpolicy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: {{ include "superadmin.fullname" . }}-api
spec:
  podSelector:
    matchLabels:
      {{- include "superadmin.selectorLabels" . | nindent 6 }}
      app.kubernetes.io/component: api
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              name: ingress-nginx
      ports:
        - protocol: TCP
          port: 3000
  egress:
    - to:
        - podSelector:
            matchLabels:
              app.kubernetes.io/name: postgresql
      ports:
        - protocol: TCP
          port: 5432
    - to:
        - podSelector:
            matchLabels:
              app.kubernetes.io/name: redis
      ports:
        - protocol: TCP
          port: 6379
    - to:
        - podSelector:
            matchLabels:
              app.kubernetes.io/name: minio
      ports:
        - protocol: TCP
          port: 9000
    - to:  # DNS
        - namespaceSelector: {}
      ports:
        - protocol: UDP
          port: 53
        - protocol: TCP
          port: 53
```

### Install with Helm

```bash
# Create namespace
kubectl create namespace superadmin

# Create secrets
kubectl create secret generic superadmin-db \
  --from-literal=password="$(openssl rand -hex 32)" \
  -n superadmin

kubectl create secret generic superadmin-redis \
  --from-literal=password="$(openssl rand -hex 32)" \
  -n superadmin

kubectl create secret generic superadmin-jwt \
  --from-literal=secret="$(openssl rand -hex 64)" \
  -n superadmin

kubectl create secret generic superadmin-encryption \
  --from-literal=key="$(openssl rand -hex 32)" \
  -n superadmin

# Install
helm install superadmin ./k8s/helm/superadmin \
  -n superadmin \
  -f k8s/helm/superadmin/values-production.yaml

# Upgrade
helm upgrade superadmin ./k8s/helm/superadmin \
  -n superadmin \
  -f k8s/helm/superadmin/values-production.yaml
```

---

## 3. Environment-Specific Configurations

### Development

```yaml
# values-development.yaml
api:
  replicas: 1
  autoscaling:
    enabled: false
  resources:
    requests:
      cpu: 100m
      memory: 256Mi
    limits:
      cpu: 500m
      memory: 1Gi

postgresql:
  primary:
    persistence:
      size: 5Gi
  readReplicas:
    replicaCount: 0

redis:
  master:
    persistence:
      size: 1Gi
  replica:
    replicaCount: 0

config:
  logLevel: debug
  rateLimitMultiplier: 10
  corsOrigins: "http://localhost:5173,http://localhost:3000"
```

### Staging

```yaml
# values-staging.yaml
api:
  replicas: 2
  autoscaling:
    enabled: true
    minReplicas: 2
    maxReplicas: 4
  resources:
    requests:
      cpu: 250m
      memory: 512Mi
    limits:
      cpu: "1"
      memory: 2Gi

postgresql:
  primary:
    persistence:
      size: 20Gi
  readReplicas:
    replicaCount: 1

redis:
  master:
    persistence:
      size: 2Gi
  replica:
    replicaCount: 1

config:
  logLevel: info
  rateLimitMultiplier: 2
  corsOrigins: "https://staging.{DOMAIN}"
```

### Production

```yaml
# values-production.yaml
api:
  replicas: 3
  autoscaling:
    enabled: true
    minReplicas: 3
    maxReplicas: 20
  resources:
    requests:
      cpu: 500m
      memory: 1Gi
    limits:
      cpu: "2"
      memory: 4Gi

postgresql:
  primary:
    persistence:
      size: 100Gi
      storageClass: ssd
  readReplicas:
    replicaCount: 2

redis:
  master:
    persistence:
      size: 8Gi
  replica:
    replicaCount: 2
  sentinel:
    enabled: true

config:
  logLevel: warn
  rateLimitMultiplier: 1
  corsOrigins: "https://{DOMAIN}"

ingress:
  enabled: true
  className: nginx
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/rate-limit: "100"
    nginx.ingress.kubernetes.io/rate-limit-window: "1m"
  hosts:
    - host: "{DOMAIN}"
      paths:
        - path: /api
          pathType: Prefix
  tls:
    - secretName: superadmin-tls
      hosts:
        - "{DOMAIN}"
```

---

## 4. SSL/TLS with Certbot and Traefik

### Traefik Configuration

```yaml
# config/traefik.yml
entryPoints:
  web:
    address: ":80"
    http:
      redirections:
        entryPoint:
          to: websecure
          scheme: https
  websecure:
    address: ":443"
    http:
      tls:
        certResolver: letsencrypt
    http2:
      maxConcurrentStreams: 250

certificatesResolvers:
  letsencrypt:
    acme:
      email: admin@{DOMAIN}
      storage: /letsencrypt/acme.json
      httpChallenge:
        entryPoint: web

providers:
  docker:
    exposedByDefault: false
    network: frontend
  file:
    directory: /etc/traefik/dynamic
    watch: true

api:
  dashboard: false

log:
  level: WARN

accessLog:
  filePath: /var/log/traefik/access.log
  bufferingSize: 100
  filters:
    statusCodes:
      - "400-599"
```

### Dynamic Configuration

```yaml
# config/traefik-dynamic.yml
http:
  routers:
    superadmin-api:
      rule: "Host(`{DOMAIN}`) && PathPrefix(`/api`)"
      service: superadmin-api
      entryPoints:
        - websecure
      tls:
        certResolver: letsencrypt
      middlewares:
        - security-headers
        - rate-limit

  services:
    superadmin-api:
      loadBalancer:
        servers:
          - url: "http://app:3000"
        healthCheck:
          path: /api/v1/health
          interval: 10s
          timeout: 3s

  middlewares:
    security-headers:
      headers:
        stsSeconds: 63072000
        stsIncludeSubdomains: true
        stsPreload: true
        forceSTSHeader: true
        contentTypeNosniff: true
        frameDeny: true
        browserXssFilter: true
        referrerPolicy: "strict-origin-when-cross-origin"
        contentSecurityPolicy: "default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'"
        customResponseHeaders:
          X-Robots-Tag: "noindex, nofollow"
          Permissions-Policy: "camera=(), microphone=(), geolocation=()"

    rate-limit:
      rateLimit:
        average: 100
        burst: 200
        period: 1m

tls:
  options:
    default:
      minVersion: VersionTLS13
      sniStrict: true
```

### Certbot Standalone (Without Traefik)

```bash
# Install Certbot
sudo apt install -y certbot

# Obtain certificate
sudo certbot certonly --standalone \
  -d {DOMAIN} \
  -d www.{DOMAIN} \
  --email admin@{DOMAIN} \
  --agree-tos \
  --no-eff-email

# Auto-renewal cron (runs twice daily)
echo "0 0,12 * * * root certbot renew --quiet --deploy-hook 'systemctl reload traefik'" | sudo tee /etc/cron.d/certbot-renew

# Verify renewal
sudo certbot renew --dry-run
```

---

## 5. Database Migration Strategy

### Migration Workflow

```
Development → Staging → Production
    ↓            ↓          ↓
 Write SQL   Test with    Run with
 migrations  prod-like    zero-downtime
             data volume  strategy
```

### Migration Commands

```bash
# Create a new migration
npm run db:migration:create -- --name add_feature_flags_table

# Run pending migrations
npm run db:migrate

# Roll back the last migration
npm run db:rollback

# Roll back all migrations
npm run db:rollback:all

# Check migration status
npm run db:status
```

### Zero-Downtime Migration Rules

To keep migrations compatible with zero-downtime deployments, follow these rules:

1. **Never drop a column in the same release that removes the code using it.** First release: stop writing to the column. Second release: drop the column.

2. **Never rename a column directly.** Instead: add the new column, backfill data, update code to use the new column, then drop the old column in a subsequent release.

3. **Always add new columns as nullable or with a default value.**

4. **Add indexes concurrently:**
   ```sql
   CREATE INDEX CONCURRENTLY idx_users_email ON users (email);
   ```

5. **Use advisory locks to prevent concurrent migration runs:**
   ```sql
   SELECT pg_advisory_lock(12345);
   -- run migration
   SELECT pg_advisory_unlock(12345);
   ```

### Kubernetes Migration Job

```yaml
# templates/migration-job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: {{ include "superadmin.fullname" . }}-migrate-{{ .Release.Revision }}
  annotations:
    "helm.sh/hook": pre-upgrade,pre-install
    "helm.sh/hook-weight": "-5"
    "helm.sh/hook-delete-policy": before-hook-creation
spec:
  backoffLimit: 3
  activeDeadlineSeconds: 600
  template:
    spec:
      restartPolicy: Never
      containers:
        - name: migrate
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          command: ["npm", "run", "db:migrate"]
          env:
            - name: DB_HOST
              valueFrom:
                configMapKeyRef:
                  name: {{ include "superadmin.fullname" . }}-config
                  key: db-host
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: {{ include "superadmin.fullname" . }}-db
                  key: password
```

---

## 6. Zero-Downtime Deployments

### Rolling Update Strategy

The Kubernetes deployment uses a rolling update with `maxUnavailable: 0` and `maxSurge: 1`, meaning new pods are created before old ones are terminated.

### Deployment Sequence

1. **Pre-deploy:** Run database migrations (Helm pre-upgrade hook)
2. **Deploy:** Kubernetes rolls out new pods one at a time
3. **Health check:** New pods must pass startup, liveness, and readiness probes
4. **Drain:** Old pods receive SIGTERM, stop accepting new connections, and finish in-flight requests
5. **Cleanup:** Old pods are terminated after graceful shutdown period

### Graceful Shutdown

The application handles SIGTERM by:

1. Stopping the HTTP server from accepting new connections
2. Waiting for in-flight requests to complete (30-second timeout)
3. Closing database connection pool
4. Closing Redis connections
5. Flushing pending audit logs
6. Exiting with code 0

```typescript
// Configured in the application
const GRACEFUL_SHUTDOWN_TIMEOUT = 30_000; // 30 seconds
```

### Blue-Green Deployment (Alternative)

For major releases, use blue-green deployment:

```bash
# Deploy to green environment
helm install superadmin-green ./k8s/helm/superadmin \
  -n superadmin \
  -f values-production.yaml \
  --set ingress.enabled=false

# Verify green is healthy
kubectl exec -it deploy/superadmin-green-api -n superadmin -- \
  curl -s http://localhost:3000/api/v1/health

# Switch traffic to green
kubectl patch ingress superadmin -n superadmin \
  --type='json' \
  -p='[{"op": "replace", "path": "/spec/rules/0/http/paths/0/backend/service/name", "value":"superadmin-green-api"}]'

# Remove blue environment after verification
helm uninstall superadmin-blue -n superadmin
```

---

## 7. Backup and Restore Procedures

### Automated Database Backups

#### Kubernetes CronJob

```yaml
# templates/cronjob-backup.yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: {{ include "superadmin.fullname" . }}-backup
spec:
  schedule: "0 2 * * *"  # Daily at 2:00 AM
  concurrencyPolicy: Forbid
  successfulJobsHistoryLimit: 7
  failedJobsHistoryLimit: 3
  jobTemplate:
    spec:
      backoffLimit: 2
      activeDeadlineSeconds: 3600
      template:
        spec:
          restartPolicy: OnFailure
          containers:
            - name: backup
              image: postgres:18-alpine
              command:
                - /bin/sh
                - -c
                - |
                  TIMESTAMP=$(date +%Y%m%d_%H%M%S)
                  FILENAME="superadmin_backup_${TIMESTAMP}.sql.gz"
                  pg_dump -h $DB_HOST -U $DB_USER -d $DB_NAME \
                    --no-owner --no-privileges --clean --if-exists \
                    | gzip > /tmp/${FILENAME}
                  mc alias set backup http://$MINIO_HOST:9000 $MINIO_ACCESS_KEY $MINIO_SECRET_KEY
                  mc cp /tmp/${FILENAME} backup/superadmin-backups/${FILENAME}
                  mc ls backup/superadmin-backups/ | head -n -30 | awk '{print $NF}' | \
                    xargs -I {} mc rm backup/superadmin-backups/{}
                  echo "Backup completed: ${FILENAME}"
              env:
                - name: DB_HOST
                  valueFrom:
                    configMapKeyRef:
                      name: {{ include "superadmin.fullname" . }}-config
                      key: db-host
                - name: DB_USER
                  value: superadmin
                - name: DB_NAME
                  value: superadmin_db
                - name: PGPASSWORD
                  valueFrom:
                    secretKeyRef:
                      name: {{ include "superadmin.fullname" . }}-db
                      key: password
                - name: MINIO_HOST
                  valueFrom:
                    configMapKeyRef:
                      name: {{ include "superadmin.fullname" . }}-config
                      key: minio-host
                - name: MINIO_ACCESS_KEY
                  valueFrom:
                    secretKeyRef:
                      name: {{ include "superadmin.fullname" . }}-minio
                      key: access-key
                - name: MINIO_SECRET_KEY
                  valueFrom:
                    secretKeyRef:
                      name: {{ include "superadmin.fullname" . }}-minio
                      key: secret-key
```

### Manual Backup

```bash
# Full backup with compression
pg_dump -h localhost -U superadmin -d superadmin_db \
  --no-owner --no-privileges --clean --if-exists \
  | gzip > superadmin_backup_$(date +%Y%m%d_%H%M%S).sql.gz

# Upload to MinIO
mc cp superadmin_backup_*.sql.gz local/superadmin-backups/
```

### Restore Procedure

```bash
# Download the backup
mc cp local/superadmin-backups/superadmin_backup_20260327_020000.sql.gz ./

# Decompress
gunzip superadmin_backup_20260327_020000.sql.gz

# Restore (this drops and recreates objects)
psql -h localhost -U superadmin -d superadmin_db \
  < superadmin_backup_20260327_020000.sql

# Run any pending migrations after restore
npm run db:migrate

# Verify data integrity
npm run db:verify
```

### Backup Retention Policy

| Type | Frequency | Retention |
|------|-----------|-----------|
| Full database | Daily at 2:00 AM | 30 days |
| WAL archives | Continuous | 7 days |
| Configuration | On every change | Indefinite |
| Audit logs | Monthly archive | Per compliance requirement (default: 7 years) |

### Redis Backup

Redis AOF persistence is enabled by default. For explicit snapshots:

```bash
# Trigger RDB snapshot
redis-cli -a $REDIS_PASSWORD BGSAVE

# Copy the dump file
docker cp superadmin-redis:/data/dump.rdb ./redis_backup_$(date +%Y%m%d).rdb
```

---

## 8. Scaling Guidelines

### Horizontal Scaling

#### API Servers

The API is stateless (sessions are stored in Redis), so horizontal scaling is straightforward:

| Users | API Pods | CPU per Pod | Memory per Pod |
|-------|----------|-------------|----------------|
| Up to 500 | 2 | 500m | 1 Gi |
| 500-2,000 | 4 | 1 | 2 Gi |
| 2,000-10,000 | 8 | 2 | 4 Gi |
| 10,000-50,000 | 16 | 2 | 4 Gi |
| 50,000+ | 20+ | 4 | 8 Gi |

#### PostgreSQL Read Replicas

Add read replicas to handle read-heavy workloads:

```yaml
postgresql:
  readReplicas:
    replicaCount: 3
    resources:
      requests:
        cpu: "1"
        memory: 2Gi
      limits:
        cpu: "4"
        memory: 8Gi
```

Route read queries to replicas in the application connection config:

```env
DB_READ_HOST=postgres-read
DB_WRITE_HOST=postgres-primary
```

#### Redis Cluster

For high-throughput caching scenarios:

```yaml
redis:
  architecture: replication
  replica:
    replicaCount: 3
  sentinel:
    enabled: true
    quorum: 2
```

### Vertical Scaling

#### PostgreSQL Tuning

Scale `shared_buffers` to 25% of available RAM and `effective_cache_size` to 75%:

| Server RAM | shared_buffers | effective_cache_size | work_mem | maintenance_work_mem |
|-----------|---------------|---------------------|---------|---------------------|
| 8 GB | 2 GB | 6 GB | 32 MB | 512 MB |
| 16 GB | 4 GB | 12 GB | 64 MB | 1 GB |
| 32 GB | 8 GB | 24 GB | 128 MB | 2 GB |
| 64 GB | 16 GB | 48 GB | 256 MB | 4 GB |

#### Node.js Cluster Mode

The application can run in cluster mode to utilize multiple CPU cores:

```env
CLUSTER_ENABLED=true
CLUSTER_WORKERS=auto  # Uses os.cpus().length
```

### Connection Pool Sizing

| Component | Formula | Example (8 pods) |
|-----------|---------|-------------------|
| DB pool per pod | `max_connections / (pods * 1.5)` | `200 / 12 ≈ 16` |
| Redis connections per pod | 10-20 | 15 |
| Total DB connections | pods * pool_max | 128 |

---

## 9. Health Check Endpoints

### Public Health Check

```
GET /api/v1/health
```

Returns basic availability status. Use this for load balancer health checks and uptime monitoring.

```json
{
  "status": "healthy",
  "version": "1.0.0",
  "uptime": 864000,
  "timestamp": "2026-03-27T10:30:00.000Z"
}
```

Possible statuses: `healthy`, `degraded`, `unhealthy`.

### Detailed Health Check (Authenticated)

```
GET /api/v1/monitoring/system-health
```

Returns component-level status with latency measurements. Use this for monitoring dashboards.

### Kubernetes Probes

| Probe | Endpoint | Purpose |
|-------|----------|---------|
| Startup | `/api/v1/health` | Determine when the pod is ready to accept traffic |
| Liveness | `/api/v1/health` | Restart the pod if it becomes unresponsive |
| Readiness | `/api/v1/health` | Remove the pod from service during issues |

### Health Check Logic

The health endpoint checks:

1. **Database:** Executes `SELECT 1` with a 5-second timeout
2. **Redis:** Executes `PING` with a 3-second timeout
3. **MinIO:** Lists buckets with a 5-second timeout
4. **Memory:** Checks process memory against the configured limit
5. **Disk:** Checks available disk space (warns below 10%)

If any critical component (database, Redis) fails, the status is `unhealthy`. If non-critical components (MinIO, SMTP) fail, the status is `degraded`.

---

## 10. Monitoring Setup (Grafana and Prometheus)

### Prometheus Configuration

```yaml
# config/prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  - "alerts/*.yml"

scrape_configs:
  - job_name: "superadmin-api"
    metrics_path: /api/v1/metrics
    bearer_token_file: /etc/prometheus/token
    static_configs:
      - targets:
          - "app:3000"
    relabel_configs:
      - source_labels: [__address__]
        target_label: instance

  - job_name: "postgresql"
    static_configs:
      - targets:
          - "postgres-exporter:9187"

  - job_name: "redis"
    static_configs:
      - targets:
          - "redis-exporter:9121"

  - job_name: "minio"
    metrics_path: /minio/v2/metrics/cluster
    static_configs:
      - targets:
          - "minio:9000"

  - job_name: "traefik"
    static_configs:
      - targets:
          - "traefik:8080"

  - job_name: "node-exporter"
    static_configs:
      - targets:
          - "node-exporter:9100"
```

### Application Metrics Exposed

The application exposes Prometheus-compatible metrics at `/api/v1/metrics`:

| Metric | Type | Description |
|--------|------|-------------|
| `http_requests_total` | Counter | Total HTTP requests by method, path, status |
| `http_request_duration_seconds` | Histogram | Request latency distribution |
| `http_active_connections` | Gauge | Current active connections |
| `db_pool_active_connections` | Gauge | Active database connections |
| `db_pool_idle_connections` | Gauge | Idle database connections |
| `db_query_duration_seconds` | Histogram | Database query latency |
| `redis_commands_total` | Counter | Redis commands executed |
| `redis_latency_seconds` | Histogram | Redis command latency |
| `auth_login_total` | Counter | Login attempts by status (success/failure) |
| `auth_mfa_verifications_total` | Counter | MFA verification attempts |
| `active_sessions_total` | Gauge | Current active user sessions |
| `rate_limit_hits_total` | Counter | Rate limit trigger count |
| `threats_detected_total` | Counter | Security threats detected by type |
| `file_uploads_total` | Counter | File upload count by status |
| `webhook_deliveries_total` | Counter | Webhook deliveries by status |

### Alert Rules

```yaml
# config/alerts/superadmin.yml
groups:
  - name: superadmin
    rules:
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m]) > 0.05
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High API error rate (> 5%)"
          description: "Error rate is {{ $value | humanizePercentage }} over the last 5 minutes."

      - alert: HighResponseLatency
        expr: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) > 2
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High API response latency (p95 > 2s)"

      - alert: DatabaseConnectionPoolExhausted
        expr: db_pool_active_connections / db_pool_active_connections > 0.9
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "Database connection pool > 90% utilized"

      - alert: HighFailedLoginRate
        expr: rate(auth_login_total{status="failure"}[15m]) > 10
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Elevated failed login rate (possible brute force)"

      - alert: RedisDown
        expr: up{job="redis"} == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Redis is unreachable"

      - alert: DiskSpaceLow
        expr: node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"} < 0.1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Disk space below 10%"
```

### Grafana Dashboard Setup

1. Deploy Grafana alongside Prometheus:

```yaml
# In docker-compose.prod.yml
grafana:
  image: grafana/grafana:11.4
  restart: always
  ports:
    - "3001:3000"
  environment:
    - GF_SECURITY_ADMIN_PASSWORD_FILE=/run/secrets/grafana_password
    - GF_USERS_ALLOW_SIGN_UP=false
  secrets:
    - grafana_password
  volumes:
    - grafana-data:/var/lib/grafana
    - ./config/grafana/provisioning:/etc/grafana/provisioning
    - ./config/grafana/dashboards:/var/lib/grafana/dashboards
  networks:
    - backend
    - frontend
```

2. Provision the Prometheus data source:

```yaml
# config/grafana/provisioning/datasources/prometheus.yml
apiVersion: 1
datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
    editable: false
```

3. Recommended dashboard panels:
   - Request rate and error rate over time
   - Response latency percentiles (p50, p95, p99)
   - Active database and Redis connections
   - Login success/failure rate
   - Active user sessions
   - Threat detection events
   - Rate limit trigger frequency
   - System resources (CPU, memory, disk)
   - Backup status and duration

---

## 11. Cloudflare Workers Edge Deployment

### Edge Functions

Deploy edge logic using Cloudflare Workers for low-latency operations:

```typescript
// workers/rate-limiter.ts
export default {
  async fetch(request: Request, env: Env): Promise<Response> {
    const clientIp = request.headers.get('CF-Connecting-IP') || 'unknown';
    const key = `rate:${clientIp}:${new URL(request.url).pathname}`;

    const current = await env.RATE_LIMIT_KV.get(key);
    const count = current ? parseInt(current, 10) : 0;

    if (count >= env.RATE_LIMIT_MAX) {
      return new Response(JSON.stringify({
        success: false,
        error: { code: 'RATE_LIMIT_EXCEEDED', message: 'Too many requests' }
      }), {
        status: 429,
        headers: {
          'Content-Type': 'application/json',
          'Retry-After': '60',
          'X-RateLimit-Limit': String(env.RATE_LIMIT_MAX),
          'X-RateLimit-Remaining': '0',
        }
      });
    }

    await env.RATE_LIMIT_KV.put(key, String(count + 1), { expirationTtl: 60 });

    const response = await fetch(request);

    const newResponse = new Response(response.body, response);
    newResponse.headers.set('X-RateLimit-Limit', String(env.RATE_LIMIT_MAX));
    newResponse.headers.set('X-RateLimit-Remaining', String(env.RATE_LIMIT_MAX - count - 1));

    return newResponse;
  },
};
```

### Wrangler Configuration

```toml
# wrangler.toml
name = "superadmin-edge"
main = "workers/rate-limiter.ts"
compatibility_date = "2026-03-01"

[vars]
RATE_LIMIT_MAX = "100"

[[kv_namespaces]]
binding = "RATE_LIMIT_KV"
id = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"

[env.production]
routes = [
  { pattern = "{DOMAIN}/api/*", zone_name = "{DOMAIN}" }
]
```

### Deploy

```bash
# Install Wrangler
npm install -g wrangler

# Authenticate
wrangler login

# Deploy to production
wrangler deploy --env production
```

### Cloudflare Tunnel (Alternative to Port Exposure)

```bash
# Install cloudflared
sudo apt install cloudflared

# Authenticate
cloudflared tunnel login

# Create tunnel
cloudflared tunnel create superadmin

# Configure
cat > ~/.cloudflared/config.yml << 'EOF'
tunnel: {TUNNEL_ID}
credentials-file: /root/.cloudflared/{TUNNEL_ID}.json

ingress:
  - hostname: {DOMAIN}
    service: http://localhost:3000
  - hostname: grafana.{DOMAIN}
    service: http://localhost:3001
  - service: http_status:404
EOF

# Run tunnel
cloudflared tunnel run superadmin
```

---

## 12. CDN and Varnish Cache Configuration

### Varnish Configuration

Deploy Varnish as a caching layer in front of the API for read-heavy endpoints:

```vcl
# config/varnish/default.vcl
vcl 4.1;

backend default {
    .host = "app";
    .port = "3000";
    .connect_timeout = 5s;
    .first_byte_timeout = 30s;
    .between_bytes_timeout = 10s;
    .probe = {
        .url = "/api/v1/health";
        .interval = 10s;
        .timeout = 3s;
        .window = 5;
        .threshold = 3;
    }
}

sub vcl_recv {
    # Only cache GET and HEAD requests
    if (req.method != "GET" && req.method != "HEAD") {
        return (pass);
    }

    # Never cache authenticated requests
    if (req.http.Authorization || req.http.Cookie ~ "session") {
        return (pass);
    }

    # Never cache admin endpoints
    if (req.url ~ "^/api/v1/(auth|users|tenants|config|database)") {
        return (pass);
    }

    # Cache public endpoints
    if (req.url ~ "^/api/v1/health") {
        return (hash);
    }

    # Cache license tiers (changes infrequently)
    if (req.url ~ "^/api/v1/licenses/tiers") {
        return (hash);
    }

    # Cache static assets
    if (req.url ~ "\.(css|js|png|jpg|jpeg|gif|ico|svg|woff2|ttf)$") {
        return (hash);
    }

    return (pass);
}

sub vcl_backend_response {
    # Cache health check for 10 seconds
    if (bereq.url ~ "^/api/v1/health") {
        set beresp.ttl = 10s;
        set beresp.grace = 30s;
    }

    # Cache license tiers for 1 hour
    if (bereq.url ~ "^/api/v1/licenses/tiers") {
        set beresp.ttl = 3600s;
        set beresp.grace = 600s;
    }

    # Cache static assets for 1 day
    if (bereq.url ~ "\.(css|js|png|jpg|jpeg|gif|ico|svg|woff2|ttf)$") {
        set beresp.ttl = 86400s;
        set beresp.grace = 3600s;
    }

    # Do not cache error responses
    if (beresp.status >= 400) {
        set beresp.uncacheable = true;
        set beresp.ttl = 0s;
    }

    return (deliver);
}

sub vcl_deliver {
    # Add cache hit/miss header for debugging
    if (obj.hits > 0) {
        set resp.http.X-Cache = "HIT";
        set resp.http.X-Cache-Hits = obj.hits;
    } else {
        set resp.http.X-Cache = "MISS";
    }

    # Remove internal headers
    unset resp.http.X-Powered-By;
    unset resp.http.Server;
    unset resp.http.X-Varnish;
    unset resp.http.Via;
}
```

### Varnish Docker Service

```yaml
# Add to docker-compose.prod.yml
varnish:
  image: varnish:7.6
  restart: always
  ports:
    - "8080:80"
  volumes:
    - ./config/varnish/default.vcl:/etc/varnish/default.vcl:ro
  command: >
    -a :80
    -f /etc/varnish/default.vcl
    -s malloc,1G
    -p default_ttl=0
    -p default_grace=300
  depends_on:
    app:
      condition: service_healthy
  networks:
    - frontend
    - backend
```

### CDN Configuration

#### Cache-Control Headers

The application sets appropriate cache headers based on endpoint type:

| Endpoint Type | Cache-Control Header |
|--------------|---------------------|
| Health check | `public, max-age=10` |
| Public static | `public, max-age=86400, immutable` |
| License tiers | `public, max-age=3600` |
| Authenticated API | `private, no-cache, no-store, must-revalidate` |
| User data | `private, no-store` |
| Audit logs | `private, no-store` |

#### CDN Integration

When using a CDN (Cloudflare, CloudFront, Fastly), configure:

1. **Origin pull:** Point the CDN to Traefik or Varnish
2. **Cache rules:** Respect the application's `Cache-Control` headers
3. **Bypass rules:** Skip cache for requests with `Authorization` header
4. **Purge API:** Integrate CDN purge with the configuration change webhook

```bash
# Cloudflare cache purge example
curl -X POST "https://api.cloudflare.com/client/v4/zones/{ZONE_ID}/purge_cache" \
  -H "Authorization: Bearer {CF_API_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{"purge_everything": true}'
```

### Performance Benchmarks

Expected performance targets with the full caching stack:

| Metric | Without Cache | With Varnish | With CDN |
|--------|--------------|-------------|----------|
| Health check latency | 5-10 ms | 0.5-1 ms | 1-5 ms (edge) |
| API p95 latency | 100-200 ms | 50-100 ms | 20-50 ms (cached) |
| Throughput (single pod) | 2,000 rps | 10,000 rps | 50,000+ rps |
| Static asset latency | 20-50 ms | 1-5 ms | 5-20 ms (global) |

---

## Additional References

- See `INFRASTRUCTURE_STACK.md` for the complete 29-technology infrastructure blueprint
- See `INSTALLATION.md` for initial setup and development environment
- See `TROUBLESHOOTING.md` for runtime issue resolution
- See Section 36 of `SUPER_ADMIN_DOCUMENTATION.md` for performance optimization details
- See Section 38 for infrastructure security hardening
