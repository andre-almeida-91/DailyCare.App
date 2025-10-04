# 👶 DailyCare - Gestão do Dia a Dia Infantil

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![.NET](https://img.shields.io/badge/.NET-9.0-purple.svg)
![React](https://img.shields.io/badge/React-18.3-blue.svg)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-blue.svg)

## 📖 Sobre o Projeto

O **DailyCare** é uma aplicação moderna e completa para gestão do dia a dia de crianças e bebés, desenvolvida para facilitar o acompanhamento do desenvolvimento infantil por pais, educadores e instituições de ensino.

### ✨ Funcionalidades Principais

- 📝 **Registo de Atividades Diárias**: Alimentação, sono, fraldas e exercícios
- 🎯 **Marcos de Desenvolvimento**: Acompanhamento de progressos importantes
- 🏥 **Gestão de Saúde**: Medicamentos, vacinas, consultas e alergias
- 📊 **Relatórios Visuais**: Gráficos e dashboards intuitivos
- 📸 **Galeria de Momentos**: Partilha de fotos e memórias
- 🔔 **Notificações em Tempo Real**: Atualizações instantâneas
- 👨‍👩‍👧‍👦 **Partilha Familiar**: Acesso controlado para familiares
- 🏫 **Multi-Tenant**: Suporte para múltiplas escolas com isolamento completo

---

## 🏗️ Arquitetura

O DailyCare utiliza uma arquitetura moderna de **microserviços** com as seguintes características:

- **Backend**: .NET 9 com microserviços independentes
- **Frontend**: React 18+ com TypeScript
- **Base de Dados**: PostgreSQL 16 (Database per Tenant)
- **Cache**: Redis 7
- **Mensageria**: RabbitMQ
- **Containerização**: Docker
- **Observabilidade**: OpenTelemetry, Elasticsearch, Prometheus, Grafana

```
├── Frontend (React + TypeScript)
│   ├── Parent App (Aplicação para Pais)
│   └── Admin App (Painel Administrativo)
├── API Gateway (YARP)
├── Microserviços (.NET 9)
│   ├── Identity Service
│   ├── School Service
│   ├── Children Service
│   ├── Activities Service
│   ├── Health Service
│   ├── Notification Service
│   ├── Reports Service
│   └── Files Service
└── Infraestrutura
    ├── PostgreSQL (Multi-tenant)
    ├── Redis (Cache)
    ├── RabbitMQ (Mensageria)
    └── Docker (Containerização)
```

---

## 🚀 Quick Start

### Pré-requisitos

- [.NET 9 SDK](https://dotnet.microsoft.com/download/dotnet/9.0)
- [Node.js 20+](https://nodejs.org/)
- [Docker Desktop](https://www.docker.com/products/docker-desktop)
- [Git](https://git-scm.com/)

### Instalação

1. **Clone o repositório**
```bash
git clone https://github.com/seu-usuario/DailyCare.App.git
cd DailyCare.App
```

2. **Inicie os serviços de infraestrutura**
```bash
docker-compose -f docker-compose.dev.yml up -d
```

3. **Backend - Identity Service** (exemplo)
```bash
cd src/backend/services/identity
dotnet restore
dotnet run
```

4. **Frontend - Parent App**
```bash
cd src/frontend/parent-app
npm install
npm run dev
```

5. **Aceda à aplicação**
- Frontend Pais: http://localhost:5173
- Frontend Admin: http://localhost:5174
- API Gateway: http://localhost:5000
- Swagger UI: http://localhost:5000/swagger

---

## 📚 Documentação Completa

A documentação completa do projeto está disponível na pasta `docs/`:

- **[00 - Índice](./docs/00-INDICE.md)** - Índice geral da documentação
- **[01 - Visão Geral](./docs/01-VISAO-GERAL.md)** - Visão geral do projeto e funcionalidades
- **[02 - Arquitetura](./docs/02-ARQUITETURA.md)** - Arquitetura detalhada do sistema
- **[03 - Stack Tecnológica](./docs/03-STACK-TECNOLOGICA.md)** - Tecnologias utilizadas
- **[04 - Frontend Pais](./docs/04-FRONTEND-PAIS.md)** - Aplicação frontend para pais
- **[05 - Frontend Admin](./docs/05-FRONTEND-ADMIN.md)** - Painel administrativo
- **[06 - Backend Microserviços](./docs/06-BACKEND-MICROSERVICOS.md)** - Arquitetura de microserviços
- **[07 - Base de Dados](./docs/07-BASE-DADOS.md)** - Estrutura de dados PostgreSQL
- **[08 - Multi-Tenancy](./docs/08-MULTI-TENANCY.md)** - Implementação multi-tenant
- **[09 - Segurança](./docs/09-SEGURANCA-AUTENTICACAO.md)** - Segurança e autenticação
- **[10 - APIs](./docs/10-API-DOCUMENTACAO.md)** - Documentação completa das APIs
- **[11 - Infraestrutura](./docs/11-INFRAESTRUTURA.md)** - DevOps e infraestrutura
- **[12 - Tarefas de Desenvolvimento](./docs/12-TAREFAS-DESENVOLVIMENTO.md)** - Lista de tarefas com prioridades
- **[13 - Guia de Desenvolvimento](./docs/13-GUIA-DESENVOLVIMENTO.md)** - Guia para desenvolvedores

### Documentos Adicionais

- **[INSTRUCOES.md](./INSTRUCOES.md)** - Instruções detalhadas para começar
- **[SUMARIO-EXECUTIVO.md](./SUMARIO-EXECUTIVO.md)** - Sumário executivo do projeto
- **[CHECKLIST.md](./CHECKLIST.md)** - Checklist de desenvolvimento e progresso

---

## 🛠️ Stack Tecnológica

### Frontend
- **React 18.3+** - Framework UI
- **TypeScript 5.3+** - Linguagem
- **Material-UI 6** - Componentes UI
- **Tailwind CSS 3.4+** - Estilização
- **Redux Toolkit** - Gestão de estado
- **React Router 6** - Roteamento
- **Vite 5** - Build tool
- **Axios** - HTTP client

### Backend
- **.NET 9** - Framework
- **ASP.NET Core** - Web framework
- **Entity Framework Core 9** - ORM
- **MediatR** - CQRS
- **FluentValidation** - Validação
- **Serilog** - Logging
- **OpenTelemetry** - Observabilidade
- **JWT** - Autenticação

### Infraestrutura
- **PostgreSQL 16** - Base de dados
- **Redis 7** - Cache
- **RabbitMQ 3.12** - Mensageria
- **Docker** - Containerização
- **Nginx** - Reverse proxy
- **Elasticsearch** - Logs
- **Kibana** - Visualização de logs
- **Prometheus** - Métricas
- **Grafana** - Dashboards
- **Jaeger** - Distributed tracing

---

## 🧪 Testes

### Backend
```bash
# Executar todos os testes
dotnet test

# Executar com cobertura
dotnet test /p:CollectCoverage=true /p:CoverageReportFormat=opencover
```

### Frontend
```bash
# Testes unitários
npm test

# Testes E2E
npm run test:e2e

# Cobertura
npm run test:coverage
```

---

## 🤝 Contribuir

Contribuições são bem-vindas! Por favor, siga estas etapas:

1. Fork o projeto
2. Crie uma branch para a sua feature (`git checkout -b feature/MinhaFeature`)
3. Commit as suas alterações (`git commit -m 'Adiciona MinhaFeature'`)
4. Push para a branch (`git push origin feature/MinhaFeature`)
5. Abra um Pull Request

### Convenções de Código

- **Idioma**: Português de Portugal para código, comentários e documentação
- **Commits**: Mensagens descritivas em português
- **Code Style**: Seguir os linters configurados (ESLint, Prettier, EditorConfig)
- **Testes**: Obrigatórios para novas funcionalidades

---

## 📋 Roadmap

### Fase 1 - MVP (Concluída ✅)
- [x] Configuração inicial do projeto
- [x] Infraestrutura base
- [x] Serviços core (Identity, School, Children, Activities)
- [x] Frontend básico para pais

### Fase 2 - Funcionalidades Core (Em Desenvolvimento 🚧)
- [ ] Health Service completo
- [ ] Notification Service com SignalR
- [ ] Relatórios e gráficos
- [ ] Painel administrativo

### Fase 3 - Funcionalidades Avançadas (Planeado 📋)
- [ ] Galeria de fotos
- [ ] Exportação de dados
- [ ] Aplicações móveis nativas
- [ ] Modo offline

### Fase 4 - Otimizações (Futuro 🔮)
- [ ] Performance improvements
- [ ] Machine learning para insights
- [ ] Integrações externas
- [ ] Suporte multi-idioma

---

## 📊 Status do Projeto

### Badges de Build e Qualidade
<!-- Adicionar badges do CI/CD quando configurado -->

### Métricas
- **Linhas de Código**: ~50,000+ (estimado)
- **Cobertura de Testes**: 80%+ (objetivo)
- **Microserviços**: 8 serviços independentes
- **Endpoints de API**: 100+ endpoints
- **Documentação**: 18 ficheiros (~315 KB)

---

## 📄 Licença

Este projeto está licenciado sob a Licença MIT - veja o ficheiro [LICENSE](LICENSE) para detalhes.

---

## 👥 Equipa

- **Arquiteto de Software**: [Seu Nome]
- **Desenvolvedores Backend**: [Equipa Backend]
- **Desenvolvedores Frontend**: [Equipa Frontend]
- **DevOps**: [Equipa DevOps]

---

## 📞 Contacto

Para questões ou suporte:

- **Email**: suporte@dailycare.pt
- **Website**: https://www.dailycare.pt
- **Issues**: https://github.com/seu-usuario/DailyCare.App/issues
- **Discussões**: https://github.com/seu-usuario/DailyCare.App/discussions

---

## 🙏 Agradecimentos

- Comunidade .NET
- Comunidade React
- Todos os contribuidores open-source
- Instituições de ensino parceiras

---

## 📸 Screenshots

<!-- Adicionar screenshots da aplicação quando disponíveis -->

---

## 🌟 Star o Projeto

Se este projeto foi útil para si, considere dar uma ⭐!

---

**Desenvolvido com ❤️ em Portugal**