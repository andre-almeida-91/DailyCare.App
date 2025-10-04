# Stack Tecnológica DailyCare

## 🎯 Visão Geral

Este documento detalha todas as tecnologias utilizadas no projeto DailyCare, incluindo justificativas, versões e configurações recomendadas.

---

## 🎨 Frontend

### React Ecosystem

#### React 18.3+
- **Versão**: 18.3.0 ou superior
- **Justificação**: 
  - Concurrent rendering
  - Suspense melhorado
  - Automatic batching
  - Hooks modernos
- **Uso**: Base do frontend

#### TypeScript 5.3+
- **Versão**: 5.3.0 ou superior
- **Justificação**:
  - Type safety
  - Melhor IntelliSense
  - Refactoring seguro
  - Documentação implícita
- **Configuração**:
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["ES2022", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "moduleResolution": "bundler",
    "jsx": "react-jsx",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true
  }
}
```

### UI Framework & Styling

#### Material-UI (MUI) 6.x
- **Versão**: 6.0.0 ou superior
- **Pacotes**:
  ```bash
  @mui/material
  @mui/icons-material
  @mui/x-data-grid
  @mui/x-date-pickers
  @mui/lab
  ```
- **Justificação**:
  - Componentes ricos e acessíveis
  - Theming robusto (light/dark)
  - Documentação extensa
  - Grande comunidade
  - Performance otimizada

#### Tailwind CSS 3.4+
- **Versão**: 3.4.0 ou superior
- **Justificação**:
  - Utility-first approach
  - Customização fácil
  - Purge CSS automático
  - Responsive design simples
  - Complementa bem MUI
- **Configuração**:
```javascript
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  important: '#root',
  theme: {
    extend: {
      colors: {
        primary: {
          light: '#6B7FE6',
          main: '#4C63D2',
          dark: '#3B4FA8',
        },
        secondary: {
          light: '#FF8A80',
          main: '#FF5252',
          dark: '#C50E29',
        },
      },
    },
  },
  plugins: [],
  corePlugins: {
    preflight: false,
  },
}
```

### Estado e Data Fetching

#### Redux Toolkit 2.0+
- **Versão**: 2.0.0 ou superior
- **Pacotes**:
  ```bash
  @reduxjs/toolkit
  react-redux
  ```
- **Justificação**:
  - Boilerplate reduzido
  - DevTools integradas
  - Immer incluído
  - Padrão da indústria

#### RTK Query
- **Incluído em**: @reduxjs/toolkit
- **Justificação**:
  - Cache automático
  - Invalidação inteligente
  - Optimistic updates
  - Polling e refetching
  - SSE/WebSocket support

#### React Query (TanStack Query) 5.x (Alternativa/Complemento)
- **Versão**: 5.0.0 ou superior
- **Uso**: Queries complexas ou real-time
- **Justificação**:
  - Gestão de cache avançada
  - Mutations simplificadas
  - Infinite queries
  - DevTools excelentes

### Roteamento

#### React Router 6.x
- **Versão**: 6.20.0 ou superior
- **Justificação**:
  - Roteamento declarativo
  - Nested routes
  - Data loading
  - Code splitting

### Formulários e Validação

#### React Hook Form 7.x
- **Versão**: 7.49.0 ou superior
- **Justificação**:
  - Performance otimizada
  - Re-renders mínimos
  - TypeScript support
  - Validação flexível

#### Zod 3.x
- **Versão**: 3.22.0 ou superior
- **Justificação**:
  - Schema validation
  - TypeScript-first
  - Inferência de tipos
  - Runtime safety

### Build Tool

#### Vite 5.x
- **Versão**: 5.0.0 ou superior
- **Justificação**:
  - Build ultra-rápido
  - HMR instantâneo
  - ESM nativo
  - Plugins ricos
- **Plugins recomendados**:
  ```bash
  @vitejs/plugin-react
  @vitejs/plugin-react-swc (alternativa mais rápida)
  vite-plugin-pwa
  vite-plugin-svgr
  ```

### Utilitários Frontend

#### date-fns 3.x
- **Versão**: 3.0.0 ou superior
- **Justificação**:
  - Manipulação de datas moderna
  - Tree-shakeable
  - i18n support
  - TypeScript support

#### axios 1.6+
- **Versão**: 1.6.0 ou superior
- **Justificação**:
  - HTTP client robusto
  - Interceptors
  - Request/Response transformation
  - Cancel tokens

#### recharts 2.x
- **Versão**: 2.10.0 ou superior
- **Justificação**:
  - Gráficos React nativos
  - Customizável
  - Responsive
  - SVG-based

#### react-i18next 14.x
- **Versão**: 14.0.0 ou superior
- **Justificação**:
  - Internacionalização
  - Lazy loading
  - Pluralização
  - Interpolação

---

## ⚙️ Backend

### .NET Ecosystem

#### .NET 9
- **Versão**: 9.0.0
- **Justificação**:
  - LTS release
  - Performance melhorada
  - Native AOT
  - Minimal APIs melhoradas
  - HTTP/3 support

#### ASP.NET Core
- **Incluído em**: .NET 9
- **Features utilizadas**:
  - Minimal APIs
  - Controllers (onde apropriado)
  - SignalR
  - gRPC (comunicação entre serviços)

### Bibliotecas Core

#### Entity Framework Core 9
- **Versão**: 9.0.0
- **Pacotes**:
  ```bash
  Microsoft.EntityFrameworkCore
  Microsoft.EntityFrameworkCore.Design
  Npgsql.EntityFrameworkCore.PostgreSQL
  ```
- **Justificação**:
  - ORM robusto
  - Migrations automáticas
  - LINQ support
  - Performance otimizada

#### Dapper 2.x (para queries complexas)
- **Versão**: 2.1.0 ou superior
- **Justificação**:
  - Micro-ORM rápido
  - Queries otimizadas
  - Complementa EF Core

#### MediatR 12.x
- **Versão**: 12.2.0 ou superior
- **Justificação**:
  - CQRS implementation
  - Pipeline behaviors
  - Desacoplamento
  - Testabilidade

#### FluentValidation 11.x
- **Versão**: 11.9.0 ou superior
- **Justificação**:
  - Validação elegante
  - Separação de concerns
  - Reutilização
  - Mensagens customizadas

#### AutoMapper 13.x
- **Versão**: 13.0.0 ou superior
- **Justificação**:
  - Mapeamento objeto-objeto
  - Configuração centralizada
  - Profile-based mapping

### API & Documentação

#### Swashbuckle.AspNetCore 6.x
- **Versão**: 6.5.0 ou superior
- **Justificação**:
  - OpenAPI/Swagger UI
  - Documentação automática
  - Try-it-out functionality

#### NSwag (alternativa)
- **Versão**: 14.0.0 ou superior
- **Justificação**:
  - Client generation
  - OpenAPI 3.0
  - TypeScript clients

### Resiliência

#### Polly 8.x
- **Versão**: 8.2.0 ou superior
- **Justificação**:
  - Retry policies
  - Circuit breaker
  - Timeout policies
  - Fallback strategies
- **Exemplo**:
```csharp
services.AddHttpClient<IActivityService, ActivityService>()
    .AddTransientHttpErrorPolicy(builder => builder.WaitAndRetryAsync(
        3, retryAttempt => TimeSpan.FromSeconds(Math.Pow(2, retryAttempt))))
    .AddTransientHttpErrorPolicy(builder => builder.CircuitBreakerAsync(
        5, TimeSpan.FromSeconds(30)));
```

### Autenticação & Autorização

#### Microsoft.AspNetCore.Authentication.JwtBearer
- **Incluído em**: .NET 9
- **Justificação**:
  - JWT authentication
  - Bearer tokens
  - Claims-based auth

#### IdentityServer4 / Duende IdentityServer (para cenários complexos)
- **Versão**: Duende IdentityServer 7.x
- **Nota**: Licença comercial necessária para produção
- **Alternativa open-source**: OpenIddict

#### OpenIddict 5.x (Recomendado - Open Source)
- **Versão**: 5.0.0 ou superior
- **Justificação**:
  - OAuth 2.0 / OpenID Connect
  - Open source
  - .NET nativo
  - Customizável

### Logging & Monitoring

#### Serilog 3.x
- **Versão**: 3.1.0 ou superior
- **Pacotes**:
  ```bash
  Serilog.AspNetCore
  Serilog.Sinks.Console
  Serilog.Sinks.File
  Serilog.Sinks.Elasticsearch
  Serilog.Enrichers.Environment
  Serilog.Enrichers.Thread
  ```
- **Configuração**:
```csharp
Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Information()
    .MinimumLevel.Override("Microsoft", LogEventLevel.Warning)
    .Enrich.FromLogContext()
    .Enrich.WithMachineName()
    .Enrich.WithEnvironmentName()
    .WriteTo.Console()
    .WriteTo.Elasticsearch(new ElasticsearchSinkOptions(new Uri("http://localhost:9200"))
    {
        AutoRegisterTemplate = true,
        IndexFormat = $"dailycare-logs-{DateTime.UtcNow:yyyy-MM}"
    })
    .CreateLogger();
```

#### OpenTelemetry 1.7+
- **Versão**: 1.7.0 ou superior
- **Pacotes**:
  ```bash
  OpenTelemetry.Extensions.Hosting
  OpenTelemetry.Instrumentation.AspNetCore
  OpenTelemetry.Instrumentation.Http
  OpenTelemetry.Instrumentation.SqlClient
  OpenTelemetry.Exporter.Jaeger
  OpenTelemetry.Exporter.Prometheus
  ```

### Testing

#### xUnit 2.6+
- **Versão**: 2.6.0 ou superior
- **Justificação**:
  - Framework de testes moderno
  - Paralelização
  - Theory e InlineData

#### FluentAssertions 6.x
- **Versão**: 6.12.0 ou superior
- **Justificação**:
  - Assertions legíveis
  - Mensagens de erro claras

#### Moq 4.x
- **Versão**: 4.20.0 ou superior
- **Justificação**:
  - Mocking framework
  - Syntax fluente

#### Testcontainers 3.x
- **Versão**: 3.6.0 ou superior
- **Justificação**:
  - Integration tests com Docker
  - PostgreSQL containers
  - Redis containers

#### Bogus 35.x
- **Versão**: 35.0.0 ou superior
- **Justificação**:
  - Fake data generation
  - Seeding de testes

---

## 🗄️ Bases de Dados & Cache

### PostgreSQL 16

#### Versão: 16.1 ou superior
- **Justificação**:
  - ACID completo
  - JSON support robusto
  - Particionamento
  - Replicação
  - Open source
  - Performance excelente

#### Npgsql 9.x
- **Versão**: 9.0.0 ou superior
- **Justificação**:
  - Driver .NET oficial
  - Alta performance
  - Full feature support

#### Configuração Recomendada:
```sql
-- Otimizações de performance
shared_buffers = 256MB
effective_cache_size = 1GB
maintenance_work_mem = 64MB
checkpoint_completion_target = 0.9
wal_buffers = 16MB
default_statistics_target = 100
random_page_cost = 1.1
effective_io_concurrency = 200
work_mem = 2621kB
min_wal_size = 1GB
max_wal_size = 4GB
max_connections = 100
```

### Redis 7.x

#### Versão: 7.2 ou superior
- **Justificação**:
  - Cache in-memory rápido
  - Pub/Sub
  - Data structures ricas
  - Persistência opcional
  - Clustering

#### StackExchange.Redis 2.x
- **Versão**: 2.7.0 ou superior
- **Justificação**:
  - Client .NET robusto
  - Multiplexing
  - Pipeline support

#### Uso:
```csharp
services.AddStackExchangeRedisCache(options =>
{
    options.Configuration = "localhost:6379";
    options.InstanceName = "DailyCare_";
});
```

---

## 🔄 Mensageria

### RabbitMQ 3.12+ (Recomendado - Open Source)

#### Versão: 3.12 ou superior
- **Justificação**:
  - Message broker robusto
  - AMQP protocol
  - Patterns diversos
  - Management UI
  - Open source

#### MassTransit 8.x
- **Versão**: 8.1.0 ou superior
- **Justificação**:
  - Abstração de mensageria
  - Sagas
  - Retry e error handling
  - Multiple transports
- **Configuração**:
```csharp
services.AddMassTransit(x =>
{
    x.AddConsumer<ActivityCreatedConsumer>();
    
    x.UsingRabbitMq((context, cfg) =>
    {
        cfg.Host("rabbitmq://localhost", h =>
        {
            h.Username("guest");
            h.Password("guest");
        });
        
        cfg.ConfigureEndpoints(context);
    });
});
```

---

## 🐳 Containerização & Orquestração

### Docker 24.x

#### Versão: 24.0 ou superior
- **Justificação**:
  - Containerização padrão
  - Isolamento
  - Portabilidade
  - Reprodutibilidade

#### Docker Compose 2.x
- **Versão**: 2.23 ou superior
- **Uso**: Desenvolvimento local
- **Exemplo**:
```yaml
version: '3.8'

services:
  postgres:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: postgres
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

  rabbitmq:
    image: rabbitmq:3-management-alpine
    ports:
      - "5672:5672"
      - "15672:15672"

volumes:
  postgres_data:
```

### Kubernetes (Produção)

#### Versão: 1.28+ ou superior
- **Justificação**:
  - Orquestração de containers
  - Auto-scaling
  - Service discovery
  - Load balancing
- **Alternativa para pequena escala**: Docker Swarm

---

## 🔍 Observabilidade

### Elasticsearch 8.x (Logs)

#### Versão: 8.11 ou superior
- **Justificação**:
  - Search e analytics
  - Log aggregation
  - Full-text search

### Kibana 8.x (Visualização)

#### Versão: 8.11 ou superior
- **Justificação**:
  - Visualização de logs
  - Dashboards
  - Alerting

### Prometheus (Métricas)

#### Versão: 2.48 ou superior
- **Justificação**:
  - Time-series database
  - Alerting
  - Open source

### Grafana (Dashboards)

#### Versão: 10.2 ou superior
- **Justificação**:
  - Visualização de métricas
  - Dashboards customizáveis
  - Multiple data sources

### Jaeger (Tracing)

#### Versão: 1.52 ou superior
- **Justificação**:
  - Distributed tracing
  - Performance analysis
  - Open source

---

## 🔐 Segurança

### Certbot (Let's Encrypt)
- **Justificação**:
  - SSL/TLS certificates gratuitos
  - Renovação automática

### Nginx 1.25+ (Reverse Proxy)
- **Versão**: 1.25 ou superior
- **Justificação**:
  - Load balancing
  - SSL termination
  - Rate limiting

---

## 🧪 Qualidade de Código

### SonarQube (Community Edition)
- **Versão**: Community Edition 10.3+
- **Justificação**:
  - Análise de código estática
  - Code smells
  - Security vulnerabilities
  - Technical debt

### ESLint 8.x (Frontend)
- **Versão**: 8.55 ou superior
- **Configuração**:
```javascript
module.exports = {
  parser: '@typescript-eslint/parser',
  extends: [
    'eslint:recommended',
    'plugin:react/recommended',
    'plugin:@typescript-eslint/recommended',
    'plugin:react-hooks/recommended',
  ],
  plugins: ['react', '@typescript-eslint'],
  rules: {
    'react/react-in-jsx-scope': 'off',
  },
};
```

### Prettier 3.x (Formatação)
- **Versão**: 3.1.0 ou superior
- **Configuração**:
```json
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 100,
  "tabWidth": 2
}
```

---

## 📦 Gestão de Pacotes

### Frontend: npm/pnpm
- **npm**: 10.2+ ou superior
- **pnpm**: 8.12+ ou superior (recomendado)
- **Justificação pnpm**:
  - Mais rápido
  - Uso eficiente de disco
  - Strict mode

### Backend: NuGet
- **Incluído em**: .NET SDK
- **Private feed**: Azure Artifacts ou MyGet (opcional)

---

## 🔧 Ferramentas de Desenvolvimento

### IDEs Recomendadas

#### Visual Studio Code
- **Versão**: Última stable
- **Extensões recomendadas**:
  - ESLint
  - Prettier
  - TypeScript
  - Tailwind CSS IntelliSense
  - Material Icon Theme
  - GitLens
  - REST Client

#### Visual Studio 2022
- **Versão**: 17.8+ ou superior
- **Workloads**:
  - ASP.NET and web development
  - .NET desktop development
  - Azure development

#### JetBrains Rider (Alternativa)
- **Versão**: 2023.3+ ou superior

### Git
- **Versão**: 2.43+ ou superior
- **Git Flow**: Para gestão de branches

### Postman / Insomnia
- **Uso**: Testes de API
- **Alternativa**: REST Client (VS Code extension)

---

## 📊 Resumo de Custos

### Tecnologias 100% Gratuitas (Open Source)

✅ React, TypeScript, Vite  
✅ Material-UI, Tailwind CSS  
✅ Redux Toolkit, React Query  
✅ .NET 9, ASP.NET Core  
✅ Entity Framework Core  
✅ PostgreSQL  
✅ Redis  
✅ RabbitMQ  
✅ Docker, Kubernetes  
✅ Elasticsearch, Kibana (Basic License)  
✅ Prometheus, Grafana  
✅ Jaeger  
✅ OpenIddict  
✅ SonarQube Community Edition  

### Tecnologias com Custo (Opcionais)

⚠️ Duende IdentityServer (licença comercial para produção)  
- **Alternativa gratuita**: OpenIddict ✅

⚠️ Cloud Providers (AWS, Azure, GCP)  
- **Alternativa**: Self-hosting em servidor próprio

---

## 🔄 Versionamento de Tecnologias

Todas as versões especificadas são mínimas recomendadas. Sempre usar a última versão stable dentro da major version especificada para receber security patches e bug fixes.

### Política de Atualização

1. **Patches (x.x.X)**: Atualizar imediatamente
2. **Minor (x.X.x)**: Avaliar e atualizar trimestralmente
3. **Major (X.x.x)**: Avaliar breaking changes e planear migração

---

## 📚 Recursos de Aprendizagem

### Documentação Oficial
- [React](https://react.dev/)
- [.NET](https://docs.microsoft.com/dotnet/)
- [PostgreSQL](https://www.postgresql.org/docs/)
- [Redis](https://redis.io/documentation)
- [Docker](https://docs.docker.com/)

### Tutoriais e Guias
- [Microsoft Learn](https://learn.microsoft.com/)
- [React Official Tutorial](https://react.dev/learn)
- [Microservices.io](https://microservices.io/)

---

**Próximo documento**: [Frontend para Pais](./04-FRONTEND-PAIS.md)  
**Anterior**: [Arquitetura](./02-ARQUITETURA.md)  
**Voltar ao**: [Índice](./00-INDICE.md)
