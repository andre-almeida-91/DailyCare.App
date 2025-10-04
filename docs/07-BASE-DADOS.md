# Estrutura de Base de Dados - DailyCare

## 📊 Visão Geral

O DailyCare utiliza PostgreSQL 16 com uma estratégia de **Database per Tenant** para isolamento completo de dados entre escolas.

---

## 🗄️ Arquitetura de Bases de Dados

### Master Database

Base de dados centralizada para gestão de tenants e utilizadores globais.

```sql
-- Master Database: dailycare_master

-- Tenants (Escolas)
CREATE TABLE tenants (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(200) NOT NULL,
    code VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(200) NOT NULL,
    phone_number VARCHAR(20),
    connection_string TEXT NOT NULL,
    is_active BOOLEAN DEFAULT TRUE,
    activated_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP,
    created_by VARCHAR(100),
    updated_by VARCHAR(100),
    is_deleted BOOLEAN DEFAULT FALSE
);

-- Utilizadores Globais
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID REFERENCES tenants(id),
    name VARCHAR(200) NOT NULL,
    email VARCHAR(200) NOT NULL UNIQUE,
    password_hash TEXT NOT NULL,
    phone_number VARCHAR(20),
    role VARCHAR(50) NOT NULL,
    email_confirmed BOOLEAN DEFAULT FALSE,
    phone_confirmed BOOLEAN DEFAULT FALSE,
    is_active BOOLEAN DEFAULT TRUE,
    last_login_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP,
    is_deleted BOOLEAN DEFAULT FALSE
);

-- Roles
CREATE TABLE roles (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL UNIQUE,
    description TEXT,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Permissions
CREATE TABLE permissions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL UNIQUE,
    resource VARCHAR(100) NOT NULL,
    action VARCHAR(50) NOT NULL,
    description TEXT,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Role Permissions
CREATE TABLE role_permissions (
    role_id UUID REFERENCES roles(id),
    permission_id UUID REFERENCES permissions(id),
    PRIMARY KEY (role_id, permission_id)
);

-- User Roles
CREATE TABLE user_roles (
    user_id UUID REFERENCES users(id),
    role_id UUID REFERENCES roles(id),
    assigned_at TIMESTAMP DEFAULT NOW(),
    PRIMARY KEY (user_id, role_id)
);

-- Refresh Tokens
CREATE TABLE refresh_tokens (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id),
    token VARCHAR(500) NOT NULL UNIQUE,
    expires_at TIMESTAMP NOT NULL,
    is_revoked BOOLEAN DEFAULT FALSE,
    revoked_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Índices
CREATE INDEX idx_users_email ON users(email) WHERE is_deleted = FALSE;
CREATE INDEX idx_users_tenant ON users(tenant_id) WHERE is_deleted = FALSE;
CREATE INDEX idx_tenants_code ON tenants(code) WHERE is_deleted = FALSE;
CREATE INDEX idx_tenants_active ON tenants(is_active) WHERE is_deleted = FALSE;
CREATE INDEX idx_refresh_tokens_user ON refresh_tokens(user_id);
CREATE INDEX idx_refresh_tokens_token ON refresh_tokens(token) WHERE is_revoked = FALSE;
```

---

## 🏫 Tenant Database Schema

Cada escola tem a sua própria base de dados com o seguinte schema.

### Schema: Children

```sql
-- Crianças
CREATE TABLE children (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    birth_date DATE NOT NULL,
    gender VARCHAR(20) NOT NULL,
    blood_type VARCHAR(10),
    photo_url TEXT,
    current_class_id UUID,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP,
    created_by VARCHAR(100),
    updated_by VARCHAR(100),
    is_deleted BOOLEAN DEFAULT FALSE
);

-- Informação de Emergência
CREATE TABLE emergency_contacts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    child_id UUID REFERENCES children(id),
    name VARCHAR(200) NOT NULL,
    phone_number VARCHAR(20) NOT NULL,
    relationship VARCHAR(50) NOT NULL,
    is_primary BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Informação Médica
CREATE TABLE child_medical_info (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    child_id UUID REFERENCES children(id),
    allergies JSONB,
    medications JSONB,
    special_needs TEXT,
    notes TEXT,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP
);

-- Membros da Família
CREATE TABLE family_members (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    child_id UUID REFERENCES children(id),
    user_id UUID NOT NULL, -- Referência ao user na master DB
    relationship VARCHAR(50) NOT NULL,
    is_primary_contact BOOLEAN DEFAULT FALSE,
    can_pickup BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Matrículas
CREATE TABLE enrollments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    child_id UUID REFERENCES children(id),
    class_id UUID NOT NULL,
    start_date DATE NOT NULL,
    end_date DATE,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Índices
CREATE INDEX idx_children_class ON children(current_class_id) WHERE is_deleted = FALSE;
CREATE INDEX idx_family_members_child ON family_members(child_id);
CREATE INDEX idx_family_members_user ON family_members(user_id);
```

### Schema: Activities

```sql
-- Atividades de Alimentação
CREATE TABLE feeding_activities (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    child_id UUID REFERENCES children(id),
    start_time TIMESTAMP NOT NULL,
    end_time TIMESTAMP,
    type VARCHAR(50) NOT NULL, -- Breakfast, Lunch, Dinner, Snack, etc.
    food_description TEXT,
    quantity DECIMAL(10,2),
    unit VARCHAR(20), -- Portion, ML, Grams
    appetite VARCHAR(20), -- Excellent, Good, Fair, Poor
    notes TEXT,
    registered_by VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP
);

-- Atividades de Sono
CREATE TABLE sleep_activities (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    child_id UUID REFERENCES children(id),
    start_time TIMESTAMP NOT NULL,
    end_time TIMESTAMP NOT NULL,
    duration_minutes INTEGER GENERATED ALWAYS AS 
        (EXTRACT(EPOCH FROM (end_time - start_time))/60) STORED,
    type VARCHAR(20) NOT NULL, -- Nap, Night
    quality VARCHAR(20), -- Excellent, Good, Fair, Poor, Restless
    notes TEXT,
    registered_by VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Atividades de Fralda
CREATE TABLE diaper_activities (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    child_id UUID REFERENCES children(id),
    time TIMESTAMP NOT NULL,
    type VARCHAR(20) NOT NULL, -- Wet, Dirty, Both
    condition VARCHAR(20), -- Normal, Dry, Loose, Hard, Unusual
    notes TEXT,
    registered_by VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Atividades Físicas
CREATE TABLE physical_activities (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    child_id UUID REFERENCES children(id),
    start_time TIMESTAMP NOT NULL,
    end_time TIMESTAMP,
    activity_type VARCHAR(50) NOT NULL,
    description TEXT,
    participation_level VARCHAR(20), -- High, Medium, Low
    notes TEXT,
    registered_by VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Marcos de Desenvolvimento
CREATE TABLE milestones (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    child_id UUID REFERENCES children(id),
    title VARCHAR(200) NOT NULL,
    description TEXT,
    category VARCHAR(50) NOT NULL, -- Physical, Cognitive, Language, Social, Emotional
    achieved_at TIMESTAMP NOT NULL,
    photo_url TEXT,
    notes TEXT,
    registered_by VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Índices para Performance
CREATE INDEX idx_feeding_child_time ON feeding_activities(child_id, start_time DESC);
CREATE INDEX idx_sleep_child_time ON sleep_activities(child_id, start_time DESC);
CREATE INDEX idx_diaper_child_time ON diaper_activities(child_id, time DESC);
CREATE INDEX idx_physical_child_time ON physical_activities(child_id, start_time DESC);
CREATE INDEX idx_milestones_child_category ON milestones(child_id, category);
```

### Schema: Health

```sql
-- Registos de Temperatura
CREATE TABLE temperature_records (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    child_id UUID REFERENCES children(id),
    measured_at TIMESTAMP NOT NULL,
    temperature DECIMAL(4,1) NOT NULL,
    unit VARCHAR(10) NOT NULL, -- Celsius, Fahrenheit
    symptoms TEXT,
    notes TEXT,
    measured_by VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Medicamentos
CREATE TABLE medications (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    child_id UUID REFERENCES children(id),
    name VARCHAR(200) NOT NULL,
    dosage VARCHAR(100) NOT NULL,
    start_date DATE NOT NULL,
    end_date DATE,
    frequency VARCHAR(100),
    prescribed_by VARCHAR(200),
    notes TEXT,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP
);

-- Doses de Medicamentos
CREATE TABLE medication_doses (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    medication_id UUID REFERENCES medications(id),
    administered_at TIMESTAMP NOT NULL,
    administered_by VARCHAR(100) NOT NULL,
    notes TEXT,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Vacinas
CREATE TABLE vaccinations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    child_id UUID REFERENCES children(id),
    vaccine_name VARCHAR(200) NOT NULL,
    scheduled_date DATE NOT NULL,
    administered_date DATE,
    status VARCHAR(20) NOT NULL, -- Scheduled, Administered, Overdue, Cancelled
    location VARCHAR(200),
    batch_number VARCHAR(100),
    notes TEXT,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP
);

-- Consultas Médicas
CREATE TABLE medical_appointments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    child_id UUID REFERENCES children(id),
    appointment_date TIMESTAMP NOT NULL,
    doctor_name VARCHAR(200),
    specialty VARCHAR(100),
    reason TEXT,
    diagnosis TEXT,
    treatment TEXT,
    notes TEXT,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP
);

-- Alergias
CREATE TABLE allergies (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    child_id UUID REFERENCES children(id),
    allergen_name VARCHAR(200) NOT NULL,
    type VARCHAR(50) NOT NULL, -- Food, Medicine, Environmental, Other
    severity VARCHAR(20) NOT NULL, -- Mild, Moderate, Severe
    reaction TEXT,
    diagnosed_at DATE,
    notes TEXT,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP
);

-- Índices
CREATE INDEX idx_temperature_child_date ON temperature_records(child_id, measured_at DESC);
CREATE INDEX idx_medications_child_active ON medications(child_id, is_active);
CREATE INDEX idx_medication_doses_medication ON medication_doses(medication_id, administered_at DESC);
CREATE INDEX idx_vaccinations_child_status ON vaccinations(child_id, status);
CREATE INDEX idx_appointments_child_date ON medical_appointments(child_id, appointment_date DESC);
```

### Schema: School

```sql
-- Turmas
CREATE TABLE classes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    description TEXT,
    min_age INTEGER,
    max_age INTEGER,
    capacity INTEGER NOT NULL,
    academic_year VARCHAR(20),
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP
);

-- Educadores de Turma
CREATE TABLE class_educators (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    class_id UUID REFERENCES classes(id),
    educator_id UUID NOT NULL, -- Referência ao user na master DB
    role VARCHAR(50) NOT NULL, -- Lead, Assistant
    assigned_at TIMESTAMP DEFAULT NOW(),
    removed_at TIMESTAMP
);

-- Configurações da Escola
CREATE TABLE school_settings (
    key VARCHAR(100) PRIMARY KEY,
    value JSONB NOT NULL,
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Índices
CREATE INDEX idx_classes_active ON classes(is_active);
CREATE INDEX idx_class_educators_class ON class_educators(class_id);
```

### Schema: Notifications

```sql
-- Notificações
CREATE TABLE notifications (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL, -- Referência ao user na master DB
    title VARCHAR(200) NOT NULL,
    message TEXT NOT NULL,
    type VARCHAR(50) NOT NULL, -- Activity, Health, Milestone, System
    related_entity_type VARCHAR(50),
    related_entity_id UUID,
    data JSONB,
    is_read BOOLEAN DEFAULT FALSE,
    read_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Preferências de Notificação
CREATE TABLE notification_preferences (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL,
    channel VARCHAR(20) NOT NULL, -- Email, Push, SMS
    type VARCHAR(50) NOT NULL,
    enabled BOOLEAN DEFAULT TRUE,
    updated_at TIMESTAMP DEFAULT NOW(),
    UNIQUE(user_id, channel, type)
);

-- Índices
CREATE INDEX idx_notifications_user_read ON notifications(user_id, is_read);
CREATE INDEX idx_notifications_created ON notifications(created_at DESC);
```

### Schema: Files

```sql
-- Ficheiros
CREATE TABLE files (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    original_name VARCHAR(255) NOT NULL,
    mime_type VARCHAR(100) NOT NULL,
    size_bytes BIGINT NOT NULL,
    path TEXT NOT NULL,
    thumbnail_path TEXT,
    related_entity_type VARCHAR(50),
    related_entity_id UUID,
    uploaded_by VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Galeria de Fotos
CREATE TABLE photo_gallery (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    child_id UUID REFERENCES children(id),
    file_id UUID REFERENCES files(id),
    title VARCHAR(200),
    description TEXT,
    taken_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Índices
CREATE INDEX idx_files_entity ON files(related_entity_type, related_entity_id);
CREATE INDEX idx_photo_gallery_child ON photo_gallery(child_id, created_at DESC);
```

---

## 🔧 Migrations com EF Core

### Estrutura de Migrations

```csharp
// Infrastructure/Persistence/Migrations/20250104000001_Initial.cs
public partial class Initial : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.CreateTable(
            name: "children",
            columns: table => new
            {
                id = table.Column<Guid>(nullable: false),
                first_name = table.Column<string>(maxLength: 100, nullable: false),
                last_name = table.Column<string>(maxLength: 100, nullable: false),
                birth_date = table.Column<DateTime>(nullable: false),
                gender = table.Column<string>(maxLength: 20, nullable: false),
                blood_type = table.Column<string>(maxLength: 10, nullable: true),
                photo_url = table.Column<string>(nullable: true),
                current_class_id = table.Column<Guid>(nullable: true),
                created_at = table.Column<DateTime>(nullable: false, defaultValueSql: "NOW()"),
                updated_at = table.Column<DateTime>(nullable: true),
                created_by = table.Column<string>(maxLength: 100, nullable: true),
                updated_by = table.Column<string>(maxLength: 100, nullable: true),
                is_deleted = table.Column<bool>(nullable: false, defaultValue: false)
            },
            constraints: table =>
            {
                table.PrimaryKey("pk_children", x => x.id);
            });

        migrationBuilder.CreateIndex(
            name: "idx_children_class",
            table: "children",
            column: "current_class_id",
            filter: "is_deleted = FALSE");
    }
}
```

### Aplicar Migrations

```bash
# Master Database
cd src/backend/services/identity
dotnet ef migrations add Initial --context MasterDbContext
dotnet ef database update --context MasterDbContext

# Tenant Database Template
cd src/backend/services/children
dotnet ef migrations add Initial --context TenantDbContext
dotnet ef database update --context TenantDbContext
```

---

## 📈 Otimizações de Performance

### 1. Índices Estratégicos

```sql
-- Índices para queries frequentes
CREATE INDEX idx_feeding_recent ON feeding_activities(child_id, start_time DESC)
    WHERE created_at > NOW() - INTERVAL '30 days';

-- Índices parciais para dados ativos
CREATE INDEX idx_medications_active ON medications(child_id)
    WHERE is_active = TRUE AND is_deleted = FALSE;

-- Índices para pesquisa full-text
CREATE INDEX idx_children_name_gin ON children 
    USING gin(to_tsvector('portuguese', first_name || ' ' || last_name))
    WHERE is_deleted = FALSE;
```

### 2. Particionamento (para grandes volumes)

```sql
-- Particionar atividades por mês
CREATE TABLE feeding_activities (
    id UUID NOT NULL,
    child_id UUID NOT NULL,
    start_time TIMESTAMP NOT NULL,
    -- outros campos...
) PARTITION BY RANGE (start_time);

-- Criar partições
CREATE TABLE feeding_activities_2025_01 
    PARTITION OF feeding_activities
    FOR VALUES FROM ('2025-01-01') TO ('2025-02-01');

CREATE TABLE feeding_activities_2025_02 
    PARTITION OF feeding_activities
    FOR VALUES FROM ('2025-02-01') TO ('2025-03-01');
```

### 3. Materialized Views para Relatórios

```sql
-- View materializada para estatísticas diárias
CREATE MATERIALIZED VIEW daily_activity_stats AS
SELECT 
    child_id,
    DATE(start_time) as activity_date,
    COUNT(*) as total_feedings,
    AVG(duration_minutes) as avg_duration,
    STRING_AGG(DISTINCT appetite, ', ') as appetites
FROM feeding_activities
WHERE created_at > NOW() - INTERVAL '90 days'
GROUP BY child_id, DATE(start_time);

CREATE UNIQUE INDEX ON daily_activity_stats(child_id, activity_date);

-- Refresh periódico
REFRESH MATERIALIZED VIEW CONCURRENTLY daily_activity_stats;
```

---

## 🔐 Segurança de Dados

### Row Level Security (RLS)

```sql
-- Ativar RLS
ALTER TABLE children ENABLE ROW LEVEL SECURITY;

-- Política para educadores verem apenas crianças das suas turmas
CREATE POLICY educator_children_policy ON children
    FOR SELECT
    USING (
        current_class_id IN (
            SELECT class_id FROM class_educators 
            WHERE educator_id = current_setting('app.current_user_id')::UUID
        )
    );

-- Política para pais verem apenas os seus filhos
CREATE POLICY parent_children_policy ON children
    FOR SELECT
    USING (
        id IN (
            SELECT child_id FROM family_members 
            WHERE user_id = current_setting('app.current_user_id')::UUID
        )
    );
```

### Encriptação de Dados Sensíveis

```sql
-- Extensão para encriptação
CREATE EXTENSION IF NOT EXISTS pgcrypto;

-- Encriptar dados médicos sensíveis
CREATE TABLE sensitive_medical_data (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    child_id UUID REFERENCES children(id),
    encrypted_data BYTEA NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Funções para encriptar/desencriptar
CREATE FUNCTION encrypt_sensitive_data(data TEXT, key TEXT)
RETURNS BYTEA AS $$
    SELECT pgp_sym_encrypt(data, key);
$$ LANGUAGE SQL;

CREATE FUNCTION decrypt_sensitive_data(encrypted_data BYTEA, key TEXT)
RETURNS TEXT AS $$
    SELECT pgp_sym_decrypt(encrypted_data, key);
$$ LANGUAGE SQL;
```

---

## 📊 Consultas Comuns

### Atividades de uma Criança (Últimos 7 dias)

```sql
SELECT 
    'feeding' as activity_type,
    f.start_time as time,
    f.type,
    f.food_description as description
FROM feeding_activities f
WHERE f.child_id = $1 
    AND f.start_time > NOW() - INTERVAL '7 days'

UNION ALL

SELECT 
    'sleep' as activity_type,
    s.start_time as time,
    s.type,
    CONCAT(s.duration_minutes, ' minutos') as description
FROM sleep_activities s
WHERE s.child_id = $1 
    AND s.start_time > NOW() - INTERVAL '7 days'

ORDER BY time DESC;
```

### Estatísticas de Alimentação (Último Mês)

```sql
SELECT 
    DATE(start_time) as date,
    COUNT(*) as total_meals,
    AVG(CASE WHEN appetite = 'Excellent' THEN 5
             WHEN appetite = 'Good' THEN 4
             WHEN appetite = 'Fair' THEN 3
             WHEN appetite = 'Poor' THEN 2
             ELSE 1 END) as avg_appetite_score
FROM feeding_activities
WHERE child_id = $1
    AND start_time > NOW() - INTERVAL '30 days'
GROUP BY DATE(start_time)
ORDER BY date DESC;
```

### Crianças com Vacinas Atrasadas

```sql
SELECT 
    c.id,
    c.first_name,
    c.last_name,
    v.vaccine_name,
    v.scheduled_date,
    AGE(NOW(), v.scheduled_date) as days_overdue
FROM children c
JOIN vaccinations v ON c.id = v.child_id
WHERE v.status = 'Scheduled'
    AND v.scheduled_date < NOW()
    AND c.is_deleted = FALSE
ORDER BY v.scheduled_date ASC;
```

---

## 🔄 Backup e Restore

### Script de Backup

```bash
#!/bin/bash
# backup-tenant.sh

TENANT_CODE=$1
DB_NAME="dailycare_tenant_${TENANT_CODE}"
BACKUP_DIR="/backups/tenants/${TENANT_CODE}"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p $BACKUP_DIR

# Full backup
pg_dump -h localhost -U postgres -d $DB_NAME \
    -F c \
    -f "${BACKUP_DIR}/full_${DATE}.backup"

# Compress
gzip "${BACKUP_DIR}/full_${DATE}.backup"

# Cleanup old backups (keep last 30 days)
find $BACKUP_DIR -name "full_*.backup.gz" -mtime +30 -delete

echo "Backup concluído: ${BACKUP_DIR}/full_${DATE}.backup.gz"
```

### Script de Restore

```bash
#!/bin/bash
# restore-tenant.sh

TENANT_CODE=$1
BACKUP_FILE=$2
DB_NAME="dailycare_tenant_${TENANT_CODE}"

# Descomprimir se necessário
if [[ $BACKUP_FILE == *.gz ]]; then
    gunzip -c $BACKUP_FILE > /tmp/restore.backup
    BACKUP_FILE=/tmp/restore.backup
fi

# Restore
pg_restore -h localhost -U postgres \
    -d $DB_NAME \
    -c \
    $BACKUP_FILE

echo "Restore concluído para ${DB_NAME}"
```

---

## 📝 Manutenção

### Vacuum e Analyze

```sql
-- Vacuum regular para limpeza
VACUUM ANALYZE children;
VACUUM ANALYZE feeding_activities;
VACUUM ANALYZE sleep_activities;

-- Vacuum FULL para recuperar espaço (offline)
VACUUM FULL children;

-- Auto-vacuum configurado no postgresql.conf
autovacuum = on
autovacuum_max_workers = 3
autovacuum_naptime = 1min
```

### Monitorização de Tamanho

```sql
-- Tamanho das tabelas
SELECT 
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size
FROM pg_tables
WHERE schemaname NOT IN ('pg_catalog', 'information_schema')
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;

-- Índices não utilizados
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan as scans,
    pg_size_pretty(pg_relation_size(indexrelid)) as size
FROM pg_stat_user_indexes
WHERE idx_scan = 0
    AND indexrelname NOT LIKE 'pg_toast%'
ORDER BY pg_relation_size(indexrelid) DESC;
```

---

**Anterior**: [Backend Microserviços](./06-BACKEND-MICROSERVICOS.md)  
**Próximo**: [Multi-Tenancy](./08-MULTI-TENANCY.md)  
**Voltar ao**: [Índice](./00-INDICE.md)
