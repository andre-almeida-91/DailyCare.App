# Infraestrutura e DevOps - DailyCare

## 🏗️ Visão Geral

Este documento detalha a infraestrutura, containerização, orquestração, CI/CD e monitorização do projeto DailyCare.

---

## 🐳 Docker e Containerização

### Docker Compose para Desenvolvimento

```yaml
# docker-compose.dev.yml
version: '3.8'

services:
  # PostgreSQL Master Database
  postgres-master:
    image: postgres:16-alpine
    container_name: dailycare-postgres-master
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: dailycare_master
    ports:
      - "5432:5432"
    volumes:
      - postgres_master_data:/var/lib/postgresql/data
      - ./scripts/init-master.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - dailycare-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Redis
  redis:
    image: redis:7-alpine
    container_name: dailycare-redis
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    command: redis-server --appendonly yes
    networks:
      - dailycare-network
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  # RabbitMQ
  rabbitmq:
    image: rabbitmq:3.12-management-alpine
    container_name: dailycare-rabbitmq
    environment:
      RABBITMQ_DEFAULT_USER: admin
      RABBITMQ_DEFAULT_PASS: admin
    ports:
      - "5672:5672"   # AMQP port
      - "15672:15672" # Management UI
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq
    networks:
      - dailycare-network
    healthcheck:
      test: rabbitmq-diagnostics -q ping
      interval: 30s
      timeout: 10s
      retries: 5

  # Elasticsearch
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    container_name: dailycare-elasticsearch
    environment:
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - xpack.security.enabled=false
    ports:
      - "9200:9200"
    volumes:
      - elasticsearch_data:/usr/share/elasticsearch/data
    networks:
      - dailycare-network
    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost:9200/_cluster/health || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 5

  # Kibana
  kibana:
    image: docker.elastic.co/kibana/kibana:8.11.0
    container_name: dailycare-kibana
    environment:
      ELASTICSEARCH_HOSTS: http://elasticsearch:9200
    ports:
      - "5601:5601"
    depends_on:
      - elasticsearch
    networks:
      - dailycare-network

  # Jaeger (Distributed Tracing)
  jaeger:
    image: jaegertracing/all-in-one:1.52
    container_name: dailycare-jaeger
    environment:
      COLLECTOR_ZIPKIN_HOST_PORT: ":9411"
    ports:
      - "5775:5775/udp"
      - "6831:6831/udp"
      - "6832:6832/udp"
      - "5778:5778"
      - "16686:16686" # UI
      - "14268:14268"
      - "14250:14250"
      - "9411:9411"
    networks:
      - dailycare-network

  # Prometheus
  prometheus:
    image: prom/prometheus:v2.48.0
    container_name: dailycare-prometheus
    volumes:
      - ./infrastructure/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
    ports:
      - "9090:9090"
    networks:
      - dailycare-network

  # Grafana
  grafana:
    image: grafana/grafana:10.2.0
    container_name: dailycare-grafana
    environment:
      GF_SECURITY_ADMIN_PASSWORD: admin
      GF_INSTALL_PLUGINS: grafana-piechart-panel
    volumes:
      - grafana_data:/var/lib/grafana
      - ./infrastructure/grafana/dashboards:/etc/grafana/provisioning/dashboards
      - ./infrastructure/grafana/datasources:/etc/grafana/provisioning/datasources
    ports:
      - "3000:3000"
    depends_on:
      - prometheus
    networks:
      - dailycare-network

volumes:
  postgres_master_data:
  redis_data:
  rabbitmq_data:
  elasticsearch_data:
  prometheus_data:
  grafana_data:

networks:
  dailycare-network:
    driver: bridge
```

### Dockerfile para Microserviços

```dockerfile
# src/backend/services/identity/Dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:9.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM mcr.microsoft.com/dotnet/sdk:9.0 AS build
WORKDIR /src

# Copiar ficheiros de projeto
COPY ["services/identity/DailyCare.Identity.API/DailyCare.Identity.API.csproj", "services/identity/DailyCare.Identity.API/"]
COPY ["services/identity/DailyCare.Identity.Application/DailyCare.Identity.Application.csproj", "services/identity/DailyCare.Identity.Application/"]
COPY ["services/identity/DailyCare.Identity.Domain/DailyCare.Identity.Domain.csproj", "services/identity/DailyCare.Identity.Domain/"]
COPY ["services/identity/DailyCare.Identity.Infrastructure/DailyCare.Identity.Infrastructure.csproj", "services/identity/DailyCare.Identity.Infrastructure/"]
COPY ["shared/DailyCare.Shared.Domain/DailyCare.Shared.Domain.csproj", "shared/DailyCare.Shared.Domain/"]
COPY ["shared/DailyCare.Shared.Infrastructure/DailyCare.Shared.Infrastructure.csproj", "shared/DailyCare.Shared.Infrastructure/"]

# Restore dependencies
RUN dotnet restore "services/identity/DailyCare.Identity.API/DailyCare.Identity.API.csproj"

# Copiar código fonte
COPY . .

# Build
WORKDIR "/src/services/identity/DailyCare.Identity.API"
RUN dotnet build "DailyCare.Identity.API.csproj" -c Release -o /app/build

# Publish
FROM build AS publish
RUN dotnet publish "DailyCare.Identity.API.csproj" -c Release -o /app/publish /p:UseAppHost=false

# Final stage
FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost/health || exit 1

ENTRYPOINT ["dotnet", "DailyCare.Identity.API.dll"]
```

### Dockerfile para Frontend

```dockerfile
# src/frontend/parent-app/Dockerfile
FROM node:20-alpine AS build
WORKDIR /app

# Copiar package files
COPY package*.json ./
COPY pnpm-lock.yaml ./

# Instalar pnpm
RUN npm install -g pnpm

# Instalar dependencies
RUN pnpm install --frozen-lockfile

# Copiar código fonte
COPY . .

# Build da aplicação
RUN pnpm run build

# Nginx para servir a aplicação
FROM nginx:1.25-alpine
COPY --from=build /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

---

## ☸️ Kubernetes

### Namespace

```yaml
# kubernetes/namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dailycare
  labels:
    name: dailycare
```

### ConfigMaps

```yaml
# kubernetes/configmaps/identity-service-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: identity-service-config
  namespace: dailycare
data:
  ASPNETCORE_ENVIRONMENT: "Production"
  Jwt__Issuer: "DailyCare"
  Jwt__Audience: "DailyCare.API"
  Jwt__ExpirationMinutes: "60"
  Redis__Configuration: "redis-service:6379"
  RabbitMQ__Host: "rabbitmq-service"
```

### Secrets

```yaml
# kubernetes/secrets/identity-service-secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: identity-service-secrets
  namespace: dailycare
type: Opaque
stringData:
  ConnectionStrings__DefaultConnection: "Host=postgres-service;Database=dailycare_master;Username=postgres;Password=<PASSWORD>"
  Jwt__Secret: "<JWT_SECRET_KEY_MIN_32_CHARS>"
  RabbitMQ__Username: "admin"
  RabbitMQ__Password: "<RABBITMQ_PASSWORD>"
```

### Deployments

```yaml
# kubernetes/deployments/identity-service.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: identity-service
  namespace: dailycare
  labels:
    app: identity-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: identity-service
  template:
    metadata:
      labels:
        app: identity-service
    spec:
      containers:
      - name: identity-service
        image: dailycare/identity-service:latest
        ports:
        - containerPort: 80
        envFrom:
        - configMapRef:
            name: identity-service-config
        - secretRef:
            name: identity-service-secrets
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health/live
            port: 80
          initialDelaySeconds: 15
          periodSeconds: 20
        readinessProbe:
          httpGet:
            path: /health/ready
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10
```

### Services

```yaml
# kubernetes/services/identity-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: identity-service
  namespace: dailycare
spec:
  selector:
    app: identity-service
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
```

### Ingress

```yaml
# kubernetes/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: dailycare-ingress
  namespace: dailycare
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/rate-limit: "100"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - api.dailycare.pt
    secretName: dailycare-tls
  rules:
  - host: api.dailycare.pt
    http:
      paths:
      - path: /api/identity
        pathType: Prefix
        backend:
          service:
            name: identity-service
            port:
              number: 80
      - path: /api/schools
        pathType: Prefix
        backend:
          service:
            name: school-service
            port:
              number: 80
      # Outros serviços...
```

### HPA (Horizontal Pod Autoscaler)

```yaml
# kubernetes/hpa/identity-service-hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: identity-service-hpa
  namespace: dailycare
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: identity-service
  minReplicas: 2
  maxReplicas: 10
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
```

---

## 🔄 CI/CD Pipeline

### GitHub Actions

```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # Backend - Identity Service
  build-identity-service:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '9.0.x'
      
      - name: Restore dependencies
        run: dotnet restore src/backend/services/identity/DailyCare.Identity.API
      
      - name: Build
        run: dotnet build src/backend/services/identity/DailyCare.Identity.API --no-restore --configuration Release
      
      - name: Run tests
        run: dotnet test tests/Identity.UnitTests --no-build --configuration Release --verbosity normal
      
      - name: Publish
        run: dotnet publish src/backend/services/identity/DailyCare.Identity.API -c Release -o ./publish
      
      - name: Docker meta
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}/identity-service
          tags: |
            type=ref,event=branch
            type=ref,event=pr
            type=semver,pattern={{version}}
            type=sha
      
      - name: Login to GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: src/backend
          file: src/backend/services/identity/Dockerfile
          push: ${{ github.event_name != 'pull_request' }}
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}

  # Frontend - Parent App
  build-parent-app:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - name: Install pnpm
        uses: pnpm/action-setup@v2
        with:
          version: 8
      
      - name: Install dependencies
        working-directory: src/frontend/parent-app
        run: pnpm install --frozen-lockfile
      
      - name: Lint
        working-directory: src/frontend/parent-app
        run: pnpm run lint
      
      - name: Run tests
        working-directory: src/frontend/parent-app
        run: pnpm run test:coverage
      
      - name: Build
        working-directory: src/frontend/parent-app
        run: pnpm run build
        env:
          VITE_API_URL: ${{ secrets.API_URL }}
      
      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: src/frontend/parent-app
          push: ${{ github.event_name != 'pull_request' }}
          tags: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}/parent-app:${{ github.sha }}

  # Code Quality
  sonarcloud:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      
      - name: SonarCloud Scan
        uses: SonarSource/sonarcloud-github-action@master
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

  # Deploy to Development
  deploy-dev:
    needs: [build-identity-service, build-parent-app]
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/develop'
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup kubectl
        uses: azure/setup-kubectl@v3
      
      - name: Configure kubectl
        run: |
          echo "${{ secrets.KUBE_CONFIG_DEV }}" | base64 -d > kubeconfig.yaml
          export KUBECONFIG=kubeconfig.yaml
      
      - name: Deploy to Development
        run: |
          kubectl set image deployment/identity-service \
            identity-service=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}/identity-service:${{ github.sha }} \
            -n dailycare-dev
          kubectl rollout status deployment/identity-service -n dailycare-dev

  # Deploy to Production
  deploy-prod:
    needs: [build-identity-service, build-parent-app]
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment: production
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy to Production
        run: |
          # Deploy steps...
          echo "Deploying to production..."
```

---

## 📊 Monitorização e Observabilidade

### Prometheus Configuration

```yaml
# infrastructure/prometheus/prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'identity-service'
    static_configs:
      - targets: ['identity-service:80']
    metrics_path: '/metrics'
  
  - job_name: 'school-service'
    static_configs:
      - targets: ['school-service:80']
  
  - job_name: 'postgresql'
    static_configs:
      - targets: ['postgres-exporter:9187']
  
  - job_name: 'redis'
    static_configs:
      - targets: ['redis-exporter:9121']

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']

rule_files:
  - 'alerts.yml'
```

### Grafana Dashboards

```json
// infrastructure/grafana/dashboards/api-performance.json
{
  "dashboard": {
    "title": "API Performance",
    "panels": [
      {
        "title": "Request Rate",
        "targets": [
          {
            "expr": "rate(http_requests_total[5m])"
          }
        ]
      },
      {
        "title": "Response Time (p95)",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))"
          }
        ]
      },
      {
        "title": "Error Rate",
        "targets": [
          {
            "expr": "rate(http_requests_total{status=~\"5..\"}[5m])"
          }
        ]
      }
    ]
  }
}
```

### Alertas

```yaml
# infrastructure/prometheus/alerts.yml
groups:
  - name: dailycare_alerts
    rules:
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.05
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Taxa de erros elevada no serviço {{ $labels.job }}"
          description: "{{ $labels.job }} tem uma taxa de erros de {{ $value }}%"
      
      - alert: ServiceDown
        expr: up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Serviço {{ $labels.job }} está down"
          description: "{{ $labels.job }} não responde há mais de 1 minuto"
      
      - alert: HighMemoryUsage
        expr: (node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes > 0.9
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Uso de memória elevado"
          description: "Memória em uso: {{ $value }}%"
```

---

## 🔒 Secrets Management

### Usando Kubernetes Secrets

```bash
# Criar secret a partir de ficheiro
kubectl create secret generic identity-service-secrets \
  --from-file=appsettings.Production.json \
  --namespace dailycare

# Criar secret a partir de valores literais
kubectl create secret generic db-credentials \
  --from-literal=username=postgres \
  --from-literal=password=supersecret \
  --namespace dailycare
```

### Usando Azure Key Vault (Opcional)

```csharp
// Program.cs
builder.Configuration.AddAzureKeyVault(
    new Uri($"https://{builder.Configuration["KeyVaultName"]}.vault.azure.net/"),
    new DefaultAzureCredential());
```

---

## 📝 Health Checks

### Configuração no Backend

```csharp
// Program.cs
builder.Services.AddHealthChecks()
    .AddNpgSql(
        builder.Configuration.GetConnectionString("DefaultConnection"),
        name: "postgres",
        tags: new[] { "db", "sql" })
    .AddRedis(
        builder.Configuration["Redis:Configuration"],
        name: "redis",
        tags: new[] { "cache" })
    .AddRabbitMQ(
        builder.Configuration["RabbitMQ:ConnectionString"],
        name: "rabbitmq",
        tags: new[] { "messaging" })
    .AddCheck<CustomHealthCheck>("custom", tags: new[] { "custom" });

app.MapHealthChecks("/health/live", new HealthCheckOptions
{
    Predicate = _ => false // Apenas verifica se a app está viva
});

app.MapHealthChecks("/health/ready", new HealthCheckOptions
{
    Predicate = check => check.Tags.Contains("db") || check.Tags.Contains("cache")
});

app.MapHealthChecks("/health", new HealthCheckOptions
{
    ResponseWriter = UIResponseWriter.WriteHealthCheckUIResponse
});
```

---

## 🔄 Backup e Disaster Recovery

### Script de Backup Automático

```bash
#!/bin/bash
# scripts/backup-all.sh

DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backups/${DATE}"

mkdir -p $BACKUP_DIR

# Backup Master Database
pg_dump -h localhost -U postgres -d dailycare_master \
    -F c -f "${BACKUP_DIR}/master.backup"

# Backup Tenant Databases
for tenant in $(psql -h localhost -U postgres -d dailycare_master \
    -t -c "SELECT code FROM tenants WHERE is_active = true"); do
    
    DB_NAME="dailycare_tenant_${tenant}"
    pg_dump -h localhost -U postgres -d $DB_NAME \
        -F c -f "${BACKUP_DIR}/${tenant}.backup"
done

# Compactar
tar -czf "${BACKUP_DIR}.tar.gz" $BACKUP_DIR
rm -rf $BACKUP_DIR

# Upload para cloud storage (opcional)
# aws s3 cp "${BACKUP_DIR}.tar.gz" s3://dailycare-backups/

echo "Backup concluído: ${BACKUP_DIR}.tar.gz"
```

---

## 📈 Escalabilidade

### Escalamento Horizontal

- **API Gateway**: 2-5 instâncias
- **Microserviços**: 2-10 instâncias cada
- **PostgreSQL**: Read replicas
- **Redis**: Redis Cluster

### Load Balancing

```nginx
# nginx.conf
upstream identity_service {
    least_conn;
    server identity-service-1:80;
    server identity-service-2:80;
    server identity-service-3:80;
}

server {
    listen 80;
    server_name api.dailycare.pt;

    location /api/identity {
        proxy_pass http://identity_service;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

---

**Anterior**: [API Documentação](./10-API-DOCUMENTACAO.md)  
**Próximo**: [Tarefas de Desenvolvimento](./12-TAREFAS-DESENVOLVIMENTO.md)  
**Voltar ao**: [Índice](./00-INDICE.md)
