# Tarefas de Desenvolvimento - DailyCare

## 📋 Visão Geral

Este documento contém todas as tarefas necessárias para o desenvolvimento completo do projeto DailyCare, organizadas por fases, com prioridades e dependências claramente definidas.

---

## 🎯 Legenda de Prioridades

- 🔴 **P0 - Crítica**: Bloqueante para o MVP
- 🟠 **P1 - Alta**: Necessária para funcionalidade core
- 🟡 **P2 - Média**: Importante mas não bloqueante
- 🟢 **P3 - Baixa**: Nice to have

## 📊 Legenda de Status

- ⬜ **TODO**: Não iniciada
- 🟦 **IN PROGRESS**: Em desenvolvimento
- 🟩 **DONE**: Concluída
- 🟥 **BLOCKED**: Bloqueada

---

## 🚀 FASE 0: Configuração Inicial do Projeto

### TASK-001: Configuração do Repositório Git
- **Prioridade**: 🔴 P0
- **Estimativa**: 2h
- **Dependências**: Nenhuma
- **Status**: ⬜ TODO

**Descrição**: Configurar repositório Git com estrutura de branches e proteções.

**Critérios de Aceitação**:
- [ ] Repositório Git inicializado
- [ ] Branch `main` protegida
- [ ] Branch `develop` criada
- [ ] Git Flow configurado
- [ ] .gitignore configurado
- [ ] README.md atualizado

**Tarefas**:
1. Inicializar repositório Git
2. Configurar branches principais (main, develop)
3. Criar .gitignore para .NET e React
4. Configurar proteção de branches
5. Documentar workflow no README

---

### TASK-002: Estrutura de Pastas do Projeto
- **Prioridade**: 🔴 P0
- **Estimativa**: 1h
- **Dependências**: TASK-001
- **Status**: ⬜ TODO

**Descrição**: Criar estrutura de pastas para todo o projeto.

**Critérios de Aceitação**:
- [ ] Pastas de backend criadas
- [ ] Pastas de frontend criadas
- [ ] Pasta de documentação criada
- [ ] Pasta de infraestrutura criada

**Estrutura**:
```
DailyCare.App/
├── docs/                       # Documentação
├── src/
│   ├── backend/
│   │   ├── services/          # Microserviços
│   │   ├── gateway/           # API Gateway
│   │   └── shared/            # Código partilhado
│   ├── frontend/
│   │   ├── parent-app/        # App para pais
│   │   └── admin-app/         # App administrativo
│   └── infrastructure/        # Docker, K8s, etc.
├── tests/
│   ├── integration/
│   └── e2e/
└── scripts/                    # Scripts úteis
```

---

### TASK-003: Configuração do Docker Compose para Desenvolvimento
- **Prioridade**: 🔴 P0
- **Estimativa**: 4h
- **Dependências**: TASK-002
- **Status**: ⬜ TODO

**Descrição**: Configurar Docker Compose com todos os serviços necessários para desenvolvimento local.

**Critérios de Aceitação**:
- [ ] PostgreSQL configurado
- [ ] Redis configurado
- [ ] RabbitMQ configurado
- [ ] Elasticsearch configurado (opcional para dev)
- [ ] Todos os serviços iniciam sem erros
- [ ] Volumes persistentes configurados
- [ ] Networks configuradas

**Arquivo**: `docker-compose.dev.yml`

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:16-alpine
    container_name: dailycare-postgres
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: dailycare_master
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - dailycare-network

  redis:
    image: redis:7-alpine
    container_name: dailycare-redis
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    networks:
      - dailycare-network

  rabbitmq:
    image: rabbitmq:3-management-alpine
    container_name: dailycare-rabbitmq
    environment:
      RABBITMQ_DEFAULT_USER: admin
      RABBITMQ_DEFAULT_PASS: admin
    ports:
      - "5672:5672"
      - "15672:15672"
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq
    networks:
      - dailycare-network

volumes:
  postgres_data:
  redis_data:
  rabbitmq_data:

networks:
  dailycare-network:
    driver: bridge
```

---

## 🏗️ FASE 1: Infraestrutura Base e Serviços Core

### TASK-004: Criar Projeto do API Gateway
- **Prioridade**: 🔴 P0
- **Estimativa**: 6h
- **Dependências**: TASK-003
- **Status**: ⬜ TODO

**Descrição**: Criar API Gateway usando .NET 9 e YARP.

**Critérios de Aceitação**:
- [ ] Projeto .NET 9 criado
- [ ] YARP configurado
- [ ] Roteamento básico configurado
- [ ] CORS configurado
- [ ] Logging configurado (Serilog)
- [ ] Health checks configurados
- [ ] Dockerfile criado

**Comandos**:
```bash
cd src/backend/gateway
dotnet new web -n DailyCare.Gateway
dotnet add package Yarp.ReverseProxy
dotnet add package Serilog.AspNetCore
dotnet add package Serilog.Sinks.Console
```

**Configuração YARP** (`appsettings.json`):
```json
{
  "ReverseProxy": {
    "Routes": {
      "identity-route": {
        "ClusterId": "identity-cluster",
        "Match": {
          "Path": "/api/identity/{**catch-all}"
        }
      },
      "activities-route": {
        "ClusterId": "activities-cluster",
        "Match": {
          "Path": "/api/activities/{**catch-all}"
        }
      }
    },
    "Clusters": {
      "identity-cluster": {
        "Destinations": {
          "destination1": {
            "Address": "http://localhost:5001"
          }
        }
      },
      "activities-cluster": {
        "Destinations": {
          "destination1": {
            "Address": "http://localhost:5002"
          }
        }
      }
    }
  }
}
```

---

### TASK-005: Criar Shared Libraries
- **Prioridade**: 🔴 P0
- **Estimativa**: 8h
- **Dependências**: TASK-004
- **Status**: ⬜ TODO

**Descrição**: Criar bibliotecas compartilhadas para uso em todos os microserviços.

**Critérios de Aceitação**:
- [ ] DailyCare.Shared.Domain criado
- [ ] DailyCare.Shared.Infrastructure criado
- [ ] DailyCare.Shared.Common criado
- [ ] Interfaces base criadas
- [ ] Base entities criadas
- [ ] Multi-tenancy middleware criado
- [ ] JWT middleware criado

**Bibliotecas**:

1. **DailyCare.Shared.Domain**
   - Base entities
   - Value objects comuns
   - Domain events
   - Repository interfaces

2. **DailyCare.Shared.Infrastructure**
   - DbContext base
   - Multi-tenancy resolver
   - UnitOfWork pattern
   - Event dispatcher

3. **DailyCare.Shared.Common**
   - Result pattern
   - Exceptions customizadas
   - Extension methods
   - Constants

**Exemplo de Base Entity**:
```csharp
public abstract class BaseEntity
{
    public Guid Id { get; protected set; }
    public DateTime CreatedAt { get; protected set; }
    public DateTime? UpdatedAt { get; protected set; }
    public string CreatedBy { get; protected set; }
    public string? UpdatedBy { get; protected set; }
    public bool IsDeleted { get; protected set; }
    
    protected BaseEntity()
    {
        Id = Guid.NewGuid();
        CreatedAt = DateTime.UtcNow;
    }
}
```

---

### TASK-006: Criar Identity Service
- **Prioridade**: 🔴 P0
- **Estimativa**: 16h
- **Dependências**: TASK-005
- **Status**: ⬜ TODO

**Descrição**: Criar microserviço de identidade e autenticação.

**Critérios de Aceitação**:
- [ ] Projeto criado com Clean Architecture
- [ ] Autenticação JWT implementada
- [ ] Refresh tokens implementados
- [ ] Registro de utilizadores
- [ ] Login implementado
- [ ] Gestão de perfis
- [ ] Gestão de roles e permissions
- [ ] Password reset
- [ ] API documentada com Swagger

**Endpoints**:
- `POST /api/identity/register` - Registo
- `POST /api/identity/login` - Login
- `POST /api/identity/refresh` - Refresh token
- `POST /api/identity/logout` - Logout
- `POST /api/identity/forgot-password` - Recuperar password
- `POST /api/identity/reset-password` - Reset password
- `GET /api/identity/me` - Perfil do utilizador
- `PUT /api/identity/me` - Atualizar perfil

**Entidades**:
```csharp
public class User : BaseEntity
{
    public string Name { get; private set; }
    public string Email { get; private set; }
    public string PasswordHash { get; private set; }
    public string PhoneNumber { get; private set; }
    public bool EmailConfirmed { get; private set; }
    public bool PhoneConfirmed { get; private set; }
    public DateTime? LastLoginAt { get; private set; }
    
    public ICollection<UserRole> Roles { get; private set; }
    public ICollection<RefreshToken> RefreshTokens { get; private set; }
}

public class Role : BaseEntity
{
    public string Name { get; private set; }
    public string Description { get; private set; }
    public ICollection<Permission> Permissions { get; private set; }
}

public class Permission : BaseEntity
{
    public string Name { get; private set; }
    public string Resource { get; private set; }
    public string Action { get; private set; }
}
```

---

### TASK-007: Criar School Service
- **Prioridade**: 🔴 P0
- **Estimativa**: 12h
- **Dependências**: TASK-006
- **Status**: ⬜ TODO

**Descrição**: Criar microserviço de gestão de escolas (tenants).

**Critérios de Aceitação**:
- [ ] Projeto criado com Clean Architecture
- [ ] CRUD de escolas
- [ ] Provisionamento de base de dados por escola
- [ ] Gestão de turmas
- [ ] Gestão de educadores
- [ ] Configurações de escola
- [ ] API documentada

**Endpoints**:
- `POST /api/schools` - Criar escola
- `GET /api/schools` - Listar escolas
- `GET /api/schools/{id}` - Obter escola
- `PUT /api/schools/{id}` - Atualizar escola
- `DELETE /api/schools/{id}` - Deletar escola
- `POST /api/schools/{id}/provision-database` - Provisionar BD
- `GET /api/schools/{schoolId}/classes` - Listar turmas
- `POST /api/schools/{schoolId}/classes` - Criar turma

**Entidades**:
```csharp
public class School : BaseEntity
{
    public string Name { get; private set; }
    public string Code { get; private set; }
    public string Email { get; private set; }
    public string PhoneNumber { get; private set; }
    public Address Address { get; private set; }
    public string ConnectionString { get; private set; }
    public bool IsActive { get; private set; }
    public DateTime? ActivatedAt { get; private set; }
    
    public SchoolSettings Settings { get; private set; }
    public ICollection<SchoolClass> Classes { get; private set; }
}

public class SchoolClass : BaseEntity
{
    public Guid SchoolId { get; private set; }
    public string Name { get; private set; }
    public string Description { get; private set; }
    public AgeRange AgeRange { get; private set; }
    public int Capacity { get; private set; }
    
    public ICollection<ClassEducator> Educators { get; private set; }
}
```

---

### TASK-008: Criar Children Service
- **Prioridade**: 🔴 P0
- **Estimativa**: 12h
- **Dependências**: TASK-007
- **Status**: ⬜ TODO

**Descrição**: Criar microserviço de gestão de crianças.

**Critérios de Aceitação**:
- [ ] CRUD de crianças
- [ ] Gestão de perfis
- [ ] Relações familiares
- [ ] Histórico de matrículas
- [ ] Upload de fotos
- [ ] Multi-tenancy implementado
- [ ] API documentada

**Endpoints**:
- `POST /api/children` - Criar criança
- `GET /api/children` - Listar crianças
- `GET /api/children/{id}` - Obter criança
- `PUT /api/children/{id}` - Atualizar criança
- `DELETE /api/children/{id}` - Deletar criança
- `POST /api/children/{id}/photo` - Upload de foto
- `GET /api/children/{id}/family` - Obter familiares
- `POST /api/children/{id}/family` - Adicionar familiar

**Entidades**:
```csharp
public class Child : BaseEntity
{
    public string FirstName { get; private set; }
    public string LastName { get; private set; }
    public DateTime BirthDate { get; private set; }
    public Gender Gender { get; private set; }
    public string PhotoUrl { get; private set; }
    public BloodType? BloodType { get; private set; }
    
    public Guid CurrentClassId { get; private set; }
    public SchoolClass CurrentClass { get; private set; }
    
    public ICollection<FamilyMember> FamilyMembers { get; private set; }
    public ICollection<Enrollment> Enrollments { get; private set; }
}

public class FamilyMember : BaseEntity
{
    public Guid ChildId { get; private set; }
    public Guid UserId { get; private set; }
    public RelationshipType Relationship { get; private set; }
    public bool IsPrimaryContact { get; private set; }
    public bool CanPickup { get; private set; }
}
```

---

### TASK-009: Criar Activities Service
- **Prioridade**: 🔴 P0
- **Estimativa**: 20h
- **Dependências**: TASK-008
- **Status**: ⬜ TODO

**Descrição**: Criar microserviço de gestão de atividades diárias.

**Critérios de Aceitação**:
- [ ] Registo de alimentação implementado
- [ ] Registo de sono implementado
- [ ] Registo de fraldas implementado
- [ ] Registo de atividades físicas implementado
- [ ] Marcos de desenvolvimento implementados
- [ ] Filtros e pesquisa
- [ ] Estatísticas básicas
- [ ] Multi-tenancy implementado
- [ ] API documentada

**Endpoints Alimentação**:
- `POST /api/activities/feeding` - Registar alimentação
- `GET /api/activities/feeding` - Listar alimentações
- `GET /api/activities/feeding/{id}` - Obter alimentação
- `PUT /api/activities/feeding/{id}` - Atualizar alimentação
- `DELETE /api/activities/feeding/{id}` - Deletar alimentação

**Endpoints Sono**:
- `POST /api/activities/sleep` - Registar sono
- `GET /api/activities/sleep` - Listar sonos
- `GET /api/activities/sleep/{id}` - Obter sono
- `PUT /api/activities/sleep/{id}` - Atualizar sono
- `DELETE /api/activities/sleep/{id}` - Deletar sono

**Endpoints Fraldas**:
- `POST /api/activities/diaper` - Registar fralda
- `GET /api/activities/diaper` - Listar fraldas
- `GET /api/activities/diaper/{id}` - Obter fralda
- `PUT /api/activities/diaper/{id}` - Atualizar fralda
- `DELETE /api/activities/diaper/{id}` - Deletar fralda

**Entidades**:
```csharp
public class FeedingActivity : BaseEntity
{
    public Guid ChildId { get; private set; }
    public DateTime StartTime { get; private set; }
    public DateTime? EndTime { get; private set; }
    public FeedingType Type { get; private set; }
    public decimal? Quantity { get; private set; }
    public QuantityUnit Unit { get; private set; }
    public string FoodDescription { get; private set; }
    public string Notes { get; private set; }
    public string RegisteredBy { get; private set; }
}

public class SleepActivity : BaseEntity
{
    public Guid ChildId { get; private set; }
    public DateTime StartTime { get; private set; }
    public DateTime EndTime { get; private set; }
    public TimeSpan Duration { get; private set; }
    public SleepType Type { get; private set; }
    public SleepQuality Quality { get; private set; }
    public string Notes { get; private set; }
    public string RegisteredBy { get; private set; }
}

public class DiaperActivity : BaseEntity
{
    public Guid ChildId { get; private set; }
    public DateTime Time { get; private set; }
    public DiaperType Type { get; private set; }
    public DiaperCondition Condition { get; private set; }
    public string Notes { get; private set; }
    public string RegisteredBy { get; private set; }
}

public class Milestone : BaseEntity
{
    public Guid ChildId { get; private set; }
    public string Title { get; private set; }
    public string Description { get; private set; }
    public MilestoneCategory Category { get; private set; }
    public DateTime AchievedAt { get; private set; }
    public string PhotoUrl { get; private set; }
    public string RegisteredBy { get; private set; }
}
```

---

### TASK-010: Criar Health Service
- **Prioridade**: 🟠 P1
- **Estimativa**: 16h
- **Dependências**: TASK-008
- **Status**: ⬜ TODO

**Descrição**: Criar microserviço de gestão de saúde.

**Critérios de Aceitação**:
- [ ] Registo de temperatura e sintomas
- [ ] Gestão de medicamentos
- [ ] Plano de vacinação
- [ ] Histórico de consultas
- [ ] Gestão de alergias
- [ ] Multi-tenancy implementado
- [ ] API documentada

**Endpoints**:
- `POST /api/health/temperature` - Registar temperatura
- `POST /api/health/medications` - Registar medicamento
- `GET /api/health/medications/{childId}` - Obter medicamentos da criança
- `POST /api/health/vaccinations` - Registar vacina
- `GET /api/health/vaccinations/{childId}` - Plano de vacinação
- `POST /api/health/appointments` - Agendar consulta
- `GET /api/health/allergies/{childId}` - Obter alergias

**Entidades**:
```csharp
public class TemperatureRecord : BaseEntity
{
    public Guid ChildId { get; private set; }
    public DateTime MeasuredAt { get; private set; }
    public decimal Temperature { get; private set; }
    public TemperatureUnit Unit { get; private set; }
    public string Symptoms { get; private set; }
    public string Notes { get; private set; }
}

public class Medication : BaseEntity
{
    public Guid ChildId { get; private set; }
    public string Name { get; private set; }
    public string Dosage { get; private set; }
    public DateTime StartDate { get; private set; }
    public DateTime? EndDate { get; private set; }
    public string Frequency { get; private set; }
    public string Notes { get; private set; }
    
    public ICollection<MedicationDose> Doses { get; private set; }
}

public class Vaccination : BaseEntity
{
    public Guid ChildId { get; private set; }
    public string VaccineName { get; private set; }
    public DateTime ScheduledDate { get; private set; }
    public DateTime? AdministeredDate { get; private set; }
    public VaccinationStatus Status { get; private set; }
    public string Location { get; private set; }
    public string Notes { get; private set; }
}

public class Allergy : BaseEntity
{
    public Guid ChildId { get; private set; }
    public string AllergenName { get; private set; }
    public AllergyType Type { get; private set; }
    public AllergySeverity Severity { get; private set; }
    public string Reaction { get; private set; }
    public DateTime DiagnosedAt { get; private set; }
}
```

---

### TASK-011: Criar Notification Service
- **Prioridade**: 🟠 P1
- **Estimativa**: 12h
- **Dependências**: TASK-006
- **Status**: ⬜ TODO

**Descrição**: Criar microserviço de notificações.

**Critérios de Aceitação**:
- [ ] SignalR configurado
- [ ] Envio de notificações push
- [ ] Envio de emails (SMTP ou SendGrid)
- [ ] Templates de mensagens
- [ ] Preferências de notificação
- [ ] Histórico de notificações
- [ ] API documentada

**Endpoints**:
- `POST /api/notifications/send` - Enviar notificação
- `GET /api/notifications` - Listar notificações do utilizador
- `PUT /api/notifications/{id}/read` - Marcar como lida
- `GET /api/notifications/preferences` - Obter preferências
- `PUT /api/notifications/preferences` - Atualizar preferências

**SignalR Hub**:
```csharp
public class NotificationHub : Hub
{
    public async Task SendToUser(string userId, Notification notification)
    {
        await Clients.User(userId).SendAsync("ReceiveNotification", notification);
    }
    
    public async Task SendToGroup(string groupId, Notification notification)
    {
        await Clients.Group(groupId).SendAsync("ReceiveNotification", notification);
    }
}
```

---

## 🎨 FASE 2: Frontend para Pais

### TASK-012: Configurar Projeto React para Pais
- **Prioridade**: 🔴 P0
- **Estimativa**: 6h
- **Dependências**: TASK-004
- **Status**: ⬜ TODO

**Descrição**: Criar e configurar projeto React para aplicação dos pais.

**Critérios de Aceitação**:
- [ ] Projeto Vite + React + TypeScript criado
- [ ] Material-UI instalado e configurado
- [ ] Tailwind CSS instalado e configurado
- [ ] Redux Toolkit configurado
- [ ] React Router configurado
- [ ] Axios configurado com interceptors
- [ ] ESLint e Prettier configurados
- [ ] Estrutura de pastas criada

**Comandos**:
```bash
cd src/frontend
npm create vite@latest parent-app -- --template react-ts
cd parent-app
npm install
npm install @mui/material @emotion/react @emotion/styled
npm install @mui/icons-material
npm install @reduxjs/toolkit react-redux
npm install react-router-dom
npm install axios
npm install date-fns
npm install react-hook-form zod @hookform/resolvers
npm install -D tailwindcss postcss autoprefixer
npm install -D @types/node
npm install -D eslint-config-prettier eslint-plugin-prettier
npx tailwindcss init -p
```

---

### TASK-013: Implementar Sistema de Temas (Light/Dark)
- **Prioridade**: 🟠 P1
- **Estimativa**: 4h
- **Dependências**: TASK-012
- **Status**: ⬜ TODO

**Descrição**: Implementar sistema de temas com detecção automática do sistema operativo.

**Critérios de Aceitação**:
- [ ] Tema light criado
- [ ] Tema dark criado
- [ ] Detecção automática do tema do SO
- [ ] Toggle manual de tema
- [ ] Preferência salva em localStorage
- [ ] Transições suaves entre temas

---

### TASK-014: Implementar Autenticação Frontend (Pais)
- **Prioridade**: 🔴 P0
- **Estimativa**: 8h
- **Dependências**: TASK-006, TASK-012
- **Status**: ⬜ TODO

**Descrição**: Implementar fluxo de autenticação no frontend.

**Critérios de Aceitação**:
- [ ] Tela de login
- [ ] Tela de registo
- [ ] Tela de recuperação de password
- [ ] Tela de reset de password
- [ ] Gestão de tokens (JWT)
- [ ] Refresh token automático
- [ ] Redirect após login
- [ ] Protected routes
- [ ] Logout

---

### TASK-015: Implementar Dashboard Principal (Pais)
- **Prioridade**: 🔴 P0
- **Estimativa**: 12h
- **Dependências**: TASK-014
- **Status**: ⬜ TODO

**Descrição**: Criar dashboard principal com resumo de atividades.

**Critérios de Aceitação**:
- [ ] Cards com crianças
- [ ] Resumo de atividades do dia
- [ ] Próximos eventos/consultas
- [ ] Gráficos simples
- [ ] Notificações recentes
- [ ] Navegação rápida

---

### TASK-016: Implementar Gestão de Atividades (Pais)
- **Prioridade**: 🔴 P0
- **Estimativa**: 16h
- **Dependências**: TASK-009, TASK-015
- **Status**: ⬜ TODO

**Descrição**: Criar interfaces para visualização de atividades.

**Critérios de Aceitação**:
- [ ] Listagem de alimentação
- [ ] Listagem de sono
- [ ] Listagem de fraldas
- [ ] Listagem de atividades físicas
- [ ] Filtros por data
- [ ] Filtros por tipo
- [ ] Detalhes de cada atividade
- [ ] Loading states
- [ ] Error handling

---

### TASK-017: Implementar Visualização de Saúde (Pais)
- **Prioridade**: 🟠 P1
- **Estimativa**: 12h
- **Dependências**: TASK-010, TASK-015
- **Status**: ⬜ TODO

**Descrição**: Criar interfaces para visualização de informações de saúde.

**Critérios de Aceitação**:
- [ ] Visualização de temperatura
- [ ] Visualização de medicamentos
- [ ] Plano de vacinação
- [ ] Histórico de consultas
- [ ] Alergias registadas
- [ ] Gráficos de temperatura

---

### TASK-018: Implementar Notificações em Tempo Real (Pais)
- **Prioridade**: 🟠 P1
- **Estimativa**: 8h
- **Dependências**: TASK-011, TASK-015
- **Status**: ⬜ TODO

**Descrição**: Integrar SignalR para notificações em tempo real.

**Critérios de Aceitação**:
- [ ] SignalR client configurado
- [ ] Conexão estabelecida após login
- [ ] Notificações exibidas em tempo real
- [ ] Toast notifications
- [ ] Badge com contador
- [ ] Centro de notificações
- [ ] Marcar como lida

---

## 🎨 FASE 3: Frontend Administrativo

### TASK-019: Configurar Projeto React para Admin
- **Prioridade**: 🟠 P1
- **Estimativa**: 6h
- **Dependências**: TASK-012
- **Status**: ⬜ TODO

**Descrição**: Criar projeto React para painel administrativo.

**Critérios de Aceitação**:
- [ ] Projeto criado (similar ao parent-app)
- [ ] Tema administrativo configurado
- [ ] Navegação lateral configurada
- [ ] Breadcrumbs implementados

---

### TASK-020: Implementar Gestão de Escolas (Admin)
- **Prioridade**: 🟠 P1
- **Estimativa**: 16h
- **Dependências**: TASK-007, TASK-019
- **Status**: ⬜ TODO

**Descrição**: Criar interfaces para gestão de escolas.

**Critérios de Aceitação**:
- [ ] Listagem de escolas
- [ ] Criar escola
- [ ] Editar escola
- [ ] Desativar escola
- [ ] Provisionar base de dados
- [ ] Visualizar estatísticas
- [ ] Formulários com validação

---

### TASK-021: Implementar Gestão de Turmas (Admin)
- **Prioridade**: 🟠 P1
- **Estimativa**: 12h
- **Dependências**: TASK-020
- **Status**: ⬜ TODO

**Descrição**: Criar interfaces para gestão de turmas.

**Critérios de Aceitação**:
- [ ] Listagem de turmas
- [ ] Criar turma
- [ ] Editar turma
- [ ] Deletar turma
- [ ] Atribuir educadores
- [ ] Visualizar crianças da turma

---

### TASK-022: Implementar Gestão de Utilizadores (Admin)
- **Prioridade**: 🟠 P1
- **Estimativa**: 12h
- **Dependências**: TASK-019
- **Status**: ⬜ TODO

**Descrição**: Criar interfaces para gestão de utilizadores e permissões.

**Critérios de Aceitação**:
- [ ] Listagem de utilizadores
- [ ] Criar utilizador
- [ ] Editar utilizador
- [ ] Desativar utilizador
- [ ] Gestão de roles
- [ ] Gestão de permissões
- [ ] Reset de password

---

### TASK-023: Implementar Registo de Atividades (Admin/Educadores)
- **Prioridade**: 🔴 P0
- **Estimativa**: 20h
- **Dependências**: TASK-009, TASK-019
- **Status**: ⬜ TODO

**Descrição**: Criar interfaces para educadores registarem atividades.

**Critérios de Aceitação**:
- [ ] Formulário de alimentação
- [ ] Formulário de sono
- [ ] Formulário de fraldas
- [ ] Formulário de atividades
- [ ] Formulário de marcos
- [ ] Validação completa
- [ ] Feedback visual
- [ ] Ações rápidas
- [ ] Modo bulk (múltiplas crianças)

---

## 🔧 FASE 4: Funcionalidades Avançadas

### TASK-024: Implementar Reports Service
- **Prioridade**: 🟡 P2
- **Estimativa**: 16h
- **Dependências**: TASK-009, TASK-010
- **Status**: ⬜ TODO

**Descrição**: Criar serviço de geração de relatórios.

**Critérios de Aceitação**:
- [ ] Relatórios de atividades
- [ ] Relatórios de saúde
- [ ] Relatórios de desenvolvimento
- [ ] Gráficos e estatísticas
- [ ] Exportação para PDF
- [ ] Exportação para Excel
- [ ] Cache de relatórios

---

### TASK-025: Implementar Files Service
- **Prioridade**: 🟡 P2
- **Estimativa**: 12h
- **Dependências**: TASK-008
- **Status**: ⬜ TODO

**Descrição**: Criar serviço de gestão de ficheiros.

**Critérios de Aceitação**:
- [ ] Upload de fotos
- [ ] Upload de documentos
- [ ] Geração de thumbnails
- [ ] Validação de tipos de ficheiro
- [ ] Gestão de quotas
- [ ] Compressão de imagens
- [ ] Storage configurável (local/cloud)

---

### TASK-026: Implementar Galeria de Fotos (Frontend)
- **Prioridade**: 🟡 P2
- **Estimativa**: 8h
- **Dependências**: TASK-025
- **Status**: ⬜ TODO

**Descrição**: Criar galeria de fotos e momentos especiais.

**Critérios de Aceitação**:
- [ ] Upload de fotos
- [ ] Visualização em grelha
- [ ] Lightbox para visualização
- [ ] Filtros por data
- [ ] Álbuns por criança
- [ ] Partilha com família

---

### TASK-027: Implementar Sistema de Relatórios Visuais (Frontend)
- **Prioridade**: 🟡 P2
- **Estimativa**: 16h
- **Dependências**: TASK-024
- **Status**: ⬜ TODO

**Descrição**: Criar dashboards e relatórios visuais.

**Critérios de Aceitação**:
- [ ] Gráficos de alimentação
- [ ] Gráficos de sono
- [ ] Gráficos de desenvolvimento
- [ ] Comparação temporal
- [ ] Filtros avançados
- [ ] Exportação de relatórios
- [ ] Impressão

---

### TASK-028: Implementar Partilha com Familiares
- **Prioridade**: 🟡 P2
- **Estimativa**: 12h
- **Dependências**: TASK-006, TASK-008
- **Status**: ⬜ TODO

**Descrição**: Sistema de convites e partilha de acesso.

**Critérios de Aceitação**:
- [ ] Convite por email
- [ ] Gestão de permissões
- [ ] Níveis de acesso diferentes
- [ ] Notificações de novos acessos
- [ ] Revogação de acesso

---

## 🔐 FASE 5: Segurança e Performance

### TASK-029: Implementar Rate Limiting
- **Prioridade**: 🟠 P1
- **Estimativa**: 6h
- **Dependências**: TASK-004
- **Status**: ⬜ TODO

**Descrição**: Adicionar rate limiting no API Gateway.

**Critérios de Aceitação**:
- [ ] Rate limiting por IP
- [ ] Rate limiting por utilizador
- [ ] Rate limiting por endpoint
- [ ] Respostas HTTP 429
- [ ] Headers informativos

---

### TASK-030: Implementar Logging Centralizado
- **Prioridade**: 🟠 P1
- **Estimativa**: 8h
- **Dependências**: TASK-003
- **Status**: ⬜ TODO

**Descrição**: Configurar Elasticsearch + Kibana para logs.

**Critérios de Aceitação**:
- [ ] Elasticsearch configurado
- [ ] Kibana configurado
- [ ] Serilog configurado em todos os serviços
- [ ] Logs estruturados
- [ ] Correlation IDs
- [ ] Dashboards básicos em Kibana

---

### TASK-031: Implementar Distributed Tracing
- **Prioridade**: 🟡 P2
- **Estimativa**: 8h
- **Dependências**: TASK-030
- **Status**: ⬜ TODO

**Descrição**: Configurar OpenTelemetry + Jaeger.

**Critérios de Aceitação**:
- [ ] OpenTelemetry configurado
- [ ] Jaeger configurado
- [ ] Tracing em todos os serviços
- [ ] Visualização de traces
- [ ] Performance analysis

---

### TASK-032: Implementar Monitorização de Métricas
- **Prioridade**: 🟡 P2
- **Estimativa**: 8h
- **Dependências**: TASK-030
- **Status**: ⬜ TODO

**Descrição**: Configurar Prometheus + Grafana.

**Critérios de Aceitação**:
- [ ] Prometheus configurado
- [ ] Grafana configurado
- [ ] Métricas expostas pelos serviços
- [ ] Dashboards criados
- [ ] Alertas configurados

---

### TASK-033: Implementar Backups Automatizados
- **Prioridade**: 🟠 P1
- **Estimativa**: 6h
- **Dependências**: TASK-003
- **Status**: ⬜ TODO

**Descrição**: Configurar backups automáticos das bases de dados.

**Critérios de Aceitação**:
- [ ] Script de backup PostgreSQL
- [ ] Backup incremental
- [ ] Backup full semanal
- [ ] Retenção configurável
- [ ] Testes de restore

---

## 🧪 FASE 6: Testes

### TASK-034: Testes Unitários - Backend
- **Prioridade**: 🟠 P1
- **Estimativa**: 40h
- **Dependências**: TASK-006 a TASK-011
- **Status**: ⬜ TODO

**Descrição**: Criar testes unitários para todos os serviços backend.

**Critérios de Aceitação**:
- [ ] Cobertura mínima de 80%
- [ ] Testes de domain entities
- [ ] Testes de services
- [ ] Testes de repositories (com Testcontainers)
- [ ] Testes de validators

---

### TASK-035: Testes de Integração - Backend
- **Prioridade**: 🟡 P2
- **Estimativa**: 24h
- **Dependências**: TASK-034
- **Status**: ⬜ TODO

**Descrição**: Criar testes de integração entre serviços.

**Critérios de Aceitação**:
- [ ] Testes de APIs
- [ ] Testes com base de dados real (Testcontainers)
- [ ] Testes de comunicação entre serviços
- [ ] Testes de eventos

---

### TASK-036: Testes Unitários - Frontend
- **Prioridade**: 🟡 P2
- **Estimativa**: 32h
- **Dependências**: TASK-012 a TASK-018
- **Status**: ⬜ TODO

**Descrição**: Criar testes unitários para componentes React.

**Critérios de Aceitação**:
- [ ] Cobertura mínima de 70%
- [ ] Testes de componentes
- [ ] Testes de hooks
- [ ] Testes de Redux slices
- [ ] Testes de utils

---

### TASK-037: Testes E2E
- **Prioridade**: 🟡 P2
- **Estimativa**: 24h
- **Dependências**: TASK-035, TASK-036
- **Status**: ⬜ TODO

**Descrição**: Criar testes end-to-end com Playwright ou Cypress.

**Critérios de Aceitação**:
- [ ] Fluxo de autenticação
- [ ] Fluxo de registo de atividade
- [ ] Fluxo de visualização de relatórios
- [ ] Testes em múltiplos browsers
- [ ] Screenshots em falhas

---

## 🚀 FASE 7: DevOps e Deployment

### TASK-038: Configurar CI Pipeline
- **Prioridade**: 🟠 P1
- **Estimativa**: 8h
- **Dependências**: TASK-001
- **Status**: ⬜ TODO

**Descrição**: Configurar pipeline de CI (GitHub Actions ou GitLab CI).

**Critérios de Aceitação**:
- [ ] Build automático
- [ ] Testes automáticos
- [ ] Análise de código (SonarQube)
- [ ] Build de imagens Docker
- [ ] Push para registry

---

### TASK-039: Configurar CD Pipeline
- **Prioridade**: 🟠 P1
- **Estimativa**: 8h
- **Dependências**: TASK-038
- **Status**: ⬜ TODO

**Descrição**: Configurar pipeline de CD para ambientes.

**Critérios de Aceitação**:
- [ ] Deploy automático para dev
- [ ] Deploy automático para staging
- [ ] Deploy manual para produção
- [ ] Rollback automático em caso de falha

---

### TASK-040: Configurar Kubernetes (Produção)
- **Prioridade**: 🟡 P2
- **Estimativa**: 16h
- **Dependências**: TASK-039
- **Status**: ⬜ TODO

**Descrição**: Preparar manifests Kubernetes para produção.

**Critérios de Aceitação**:
- [ ] Deployments criados
- [ ] Services criados
- [ ] ConfigMaps e Secrets
- [ ] Ingress configurado
- [ ] HPA (Horizontal Pod Autoscaler)
- [ ] Persistent Volumes

---

## 📖 FASE 8: Documentação e Finalização

### TASK-041: Documentar APIs (Swagger/OpenAPI)
- **Prioridade**: 🟠 P1
- **Estimativa**: 16h
- **Dependências**: TASK-006 a TASK-011
- **Status**: ⬜ TODO

**Descrição**: Documentar completamente todas as APIs.

**Critérios de Aceitação**:
- [ ] Todos os endpoints documentados
- [ ] Exemplos de requisições
- [ ] Exemplos de respostas
- [ ] Códigos de erro documentados
- [ ] Schemas de dados completos

---

### TASK-042: Criar Guias de Utilizador
- **Prioridade**: 🟢 P3
- **Estimativa**: 16h
- **Dependências**: TASK-018, TASK-023
- **Status**: ⬜ TODO

**Descrição**: Criar documentação para utilizadores finais.

**Critérios de Aceitação**:
- [ ] Guia para pais
- [ ] Guia para educadores
- [ ] Guia para administradores
- [ ] FAQs
- [ ] Tutoriais em vídeo (opcional)

---

### TASK-043: Criar Documentação Técnica
- **Prioridade**: 🟠 P1
- **Estimativa**: 12h
- **Dependências**: TASK-041
- **Status**: ⬜ TODO

**Descrição**: Completar documentação técnica do projeto.

**Critérios de Aceitação**:
- [ ] Architecture Decision Records (ADRs)
- [ ] Diagramas atualizados
- [ ] Guias de deployment
- [ ] Troubleshooting guides
- [ ] Contributing guidelines

---

## 📊 Resumo de Estimativas

### Por Fase

| Fase | Descrição | Tarefas | Estimativa Total |
|------|-----------|---------|------------------|
| 0 | Configuração Inicial | 3 | 7h |
| 1 | Infraestrutura Base | 8 | 108h |
| 2 | Frontend Pais | 7 | 66h |
| 3 | Frontend Admin | 5 | 66h |
| 4 | Funcionalidades Avançadas | 5 | 64h |
| 5 | Segurança e Performance | 5 | 36h |
| 6 | Testes | 4 | 120h |
| 7 | DevOps | 3 | 32h |
| 8 | Documentação | 3 | 44h |
| **TOTAL** | | **43** | **543h** |

### Por Prioridade

| Prioridade | Tarefas | Estimativa |
|------------|---------|------------|
| 🔴 P0 | 12 | 146h |
| 🟠 P1 | 17 | 234h |
| 🟡 P2 | 11 | 140h |
| 🟢 P3 | 1 | 16h |

---

## 📅 Roadmap Sugerido

### Sprint 1-2 (4 semanas): Infraestrutura
- TASK-001 a TASK-005
- TASK-029 a TASK-030

### Sprint 3-5 (6 semanas): Serviços Core
- TASK-006 a TASK-009

### Sprint 6-7 (4 semanas): Serviços Complementares
- TASK-010 a TASK-011

### Sprint 8-10 (6 semanas): Frontend Pais
- TASK-012 a TASK-018

### Sprint 11-13 (6 semanas): Frontend Admin
- TASK-019 a TASK-023

### Sprint 14-15 (4 semanas): Features Avançadas
- TASK-024 a TASK-028

### Sprint 16-18 (6 semanas): Testes e Qualidade
- TASK-034 a TASK-037

### Sprint 19-20 (4 semanas): DevOps e Docs
- TASK-038 a TASK-043

**Duração Total Estimada**: 40 semanas (10 meses)

---

## 🔄 Processo de Desenvolvimento

### Para cada tarefa:

1. **Planeamento**
   - Ler critérios de aceitação
   - Verificar dependências
   - Estimar tempo necessário

2. **Desenvolvimento**
   - Criar branch feature/TASK-XXX
   - Implementar funcionalidade
   - Escrever testes
   - Commitar com mensagens descritivas

3. **Review**
   - Code review
   - Testes automáticos (CI)
   - Validação de qualidade

4. **Deploy**
   - Merge para develop
   - Deploy para ambiente de dev
   - Testes de integração

5. **Documentação**
   - Atualizar documentação técnica
   - Atualizar documentação de API
   - Atualizar changelog

---

## 📝 Notas Importantes

1. **Todas as tarefas devem seguir os padrões definidos na documentação**
2. **Código e comentários em português de Portugal**
3. **Commits descritivos e em português**
4. **Testes são obrigatórios para tarefas P0 e P1**
5. **Code review obrigatório antes de merge**
6. **Documentação da API é obrigatória**

---

**Anterior**: [Infraestrutura](./11-INFRAESTRUTURA.md)  
**Próximo**: [Guia de Desenvolvimento](./13-GUIA-DESENVOLVIMENTO.md)  
**Voltar ao**: [Índice](./00-INDICE.md)
