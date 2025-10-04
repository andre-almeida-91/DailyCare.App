# Documentação das APIs - DailyCare

## 📖 Visão Geral

Este documento contém a especificação completa de todas as APIs do projeto DailyCare, organizada por microserviço.

Todas as APIs seguem o padrão RESTful e utilizam JSON como formato de dados.

---

## 🌐 URL Base

### Desenvolvimento
```
http://localhost:5000/api
```

### Produção
```
https://api.dailycare.pt/api
```

---

## 🔐 Autenticação

Todas as APIs (exceto login e registo) requerem autenticação via JWT Bearer Token.

### Header de Autenticação
```http
Authorization: Bearer {access_token}
```

### Header de Tenant (para APIs multi-tenant)
```http
X-Tenant-Id: {school_id}
```

---

## 📚 Serviços

### 1. Identity Service
### 2. School Service
### 3. Children Service
### 4. Activities Service
### 5. Health Service
### 6. Notification Service
### 7. Reports Service
### 8. Files Service

---

## 1️⃣ Identity Service

**Base Path**: `/api/identity`

### 1.1 Autenticação

#### POST /auth/register
Regista um novo utilizador na plataforma.

**Request Body**:
```json
{
  "name": "João Silva",
  "email": "joao.silva@example.com",
  "password": "SenhaSegura123!",
  "confirmPassword": "SenhaSegura123!",
  "phoneNumber": "+351912345678",
  "role": "Parent"
}
```

**Response** (201 Created):
```json
{
  "success": true,
  "message": "Utilizador registado com sucesso. Verifique o seu email.",
  "data": {
    "userId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "email": "joao.silva@example.com"
  }
}
```

**Erros**:
- `400 Bad Request`: Dados inválidos
- `409 Conflict`: Email já registado

---

#### POST /auth/login
Autentica um utilizador e retorna tokens de acesso.

**Request Body**:
```json
{
  "email": "joao.silva@example.com",
  "password": "SenhaSegura123!"
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refreshToken": "f3d2e1c0b9a8...",
    "expiresIn": 3600,
    "tokenType": "Bearer",
    "user": {
      "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
      "name": "João Silva",
      "email": "joao.silva@example.com",
      "role": "Parent",
      "emailConfirmed": true,
      "photoUrl": "https://storage.dailycare.pt/users/avatar.jpg"
    }
  }
}
```

**Erros**:
- `400 Bad Request`: Dados inválidos
- `401 Unauthorized`: Credenciais incorretas
- `403 Forbidden`: Conta não verificada ou bloqueada

---

#### POST /auth/refresh
Renova o access token usando o refresh token.

**Request Body**:
```json
{
  "refreshToken": "f3d2e1c0b9a8..."
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refreshToken": "a8b9c0d1e2f3...",
    "expiresIn": 3600,
    "tokenType": "Bearer"
  }
}
```

**Erros**:
- `401 Unauthorized`: Refresh token inválido ou expirado

---

#### POST /auth/logout
Invalida o refresh token do utilizador.

**Headers**: `Authorization: Bearer {token}`

**Response** (200 OK):
```json
{
  "success": true,
  "message": "Logout realizado com sucesso"
}
```

---

#### POST /auth/forgot-password
Solicita recuperação de password.

**Request Body**:
```json
{
  "email": "joao.silva@example.com"
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "message": "Instruções enviadas para o seu email"
}
```

---

#### POST /auth/reset-password
Reset de password usando token recebido por email.

**Request Body**:
```json
{
  "token": "abc123def456",
  "newPassword": "NovaSenhaSegura123!",
  "confirmPassword": "NovaSenhaSegura123!"
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "message": "Password alterada com sucesso"
}
```

---

### 1.2 Perfil do Utilizador

#### GET /me
Obtém o perfil do utilizador autenticado.

**Headers**: `Authorization: Bearer {token}`

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "name": "João Silva",
    "email": "joao.silva@example.com",
    "phoneNumber": "+351912345678",
    "photoUrl": "https://storage.dailycare.pt/users/avatar.jpg",
    "role": "Parent",
    "emailConfirmed": true,
    "phoneConfirmed": false,
    "createdAt": "2024-01-01T10:00:00Z",
    "lastLoginAt": "2024-01-15T14:30:00Z"
  }
}
```

---

#### PUT /me
Atualiza o perfil do utilizador.

**Headers**: `Authorization: Bearer {token}`

**Request Body**:
```json
{
  "name": "João Pedro Silva",
  "phoneNumber": "+351913456789"
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "message": "Perfil atualizado com sucesso",
  "data": {
    "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "name": "João Pedro Silva",
    "email": "joao.silva@example.com",
    "phoneNumber": "+351913456789",
    "photoUrl": "https://storage.dailycare.pt/users/avatar.jpg"
  }
}
```

---

#### POST /me/change-password
Altera a password do utilizador.

**Headers**: `Authorization: Bearer {token}`

**Request Body**:
```json
{
  "currentPassword": "SenhaAtual123!",
  "newPassword": "NovaSenha123!",
  "confirmPassword": "NovaSenha123!"
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "message": "Password alterada com sucesso"
}
```

---

## 2️⃣ School Service

**Base Path**: `/api/schools`

**Headers obrigatórios**: 
- `Authorization: Bearer {token}`
- `X-Tenant-Id: {school_id}` (exceto em endpoints de criação/listagem de escolas)

### 2.1 Gestão de Escolas

#### POST /
Cria uma nova escola (tenant).

**Permissão**: SuperAdmin

**Request Body**:
```json
{
  "name": "Jardim de Infância Estrelinhas",
  "code": "JIE001",
  "email": "geral@estrelinhas.pt",
  "phoneNumber": "+351210123456",
  "address": {
    "street": "Rua das Flores",
    "number": "123",
    "postalCode": "1000-001",
    "city": "Lisboa",
    "country": "Portugal"
  },
  "adminUser": {
    "name": "Maria Santos",
    "email": "maria.santos@estrelinhas.pt",
    "phoneNumber": "+351912345678"
  },
  "settings": {
    "timezone": "Europe/Lisbon",
    "language": "pt-PT",
    "currency": "EUR"
  }
}
```

**Response** (201 Created):
```json
{
  "success": true,
  "message": "Escola criada com sucesso",
  "data": {
    "id": "7c9e6679-7425-40de-944b-e07fc1f90ae7",
    "name": "Jardim de Infância Estrelinhas",
    "code": "JIE001",
    "email": "geral@estrelinhas.pt",
    "phoneNumber": "+351210123456",
    "isActive": true,
    "createdAt": "2024-01-15T10:00:00Z",
    "databaseProvisioned": true
  }
}
```

---

#### GET /
Lista todas as escolas.

**Permissão**: SuperAdmin

**Query Parameters**:
- `page` (int, default: 1)
- `pageSize` (int, default: 20, max: 100)
- `search` (string, opcional)
- `isActive` (boolean, opcional)

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "7c9e6679-7425-40de-944b-e07fc1f90ae7",
        "name": "Jardim de Infância Estrelinhas",
        "code": "JIE001",
        "email": "geral@estrelinhas.pt",
        "phoneNumber": "+351210123456",
        "isActive": true,
        "totalClasses": 5,
        "totalChildren": 45,
        "totalEducators": 8,
        "createdAt": "2024-01-15T10:00:00Z"
      }
    ],
    "pagination": {
      "currentPage": 1,
      "pageSize": 20,
      "totalPages": 1,
      "totalItems": 1
    }
  }
}
```

---

#### GET /{id}
Obtém detalhes de uma escola específica.

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "id": "7c9e6679-7425-40de-944b-e07fc1f90ae7",
    "name": "Jardim de Infância Estrelinhas",
    "code": "JIE001",
    "email": "geral@estrelinhas.pt",
    "phoneNumber": "+351210123456",
    "address": {
      "street": "Rua das Flores",
      "number": "123",
      "postalCode": "1000-001",
      "city": "Lisboa",
      "country": "Portugal"
    },
    "isActive": true,
    "settings": {
      "timezone": "Europe/Lisbon",
      "language": "pt-PT",
      "currency": "EUR"
    },
    "statistics": {
      "totalClasses": 5,
      "totalChildren": 45,
      "totalEducators": 8,
      "totalParents": 67
    },
    "createdAt": "2024-01-15T10:00:00Z"
  }
}
```

---

### 2.2 Gestão de Turmas

#### POST /{schoolId}/classes
Cria uma nova turma.

**Request Body**:
```json
{
  "name": "Sala dos Patinhos",
  "description": "Turma para crianças dos 3 aos 4 anos",
  "ageRange": {
    "minAge": 3,
    "maxAge": 4
  },
  "capacity": 15,
  "academicYear": "2024/2025"
}
```

**Response** (201 Created):
```json
{
  "success": true,
  "message": "Turma criada com sucesso",
  "data": {
    "id": "8b7a5432-1234-5678-9abc-def012345678",
    "schoolId": "7c9e6679-7425-40de-944b-e07fc1f90ae7",
    "name": "Sala dos Patinhos",
    "description": "Turma para crianças dos 3 aos 4 anos",
    "capacity": 15,
    "currentEnrollment": 0,
    "academicYear": "2024/2025",
    "createdAt": "2024-01-15T11:00:00Z"
  }
}
```

---

#### GET /{schoolId}/classes
Lista todas as turmas de uma escola.

**Query Parameters**:
- `page` (int)
- `pageSize` (int)
- `academicYear` (string, opcional)

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "8b7a5432-1234-5678-9abc-def012345678",
        "name": "Sala dos Patinhos",
        "capacity": 15,
        "currentEnrollment": 12,
        "academicYear": "2024/2025",
        "educators": [
          {
            "id": "1a2b3c4d-5e6f-7g8h-9i0j-k1l2m3n4o5p6",
            "name": "Ana Costa",
            "role": "Lead Educator"
          }
        ]
      }
    ],
    "pagination": {
      "currentPage": 1,
      "pageSize": 20,
      "totalPages": 1,
      "totalItems": 1
    }
  }
}
```

---

## 3️⃣ Children Service

**Base Path**: `/api/children`

**Headers obrigatórios**: 
- `Authorization: Bearer {token}`
- `X-Tenant-Id: {school_id}`

### 3.1 Gestão de Crianças

#### POST /
Cria um novo perfil de criança.

**Request Body**:
```json
{
  "firstName": "Miguel",
  "lastName": "Rodrigues",
  "birthDate": "2020-05-15",
  "gender": "Male",
  "bloodType": "A+",
  "classId": "8b7a5432-1234-5678-9abc-def012345678",
  "emergencyContact": {
    "name": "Ana Rodrigues",
    "phoneNumber": "+351912345678",
    "relationship": "Mother"
  },
  "medicalInfo": {
    "allergies": ["Amendoim", "Leite"],
    "medications": [],
    "specialNeeds": "Nenhuma"
  }
}
```

**Response** (201 Created):
```json
{
  "success": true,
  "message": "Criança registada com sucesso",
  "data": {
    "id": "9c8d7e6f-5a4b-3c2d-1e0f-1a2b3c4d5e6f",
    "firstName": "Miguel",
    "lastName": "Rodrigues",
    "fullName": "Miguel Rodrigues",
    "birthDate": "2020-05-15",
    "age": 3,
    "gender": "Male",
    "bloodType": "A+",
    "photoUrl": null,
    "classId": "8b7a5432-1234-5678-9abc-def012345678",
    "className": "Sala dos Patinhos",
    "createdAt": "2024-01-15T12:00:00Z"
  }
}
```

---

#### GET /
Lista todas as crianças.

**Query Parameters**:
- `page` (int)
- `pageSize` (int)
- `classId` (guid, opcional)
- `search` (string, opcional)

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "9c8d7e6f-5a4b-3c2d-1e0f-1a2b3c4d5e6f",
        "firstName": "Miguel",
        "lastName": "Rodrigues",
        "fullName": "Miguel Rodrigues",
        "birthDate": "2020-05-15",
        "age": 3,
        "gender": "Male",
        "photoUrl": "https://storage.dailycare.pt/children/miguel.jpg",
        "className": "Sala dos Patinhos",
        "parents": [
          {
            "id": "1a2b3c4d-5e6f-7g8h-9i0j-k1l2m3n4o5p6",
            "name": "Ana Rodrigues",
            "relationship": "Mother"
          }
        ]
      }
    ],
    "pagination": {
      "currentPage": 1,
      "pageSize": 20,
      "totalPages": 1,
      "totalItems": 1
    }
  }
}
```

---

#### GET /{id}
Obtém detalhes completos de uma criança.

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "id": "9c8d7e6f-5a4b-3c2d-1e0f-1a2b3c4d5e6f",
    "firstName": "Miguel",
    "lastName": "Rodrigues",
    "fullName": "Miguel Rodrigues",
    "birthDate": "2020-05-15",
    "age": 3,
    "gender": "Male",
    "bloodType": "A+",
    "photoUrl": "https://storage.dailycare.pt/children/miguel.jpg",
    "class": {
      "id": "8b7a5432-1234-5678-9abc-def012345678",
      "name": "Sala dos Patinhos"
    },
    "emergencyContact": {
      "name": "Ana Rodrigues",
      "phoneNumber": "+351912345678",
      "relationship": "Mother"
    },
    "medicalInfo": {
      "allergies": ["Amendoim", "Leite"],
      "medications": [],
      "specialNeeds": "Nenhuma"
    },
    "familyMembers": [
      {
        "id": "1a2b3c4d-5e6f-7g8h-9i0j-k1l2m3n4o5p6",
        "name": "Ana Rodrigues",
        "email": "ana.rodrigues@example.com",
        "phoneNumber": "+351912345678",
        "relationship": "Mother",
        "isPrimaryContact": true,
        "canPickup": true
      },
      {
        "id": "2b3c4d5e-6f7g-8h9i-0j1k-l2m3n4o5p6q7",
        "name": "Pedro Rodrigues",
        "email": "pedro.rodrigues@example.com",
        "phoneNumber": "+351913456789",
        "relationship": "Father",
        "isPrimaryContact": false,
        "canPickup": true
      }
    ],
    "enrollmentHistory": [
      {
        "classId": "8b7a5432-1234-5678-9abc-def012345678",
        "className": "Sala dos Patinhos",
        "startDate": "2023-09-01",
        "endDate": null,
        "isActive": true
      }
    ],
    "createdAt": "2024-01-15T12:00:00Z",
    "updatedAt": "2024-01-15T12:00:00Z"
  }
}
```

---

#### PUT /{id}
Atualiza informações de uma criança.

**Request Body**:
```json
{
  "firstName": "Miguel",
  "lastName": "Rodrigues Silva",
  "bloodType": "A+",
  "classId": "8b7a5432-1234-5678-9abc-def012345678",
  "emergencyContact": {
    "name": "Ana Rodrigues",
    "phoneNumber": "+351912345678",
    "relationship": "Mother"
  },
  "medicalInfo": {
    "allergies": ["Amendoim"],
    "medications": [],
    "specialNeeds": "Nenhuma"
  }
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "message": "Informações atualizadas com sucesso",
  "data": {
    "id": "9c8d7e6f-5a4b-3c2d-1e0f-1a2b3c4d5e6f",
    "firstName": "Miguel",
    "lastName": "Rodrigues Silva",
    "fullName": "Miguel Rodrigues Silva",
    "updatedAt": "2024-01-16T10:30:00Z"
  }
}
```

---

#### POST /{id}/photo
Upload de foto de perfil da criança.

**Request**: `multipart/form-data`
- `file`: Ficheiro de imagem (JPEG, PNG)

**Response** (200 OK):
```json
{
  "success": true,
  "message": "Foto atualizada com sucesso",
  "data": {
    "photoUrl": "https://storage.dailycare.pt/children/miguel-updated.jpg"
  }
}
```

---

## 4️⃣ Activities Service

**Base Path**: `/api/activities`

**Headers obrigatórios**: 
- `Authorization: Bearer {token}`
- `X-Tenant-Id: {school_id}`

### 4.1 Alimentação

#### POST /feeding
Regista uma atividade de alimentação.

**Request Body**:
```json
{
  "childId": "9c8d7e6f-5a4b-3c2d-1e0f-1a2b3c4d5e6f",
  "startTime": "2024-01-15T12:00:00Z",
  "endTime": "2024-01-15T12:30:00Z",
  "type": "Lunch",
  "foodDescription": "Sopa de legumes, arroz com frango, fruta",
  "quantity": 1,
  "unit": "Portion",
  "appetite": "Good",
  "notes": "Comeu tudo com bom apetite"
}
```

**Response** (201 Created):
```json
{
  "success": true,
  "message": "Alimentação registada com sucesso",
  "data": {
    "id": "a1b2c3d4-e5f6-7g8h-9i0j-k1l2m3n4o5p6",
    "childId": "9c8d7e6f-5a4b-3c2d-1e0f-1a2b3c4d5e6f",
    "childName": "Miguel Rodrigues",
    "startTime": "2024-01-15T12:00:00Z",
    "endTime": "2024-01-15T12:30:00Z",
    "duration": "00:30:00",
    "type": "Lunch",
    "foodDescription": "Sopa de legumes, arroz com frango, fruta",
    "quantity": 1,
    "unit": "Portion",
    "appetite": "Good",
    "notes": "Comeu tudo com bom apetite",
    "registeredBy": "Ana Costa",
    "registeredAt": "2024-01-15T12:35:00Z"
  }
}
```

**Tipos de Alimentação**:
- `Breakfast` - Pequenoalmoço
- `MorningSnack` - Lanche da manhã
- `Lunch` - Almoço
- `AfternoonSnack` - Lanche da tarde
- `Dinner` - Jantar
- `Breastfeeding` - Amamentação
- `BottleFeeding` - Biberão

**Apetite**:
- `Excellent` - Excelente
- `Good` - Bom
- `Fair` - Razoável
- `Poor` - Fraco
- `None` - Nenhum

---

#### GET /feeding
Lista atividades de alimentação.

**Query Parameters**:
- `childId` (guid, obrigatório)
- `startDate` (date, opcional)
- `endDate` (date, opcional)
- `type` (enum, opcional)
- `page` (int)
- `pageSize` (int)

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "a1b2c3d4-e5f6-7g8h-9i0j-k1l2m3n4o5p6",
        "childId": "9c8d7e6f-5a4b-3c2d-1e0f-1a2b3c4d5e6f",
        "childName": "Miguel Rodrigues",
        "startTime": "2024-01-15T12:00:00Z",
        "endTime": "2024-01-15T12:30:00Z",
        "type": "Lunch",
        "foodDescription": "Sopa de legumes, arroz com frango, fruta",
        "appetite": "Good",
        "registeredBy": "Ana Costa",
        "registeredAt": "2024-01-15T12:35:00Z"
      }
    ],
    "pagination": {
      "currentPage": 1,
      "pageSize": 20,
      "totalPages": 1,
      "totalItems": 3
    }
  }
}
```

---

### 4.2 Sono

#### POST /sleep
Regista uma atividade de sono.

**Request Body**:
```json
{
  "childId": "9c8d7e6f-5a4b-3c2d-1e0f-1a2b3c4d5e6f",
  "startTime": "2024-01-15T14:00:00Z",
  "endTime": "2024-01-15T16:00:00Z",
  "type": "Nap",
  "quality": "Good",
  "notes": "Dormiu tranquilamente"
}
```

**Response** (201 Created):
```json
{
  "success": true,
  "message": "Sono registado com sucesso",
  "data": {
    "id": "b2c3d4e5-f6g7-8h9i-0j1k-l2m3n4o5p6q7",
    "childId": "9c8d7e6f-5a4b-3c2d-1e0f-1a2b3c4d5e6f",
    "childName": "Miguel Rodrigues",
    "startTime": "2024-01-15T14:00:00Z",
    "endTime": "2024-01-15T16:00:00Z",
    "duration": "02:00:00",
    "type": "Nap",
    "quality": "Good",
    "notes": "Dormiu tranquilamente",
    "registeredBy": "Ana Costa",
    "registeredAt": "2024-01-15T16:05:00Z"
  }
}
```

**Tipos de Sono**:
- `Nap` - Soneca
- `Night` - Noite

**Qualidade**:
- `Excellent` - Excelente
- `Good` - Boa
- `Fair` - Razoável
- `Poor` - Fraca
- `Restless` - Agitado

---

#### GET /sleep
Lista atividades de sono.

**Query Parameters**: (similares aos de feeding)

---

### 4.3 Fraldas

#### POST /diaper
Regista uma troca de fralda.

**Request Body**:
```json
{
  "childId": "9c8d7e6f-5a4b-3c2d-1e0f-1a2b3c4d5e6f",
  "time": "2024-01-15T10:30:00Z",
  "type": "Both",
  "condition": "Normal",
  "notes": ""
}
```

**Response** (201 Created):
```json
{
  "success": true,
  "message": "Fralda registada com sucesso",
  "data": {
    "id": "c3d4e5f6-g7h8-9i0j-1k2l-m3n4o5p6q7r8",
    "childId": "9c8d7e6f-5a4b-3c2d-1e0f-1a2b3c4d5e6f",
    "childName": "Miguel Rodrigues",
    "time": "2024-01-15T10:30:00Z",
    "type": "Both",
    "condition": "Normal",
    "notes": "",
    "registeredBy": "Ana Costa",
    "registeredAt": "2024-01-15T10:32:00Z"
  }
}
```

**Tipos**:
- `Wet` - Xixi
- `Dirty` - Cocó
- `Both` - Ambos

**Condição**:
- `Normal` - Normal
- `Dry` - Seca
- `Loose` - Solto
- `Hard` - Duro
- `Unusual` - Anormal

---

### 4.4 Marcos de Desenvolvimento

#### POST /milestones
Regista um marco de desenvolvimento.

**Request Body**:
```json
{
  "childId": "9c8d7e6f-5a4b-3c2d-1e0f-1a2b3c4d5e6f",
  "title": "Primeiras palavras",
  "description": "Disse 'mamã' pela primeira vez!",
  "category": "Language",
  "achievedAt": "2024-01-15T15:30:00Z",
  "notes": "Momento emocionante!"
}
```

**Response** (201 Created):
```json
{
  "success": true,
  "message": "Marco registado com sucesso",
  "data": {
    "id": "d4e5f6g7-h8i9-0j1k-2l3m-n4o5p6q7r8s9",
    "childId": "9c8d7e6f-5a4b-3c2d-1e0f-1a2b3c4d5e6f",
    "childName": "Miguel Rodrigues",
    "title": "Primeiras palavras",
    "description": "Disse 'mamã' pela primeira vez!",
    "category": "Language",
    "achievedAt": "2024-01-15T15:30:00Z",
    "photoUrl": null,
    "notes": "Momento emocionante!",
    "registeredBy": "Ana Costa",
    "registeredAt": "2024-01-15T15:35:00Z"
  }
}
```

**Categorias**:
- `Physical` - Físico
- `Cognitive` - Cognitivo
- `Language` - Linguagem
- `Social` - Social
- `Emotional` - Emocional

---

## 5️⃣ Health Service

**Base Path**: `/api/health`

### 5.1 Temperatura

#### POST /temperature
Regista medição de temperatura.

**Request Body**:
```json
{
  "childId": "9c8d7e6f-5a4b-3c2d-1e0f-1a2b3c4d5e6f",
  "measuredAt": "2024-01-15T10:00:00Z",
  "temperature": 37.5,
  "unit": "Celsius",
  "symptoms": "Tosse leve",
  "notes": "Criança um pouco quente"
}
```

**Response** (201 Created):
```json
{
  "success": true,
  "message": "Temperatura registada com sucesso",
  "data": {
    "id": "e5f6g7h8-i9j0-1k2l-3m4n-o5p6q7r8s9t0",
    "childId": "9c8d7e6f-5a4b-3c2d-1e0f-1a2b3c4d5e6f",
    "childName": "Miguel Rodrigues",
    "measuredAt": "2024-01-15T10:00:00Z",
    "temperature": 37.5,
    "unit": "Celsius",
    "symptoms": "Tosse leve",
    "notes": "Criança um pouco quente",
    "registeredBy": "Ana Costa",
    "registeredAt": "2024-01-15T10:05:00Z"
  }
}
```

---

### 5.2 Medicamentos

#### POST /medications
Regista administração de medicamento.

**Request Body**:
```json
{
  "childId": "9c8d7e6f-5a4b-3c2d-1e0f-1a2b3c4d5e6f",
  "medicationName": "Paracetamol",
  "dosage": "125mg",
  "administeredAt": "2024-01-15T11:00:00Z",
  "prescribedBy": "Dr. João Silva",
  "notes": "Administrado devido a febre"
}
```

**Response** (201 Created):
```json
{
  "success": true,
  "message": "Medicamento registado com sucesso",
  "data": {
    "id": "f6g7h8i9-j0k1-2l3m-4n5o-p6q7r8s9t0u1",
    "childId": "9c8d7e6f-5a4b-3c2d-1e0f-1a2b3c4d5e6f",
    "childName": "Miguel Rodrigues",
    "medicationName": "Paracetamol",
    "dosage": "125mg",
    "administeredAt": "2024-01-15T11:00:00Z",
    "prescribedBy": "Dr. João Silva",
    "notes": "Administrado devido a febre",
    "registeredBy": "Ana Costa",
    "registeredAt": "2024-01-15T11:05:00Z"
  }
}
```

---

### 5.3 Vacinação

#### GET /vaccinations/{childId}
Obtém o plano de vacinação de uma criança.

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "childId": "9c8d7e6f-5a4b-3c2d-1e0f-1a2b3c4d5e6f",
    "childName": "Miguel Rodrigues",
    "birthDate": "2020-05-15",
    "vaccinations": [
      {
        "id": "g7h8i9j0-k1l2-3m4n-5o6p-q7r8s9t0u1v2",
        "vaccineName": "VASPR",
        "scheduledDate": "2024-02-15",
        "administeredDate": null,
        "status": "Scheduled",
        "location": null,
        "notes": ""
      },
      {
        "id": "h8i9j0k1-l2m3-4n5o-6p7q-r8s9t0u1v2w3",
        "vaccineName": "Hepatite B",
        "scheduledDate": "2020-06-15",
        "administeredDate": "2020-06-15",
        "status": "Administered",
        "location": "Centro de Saúde de Lisboa",
        "notes": "Sem reações adversas"
      }
    ]
  }
}
```

**Status de Vacinação**:
- `Scheduled` - Agendada
- `Administered` - Administrada
- `Overdue` - Atrasada
- `Cancelled` - Cancelada

---

## 6️⃣ Notification Service

**Base Path**: `/api/notifications`

### 6.1 Notificações

#### GET /
Lista notificações do utilizador.

**Query Parameters**:
- `page` (int)
- `pageSize` (int)
- `isRead` (boolean, opcional)
- `type` (string, opcional)

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "i9j0k1l2-m3n4-5o6p-7q8r-s9t0u1v2w3x4",
        "title": "Nova atividade registada",
        "message": "Miguel almoçou muito bem!",
        "type": "Activity",
        "isRead": false,
        "createdAt": "2024-01-15T12:35:00Z",
        "data": {
          "activityId": "a1b2c3d4-e5f6-7g8h-9i0j-k1l2m3n4o5p6",
          "activityType": "Feeding",
          "childId": "9c8d7e6f-5a4b-3c2d-1e0f-1a2b3c4d5e6f"
        }
      }
    ],
    "pagination": {
      "currentPage": 1,
      "pageSize": 20,
      "totalPages": 1,
      "totalItems": 5
    },
    "unreadCount": 3
  }
}
```

---

#### PUT /{id}/read
Marca uma notificação como lida.

**Response** (200 OK):
```json
{
  "success": true,
  "message": "Notificação marcada como lida"
}
```

---

#### PUT /read-all
Marca todas as notificações como lidas.

**Response** (200 OK):
```json
{
  "success": true,
  "message": "Todas as notificações foram marcadas como lidas"
}
```

---

## 📊 Códigos de Status HTTP

### Sucesso
- `200 OK`: Requisição bem-sucedida
- `201 Created`: Recurso criado com sucesso
- `204 No Content`: Requisição bem-sucedida sem conteúdo de retorno

### Erros do Cliente
- `400 Bad Request`: Dados inválidos ou requisição malformada
- `401 Unauthorized`: Autenticação necessária ou token inválido
- `403 Forbidden`: Sem permissão para o recurso
- `404 Not Found`: Recurso não encontrado
- `409 Conflict`: Conflito com o estado atual (ex: email duplicado)
- `422 Unprocessable Entity`: Validação de dados falhou
- `429 Too Many Requests`: Rate limit excedido

### Erros do Servidor
- `500 Internal Server Error`: Erro interno do servidor
- `502 Bad Gateway`: Erro de comunicação entre serviços
- `503 Service Unavailable`: Serviço temporariamente indisponível

---

## 🔧 Formato de Erro Padrão

Todos os erros seguem o formato:

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Os dados fornecidos são inválidos",
    "details": [
      {
        "field": "email",
        "message": "Email é obrigatório"
      },
      {
        "field": "password",
        "message": "Password deve ter no mínimo 8 caracteres"
      }
    ]
  },
  "traceId": "0HMVFE42N8J9N:00000001"
}
```

---

## 📝 Notas Importantes

1. **Datas**: Todas as datas devem estar no formato ISO 8601 (UTC)
2. **Paginação**: Default `page=1`, `pageSize=20`, máximo `pageSize=100`
3. **Rate Limiting**: 100 requisições por minuto por IP
4. **Versionamento**: Atualmente v1, future versions em `/api/v2/...`
5. **CORS**: Configurado para permitir origens específicas
6. **Timeout**: Requisições têm timeout de 30 segundos

---

## 🔐 Segurança

### Headers de Segurança
- `X-Content-Type-Options: nosniff`
- `X-Frame-Options: DENY`
- `X-XSS-Protection: 1; mode=block`
- `Strict-Transport-Security: max-age=31536000`

### Rate Limiting
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1642253400
```

---

**Anterior**: [Segurança e Autenticação](./09-SEGURANCA-AUTENTICACAO.md)  
**Próximo**: [Infraestrutura](./11-INFRAESTRUTURA.md)  
**Voltar ao**: [Índice](./00-INDICE.md)
