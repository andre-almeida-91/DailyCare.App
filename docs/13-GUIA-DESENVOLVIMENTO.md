# Guia de Desenvolvimento - DailyCare

## 🎯 Objetivo

Este guia contém todas as informações necessárias para configurar o ambiente de desenvolvimento e começar a contribuir para o projeto DailyCare.

---

## 💻 Configuração do Ambiente

### Pré-requisitos

#### Ferramentas Obrigatórias

1. **.NET 9 SDK**
   ```bash
   # Verificar instalação
   dotnet --version
   # Deve retornar 9.0.x ou superior
   ```
   Download: https://dotnet.microsoft.com/download/dotnet/9.0

2. **Node.js 20+ e npm/pnpm**
   ```bash
   node --version  # v20.x.x ou superior
   npm --version   # 10.x.x ou superior
   ```
   Download: https://nodejs.org/

3. **Docker Desktop**
   ```bash
   docker --version
   docker-compose --version
   ```
   Download: https://www.docker.com/products/docker-desktop

4. **Git**
   ```bash
   git --version
   ```
   Download: https://git-scm.com/

#### Ferramentas Recomendadas

- **Visual Studio Code** com extensões:
  - C# Dev Kit
  - ESLint
  - Prettier
  - Tailwind CSS IntelliSense
  - GitLens
  - Docker
  - REST Client
  
- **Visual Studio 2022** (alternativa para backend)
- **PostgreSQL Client** (pgAdmin, DBeaver, ou CLI)
- **Redis Desktop Manager** ou RedisInsight
- **Postman** ou **Insomnia** (testes de API)

---

## 🚀 Configuração Inicial

### 1. Clonar o Repositório

```bash
git clone https://github.com/seu-usuario/DailyCare.App.git
cd DailyCare.App
```

### 2. Configurar Git Flow

```bash
# Instalar git-flow (se não tiver)
# Windows (via Chocolatey)
choco install gitflow-avh

# Mac (via Homebrew)
brew install git-flow-avh

# Inicializar git-flow
git flow init -d
```

### 3. Iniciar Infraestrutura

```bash
# Iniciar todos os serviços de infraestrutura
docker-compose -f docker-compose.dev.yml up -d

# Verificar se os serviços estão a correr
docker-compose -f docker-compose.dev.yml ps

# Ver logs
docker-compose -f docker-compose.dev.yml logs -f
```

### 4. Configurar Backend

#### 4.1 Restaurar Dependências

```bash
cd src/backend

# Restaurar todas as soluções
dotnet restore

# Ou restaurar serviço específico
cd services/identity
dotnet restore
```

#### 4.2 Configurar Connection Strings

Criar ficheiro `appsettings.Development.json` em cada microserviço:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Host=localhost;Database=dailycare_master;Username=postgres;Password=postgres"
  },
  "Redis": {
    "Configuration": "localhost:6379"
  },
  "RabbitMQ": {
    "Host": "localhost",
    "Username": "admin",
    "Password": "admin"
  },
  "Jwt": {
    "Secret": "your-super-secret-key-change-in-production-min-32-chars",
    "Issuer": "DailyCare",
    "Audience": "DailyCare.API",
    "ExpirationMinutes": 60
  }
}
```

#### 4.3 Executar Migrations

```bash
cd src/backend/services/identity
dotnet ef database update

# Repetir para cada microserviço
```

#### 4.4 Executar Serviços

```bash
# Terminal 1 - API Gateway
cd src/backend/gateway
dotnet run

# Terminal 2 - Identity Service
cd src/backend/services/identity
dotnet run

# Terminal 3 - School Service
cd src/backend/services/school
dotnet run

# etc...
```

### 5. Configurar Frontend

#### 5.1 Parent App

```bash
cd src/frontend/parent-app

# Instalar dependências (recomendado: pnpm)
pnpm install
# ou
npm install

# Copiar ficheiro de ambiente
cp .env.example .env.local

# Editar .env.local
# VITE_API_URL=http://localhost:5000/api
# VITE_SIGNALR_URL=http://localhost:5000

# Executar em modo desenvolvimento
pnpm dev
# ou
npm run dev
```

#### 5.2 Admin App

```bash
cd src/frontend/admin-app

pnpm install
cp .env.example .env.local

# Editar .env.local conforme necessário

pnpm dev
```

---

## 📁 Estrutura do Projeto

### Backend

```
src/backend/
├── gateway/                    # API Gateway
│   └── DailyCare.Gateway/
├── services/                   # Microserviços
│   ├── identity/
│   │   ├── DailyCare.Identity.API/
│   │   ├── DailyCare.Identity.Application/
│   │   ├── DailyCare.Identity.Domain/
│   │   └── DailyCare.Identity.Infrastructure/
│   ├── school/
│   ├── children/
│   ├── activities/
│   ├── health/
│   ├── notification/
│   ├── reports/
│   └── files/
└── shared/                     # Bibliotecas partilhadas
    ├── DailyCare.Shared.Domain/
    ├── DailyCare.Shared.Infrastructure/
    └── DailyCare.Shared.Common/
```

### Frontend

```
src/frontend/
├── parent-app/                 # App para pais
│   ├── public/
│   ├── src/
│   │   ├── assets/
│   │   ├── components/
│   │   ├── features/
│   │   ├── hooks/
│   │   ├── layouts/
│   │   ├── pages/
│   │   ├── routes/
│   │   ├── services/
│   │   ├── store/
│   │   ├── styles/
│   │   ├── types/
│   │   └── utils/
│   ├── package.json
│   └── vite.config.ts
└── admin-app/                  # App administrativa
    └── (estrutura similar)
```

---

## 🔨 Convenções de Desenvolvimento

### Git Workflow

#### Branches

- `main` - Código em produção
- `develop` - Desenvolvimento ativo
- `feature/*` - Novas funcionalidades
- `bugfix/*` - Correções de bugs
- `hotfix/*` - Correções urgentes de produção
- `release/*` - Preparação de releases

#### Fluxo de Trabalho

```bash
# 1. Criar feature branch a partir de develop
git checkout develop
git pull origin develop
git checkout -b feature/TASK-001-criar-login

# 2. Desenvolver e commitar
git add .
git commit -m "feat: implementa autenticação JWT no Identity Service"

# 3. Push para o remoto
git push origin feature/TASK-001-criar-login

# 4. Criar Pull Request no GitHub/GitLab
# 5. Após aprovação, merge para develop
```

### Mensagens de Commit

Seguir [Conventional Commits](https://www.conventionalcommits.org/):

```
<tipo>(<escopo>): <descrição>

[corpo opcional]

[rodapé opcional]
```

**Tipos**:
- `feat`: Nova funcionalidade
- `fix`: Correção de bug
- `docs`: Documentação
- `style`: Formatação
- `refactor`: Refatoração
- `test`: Testes
- `chore`: Tarefas de manutenção

**Exemplos**:
```bash
feat(identity): adiciona refresh token
fix(activities): corrige validação de data
docs(api): atualiza documentação de endpoints
test(children): adiciona testes unitários para ChildService
refactor(health): melhora estrutura do serviço
```

---

## 🎨 Convenções de Código

### Backend (.NET)

#### Naming Conventions

```csharp
// Classes e Métodos: PascalCase
public class UserService { }
public void CreateUser() { }

// Variáveis locais e parâmetros: camelCase
var userName = "João";
public void UpdateUser(string userId) { }

// Constantes: PascalCase
public const int MaxRetries = 3;

// Interfaces: Prefixo I
public interface IUserRepository { }

// Async methods: sufixo Async
public async Task<User> GetUserAsync(Guid id) { }
```

#### Organização de Classes

```csharp
public class UserService : IUserService
{
    // 1. Campos privados
    private readonly IUserRepository _userRepository;
    private readonly ILogger<UserService> _logger;

    // 2. Construtor
    public UserService(
        IUserRepository userRepository,
        ILogger<UserService> logger)
    {
        _userRepository = userRepository;
        _logger = logger;
    }

    // 3. Métodos públicos
    public async Task<User> GetUserAsync(Guid id)
    {
        // Implementação
    }

    // 4. Métodos privados
    private void ValidateUser(User user)
    {
        // Implementação
    }
}
```

#### Comentários em Português

```csharp
/// <summary>
/// Cria um novo utilizador no sistema
/// </summary>
/// <param name="request">Dados do utilizador</param>
/// <returns>Utilizador criado</returns>
public async Task<User> CreateUserAsync(CreateUserRequest request)
{
    // Valida os dados de entrada
    ValidateRequest(request);
    
    // Cria o utilizador
    var user = User.Create(request.Name, request.Email);
    
    // Persiste na base de dados
    await _userRepository.AddAsync(user);
    
    return user;
}
```

### Frontend (React/TypeScript)

#### Naming Conventions

```typescript
// Componentes: PascalCase
function UserCard() { }

// Hooks: prefixo use + PascalCase
function useAuth() { }

// Variáveis e funções: camelCase
const userName = 'João';
function handleClick() { }

// Tipos e Interfaces: PascalCase
interface User { }
type UserRole = 'Admin' | 'Parent';

// Constantes: UPPER_SNAKE_CASE
const API_BASE_URL = 'http://localhost:5000';
```

#### Estrutura de Componentes

```typescript
// Imports
import { FC, useState, useEffect } from 'react';
import { Box, Typography } from '@mui/material';
import { useAuth } from '@/hooks/useAuth';

// Types
interface UserCardProps {
  userId: string;
  onSelect?: (user: User) => void;
}

// Component
export const UserCard: FC<UserCardProps> = ({ userId, onSelect }) => {
  // Hooks
  const { user } = useAuth();
  const [loading, setLoading] = useState(false);

  // Effects
  useEffect(() => {
    // Lógica do effect
  }, [userId]);

  // Handlers
  const handleClick = () => {
    onSelect?.(user);
  };

  // Render
  return (
    <Box onClick={handleClick}>
      <Typography>{user.name}</Typography>
    </Box>
  );
};
```

#### Comentários em Português

```typescript
/**
 * Hook para gestão de autenticação
 * 
 * @returns {Object} Estado e métodos de autenticação
 */
export const useAuth = () => {
  // Estado do utilizador autenticado
  const [user, setUser] = useState<User | null>(null);

  // Função para fazer login
  const login = async (credentials: LoginRequest) => {
    // Implementação
  };

  return { user, login, logout };
};
```

---

## 🧪 Testes

### Backend

#### Testes Unitários

```csharp
public class UserServiceTests
{
    private readonly Mock<IUserRepository> _userRepositoryMock;
    private readonly UserService _userService;

    public UserServiceTests()
    {
        _userRepositoryMock = new Mock<IUserRepository>();
        _userService = new UserService(_userRepositoryMock.Object);
    }

    [Fact]
    public async Task CreateUser_ComDadosValidos_DeveCriarUtilizador()
    {
        // Arrange
        var request = new CreateUserRequest
        {
            Name = "João Silva",
            Email = "joao@example.com"
        };

        _userRepositoryMock
            .Setup(x => x.AddAsync(It.IsAny<User>()))
            .ReturnsAsync((User u) => u);

        // Act
        var result = await _userService.CreateUserAsync(request);

        // Assert
        Assert.NotNull(result);
        Assert.Equal(request.Name, result.Name);
        _userRepositoryMock.Verify(x => x.AddAsync(It.IsAny<User>()), Times.Once);
    }
}
```

#### Executar Testes

```bash
# Todos os testes
dotnet test

# Teste específico
dotnet test --filter "FullyQualifiedName~UserServiceTests"

# Com cobertura
dotnet test /p:CollectCoverage=true /p:CoverageReportFormat=opencover

# Watch mode
dotnet watch test
```

### Frontend

#### Testes de Componentes

```typescript
import { render, screen, fireEvent } from '@testing-library/react';
import { UserCard } from './UserCard';

describe('UserCard', () => {
  it('deve renderizar o nome do utilizador', () => {
    const user = { id: '1', name: 'João Silva' };
    
    render(<UserCard user={user} />);
    
    expect(screen.getByText('João Silva')).toBeInTheDocument();
  });

  it('deve chamar onSelect quando clicado', () => {
    const handleSelect = jest.fn();
    const user = { id: '1', name: 'João Silva' };
    
    render(<UserCard user={user} onSelect={handleSelect} />);
    
    fireEvent.click(screen.getByText('João Silva'));
    
    expect(handleSelect).toHaveBeenCalledWith(user);
  });
});
```

#### Executar Testes

```bash
# Todos os testes
npm test

# Watch mode
npm test -- --watch

# Cobertura
npm run test:coverage

# E2E
npm run test:e2e
```

---

## 🔍 Debugging

### Backend (Visual Studio Code)

Criar `.vscode/launch.json`:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": ".NET Core Launch (Identity Service)",
      "type": "coreclr",
      "request": "launch",
      "preLaunchTask": "build",
      "program": "${workspaceFolder}/src/backend/services/identity/DailyCare.Identity.API/bin/Debug/net9.0/DailyCare.Identity.API.dll",
      "args": [],
      "cwd": "${workspaceFolder}/src/backend/services/identity/DailyCare.Identity.API",
      "stopAtEntry": false,
      "serverReadyAction": {
        "action": "openExternally",
        "pattern": "\\bNow listening on:\\s+(https?://\\S+)"
      },
      "env": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    }
  ]
}
```

### Frontend (Chrome DevTools)

1. Abrir Chrome DevTools (F12)
2. Ir para Sources
3. Adicionar breakpoints no código TypeScript
4. React Developer Tools (extensão recomendada)

---

## 📊 Análise de Código

### Backend - SonarQube (Local)

```bash
# Instalar ferramenta global
dotnet tool install --global dotnet-sonarscanner

# Iniciar análise
dotnet sonarscanner begin /k:"DailyCare" /d:sonar.host.url="http://localhost:9000"

# Build
dotnet build

# Finalizar análise
dotnet sonarscanner end
```

### Frontend - ESLint

```bash
# Verificar código
npm run lint

# Fix automático
npm run lint:fix
```

---

## 📦 Build e Deploy

### Backend

```bash
# Build Release
dotnet build -c Release

# Publicar
dotnet publish -c Release -o ./publish

# Criar imagem Docker
docker build -t dailycare/identity-service:latest .

# Push para registry
docker push dailycare/identity-service:latest
```

### Frontend

```bash
# Build production
npm run build

# Preview build
npm run preview

# Análise de bundle
npm run build -- --analyze
```

---

## 🐛 Troubleshooting

### Problemas Comuns

#### 1. Erro de Conexão ao PostgreSQL

```bash
# Verificar se o container está a correr
docker ps | grep postgres

# Ver logs
docker logs dailycare-postgres

# Reiniciar
docker-compose restart postgres
```

#### 2. Porta já em uso

```bash
# Windows - Encontrar processo
netstat -ano | findstr :5000

# Matar processo
taskkill /PID <PID> /F

# Linux/Mac
lsof -ti:5000 | xargs kill -9
```

#### 3. Migrations falhadas

```bash
# Limpar migrations
dotnet ef database drop
dotnet ef migrations remove

# Recriar
dotnet ef migrations add Initial
dotnet ef database update
```

---

## 📚 Recursos Úteis

### Documentação Oficial
- [.NET Documentation](https://docs.microsoft.com/dotnet/)
- [React Documentation](https://react.dev/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)

### Tutoriais e Guias
- [Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [DDD Pattern](https://martinfowler.com/bliki/DomainDrivenDesign.html)
- [React Best Practices](https://react.dev/learn/thinking-in-react)

### Comunidade
- [Stack Overflow](https://stackoverflow.com/)
- [.NET Community](https://dotnet.microsoft.com/platform/community)
- [Reactiflux Discord](https://www.reactiflux.com/)

---

## ✅ Checklist do Desenvolvedor

Antes de submeter um Pull Request:

- [ ] Código segue as convenções estabelecidas
- [ ] Comentários em português de Portugal
- [ ] Testes unitários escritos e a passar
- [ ] Sem warnings do linter
- [ ] Build de produção bem-sucedido
- [ ] Documentação atualizada
- [ ] Changelog atualizado (se aplicável)
- [ ] Branch atualizada com develop
- [ ] Descrição clara no Pull Request

---

## 🎓 Onboarding de Novos Desenvolvedores

### Dia 1
1. Configurar ambiente de desenvolvimento
2. Clonar repositório
3. Executar aplicação localmente
4. Explorar a estrutura do código

### Dia 2-3
1. Ler documentação completa
2. Revisar código existente
3. Fazer pair programming com a equipa
4. Corrigir um bug simples

### Semana 1
1. Implementar uma feature pequena
2. Escrever testes
3. Submeter primeiro Pull Request
4. Participar em code review

---

**Anterior**: [Tarefas de Desenvolvimento](./12-TAREFAS-DESENVOLVIMENTO.md)  
**Voltar ao**: [Índice](./00-INDICE.md)
