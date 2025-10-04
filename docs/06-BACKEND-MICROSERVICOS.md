# Backend - Arquitetura de Microserviços

## 📐 Visão Geral

O backend do DailyCare é construído com uma arquitetura de microserviços usando .NET 9, seguindo os princípios de Domain-Driven Design (DDD), Clean Architecture e CQRS.

---

## 🏗️ Estrutura de Microserviços

### 1. Identity Service (Serviço de Identidade)

**Responsabilidade**: Autenticação, autorização e gestão de utilizadores

#### Estrutura do Projeto

```
DailyCare.Identity/
├── DailyCare.Identity.API/
│   ├── Controllers/
│   │   ├── AuthController.cs
│   │   └── ProfileController.cs
│   ├── Middleware/
│   │   ├── JwtMiddleware.cs
│   │   └── ExceptionHandlerMiddleware.cs
│   ├── Program.cs
│   └── appsettings.json
├── DailyCare.Identity.Application/
│   ├── Commands/
│   │   ├── RegisterUser/
│   │   │   ├── RegisterUserCommand.cs
│   │   │   ├── RegisterUserCommandHandler.cs
│   │   │   └── RegisterUserCommandValidator.cs
│   │   └── LoginUser/
│   │       ├── LoginUserCommand.cs
│   │       └── LoginUserCommandHandler.cs
│   ├── Queries/
│   │   └── GetUserProfile/
│   │       ├── GetUserProfileQuery.cs
│   │       └── GetUserProfileQueryHandler.cs
│   ├── DTOs/
│   │   ├── LoginRequest.cs
│   │   ├── LoginResponse.cs
│   │   └── UserDto.cs
│   ├── Services/
│   │   ├── TokenService.cs
│   │   └── PasswordHasher.cs
│   └── Validators/
│       └── LoginRequestValidator.cs
├── DailyCare.Identity.Domain/
│   ├── Entities/
│   │   ├── User.cs
│   │   ├── Role.cs
│   │   ├── Permission.cs
│   │   └── RefreshToken.cs
│   ├── ValueObjects/
│   │   └── Email.cs
│   ├── Events/
│   │   ├── UserRegisteredEvent.cs
│   │   └── UserLoggedInEvent.cs
│   ├── Interfaces/
│   │   ├── IUserRepository.cs
│   │   └── ITokenService.cs
│   └── Exceptions/
│       └── InvalidCredentialsException.cs
└── DailyCare.Identity.Infrastructure/
    ├── Persistence/
    │   ├── IdentityDbContext.cs
    │   ├── Repositories/
    │   │   └── UserRepository.cs
    │   └── Configurations/
    │       ├── UserConfiguration.cs
    │       └── RoleConfiguration.cs
    └── Services/
        └── JwtTokenService.cs
```

#### Entidades Principais

```csharp
// Domain/Entities/User.cs
public class User : BaseEntity
{
    public string Name { get; private set; }
    public Email Email { get; private set; }
    public string PasswordHash { get; private set; }
    public string PhoneNumber { get; private set; }
    public bool EmailConfirmed { get; private set; }
    public bool PhoneConfirmed { get; private set; }
    public DateTime? LastLoginAt { get; private set; }
    public bool IsActive { get; private set; }
    
    public ICollection<UserRole> Roles { get; private set; }
    public ICollection<RefreshToken> RefreshTokens { get; private set; }

    private User() { } // EF Core

    public static User Create(string name, Email email, string password)
    {
        var user = new User
        {
            Id = Guid.NewGuid(),
            Name = name,
            Email = email,
            PasswordHash = BCrypt.Net.BCrypt.HashPassword(password),
            IsActive = true,
            CreatedAt = DateTime.UtcNow
        };

        user.AddDomainEvent(new UserRegisteredEvent(user.Id, user.Email.Value));
        return user;
    }

    public void UpdateProfile(string name, string phoneNumber)
    {
        Name = name;
        PhoneNumber = phoneNumber;
        UpdatedAt = DateTime.UtcNow;
    }

    public void ConfirmEmail()
    {
        EmailConfirmed = true;
    }

    public RefreshToken GenerateRefreshToken()
    {
        var refreshToken = RefreshToken.Create(Id);
        RefreshTokens.Add(refreshToken);
        return refreshToken;
    }
}

public class RefreshToken : BaseEntity
{
    public Guid UserId { get; private set; }
    public string Token { get; private set; }
    public DateTime ExpiresAt { get; private set; }
    public bool IsRevoked { get; private set; }
    public DateTime? RevokedAt { get; private set; }

    public static RefreshToken Create(Guid userId)
    {
        return new RefreshToken
        {
            Id = Guid.NewGuid(),
            UserId = userId,
            Token = GenerateToken(),
            ExpiresAt = DateTime.UtcNow.AddDays(30),
            CreatedAt = DateTime.UtcNow
        };
    }

    private static string GenerateToken()
    {
        var randomNumber = new byte[32];
        using var rng = RandomNumberGenerator.Create();
        rng.GetBytes(randomNumber);
        return Convert.ToBase64String(randomNumber);
    }
}
```

#### Command Handler Example

```csharp
// Application/Commands/LoginUser/LoginUserCommandHandler.cs
public class LoginUserCommandHandler : IRequestHandler<LoginUserCommand, LoginResponse>
{
    private readonly IUserRepository _userRepository;
    private readonly ITokenService _tokenService;
    private readonly ILogger<LoginUserCommandHandler> _logger;

    public LoginUserCommandHandler(
        IUserRepository userRepository,
        ITokenService tokenService,
        ILogger<LoginUserCommandHandler> logger)
    {
        _userRepository = userRepository;
        _tokenService = tokenService;
        _logger = logger;
    }

    public async Task<LoginResponse> Handle(LoginUserCommand request, CancellationToken cancellationToken)
    {
        _logger.LogInformation("Tentativa de login para email {Email}", request.Email);

        var user = await _userRepository.GetByEmailAsync(request.Email, cancellationToken);
        
        if (user == null || !BCrypt.Net.BCrypt.Verify(request.Password, user.PasswordHash))
        {
            _logger.LogWarning("Falha de autenticação para email {Email}", request.Email);
            throw new InvalidCredentialsException("Email ou password incorretos");
        }

        if (!user.IsActive)
        {
            throw new UserInactiveException("Conta de utilizador inativa");
        }

        var accessToken = _tokenService.GenerateAccessToken(user);
        var refreshToken = user.GenerateRefreshToken();
        
        await _userRepository.UpdateAsync(user, cancellationToken);

        _logger.LogInformation("Login bem-sucedido para utilizador {UserId}", user.Id);

        return new LoginResponse
        {
            AccessToken = accessToken,
            RefreshToken = refreshToken.Token,
            ExpiresIn = 3600,
            User = new UserDto
            {
                Id = user.Id,
                Name = user.Name,
                Email = user.Email.Value,
                Role = user.Roles.FirstOrDefault()?.Role.Name
            }
        };
    }
}
```

---

### 2. School Service (Serviço de Escolas)

**Responsabilidade**: Gestão de escolas (tenants), turmas e educadores

#### Entidades Principais

```csharp
// Domain/Entities/School.cs
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

    public static School Create(string name, string code, string email, Address address)
    {
        var school = new School
        {
            Id = Guid.NewGuid(),
            Name = name,
            Code = code,
            Email = email,
            Address = address,
            IsActive = false,
            CreatedAt = DateTime.UtcNow,
            Classes = new List<SchoolClass>()
        };

        school.AddDomainEvent(new SchoolCreatedEvent(school.Id, school.Name));
        return school;
    }

    public void Activate(string connectionString)
    {
        ConnectionString = connectionString;
        IsActive = true;
        ActivatedAt = DateTime.UtcNow;
        
        AddDomainEvent(new SchoolActivatedEvent(Id));
    }

    public SchoolClass AddClass(string name, AgeRange ageRange, int capacity)
    {
        var schoolClass = SchoolClass.Create(Id, name, ageRange, capacity);
        Classes.Add(schoolClass);
        return schoolClass;
    }
}

// Domain/Entities/SchoolClass.cs
public class SchoolClass : BaseEntity
{
    public Guid SchoolId { get; private set; }
    public string Name { get; private set; }
    public string Description { get; private set; }
    public AgeRange AgeRange { get; private set; }
    public int Capacity { get; private set; }
    public string AcademicYear { get; private set; }
    
    public ICollection<ClassEducator> Educators { get; private set; }

    public static SchoolClass Create(Guid schoolId, string name, AgeRange ageRange, int capacity)
    {
        return new SchoolClass
        {
            Id = Guid.NewGuid(),
            SchoolId = schoolId,
            Name = name,
            AgeRange = ageRange,
            Capacity = capacity,
            CreatedAt = DateTime.UtcNow,
            Educators = new List<ClassEducator>()
        };
    }

    public void AssignEducator(Guid educatorId, EducatorRole role)
    {
        var classEducator = new ClassEducator
        {
            ClassId = Id,
            EducatorId = educatorId,
            Role = role,
            AssignedAt = DateTime.UtcNow
        };
        
        Educators.Add(classEducator);
    }
}

// Domain/ValueObjects/Address.cs
public class Address : ValueObject
{
    public string Street { get; private set; }
    public string Number { get; private set; }
    public string PostalCode { get; private set; }
    public string City { get; private set; }
    public string Country { get; private set; }

    public Address(string street, string number, string postalCode, string city, string country)
    {
        Street = street;
        Number = number;
        PostalCode = postalCode;
        City = city;
        Country = country;
    }

    protected override IEnumerable<object> GetEqualityComponents()
    {
        yield return Street;
        yield return Number;
        yield return PostalCode;
        yield return City;
        yield return Country;
    }
}
```

---

### 3. Children Service (Serviço de Crianças)

**Responsabilidade**: Gestão de perfis de crianças e relações familiares

#### Entidades Principais

```csharp
// Domain/Entities/Child.cs
public class Child : BaseEntity
{
    public string FirstName { get; private set; }
    public string LastName { get; private set; }
    public DateTime BirthDate { get; private set; }
    public Gender Gender { get; private set; }
    public string PhotoUrl { get; private set; }
    public BloodType? BloodType { get; private set; }
    
    public Guid CurrentClassId { get; private set; }
    
    public EmergencyContact EmergencyContact { get; private set; }
    public MedicalInfo MedicalInfo { get; private set; }
    public ICollection<FamilyMember> FamilyMembers { get; private set; }
    public ICollection<Enrollment> Enrollments { get; private set; }

    public string FullName => $"{FirstName} {LastName}";
    public int Age => DateTime.UtcNow.Year - BirthDate.Year;

    public static Child Create(string firstName, string lastName, DateTime birthDate, Gender gender)
    {
        var child = new Child
        {
            Id = Guid.NewGuid(),
            FirstName = firstName,
            LastName = lastName,
            BirthDate = birthDate,
            Gender = gender,
            FamilyMembers = new List<FamilyMember>(),
            Enrollments = new List<Enrollment>(),
            CreatedAt = DateTime.UtcNow
        };

        child.AddDomainEvent(new ChildCreatedEvent(child.Id, child.FullName));
        return child;
    }

    public void UpdatePhoto(string photoUrl)
    {
        PhotoUrl = photoUrl;
        UpdatedAt = DateTime.UtcNow;
    }

    public void AddFamilyMember(Guid userId, RelationshipType relationship, bool isPrimaryContact)
    {
        var familyMember = new FamilyMember
        {
            ChildId = Id,
            UserId = userId,
            Relationship = relationship,
            IsPrimaryContact = isPrimaryContact,
            CanPickup = true,
            CreatedAt = DateTime.UtcNow
        };

        FamilyMembers.Add(familyMember);
    }

    public void EnrollInClass(Guid classId, DateTime startDate)
    {
        CurrentClassId = classId;
        
        var enrollment = new Enrollment
        {
            ChildId = Id,
            ClassId = classId,
            StartDate = startDate,
            IsActive = true
        };

        Enrollments.Add(enrollment);
        AddDomainEvent(new ChildEnrolledEvent(Id, classId));
    }
}

// Domain/Entities/FamilyMember.cs
public class FamilyMember : BaseEntity
{
    public Guid ChildId { get; private set; }
    public Guid UserId { get; private set; }
    public RelationshipType Relationship { get; private set; }
    public bool IsPrimaryContact { get; private set; }
    public bool CanPickup { get; private set; }
}

// Domain/ValueObjects/MedicalInfo.cs
public class MedicalInfo : ValueObject
{
    public List<string> Allergies { get; private set; }
    public List<string> Medications { get; private set; }
    public string SpecialNeeds { get; private set; }

    public MedicalInfo(List<string> allergies, List<string> medications, string specialNeeds)
    {
        Allergies = allergies ?? new List<string>();
        Medications = medications ?? new List<string>();
        SpecialNeeds = specialNeeds;
    }

    protected override IEnumerable<object> GetEqualityComponents()
    {
        yield return string.Join(",", Allergies);
        yield return string.Join(",", Medications);
        yield return SpecialNeeds;
    }
}
```

---

### 4. Activities Service (Serviço de Atividades)

**Responsabilidade**: Registo e gestão de atividades diárias

#### Entidades Principais

```csharp
// Domain/Entities/FeedingActivity.cs
public class FeedingActivity : BaseEntity
{
    public Guid ChildId { get; private set; }
    public DateTime StartTime { get; private set; }
    public DateTime? EndTime { get; private set; }
    public FeedingType Type { get; private set; }
    public decimal? Quantity { get; private set; }
    public QuantityUnit Unit { get; private set; }
    public string FoodDescription { get; private set; }
    public Appetite Appetite { get; private set; }
    public string Notes { get; private set; }
    public string RegisteredBy { get; private set; }

    public TimeSpan? Duration => EndTime.HasValue ? EndTime.Value - StartTime : null;

    public static FeedingActivity Create(
        Guid childId,
        DateTime startTime,
        FeedingType type,
        string foodDescription)
    {
        var activity = new FeedingActivity
        {
            Id = Guid.NewGuid(),
            ChildId = childId,
            StartTime = startTime,
            Type = type,
            FoodDescription = foodDescription,
            CreatedAt = DateTime.UtcNow
        };

        activity.AddDomainEvent(new FeedingActivityCreatedEvent(activity.Id, childId));
        return activity;
    }

    public void Complete(DateTime endTime, Appetite appetite)
    {
        EndTime = endTime;
        Appetite = appetite;
        UpdatedAt = DateTime.UtcNow;
    }
}

// Domain/Entities/SleepActivity.cs
public class SleepActivity : BaseEntity
{
    public Guid ChildId { get; private set; }
    public DateTime StartTime { get; private set; }
    public DateTime EndTime { get; private set; }
    public SleepType Type { get; private set; }
    public SleepQuality Quality { get; private set; }
    public string Notes { get; private set; }
    public string RegisteredBy { get; private set; }

    public TimeSpan Duration => EndTime - StartTime;

    public static SleepActivity Create(
        Guid childId,
        DateTime startTime,
        DateTime endTime,
        SleepType type)
    {
        var activity = new SleepActivity
        {
            Id = Guid.NewGuid(),
            ChildId = childId,
            StartTime = startTime,
            EndTime = endTime,
            Type = type,
            CreatedAt = DateTime.UtcNow
        };

        activity.AddDomainEvent(new SleepActivityCreatedEvent(activity.Id, childId));
        return activity;
    }
}

// Domain/Entities/Milestone.cs
public class Milestone : BaseEntity
{
    public Guid ChildId { get; private set; }
    public string Title { get; private set; }
    public string Description { get; private set; }
    public MilestoneCategory Category { get; private set; }
    public DateTime AchievedAt { get; private set; }
    public string PhotoUrl { get; private set; }
    public string RegisteredBy { get; private set; }

    public static Milestone Create(
        Guid childId,
        string title,
        string description,
        MilestoneCategory category,
        DateTime achievedAt)
    {
        var milestone = new Milestone
        {
            Id = Guid.NewGuid(),
            ChildId = childId,
            Title = title,
            Description = description,
            Category = category,
            AchievedAt = achievedAt,
            CreatedAt = DateTime.UtcNow
        };

        milestone.AddDomainEvent(new MilestoneAchievedEvent(milestone.Id, childId, title));
        return milestone;
    }
}
```

---

### 5. Health Service (Serviço de Saúde)

**Responsabilidade**: Gestão de informações de saúde e bem-estar

#### Entidades Principais

```csharp
// Domain/Entities/TemperatureRecord.cs
public class TemperatureRecord : BaseEntity
{
    public Guid ChildId { get; private set; }
    public DateTime MeasuredAt { get; private set; }
    public decimal Temperature { get; private set; }
    public TemperatureUnit Unit { get; private set; }
    public string Symptoms { get; private set; }
    public string Notes { get; private set; }
    public string MeasuredBy { get; private set; }

    public bool IsFever => (Unit == TemperatureUnit.Celsius && Temperature >= 37.5m) ||
                           (Unit == TemperatureUnit.Fahrenheit && Temperature >= 99.5m);

    public static TemperatureRecord Create(
        Guid childId,
        DateTime measuredAt,
        decimal temperature,
        TemperatureUnit unit)
    {
        var record = new TemperatureRecord
        {
            Id = Guid.NewGuid(),
            ChildId = childId,
            MeasuredAt = measuredAt,
            Temperature = temperature,
            Unit = unit,
            CreatedAt = DateTime.UtcNow
        };

        if (record.IsFever)
        {
            record.AddDomainEvent(new FeverDetectedEvent(childId, temperature));
        }

        return record;
    }
}

// Domain/Entities/Medication.cs
public class Medication : BaseEntity
{
    public Guid ChildId { get; private set; }
    public string Name { get; private set; }
    public string Dosage { get; private set; }
    public DateTime StartDate { get; private set; }
    public DateTime? EndDate { get; private set; }
    public string Frequency { get; private set; }
    public string PrescribedBy { get; private set; }
    public string Notes { get; private set; }
    
    public ICollection<MedicationDose> Doses { get; private set; }

    public bool IsActive => !EndDate.HasValue || EndDate.Value > DateTime.UtcNow;

    public void AddDose(DateTime administeredAt, string administeredBy)
    {
        var dose = new MedicationDose
        {
            MedicationId = Id,
            AdministeredAt = administeredAt,
            AdministeredBy = administeredBy
        };

        Doses.Add(dose);
    }
}

// Domain/Entities/Vaccination.cs
public class Vaccination : BaseEntity
{
    public Guid ChildId { get; private set; }
    public string VaccineName { get; private set; }
    public DateTime ScheduledDate { get; private set; }
    public DateTime? AdministeredDate { get; private set; }
    public VaccinationStatus Status { get; private set; }
    public string Location { get; private set; }
    public string Notes { get; private set; }

    public bool IsOverdue => Status == VaccinationStatus.Scheduled && 
                             ScheduledDate < DateTime.UtcNow;

    public void MarkAsAdministered(DateTime administeredDate, string location)
    {
        AdministeredDate = administeredDate;
        Location = location;
        Status = VaccinationStatus.Administered;
        UpdatedAt = DateTime.UtcNow;
    }
}
```

---

## 🔄 Comunicação Entre Serviços

### Eventos de Domínio

```csharp
// Shared/Domain/Events/IDomainEvent.cs
public interface IDomainEvent
{
    Guid EventId { get; }
    DateTime OccurredOn { get; }
}

// Identity Service - Domain/Events/UserRegisteredEvent.cs
public class UserRegisteredEvent : IDomainEvent
{
    public Guid EventId { get; }
    public DateTime OccurredOn { get; }
    public Guid UserId { get; }
    public string Email { get; }

    public UserRegisteredEvent(Guid userId, string email)
    {
        EventId = Guid.NewGuid();
        OccurredOn = DateTime.UtcNow;
        UserId = userId;
        Email = email;
    }
}

// Event Handler em outro serviço
public class UserRegisteredEventHandler : INotificationHandler<UserRegisteredEvent>
{
    private readonly IEmailService _emailService;

    public async Task Handle(UserRegisteredEvent notification, CancellationToken cancellationToken)
    {
        await _emailService.SendWelcomeEmailAsync(notification.Email, cancellationToken);
    }
}
```

### Mensageria (RabbitMQ)

```csharp
// MassTransit Configuration
services.AddMassTransit(x =>
{
    // Consumers
    x.AddConsumer<ChildEnrolledConsumer>();
    x.AddConsumer<ActivityCreatedConsumer>();

    x.UsingRabbitMq((context, cfg) =>
    {
        cfg.Host("rabbitmq://localhost", h =>
        {
            h.Username("admin");
            h.Password("admin");
        });

        // Configurar exchanges e queues
        cfg.Message<ChildEnrolledEvent>(e => e.SetEntityName("child.enrolled"));
        cfg.Publish<ChildEnrolledEvent>(e => e.ExchangeType = "fanout");

        cfg.ConfigureEndpoints(context);
    });
});

// Consumer Example
public class ChildEnrolledConsumer : IConsumer<ChildEnrolledEvent>
{
    private readonly INotificationService _notificationService;

    public async Task Consume(ConsumeContext<ChildEnrolledEvent> context)
    {
        var message = context.Message;
        
        await _notificationService.NotifyParentsAsync(
            message.ChildId,
            "Matrícula confirmada",
            $"A criança foi matriculada na turma {message.ClassName}");
    }
}
```

---

## 📦 Shared Libraries

### DailyCare.Shared.Domain

```csharp
// Shared/Domain/BaseEntity.cs
public abstract class BaseEntity
{
    public Guid Id { get; protected set; }
    public DateTime CreatedAt { get; protected set; }
    public DateTime? UpdatedAt { get; protected set; }
    public string CreatedBy { get; protected set; }
    public string UpdatedBy { get; protected set; }
    public bool IsDeleted { get; protected set; }

    private readonly List<IDomainEvent> _domainEvents = new();
    public IReadOnlyCollection<IDomainEvent> DomainEvents => _domainEvents.AsReadOnly();

    protected void AddDomainEvent(IDomainEvent eventItem)
    {
        _domainEvents.Add(eventItem);
    }

    public void ClearDomainEvents()
    {
        _domainEvents.Clear();
    }

    public void Delete()
    {
        IsDeleted = true;
        UpdatedAt = DateTime.UtcNow;
    }
}

// Shared/Domain/ValueObject.cs
public abstract class ValueObject
{
    protected abstract IEnumerable<object> GetEqualityComponents();

    public override bool Equals(object obj)
    {
        if (obj == null || obj.GetType() != GetType())
            return false;

        var other = (ValueObject)obj;
        return GetEqualityComponents().SequenceEqual(other.GetEqualityComponents());
    }

    public override int GetHashCode()
    {
        return GetEqualityComponents()
            .Select(x => x?.GetHashCode() ?? 0)
            .Aggregate((x, y) => x ^ y);
    }
}
```

### DailyCare.Shared.Application

```csharp
// Shared/Application/Result.cs
public class Result<T>
{
    public bool IsSuccess { get; }
    public T Data { get; }
    public string Error { get; }
    public List<string> Errors { get; }

    protected Result(bool isSuccess, T data, string error, List<string> errors)
    {
        IsSuccess = isSuccess;
        Data = data;
        Error = error;
        Errors = errors ?? new List<string>();
    }

    public static Result<T> Success(T data) => new(true, data, null, null);
    public static Result<T> Failure(string error) => new(false, default, error, null);
    public static Result<T> Failure(List<string> errors) => new(false, default, null, errors);
}

// Shared/Application/PagedResult.cs
public class PagedResult<T>
{
    public List<T> Items { get; set; }
    public int TotalItems { get; set; }
    public int CurrentPage { get; set; }
    public int PageSize { get; set; }
    public int TotalPages => (int)Math.Ceiling(TotalItems / (double)PageSize);
    public bool HasPrevious => CurrentPage > 1;
    public bool HasNext => CurrentPage < TotalPages;
}
```

---

## 🧪 Testes

### Testes Unitários

```csharp
// Tests/Identity.UnitTests/Domain/UserTests.cs
public class UserTests
{
    [Fact]
    public void Create_ComDadosValidos_DeveCriarUtilizador()
    {
        // Arrange
        var name = "João Silva";
        var email = Email.Create("joao@example.com");
        var password = "Password123!";

        // Act
        var user = User.Create(name, email, password);

        // Assert
        user.Should().NotBeNull();
        user.Name.Should().Be(name);
        user.Email.Should().Be(email);
        user.IsActive.Should().BeTrue();
        user.DomainEvents.Should().ContainSingle(e => e is UserRegisteredEvent);
    }

    [Fact]
    public void ConfirmEmail_DeveMarcarEmailComoConfirmado()
    {
        // Arrange
        var user = User.Create("Test", Email.Create("test@example.com"), "Pass123!");

        // Act
        user.ConfirmEmail();

        // Assert
        user.EmailConfirmed.Should().BeTrue();
    }
}
```

### Testes de Integração

```csharp
// Tests/Identity.IntegrationTests/Controllers/AuthControllerTests.cs
public class AuthControllerTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly HttpClient _client;

    public AuthControllerTests(WebApplicationFactory<Program> factory)
    {
        _client = factory.CreateClient();
    }

    [Fact]
    public async Task Login_ComCredenciaisValidas_DeveRetornarToken()
    {
        // Arrange
        var request = new LoginRequest
        {
            Email = "test@example.com",
            Password = "Password123!"
        };

        // Act
        var response = await _client.PostAsJsonAsync("/api/auth/login", request);

        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.OK);
        var result = await response.Content.ReadFromJsonAsync<LoginResponse>();
        result.AccessToken.Should().NotBeNullOrEmpty();
    }
}
```

---

## 📝 Boas Práticas

### 1. SOLID Principles
- ✅ Single Responsibility
- ✅ Open/Closed
- ✅ Liskov Substitution
- ✅ Interface Segregation
- ✅ Dependency Inversion

### 2. DDD Patterns
- ✅ Entities
- ✅ Value Objects
- ✅ Aggregates
- ✅ Domain Events
- ✅ Repositories

### 3. CQRS
- ✅ Separação Commands/Queries
- ✅ MediatR para mediação
- ✅ Validação com FluentValidation

### 4. Clean Code
- ✅ Nomes descritivos
- ✅ Métodos pequenos
- ✅ Comentários em português
- ✅ Testes unitários

---

**Anterior**: [Frontend Admin](./05-FRONTEND-ADMIN.md)  
**Próximo**: [Base de Dados](./07-BASE-DADOS.md)  
**Voltar ao**: [Índice](./00-INDICE.md)
