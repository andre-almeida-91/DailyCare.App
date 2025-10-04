# Multi-Tenancy - DailyCare

## 📖 Visão Geral

O DailyCare implementa uma estratégia de multi-tenancy baseada em **Database per Tenant**, garantindo isolamento completo de dados entre escolas.

---

## 🏗️ Arquitetura Multi-Tenant

### Estratégia: Database per Tenant

Cada escola (tenant) possui a sua própria base de dados PostgreSQL isolada.

```
┌─────────────────────────────────────────────────────┐
│           Master Database (Shared)                   │
│  ┌───────────────────────────────────────────────┐  │
│  │ Tabela: Tenants                               │  │
│  ├───────────────────────────────────────────────┤  │
│  │ Id          | Name      | ConnectionString   │  │
│  │ school-001  | Escola A  | Server=...;DB=...  │  │
│  │ school-002  | Escola B  | Server=...;DB=...  │  │
│  └───────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────┐  │
│  │ Tabela: Users (Global)                        │  │
│  ├───────────────────────────────────────────────┤  │
│  │ Id | Email | Role | TenantId                  │  │
│  └───────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────┘
                      │
        ┌─────────────┼─────────────┐
        │             │             │
        ▼             ▼             ▼
┌─────────────┐ ┌─────────────┐ ┌─────────────┐
│  Tenant DB  │ │  Tenant DB  │ │  Tenant DB  │
│  Escola A   │ │  Escola B   │ │  Escola N   │
├─────────────┤ ├─────────────┤ ├─────────────┤
│ • Children  │ │ • Children  │ │ • Children  │
│ • Activities│ │ • Activities│ │ • Activities│
│ • Health    │ │ • Health    │ │ • Health    │
│ • Classes   │ │ • Classes   │ │ • Classes   │
└─────────────┘ └─────────────┘ └─────────────┘
```

---

## 🔑 Vantagens da Abordagem

### ✅ Isolamento Total
- Dados completamente isolados entre escolas
- Impossibilidade de acesso cruzado acidental
- Compliance com RGPD facilitado

### ✅ Performance Otimizada
- Cada escola tem recursos dedicados
- Sem queries com filtros de tenant
- Índices otimizados por escola

### ✅ Backup e Restore Independente
- Backup individual por escola
- Restore sem afetar outros tenants
- Gestão de dados simplificada

### ✅ Escalabilidade
- Distribuição de cargas por diferentes servidores
- Possibilidade de tiers diferentes (básico, premium)
- Migração facilitada

### ✅ Segurança
- Risco de data leakage minimizado
- Auditoria independente
- Conformidade com regulações

---

## ⚙️ Implementação

### 1. Master Database

#### Schema: Tenants

```sql
CREATE TABLE Tenants (
    Id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    Name VARCHAR(200) NOT NULL,
    Code VARCHAR(50) NOT NULL UNIQUE,
    Email VARCHAR(200) NOT NULL,
    PhoneNumber VARCHAR(20),
    ConnectionString TEXT NOT NULL,
    IsActive BOOLEAN DEFAULT TRUE,
    ActivatedAt TIMESTAMP,
    CreatedAt TIMESTAMP DEFAULT NOW(),
    UpdatedAt TIMESTAMP,
    IsDeleted BOOLEAN DEFAULT FALSE
);

CREATE INDEX idx_tenants_code ON Tenants(Code) WHERE IsDeleted = FALSE;
CREATE INDEX idx_tenants_active ON Tenants(IsActive) WHERE IsDeleted = FALSE;
```

#### Schema: Users (Global)

```sql
CREATE TABLE Users (
    Id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    TenantId UUID REFERENCES Tenants(Id),
    Name VARCHAR(200) NOT NULL,
    Email VARCHAR(200) NOT NULL UNIQUE,
    PasswordHash TEXT NOT NULL,
    PhoneNumber VARCHAR(20),
    Role VARCHAR(50) NOT NULL,
    EmailConfirmed BOOLEAN DEFAULT FALSE,
    PhoneConfirmed BOOLEAN DEFAULT FALSE,
    IsActive BOOLEAN DEFAULT TRUE,
    CreatedAt TIMESTAMP DEFAULT NOW(),
    UpdatedAt TIMESTAMP,
    LastLoginAt TIMESTAMP,
    IsDeleted BOOLEAN DEFAULT FALSE
);

CREATE INDEX idx_users_email ON Users(Email) WHERE IsDeleted = FALSE;
CREATE INDEX idx_users_tenant ON Users(TenantId) WHERE IsDeleted = FALSE;
```

---

### 2. Tenant Resolution Middleware

```csharp
// src/backend/shared/DailyCare.Shared.Infrastructure/MultiTenancy/TenantResolutionMiddleware.cs

public class TenantResolutionMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ITenantResolver _tenantResolver;

    public TenantResolutionMiddleware(RequestDelegate next, ITenantResolver tenantResolver)
    {
        _next = next;
        _tenantResolver = tenantResolver;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        // Extrai o TenantId do header X-Tenant-Id
        var tenantId = context.Request.Headers["X-Tenant-Id"].FirstOrDefault();
        
        if (string.IsNullOrEmpty(tenantId))
        {
            // Tenta obter do claim do JWT
            tenantId = context.User?.FindFirst("tenant_id")?.Value;
        }

        if (!string.IsNullOrEmpty(tenantId))
        {
            var tenant = await _tenantResolver.ResolveAsync(tenantId);
            
            if (tenant == null)
            {
                context.Response.StatusCode = 400;
                await context.Response.WriteAsJsonAsync(new
                {
                    success = false,
                    error = new
                    {
                        code = "INVALID_TENANT",
                        message = "Tenant inválido ou não encontrado"
                    }
                });
                return;
            }

            // Armazena o tenant no contexto HTTP
            context.Items["Tenant"] = tenant;
        }

        await _next(context);
    }
}
```

---

### 3. Tenant Context

```csharp
// src/backend/shared/DailyCare.Shared.Domain/MultiTenancy/TenantContext.cs

public class TenantContext : ITenantContext
{
    private readonly IHttpContextAccessor _httpContextAccessor;

    public TenantContext(IHttpContextAccessor httpContextAccessor)
    {
        _httpContextAccessor = httpContextAccessor;
    }

    public Tenant? CurrentTenant
    {
        get
        {
            if (_httpContextAccessor.HttpContext?.Items.TryGetValue("Tenant", out var tenant) == true)
            {
                return tenant as Tenant;
            }
            return null;
        }
    }

    public string? TenantId => CurrentTenant?.Id.ToString();
    
    public string? ConnectionString => CurrentTenant?.ConnectionString;
}
```

---

### 4. Dynamic DbContext

```csharp
// src/backend/shared/DailyCare.Shared.Infrastructure/Persistence/TenantDbContext.cs

public class TenantDbContext : DbContext
{
    private readonly ITenantContext _tenantContext;

    public TenantDbContext(
        DbContextOptions<TenantDbContext> options,
        ITenantContext tenantContext) : base(options)
    {
        _tenantContext = tenantContext;
    }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        if (!optionsBuilder.IsConfigured && !string.IsNullOrEmpty(_tenantContext.ConnectionString))
        {
            optionsBuilder.UseNpgsql(_tenantContext.ConnectionString);
        }
    }

    public override async Task<int> SaveChangesAsync(CancellationToken cancellationToken = default)
    {
        // Auditoria automática
        var entries = ChangeTracker
            .Entries()
            .Where(e => e.Entity is BaseEntity && 
                       (e.State == EntityState.Added || e.State == EntityState.Modified));

        foreach (var entry in entries)
        {
            var entity = (BaseEntity)entry.Entity;
            
            if (entry.State == EntityState.Added)
            {
                entity.CreatedAt = DateTime.UtcNow;
                entity.CreatedBy = _tenantContext.CurrentTenant?.Id.ToString();
            }
            
            entity.UpdatedAt = DateTime.UtcNow;
            entity.UpdatedBy = _tenantContext.CurrentTenant?.Id.ToString();
        }

        return await base.SaveChangesAsync(cancellationToken);
    }
}
```

---

### 5. Tenant Provisioning Service

```csharp
// src/backend/services/school/DailyCare.School.Application/Services/TenantProvisioningService.cs

public class TenantProvisioningService : ITenantProvisioningService
{
    private readonly IMasterDbContext _masterDbContext;
    private readonly IConfiguration _configuration;
    private readonly ILogger<TenantProvisioningService> _logger;

    public async Task<Tenant> ProvisionTenantAsync(CreateTenantRequest request)
    {
        _logger.LogInformation("Iniciando provisionamento do tenant {TenantName}", request.Name);

        try
        {
            // 1. Criar tenant na master database
            var tenant = new Tenant
            {
                Id = Guid.NewGuid(),
                Name = request.Name,
                Code = request.Code,
                Email = request.Email,
                PhoneNumber = request.PhoneNumber,
                IsActive = false // Ativo apenas após provisionamento completo
            };

            // 2. Gerar connection string
            var baseConnectionString = _configuration.GetConnectionString("TenantTemplate");
            var databaseName = $"dailycare_tenant_{tenant.Code.ToLower()}";
            tenant.ConnectionString = baseConnectionString.Replace("{DatabaseName}", databaseName);

            // 3. Criar base de dados
            await CreateDatabaseAsync(tenant.ConnectionString);

            // 4. Executar migrations
            await RunMigrationsAsync(tenant.ConnectionString);

            // 5. Seed dados iniciais
            await SeedInitialDataAsync(tenant.ConnectionString, request);

            // 6. Criar utilizador admin
            await CreateAdminUserAsync(tenant.Id, request.AdminUser);

            // 7. Ativar tenant
            tenant.IsActive = true;
            tenant.ActivatedAt = DateTime.UtcNow;

            await _masterDbContext.Tenants.AddAsync(tenant);
            await _masterDbContext.SaveChangesAsync();

            _logger.LogInformation("Tenant {TenantName} provisionado com sucesso", request.Name);

            return tenant;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Erro ao provisionar tenant {TenantName}", request.Name);
            
            // Rollback: tentar limpar recursos criados
            await CleanupFailedProvisioningAsync(request.Code);
            
            throw;
        }
    }

    private async Task CreateDatabaseAsync(string connectionString)
    {
        var builder = new NpgsqlConnectionStringBuilder(connectionString);
        var databaseName = builder.Database;
        builder.Database = "postgres"; // Conecta à DB padrão para criar nova

        using var connection = new NpgsqlConnection(builder.ToString());
        await connection.OpenAsync();

        using var command = connection.CreateCommand();
        command.CommandText = $@"
            CREATE DATABASE {databaseName}
            WITH OWNER = postgres
            ENCODING = 'UTF8'
            CONNECTION LIMIT = 100;
        ";

        await command.ExecuteNonQueryAsync();
        
        _logger.LogInformation("Base de dados {DatabaseName} criada", databaseName);
    }

    private async Task RunMigrationsAsync(string connectionString)
    {
        var optionsBuilder = new DbContextOptionsBuilder<TenantDbContext>();
        optionsBuilder.UseNpgsql(connectionString);

        using var context = new TenantDbContext(optionsBuilder.Options, null!);
        await context.Database.MigrateAsync();
        
        _logger.LogInformation("Migrations executadas com sucesso");
    }

    private async Task SeedInitialDataAsync(string connectionString, CreateTenantRequest request)
    {
        // Seed de dados mestre: roles, permissions, configurações padrão
        _logger.LogInformation("Dados iniciais criados");
    }

    private async Task CreateAdminUserAsync(Guid tenantId, AdminUserRequest adminUser)
    {
        var user = new User
        {
            TenantId = tenantId,
            Name = adminUser.Name,
            Email = adminUser.Email,
            Role = "SchoolAdmin",
            EmailConfirmed = true,
            IsActive = true
        };

        // Criar password hash
        user.PasswordHash = BCrypt.Net.BCrypt.HashPassword(GenerateTemporaryPassword());

        await _masterDbContext.Users.AddAsync(user);
        await _masterDbContext.SaveChangesAsync();

        // Enviar email com password temporária
        // await _emailService.SendWelcomeEmailAsync(user);
        
        _logger.LogInformation("Utilizador admin criado para tenant");
    }

    private async Task CleanupFailedProvisioningAsync(string tenantCode)
    {
        try
        {
            var databaseName = $"dailycare_tenant_{tenantCode.ToLower()}";
            // Código para dropar a database se foi criada
            _logger.LogInformation("Limpeza de provisionamento falhado executada");
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Erro ao limpar provisionamento falhado");
        }
    }

    private string GenerateTemporaryPassword()
    {
        // Gerar password temporária segura
        var chars = "ABCDEFGHJKLMNOPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz0123456789!@#$%";
        var random = new Random();
        return new string(Enumerable.Repeat(chars, 12)
            .Select(s => s[random.Next(s.Length)]).ToArray());
    }
}
```

---

## 🔄 Connection String Management

### Configuração no appsettings.json

```json
{
  "ConnectionStrings": {
    "MasterDatabase": "Host=localhost;Database=dailycare_master;Username=postgres;Password=postgres",
    "TenantTemplate": "Host=localhost;Database={DatabaseName};Username=postgres;Password=postgres"
  },
  "MultiTenancy": {
    "ConnectionPoolSize": 100,
    "ConnectionTimeout": 30,
    "CommandTimeout": 60,
    "EnableConnectionPooling": true
  }
}
```

---

## 🎯 Tenant Identification Flow

### 1. Request Flow

```
┌─────────────┐
│   Client    │
└──────┬──────┘
       │ 1. Request com X-Tenant-Id header
       ▼
┌─────────────────────┐
│   API Gateway       │
│  (Valida JWT)       │
└──────┬──────────────┘
       │ 2. Adiciona tenant_id ao contexto
       ▼
┌─────────────────────┐
│  Microservice       │
│  Middleware         │
└──────┬──────────────┘
       │ 3. Resolve tenant via TenantResolver
       ▼
┌─────────────────────┐
│  Master Database    │
│  (Busca tenant)     │
└──────┬──────────────┘
       │ 4. Retorna ConnectionString
       ▼
┌─────────────────────┐
│  Tenant Database    │
│  (Query dados)      │
└──────┬──────────────┘
       │ 5. Retorna dados
       ▼
┌─────────────┐
│   Client    │
└─────────────┘
```

---

## 🔐 Segurança Multi-Tenant

### 1. Prevenção de Acesso Cruzado

```csharp
// Validação automática no DbContext
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Query filter global - nunca retorna dados de outro tenant
    foreach (var entityType in modelBuilder.Model.GetEntityTypes())
    {
        if (typeof(ITenantEntity).IsAssignableFrom(entityType.ClrType))
        {
            var parameter = Expression.Parameter(entityType.ClrType, "e");
            var tenantIdProperty = Expression.Property(parameter, nameof(ITenantEntity.TenantId));
            var currentTenantId = Expression.Constant(_tenantContext.TenantId);
            var body = Expression.Equal(tenantIdProperty, currentTenantId);
            var lambda = Expression.Lambda(body, parameter);

            modelBuilder.Entity(entityType.ClrType).HasQueryFilter(lambda);
        }
    }
}
```

### 2. Auditoria de Acesso

```csharp
public class TenantAccessAudit
{
    public Guid Id { get; set; }
    public Guid TenantId { get; set; }
    public Guid UserId { get; set; }
    public string Action { get; set; }
    public string Resource { get; set; }
    public string IpAddress { get; set; }
    public DateTime Timestamp { get; set; }
    public bool Success { get; set; }
}
```

---

## 📊 Monitorização

### Métricas por Tenant

- Número de utilizadores ativos
- Número de crianças registadas
- Número de atividades registadas
- Uso de storage
- Performance de queries
- Uptime da base de dados

### Alertas

- Crescimento anormal de dados
- Tentativas de acesso não autorizado
- Performance degradada
- Erros de conexão

---

## 🔄 Migração de Tenants

### Processo de Migração

```csharp
public class TenantMigrationService
{
    public async Task MigrateTenantAsync(Guid tenantId, string newConnectionString)
    {
        var tenant = await _masterDbContext.Tenants.FindAsync(tenantId);
        
        // 1. Backup da base de dados atual
        await BackupDatabaseAsync(tenant.ConnectionString);
        
        // 2. Criar nova base de dados
        await CreateDatabaseAsync(newConnectionString);
        
        // 3. Copiar dados
        await CopyDataAsync(tenant.ConnectionString, newConnectionString);
        
        // 4. Validar integridade
        await ValidateDataIntegrityAsync(newConnectionString);
        
        // 5. Atualizar connection string
        tenant.ConnectionString = newConnectionString;
        await _masterDbContext.SaveChangesAsync();
        
        // 6. Limpar conexões antigas
        await ClearConnectionPoolAsync(tenant.Id);
    }
}
```

---

## 🗄️ Backup e Restore

### Estratégia de Backup

```bash
#!/bin/bash
# Script de backup automático por tenant

TENANT_CODE=$1
BACKUP_DIR="/backups/tenants"
DATE=$(date +%Y%m%d_%H%M%S)
DB_NAME="dailycare_tenant_${TENANT_CODE}"

# Full backup
pg_dump -h localhost -U postgres -d $DB_NAME | gzip > \
    "${BACKUP_DIR}/${TENANT_CODE}/full_${DATE}.sql.gz"

# Incremental backup (usando WAL archiving)
pg_basebackup -h localhost -U postgres -D \
    "${BACKUP_DIR}/${TENANT_CODE}/incremental_${DATE}" -F tar -z -P

# Retenção: 30 dias de backups diários, 1 ano de backups semanais
find "${BACKUP_DIR}/${TENANT_CODE}" -name "full_*.sql.gz" -mtime +30 -delete
```

---

## 💾 Cache Multi-Tenant

### Redis Keys com Tenant Prefix

```csharp
public class TenantCacheService
{
    private readonly IDistributedCache _cache;
    private readonly ITenantContext _tenantContext;

    public async Task<T?> GetAsync<T>(string key)
    {
        var tenantKey = GetTenantKey(key);
        var data = await _cache.GetStringAsync(tenantKey);
        
        return data == null ? default : JsonSerializer.Deserialize<T>(data);
    }

    public async Task SetAsync<T>(string key, T value, TimeSpan? expiration = null)
    {
        var tenantKey = GetTenantKey(key);
        var data = JsonSerializer.Serialize(value);
        
        var options = new DistributedCacheEntryOptions
        {
            AbsoluteExpirationRelativeToNow = expiration ?? TimeSpan.FromHours(1)
        };
        
        await _cache.SetStringAsync(tenantKey, data, options);
    }

    private string GetTenantKey(string key)
    {
        return $"tenant:{_tenantContext.TenantId}:{key}";
    }
}
```

---

## 📈 Escalabilidade

### Distribuição de Tenants

```
┌─────────────────────────────────────────┐
│   PostgreSQL Cluster - Primary          │
│   • Tenants Premium (Tier 1)           │
│   • SLA 99.99%                         │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│   PostgreSQL Server 1                   │
│   • Tenants Standard (Tier 2)          │
│   • Escolas 1-100                       │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│   PostgreSQL Server 2                   │
│   • Tenants Standard (Tier 2)          │
│   • Escolas 101-200                     │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│   PostgreSQL Server 3                   │
│   • Tenants Basic (Tier 3)             │
│   • Escolas com menos recursos          │
└─────────────────────────────────────────┘
```

---

## 🧪 Testes Multi-Tenant

### Teste de Isolamento

```csharp
[Fact]
public async Task Should_Not_Access_Data_From_Different_Tenant()
{
    // Arrange
    var tenant1Id = Guid.NewGuid();
    var tenant2Id = Guid.NewGuid();
    
    // Criar dados no tenant 1
    using (var scope = CreateScopeWithTenant(tenant1Id))
    {
        var context = scope.ServiceProvider.GetRequiredService<TenantDbContext>();
        context.Children.Add(new Child { Name = "Child from Tenant 1" });
        await context.SaveChangesAsync();
    }
    
    // Tentar aceder com tenant 2
    using (var scope = CreateScopeWithTenant(tenant2Id))
    {
        var context = scope.ServiceProvider.GetRequiredService<TenantDbContext>();
        var children = await context.Children.ToListAsync();
        
        // Assert
        Assert.Empty(children); // Não deve ver dados do tenant 1
    }
}
```

---

## 📝 Checklist de Implementação

- [ ] Master database configurada
- [ ] Tenant resolution middleware implementado
- [ ] Dynamic connection string funcionando
- [ ] Provisioning service testado
- [ ] Migrations automáticas
- [ ] Backup/restore testado
- [ ] Cache multi-tenant implementado
- [ ] Auditoria de acessos
- [ ] Testes de isolamento
- [ ] Documentação de APIs
- [ ] Monitorização configurada

---

**Anterior**: [Base de Dados](./07-BASE-DADOS.md)  
**Próximo**: [Segurança e Autenticação](./09-SEGURANCA-AUTENTICACAO.md)  
**Voltar ao**: [Índice](./00-INDICE.md)
