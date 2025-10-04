# Arquitetura do Sistema DailyCare

## 📐 Visão Geral da Arquitetura

O DailyCare foi projetado com uma arquitetura moderna, escalável e resiliente, baseada em microserviços. A arquitetura garante isolamento de dados (multi-tenancy), alta disponibilidade e facilidade de manutenção.

---

## 🏗️ Arquitetura de Alto Nível

```
┌─────────────────────────────────────────────────────────────────┐
│                         Camada de Cliente                        │
├─────────────────────────────────────────────────────────────────┤
│  ┌──────────────────┐         ┌──────────────────┐             │
│  │  Frontend Pais   │         │  Frontend Admin  │             │
│  │   (React SPA)    │         │   (React SPA)    │             │
│  └──────────────────┘         └──────────────────┘             │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      API Gateway / BFF                           │
├─────────────────────────────────────────────────────────────────┤
│  • Roteamento de requisições                                    │
│  • Autenticação e autorização                                   │
│  • Rate limiting                                                 │
│  • Agregação de dados                                           │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Camada de Microserviços                       │
├─────────────────────────────────────────────────────────────────┤
│  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌──────────┐ │
│  │ Identity   │  │ Activities │  │ Health     │  │Children  │ │
│  │ Service    │  │ Service    │  │ Service    │  │Service   │ │
│  └────────────┘  └────────────┘  └────────────┘  └──────────┘ │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌──────────┐ │
│  │ School     │  │ Notification│  │ Reports    │  │Files     │ │
│  │ Service    │  │ Service    │  │ Service    │  │Service   │ │
│  └────────────┘  └────────────┘  └────────────┘  └──────────┘ │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      Camada de Dados                             │
├─────────────────────────────────────────────────────────────────┤
│  ┌──────────────────┐         ┌──────────────────┐             │
│  │   PostgreSQL     │         │      Redis       │             │
│  │  (Multi-Tenant)  │         │     (Cache)      │             │
│  └──────────────────┘         └──────────────────┘             │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔧 Componentes Principais

### 1. Camada de Cliente

#### 1.1 Frontend para Pais (Parent App)
- **Tecnologia**: React 18+ com TypeScript
- **UI Framework**: Material-UI (MUI) + Tailwind CSS
- **Estado**: Redux Toolkit + RTK Query
- **Responsabilidades**:
  - Interface para pais/encarregados de educação
  - Visualização de atividades e relatórios
  - Acesso a informações das crianças
  - Recepção de notificações

#### 1.2 Frontend Administrativo (Admin Panel)
- **Tecnologia**: React 18+ com TypeScript
- **UI Framework**: Material-UI (MUI) + Tailwind CSS
- **Estado**: Redux Toolkit + RTK Query
- **Responsabilidades**:
  - Gestão de escolas e configurações
  - Administração de utilizadores
  - Gestão de dados mestre
  - Relatórios administrativos

### 2. API Gateway / BFF (Backend for Frontend)

- **Tecnologia**: .NET 9 com Ocelot ou YARP
- **Responsabilidades**:
  - Roteamento de requisições para microserviços
  - Autenticação centralizada (JWT)
  - Rate limiting e throttling
  - Agregação de dados de múltiplos serviços
  - Transformação de dados para o frontend
  - Gestão de CORS
  - Logging centralizado

### 3. Camada de Microserviços

#### 3.1 Identity Service (Serviço de Identidade)
- **Responsabilidades**:
  - Autenticação de utilizadores
  - Gestão de tokens JWT
  - Gestão de permissões e roles
  - Refresh tokens
  - Gestão de sessões
- **Base de dados**: PostgreSQL (schema: identity)
- **Cache**: Redis para tokens e sessões

#### 3.2 School Service (Serviço de Escolas)
- **Responsabilidades**:
  - Gestão de escolas (tenants)
  - Provisionamento de bases de dados
  - Gestão de turmas
  - Configurações da escola
  - Gestão de educadores
- **Base de dados**: PostgreSQL (schema: schools)

#### 3.3 Children Service (Serviço de Crianças)
- **Responsabilidades**:
  - Gestão de perfis de crianças
  - Informações demográficas
  - Relações familiares
  - Histórico de matrículas
- **Base de dados**: PostgreSQL por tenant (schema: children)

#### 3.4 Activities Service (Serviço de Atividades)
- **Responsabilidades**:
  - Registo de alimentação
  - Registo de sono
  - Registo de fraldas
  - Registo de atividades físicas
  - Marcos de desenvolvimento
- **Base de dados**: PostgreSQL por tenant (schema: activities)
- **Cache**: Redis para atividades recentes

#### 3.5 Health Service (Serviço de Saúde)
- **Responsabilidades**:
  - Registo de temperatura e sintomas
  - Gestão de medicamentos
  - Histórico de vacinas
  - Consultas médicas
  - Alergias
- **Base de dados**: PostgreSQL por tenant (schema: health)

#### 3.6 Notification Service (Serviço de Notificações)
- **Responsabilidades**:
  - Envio de notificações push
  - Envio de emails
  - Gestão de preferências de notificação
  - Templates de mensagens
  - Histórico de notificações
- **Tecnologia**: SignalR para real-time
- **Fila**: RabbitMQ ou Azure Service Bus

#### 3.7 Reports Service (Serviço de Relatórios)
- **Responsabilidades**:
  - Geração de relatórios
  - Gráficos e dashboards
  - Análises estatísticas
  - Exportação de dados (PDF, Excel)
- **Cache**: Redis para relatórios pré-calculados

#### 3.8 Files Service (Serviço de Ficheiros)
- **Responsabilidades**:
  - Upload de fotos e documentos
  - Armazenamento de ficheiros
  - Geração de thumbnails
  - Gestão de quotas
- **Armazenamento**: Sistema de ficheiros ou object storage (MinIO)

### 4. Camada de Dados

#### 4.1 PostgreSQL
- **Estratégia Multi-Tenant**: Base de dados por tenant
- **Master Database**: Gestão de tenants e roteamento
- **Tenant Databases**: Dados isolados por escola
- **Migrations**: EF Core Migrations
- **Backup**: Automatizado e incremental

#### 4.2 Redis
- **Uso**:
  - Cache de dados frequentemente acedidos
  - Cache de sessões
  - Cache de tokens
  - Rate limiting
  - Pub/Sub para eventos
- **Estratégia**: Cache-aside pattern

---

## 🔄 Padrões Arquiteturais

### 1. Domain-Driven Design (DDD)

Cada microserviço segue os princípios de DDD:

```
Service/
├── Domain/
│   ├── Entities/          # Entidades de domínio
│   ├── ValueObjects/      # Objetos de valor
│   ├── Aggregates/        # Agregados
│   ├── Events/            # Eventos de domínio
│   └── Interfaces/        # Interfaces de repositórios
├── Application/
│   ├── Commands/          # Comandos (CQRS)
│   ├── Queries/           # Queries (CQRS)
│   ├── DTOs/              # Data Transfer Objects
│   ├── Validators/        # Validadores (FluentValidation)
│   └── Services/          # Serviços de aplicação
├── Infrastructure/
│   ├── Persistence/       # EF Core, repositórios
│   ├── ExternalServices/  # Integração com serviços externos
│   └── Messaging/         # Mensageria
└── API/
    ├── Controllers/       # Endpoints REST
    ├── Middleware/        # Middleware customizado
    └── Filters/           # Filtros de ação
```

### 2. CQRS (Command Query Responsibility Segregation)

- **Commands**: Operações que alteram estado
- **Queries**: Operações de leitura
- **Benefícios**:
  - Otimização independente de leitura e escrita
  - Escalabilidade
  - Clareza de código

### 3. Event Sourcing (Opcional para eventos críticos)

- Armazenamento de eventos de domínio
- Reconstrução de estado a partir de eventos
- Auditoria completa

### 4. Repository Pattern

- Abstração da camada de dados
- Facilita testes unitários
- Desacoplamento

### 5. Unit of Work Pattern

- Gestão transacional
- Consistência de dados
- Rollback automático em caso de erro

---

## 🔐 Multi-Tenancy

### Estratégia: Database per Tenant

```
┌─────────────────────────────────────────────┐
│         Master Database (PostgreSQL)         │
├─────────────────────────────────────────────┤
│  • Registo de tenants                       │
│  • Connection strings                       │
│  • Configurações globais                    │
│  • Utilizadores e roles                     │
└─────────────────────────────────────────────┘
                    │
        ┌───────────┼───────────┐
        │           │           │
        ▼           ▼           ▼
┌──────────┐  ┌──────────┐  ┌──────────┐
│ Tenant 1 │  │ Tenant 2 │  │ Tenant N │
│   DB     │  │   DB     │  │   DB     │
└──────────┘  └──────────┘  └──────────┘
```

**Vantagens**:
- Isolamento completo de dados
- Facilidade de backup por escola
- Compliance com RGPD
- Performance otimizada por tenant

**Implementação**:
- Middleware de resolução de tenant
- Connection string dinâmica por requisição
- Pool de conexões por tenant

---

## 📡 Comunicação entre Serviços

### 1. Síncrona (HTTP/REST)

- **Quando usar**: Operações que requerem resposta imediata
- **Tecnologia**: HTTP/REST com retry e circuit breaker
- **Biblioteca**: Polly para resiliência

### 2. Assíncrona (Mensageria)

- **Quando usar**: Operações que podem ser processadas posteriormente
- **Tecnologia**: RabbitMQ ou Azure Service Bus
- **Padrões**:
  - Publish/Subscribe
  - Request/Reply
  - Dead Letter Queue

### 3. Eventos de Domínio

```csharp
// Exemplo de evento de domínio
public class ChildEnrolledEvent : IDomainEvent
{
    public Guid ChildId { get; set; }
    public Guid SchoolId { get; set; }
    public Guid ClassId { get; set; }
    public DateTime EnrolledAt { get; set; }
}
```

---

## 🛡️ Resiliência e Tolerância a Falhas

### 1. Circuit Breaker Pattern

```csharp
services.AddHttpClient<IActivityService, ActivityService>()
    .AddTransientHttpErrorPolicy(p => 
        p.CircuitBreakerAsync(5, TimeSpan.FromSeconds(30)));
```

### 2. Retry Pattern

```csharp
services.AddHttpClient<IHealthService, HealthService>()
    .AddTransientHttpErrorPolicy(p => 
        p.WaitAndRetryAsync(3, retryAttempt => 
            TimeSpan.FromSeconds(Math.Pow(2, retryAttempt))));
```

### 3. Timeout Policy

```csharp
services.AddHttpClient<IReportService, ReportService>()
    .AddPolicyHandler(Policy.TimeoutAsync<HttpResponseMessage>(10));
```

### 4. Fallback Pattern

- Cache de dados em caso de falha do serviço
- Resposta degradada mas funcional
- Mensagens de erro amigáveis

---

## 📊 Monitorização e Observabilidade

### 1. Logging Estruturado

- **Tecnologia**: Serilog
- **Sink**: Elasticsearch ou Seq
- **Níveis**:
  - Trace: Informação muito detalhada
  - Debug: Informação de debug
  - Information: Informação geral
  - Warning: Avisos
  - Error: Erros
  - Critical: Erros críticos

```csharp
Log.Information("Activity registered for child {ChildId} by {UserId}", 
    childId, userId);
```

### 2. Distributed Tracing

- **Tecnologia**: OpenTelemetry
- **Backend**: Jaeger ou Zipkin
- **Benefícios**:
  - Rastreamento de requisições entre serviços
  - Identificação de bottlenecks
  - Análise de latência

### 3. Métricas

- **Tecnologia**: Prometheus
- **Visualização**: Grafana
- **Métricas**:
  - Request rate
  - Error rate
  - Duration (latência)
  - Saturation (uso de recursos)

### 4. Health Checks

```csharp
services.AddHealthChecks()
    .AddNpgSql(connectionString)
    .AddRedis(redisConnection)
    .AddRabbitMQ(rabbitConnection);
```

---

## 🔒 Segurança

### 1. Autenticação

- **JWT (JSON Web Tokens)**
- **Refresh Tokens**
- **Token rotation**

### 2. Autorização

- **Role-Based Access Control (RBAC)**
- **Claims-Based Authorization**
- **Políticas personalizadas**

### 3. Proteção de Dados

- **Encriptação em trânsito**: TLS 1.3
- **Encriptação em repouso**: PostgreSQL encryption
- **Hashing de passwords**: BCrypt/Argon2

### 4. Rate Limiting

- Por IP
- Por utilizador
- Por endpoint

---

## 🚀 Escalabilidade

### Horizontal Scaling

```
┌─────────┐  ┌─────────┐  ┌─────────┐
│Service 1│  │Service 1│  │Service 1│
│Instance │  │Instance │  │Instance │
│    1    │  │    2    │  │    N    │
└─────────┘  └─────────┘  └─────────┘
      │            │            │
      └────────────┴────────────┘
                   │
            ┌──────────────┐
            │Load Balancer │
            └──────────────┘
```

### Estratégias de Escalabilidade

1. **Load Balancing**: Nginx ou cloud load balancer
2. **Auto-scaling**: Baseado em métricas
3. **Database replication**: Read replicas
4. **Cache distribuído**: Redis Cluster
5. **CDN**: Para conteúdo estático

---

## 📦 Deployment

### Containerização

```dockerfile
# Exemplo de Dockerfile para microserviço
FROM mcr.microsoft.com/dotnet/aspnet:9.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM mcr.microsoft.com/dotnet/sdk:9.0 AS build
WORKDIR /src
COPY ["Service.API/Service.API.csproj", "Service.API/"]
RUN dotnet restore
COPY . .
RUN dotnet build -c Release -o /app/build

FROM build AS publish
RUN dotnet publish -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "Service.API.dll"]
```

### Orquestração

- **Desenvolvimento**: Docker Compose
- **Produção**: Kubernetes ou Docker Swarm

---

## 🔄 CI/CD

### Pipeline de CI

1. **Build**: Compilação do código
2. **Test**: Execução de testes
3. **Quality Gate**: Análise de qualidade (SonarQube)
4. **Package**: Criação de imagens Docker
5. **Push**: Envio para registry

### Pipeline de CD

1. **Deploy to Dev**: Automático após CI
2. **Deploy to Staging**: Automático após Dev
3. **Deploy to Production**: Manual com aprovação

---

## 📚 Decisões Arquiteturais (ADRs)

### ADR-001: Escolha de Microserviços
**Status**: Aceite  
**Contexto**: Necessidade de escalabilidade e isolamento  
**Decisão**: Arquitetura de microserviços  
**Consequências**: Complexidade aumentada, mas maior flexibilidade

### ADR-002: Database per Tenant
**Status**: Aceite  
**Contexto**: Requisito de isolamento total de dados  
**Decisão**: Uma base de dados por escola  
**Consequências**: Maior uso de recursos, mas compliance garantido

### ADR-003: CQRS
**Status**: Aceite  
**Contexto**: Otimização de leitura e escrita  
**Decisão**: Implementar CQRS em serviços core  
**Consequências**: Código mais complexo, mas melhor performance

---

## 📖 Referências

- [Domain-Driven Design](https://martinfowler.com/tags/domain%20driven%20design.html)
- [Microservices Patterns](https://microservices.io/patterns/index.html)
- [.NET Microservices Architecture](https://docs.microsoft.com/en-us/dotnet/architecture/microservices/)
- [Multi-Tenancy Patterns](https://docs.microsoft.com/en-us/azure/architecture/guide/multitenant/overview)

---

**Próximo documento**: [Stack Tecnológica](./03-STACK-TECNOLOGICA.md)  
**Anterior**: [Visão Geral](./01-VISAO-GERAL.md)  
**Voltar ao**: [Índice](./00-INDICE.md)
