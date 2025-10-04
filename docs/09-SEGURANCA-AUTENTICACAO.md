# Segurança e Autenticação - DailyCare

## 🔐 Visão Geral

A segurança no DailyCare é implementada em múltiplas camadas, garantindo proteção de dados, autenticação robusta e autorização granular, com total compliance RGPD.

---

## 🎫 Autenticação JWT

### Estrutura do Token

```json
{
  "header": {
    "alg": "HS256",
    "typ": "JWT"
  },
  "payload": {
    "sub": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "email": "joao.silva@example.com",
    "name": "João Silva",
    "role": "Parent",
    "tenant_id": "7c9e6679-7425-40de-944b-e07fc1f90ae7",
    "permissions": ["read:children", "read:activities"],
    "iat": 1704369600,
    "exp": 1704373200,
    "iss": "DailyCare",
    "aud": "DailyCare.API"
  }
}
```

### Implementação

```csharp
// Infrastructure/Services/JwtTokenService.cs
public class JwtTokenService : ITokenService
{
    private readonly IConfiguration _configuration;

    public string GenerateAccessToken(User user)
    {
        var jwtSettings = _configuration.GetSection("Jwt");
        var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(jwtSettings["Secret"]));
        var credentials = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);

        var claims = new List<Claim>
        {
            new Claim(JwtRegisteredClaimNames.Sub, user.Id.ToString()),
            new Claim(JwtRegisteredClaimNames.Email, user.Email.Value),
            new Claim(JwtRegisteredClaimNames.Name, user.Name),
            new Claim("role", user.Roles.First().Role.Name),
            new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString())
        };

        if (user.TenantId.HasValue)
        {
            claims.Add(new Claim("tenant_id", user.TenantId.Value.ToString()));
        }

        var token = new JwtSecurityToken(
            issuer: jwtSettings["Issuer"],
            audience: jwtSettings["Audience"],
            claims: claims,
            expires: DateTime.UtcNow.AddMinutes(60),
            signingCredentials: credentials
        );

        return new JwtSecurityTokenHandler().WriteToken(token);
    }
}
```

---

## 🔄 Refresh Tokens

```csharp
public class RefreshToken : BaseEntity
{
    public Guid UserId { get; private set; }
    public string Token { get; private set; }
    public DateTime ExpiresAt { get; private set; }
    public bool IsRevoked { get; private set; }

    public static RefreshToken Create(Guid userId)
    {
        var randomBytes = new byte[32];
        using var rng = RandomNumberGenerator.Create();
        rng.GetBytes(randomBytes);

        return new RefreshToken
        {
            Id = Guid.NewGuid(),
            UserId = userId,
            Token = Convert.ToBase64String(randomBytes),
            ExpiresAt = DateTime.UtcNow.AddDays(30)
        };
    }

    public void Revoke()
    {
        IsRevoked = true;
        RevokedAt = DateTime.UtcNow;
    }
}
```

---

## 🛡️ Autorização RBAC

### Roles e Permissões

```csharp
public class Role : BaseEntity
{
    public string Name { get; private set; }
    public ICollection<Permission> Permissions { get; private set; }

    public static class Roles
    {
        public const string SuperAdmin = "SuperAdmin";
        public const string SchoolAdmin = "SchoolAdmin";
        public const string Educator = "Educator";
        public const string Parent = "Parent";
    }
}

public class Permission : BaseEntity
{
    public string Name { get; private set; }
    public string Resource { get; private set; }
    public string Action { get; private set; }
}
```

### Políticas

```csharp
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("ManageSchools", policy =>
        policy.RequireAssertion(context =>
            context.User.HasClaim("permission", "write:schools") ||
            context.User.IsInRole("SuperAdmin")));

    options.AddPolicy("ViewChildren", policy =>
        policy.RequireAssertion(context =>
            context.User.HasClaim("permission", "read:children")));
});
```

---

## 🔒 Password Hashing

```csharp
public class PasswordHasher : IPasswordHasher
{
    private const int WorkFactor = 12;

    public string HashPassword(string password)
    {
        return BCrypt.Net.BCrypt.HashPassword(password, WorkFactor);
    }

    public bool VerifyPassword(string password, string hash)
    {
        return BCrypt.Net.BCrypt.Verify(password, hash);
    }
}
```

### Políticas de Password

- Mínimo 8 caracteres
- Pelo menos 1 maiúscula
- Pelo menos 1 minúscula
- Pelo menos 1 número
- Pelo menos 1 caractere especial

---

## 🚦 Rate Limiting

```csharp
builder.Services.AddRateLimiter(options =>
{
    options.GlobalLimiter = PartitionedRateLimiter.Create<HttpContext, string>(context =>
        RateLimitPartition.GetFixedWindowLimiter(
            partitionKey: context.User.Identity?.Name ?? context.Request.Headers.Host.ToString(),
            factory: _ => new FixedWindowRateLimiterOptions
            {
                PermitLimit = 100,
                Window = TimeSpan.FromMinutes(1)
            }));
});
```

---

## 🔐 Headers de Segurança

```csharp
app.Use(async (context, next) =>
{
    context.Response.Headers.Add("Strict-Transport-Security", "max-age=31536000");
    context.Response.Headers.Add("X-Content-Type-Options", "nosniff");
    context.Response.Headers.Add("X-Frame-Options", "DENY");
    context.Response.Headers.Add("X-XSS-Protection", "1; mode=block");
    await next();
});
```

---

## 📝 Auditoria

```csharp
public class AuditLog : BaseEntity
{
    public Guid UserId { get; private set; }
    public string Action { get; private set; }
    public string Resource { get; private set; }
    public Guid? ResourceId { get; private set; }
    public string IpAddress { get; private set; }
    public DateTime Timestamp { get; private set; }
}
```

---

## 🔐 RGPD Compliance

### Consentimento

```csharp
public class UserConsent : BaseEntity
{
    public Guid UserId { get; private set; }
    public ConsentType Type { get; private set; }
    public bool IsGranted { get; private set; }
    public DateTime GrantedAt { get; private set; }
}
```

### Direito ao Esquecimento

```csharp
public class DeleteUserDataCommand : IRequest
{
    public Guid UserId { get; set; }
}
```

---

## ✅ Checklist de Segurança

- [x] Autenticação JWT
- [x] Refresh tokens
- [x] Password hashing (BCrypt)
- [x] RBAC
- [x] HTTPS obrigatório
- [x] Rate limiting
- [x] Headers de segurança
- [x] Auditoria
- [x] RGPD compliance
- [x] Multi-tenancy isolado

---

**Anterior**: [Multi-Tenancy](./08-MULTI-TENANCY.md)  
**Próximo**: [API Documentação](./10-API-DOCUMENTACAO.md)  
**Voltar ao**: [Índice](./00-INDICE.md)
