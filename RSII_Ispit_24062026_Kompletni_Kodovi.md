# RSII 24.06.2026. – KOMPLETNI KODOVI, KORAK PO KORAK

Prije svega: u cijelom dokumentu uradi **Replace All**:

`IBXXXXXX` → tvoj indeks, npr. `IB210001`

Zatim idi redom. Svaki korak je **cijeli fajl** (ili cijela komanda). Ne nagađaj šta fali.

Template na kojem ovo radi: `rsII_exam_template_2025_26`.

---

# KORAK 1 – Connection string

**Fajl:** `eCommerce/eCommerce.WebAPI/appsettings.Development.json`

**Šta radiš:** otvori fajl, obriši sve, zalijepi ovo (Database = TVOJ indeks).

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "ConnectionStrings": {
    "DefaultConnection": "Server=192.168.0.1\\Exams,1999;Database=IBXXXXXX;User Id=john;Password=doe2025;TrustServerCertificate=True;"
  }
}
```

Visual Studio → Set Startup Project = `eCommerce.WebAPI`.

Package Manager Console → Default project = `eCommerce.Services`.

```
Update-Database
```

Ako prođe, baza tvog indeksa postoji. Login u aplikaciji: `customer1` / `Test123`.

---

# KORAK 2 – NOVI fajl: Entity kartice

**Fajl:** `eCommerce/eCommerce.Services/Database/PaymentCardIBXXXXXX.cs`

Desni klik na folder `Database` → Add → Class → ime `PaymentCardIBXXXXXX`.

Obriši generisani sadržaj, zalijepi:

```csharp
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace eCommerce.Services.Database
{
    public class PaymentCardIBXXXXXX
    {
        [Key]
        public int Id { get; set; }

        public int UserId { get; set; }

        [ForeignKey(nameof(UserId))]
        public User User { get; set; } = null!;

        [Required]
        [MaxLength(12)]
        public string CardNumber { get; set; } = string.Empty;

        [Required]
        [MaxLength(3)]
        public string Cvc { get; set; } = string.Empty;

        [Required]
        public DateTime ExpiryDate { get; set; }

        [Required]
        [Column(TypeName = "decimal(18,2)")]
        public decimal InitialBalance { get; set; }

        public DateTime CreatedAt { get; set; } = DateTime.UtcNow;

        public DateTime? UpdatedAt { get; set; }

        public ICollection<Order> Orders { get; set; } = new List<Order>();
    }
}
```

---

# KORAK 3 – CIJELI `User.cs` (zamijeni postojeći sadržaj)

**Fajl:** `eCommerce/eCommerce.Services/Database/User.cs`

```csharp
using System;
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;

namespace eCommerce.Services.Database
{
    public class User
    {
        [Key]
        public int Id { get; set; }

        [Required]
        [MaxLength(50)]
        public string FirstName { get; set; } = string.Empty;

        [Required]
        [MaxLength(50)]
        public string LastName { get; set; } = string.Empty;

        [Required]
        [MaxLength(100)]
        [EmailAddress]
        public string Email { get; set; } = string.Empty;

        [Required]
        [MaxLength(100)]
        public string Username { get; set; } = string.Empty;

        public string PasswordHash { get; set; } = string.Empty;

        public string PasswordSalt { get; set; } = string.Empty;

        public bool IsActive { get; set; } = true;

        public DateTime CreatedAt { get; set; } = DateTime.UtcNow;

        public DateTime? LastLoginAt { get; set; }

        [Phone]
        [MaxLength(20)]
        public string? PhoneNumber { get; set; }

        public string? ProfileImageBase64 { get; set; }

        public ICollection<UserRole> UserRoles { get; set; } = new List<UserRole>();

        public ICollection<RefreshToken> RefreshTokens { get; set; } = new List<RefreshToken>();

        public ICollection<PaymentCardIBXXXXXX> PaymentCardsIBXXXXXX { get; set; } = new List<PaymentCardIBXXXXXX>();
    }
}
```

Jedina nova stvar je zadnji `ICollection`.

---

# KORAK 4 – CIJELI `Order.cs`

**Fajl:** `eCommerce/eCommerce.Services/Database/Order.cs`

```csharp
using System;
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace eCommerce.Services.Database
{
    public class Order
    {
        [Key]
        public int Id { get; set; }

        [Required]
        public DateTime OrderDate { get; set; } = DateTime.UtcNow;

        [MaxLength(20)]
        public string OrderNumber { get; set; } = string.Empty;

        [Required]
        public OrderStatus Status { get; set; } = OrderStatus.Pending;

        [Required]
        [Column(TypeName = "decimal(18,2)")]
        public decimal TotalAmount { get; set; }

        public int UserId { get; set; }

        [ForeignKey("UserId")]
        public User User { get; set; } = null!;

        public ICollection<OrderItem> OrderItems { get; set; } = new List<OrderItem>();

        public ICollection<ProductReview> ProductReviews { get; set; } = new List<ProductReview>();

        [MaxLength(200)]
        public string ShippingAddress { get; set; } = string.Empty;

        [MaxLength(100)]
        public string ShippingCity { get; set; } = string.Empty;

        [MaxLength(50)]
        public string ShippingState { get; set; } = string.Empty;

        [MaxLength(20)]
        public string ShippingZipCode { get; set; } = string.Empty;

        [MaxLength(100)]
        public string ShippingCountry { get; set; } = string.Empty;

        [MaxLength(20)]
        public string? PaymentTransactionId { get; set; }

        public DateTime? PaymentDate { get; set; }

        public string? Notes { get; set; }

        public int? PaymentCardIBXXXXXXId { get; set; }

        [ForeignKey(nameof(PaymentCardIBXXXXXXId))]
        public PaymentCardIBXXXXXX? PaymentCardIBXXXXXX { get; set; }
    }

    public enum OrderStatus
    {
        Pending,
        Processing,
        Shipped,
        Delivered,
        Cancelled,
        Returned
    }
}
```

Novo: `PaymentCardIBXXXXXXId` i navigation. Ostalo je isto kao u templateu.

---

# KORAK 5 – CIJELI `eCommerceConfiguration.cs`

**Fajl:** `eCommerce/eCommerce.Services/Database/eCommerceConfiguration.cs`

```csharp
using Microsoft.EntityFrameworkCore;

namespace eCommerce.Services.Database
{
    public partial class ECommerceDbContext : DbContext
    {
        private void CreateConfiguration(ModelBuilder modelBuilder)
        {
            modelBuilder.Entity<Category>()
                .HasOne(c => c.ParentCategory)
                .WithMany(c => c.ChildCategories)
                .HasForeignKey(c => c.ParentCategoryId)
                .OnDelete(DeleteBehavior.Restrict);

            modelBuilder.Entity<ProductCategory>()
                .HasOne(pc => pc.Product)
                .WithMany(pc => pc.ProductCategories)
                .HasForeignKey(pc => pc.ProductId)
                .OnDelete(DeleteBehavior.Cascade);

            modelBuilder.Entity<ProductCategory>()
                .HasOne(pc => pc.Category)
                .WithMany(pc => pc.ProductCategories)
                .HasForeignKey(pc => pc.CategoryId)
                .OnDelete(DeleteBehavior.Cascade);

            modelBuilder.Entity<UserRole>()
                .HasOne(ur => ur.User)
                .WithMany(ur => ur.UserRoles)
                .HasForeignKey(ur => ur.UserId)
                .OnDelete(DeleteBehavior.Cascade);

            modelBuilder.Entity<UserRole>()
                .HasOne(ur => ur.Role)
                .WithMany(ur => ur.UserRoles)
                .HasForeignKey(ur => ur.RoleId)
                .OnDelete(DeleteBehavior.Cascade);

            modelBuilder.Entity<Asset>()
               .HasOne(a => a.Product)
               .WithMany(p => p.Assets)
               .HasForeignKey(a => a.ProductId)
               .OnDelete(DeleteBehavior.Cascade);

            modelBuilder.Entity<ProductReview>()
                .HasOne(pr => pr.Order)
                .WithMany(o => o.ProductReviews)
                .HasForeignKey(pr => pr.OrderId)
                .OnDelete(DeleteBehavior.Restrict);

            modelBuilder.Entity<PaymentCardIBXXXXXX>()
                .HasOne(c => c.User)
                .WithMany(u => u.PaymentCardsIBXXXXXX)
                .HasForeignKey(c => c.UserId)
                .OnDelete(DeleteBehavior.Cascade);

            modelBuilder.Entity<Order>()
                .HasOne(o => o.PaymentCardIBXXXXXX)
                .WithMany(c => c.Orders)
                .HasForeignKey(o => o.PaymentCardIBXXXXXXId)
                .OnDelete(DeleteBehavior.Restrict);
        }
    }
}
```

---

# KORAK 6 – CIJELI `eCommerceDbContext.cs`

**Fajl:** `eCommerce/eCommerce.Services/Database/eCommerceDbContext.cs`

```csharp
using Microsoft.EntityFrameworkCore;

namespace eCommerce.Services.Database
{
    public partial class ECommerceDbContext : DbContext
    {
        public ECommerceDbContext(DbContextOptions<ECommerceDbContext> options) : base(options)
        {
        }

        public DbSet<Category> Categories { get; set; }
        public DbSet<Product> Products { get; set; }
        public DbSet<ProductType> ProductTypes { get; set; }
        public DbSet<UnitOfMeasure> UnitOfMeasures { get; set; }
        public DbSet<ProductCategory> ProductCategories { get; set; }
        public DbSet<ProductReview> ProductReviews { get; set; }
        public DbSet<User> Users { get; set; }
        public DbSet<Role> Roles { get; set; }
        public DbSet<UserRole> UserRoles { get; set; }
        public DbSet<Cart> Carts { get; set; }
        public DbSet<CartItem> CartItems { get; set; }
        public DbSet<Order> Orders { get; set; }
        public DbSet<OrderItem> OrderItems { get; set; }
        public DbSet<Asset> Assets { get; set; }
        public DbSet<RefreshToken> RefreshTokens { get; set; }
        public DbSet<PaymentCardIBXXXXXX> PaymentCardsIBXXXXXX { get; set; }

        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            base.OnModelCreating(modelBuilder);

            CreateConfiguration(modelBuilder);

            CreateSeed(modelBuilder);
        }
    }
}
```

Sada **Build** (Ctrl+Shift+B). Ako ima grešaka, ne idi dalje.

---

# KORAK 7 – Migracija

PMC, Default project = `eCommerce.Services`:

```
Add-Migration AddPaymentCardsIBXXXXXX
Update-Database
```

U SSMS-u moraš vidjeti novu tabelu i novu kolonu na `Orders`.

---

# KORAK 8 – NOVI InsertRequest

**Fajl:** `eCommerce/eCommerce.Model/Requests/PaymentCardIBXXXXXXInsertRequest.cs`

```csharp
namespace eCommerce.Model.Requests
{
    public class PaymentCardIBXXXXXXInsertRequest
    {
        public string CardNumber { get; set; } = string.Empty;
        public string Cvc { get; set; } = string.Empty;
        public DateTime ExpiryDate { get; set; }
        public decimal InitialBalance { get; set; }
    }
}
```

---

# KORAK 9 – NOVI UpdateRequest

**Fajl:** `eCommerce/eCommerce.Model/Requests/PaymentCardIBXXXXXXUpdateRequest.cs`

```csharp
namespace eCommerce.Model.Requests
{
    public class PaymentCardIBXXXXXXUpdateRequest
    {
        public string CardNumber { get; set; } = string.Empty;
        public string Cvc { get; set; } = string.Empty;
        public DateTime ExpiryDate { get; set; }
        public decimal InitialBalance { get; set; }
    }
}
```

---

# KORAK 10 – NOVI Response

**Fajl:** `eCommerce/eCommerce.Model/Responses/PaymentCardIBXXXXXXResponse.cs`

```csharp
namespace eCommerce.Model.Responses
{
    public class PaymentCardIBXXXXXXResponse
    {
        public int Id { get; set; }
        public int UserId { get; set; }
        public string CardNumber { get; set; } = string.Empty;
        public string Cvc { get; set; } = string.Empty;
        public DateTime ExpiryDate { get; set; }
        public decimal InitialBalance { get; set; }
        public decimal AvailableBalance { get; set; }
        public DateTime CreatedAt { get; set; }
        public DateTime? UpdatedAt { get; set; }
        public List<PaymentCardTransactionResponse> Transactions { get; set; } = new();
    }

    public class PaymentCardTransactionResponse
    {
        public DateTime Date { get; set; }
        public decimal Amount { get; set; }
    }
}
```

---

# KORAK 11 – NOVI SearchObject

**Fajl:** `eCommerce/eCommerce.Model/SearchObjects/PaymentCardIBXXXXXXSearch.cs`

```csharp
namespace eCommerce.Model.SearchObjects
{
    public class PaymentCardIBXXXXXXSearch : BaseSearchObject
    {
        public int? UserId { get; set; }
    }
}
```

---

# KORAK 12 – NOVI InsertValidator

**Fajl:** `eCommerce/eCommerce.Services/Validators/PaymentCardIBXXXXXXInsertValidator.cs`

```csharp
using eCommerce.Model.Requests;
using FluentValidation;

namespace eCommerce.Services.Validators
{
    public class PaymentCardIBXXXXXXInsertValidator : AbstractValidator<PaymentCardIBXXXXXXInsertRequest>
    {
        public PaymentCardIBXXXXXXInsertValidator()
        {
            RuleFor(x => x.CardNumber)
                .NotEmpty().WithMessage("Card number is required.")
                .Matches("^[0-9]{12}$").WithMessage("Card number must be exactly 12 digits.");

            RuleFor(x => x.Cvc)
                .NotEmpty().WithMessage("CVC is required.")
                .Matches("^[0-9]{3}$").WithMessage("CVC must be exactly 3 digits.");

            RuleFor(x => x.ExpiryDate)
                .Must(d => d > DateTime.UtcNow)
                .WithMessage("Card is expired.");

            RuleFor(x => x.InitialBalance)
                .GreaterThanOrEqualTo(0)
                .WithMessage("Initial balance must be 0 or greater.");
        }
    }
}
```

---

# KORAK 13 – NOVI UpdateValidator

**Fajl:** `eCommerce/eCommerce.Services/Validators/PaymentCardIBXXXXXXUpdateValidator.cs`

```csharp
using eCommerce.Model.Requests;
using FluentValidation;

namespace eCommerce.Services.Validators
{
    public class PaymentCardIBXXXXXXUpdateValidator : AbstractValidator<PaymentCardIBXXXXXXUpdateRequest>
    {
        public PaymentCardIBXXXXXXUpdateValidator()
        {
            RuleFor(x => x.CardNumber)
                .NotEmpty().WithMessage("Card number is required.")
                .Matches("^[0-9]{12}$").WithMessage("Card number must be exactly 12 digits.");

            RuleFor(x => x.Cvc)
                .NotEmpty().WithMessage("CVC is required.")
                .Matches("^[0-9]{3}$").WithMessage("CVC must be exactly 3 digits.");

            RuleFor(x => x.ExpiryDate)
                .Must(d => d > DateTime.UtcNow)
                .WithMessage("Card is expired.");

            RuleFor(x => x.InitialBalance)
                .GreaterThanOrEqualTo(0)
                .WithMessage("Initial balance must be 0 or greater.");
        }
    }
}
```

---

# KORAK 14 – NOVI interface

**Fajl:** `eCommerce/eCommerce.Services/IPaymentCardIBXXXXXXService.cs`

```csharp
using eCommerce.Model.Requests;
using eCommerce.Model.Responses;
using eCommerce.Model.SearchObjects;

namespace eCommerce.Services
{
    public interface IPaymentCardIBXXXXXXService
        : IBaseCRUDService<
            PaymentCardIBXXXXXXResponse,
            PaymentCardIBXXXXXXSearch,
            PaymentCardIBXXXXXXInsertRequest,
            PaymentCardIBXXXXXXUpdateRequest>
    {
    }
}
```

---

# KORAK 15 – NOVI servis (cijeli fajl)

**Fajl:** `eCommerce/eCommerce.Services/PaymentCardIBXXXXXXService.cs`

```csharp
using eCommerce.Model.Requests;
using eCommerce.Model.Responses;
using eCommerce.Model.SearchObjects;
using eCommerce.Services.Database;
using FluentValidation;
using MapsterMapper;
using Microsoft.EntityFrameworkCore;

namespace eCommerce.Services
{
    public class PaymentCardIBXXXXXXService
        : BaseCRUDService<
            PaymentCardIBXXXXXX,
            PaymentCardIBXXXXXXResponse,
            PaymentCardIBXXXXXXSearch,
            PaymentCardIBXXXXXXInsertRequest,
            PaymentCardIBXXXXXXUpdateRequest>,
          IPaymentCardIBXXXXXXService
    {
        private readonly IAuthenticatedUserAccessor _userAccessor;

        public PaymentCardIBXXXXXXService(
            ECommerceDbContext dbContext,
            IMapper mapper,
            IValidator<PaymentCardIBXXXXXXInsertRequest> insertValidator,
            IValidator<PaymentCardIBXXXXXXUpdateRequest> updateValidator,
            IAuthenticatedUserAccessor userAccessor)
            : base(dbContext, mapper, insertValidator, updateValidator)
        {
            _userAccessor = userAccessor;
        }

        private int RequireUserId()
        {
            return _userAccessor.GetUserId()
                   ?? throw new InvalidOperationException("User id claim is missing.");
        }

        protected override IEnumerable<PaymentCardIBXXXXXX> ApplyFilters(
            IEnumerable<PaymentCardIBXXXXXX> query,
            PaymentCardIBXXXXXXSearch? search)
        {
            var userId = _userAccessor.GetUserId();
            if (!userId.HasValue)
            {
                return Enumerable.Empty<PaymentCardIBXXXXXX>();
            }

            return query.Where(c => c.UserId == userId.Value);
        }

        protected override PaymentCardIBXXXXXX MapInsertRequestToEntity(
            PaymentCardIBXXXXXXInsertRequest request)
        {
            var entity = base.MapInsertRequestToEntity(request);
            entity.UserId = RequireUserId();
            return entity;
        }

        public override async Task<PageResult<PaymentCardIBXXXXXXResponse>> GetAllAsync(
            PaymentCardIBXXXXXXSearch? search = null)
        {
            search ??= new PaymentCardIBXXXXXXSearch();
            search.PageSize = 50;

            var page = await base.GetAllAsync(search);

            foreach (var item in page.Items)
            {
                item.AvailableBalance = await CalculateAvailableAsync(item.Id, item.InitialBalance);
            }

            return page;
        }

        public override async Task<PaymentCardIBXXXXXXResponse> GetByIdAsync(int id)
        {
            var userId = RequireUserId();

            var entity = await _dbContext.Set<PaymentCardIBXXXXXX>()
                .AsNoTracking()
                .Include(c => c.Orders)
                .FirstOrDefaultAsync(c => c.Id == id && c.UserId == userId);

            if (entity == null)
            {
                throw new KeyNotFoundException($"PaymentCard with id {id} not found.");
            }

            return ToResponse(entity);
        }

        public override async Task<PaymentCardIBXXXXXXResponse> UpdateAsync(
            int id,
            PaymentCardIBXXXXXXUpdateRequest request)
        {
            var userId = RequireUserId();
            var entity = await _dbContext.Set<PaymentCardIBXXXXXX>().FindAsync(id);

            if (entity == null || entity.UserId != userId)
            {
                throw new KeyNotFoundException($"PaymentCard with id {id} not found.");
            }

            return await base.UpdateAsync(id, request);
        }

        private PaymentCardIBXXXXXXResponse ToResponse(PaymentCardIBXXXXXX entity)
        {
            var response = _mapper.Map<PaymentCardIBXXXXXXResponse>(entity);

            var successful = entity.Orders
                .Where(o => o.Status != OrderStatus.Cancelled)
                .OrderByDescending(o => o.OrderDate)
                .ToList();

            response.AvailableBalance = entity.InitialBalance - successful.Sum(o => o.TotalAmount);
            response.Transactions = successful.Select(o => new PaymentCardTransactionResponse
            {
                Date = o.OrderDate,
                Amount = o.TotalAmount
            }).ToList();

            return response;
        }

        private async Task<decimal> CalculateAvailableAsync(int cardId, decimal initial)
        {
            var spent = await _dbContext.Orders
                .Where(o => o.PaymentCardIBXXXXXXId == cardId && o.Status != OrderStatus.Cancelled)
                .SumAsync(o => (decimal?)o.TotalAmount) ?? 0;

            return initial - spent;
        }
    }
}
```

---

# KORAK 16 – CIJELI `Program.cs`

**Fajl:** `eCommerce/eCommerce.WebAPI/Program.cs`

Zamijeni cijeli fajl ovim. Nove su samo 3 linije `AddScoped` za karticu.

```csharp
using eCommerce.Common.Services.CryptoService;
using eCommerce.Model.Requests;
using eCommerce.Model.Responses;
using eCommerce.Services;
using eCommerce.Services.Database;
using eCommerce.Services.ProductStateMachine;
using eCommerce.Services.Validators;
using eCommerce.WebAPI.Filters;
using eCommerce.WebAPI.Services;
using eCommerce.WebAPI.Services.AccessManager;
using FluentValidation;
using Mapster;
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.AspNetCore.Mvc.Filters;
using Microsoft.EntityFrameworkCore;
using Microsoft.IdentityModel.Tokens;
using Microsoft.OpenApi.Models;
using Scalar.AspNetCore;
using System.Text;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddHttpContextAccessor();
builder.Services.AddScoped<IAuthenticatedUserAccessor, HttpAuthenticatedUserAccessor>();

builder.Services.AddControllers(
   options => options.Filters.Add<ExceptionFilter>()
);

var connectionString = builder.Configuration.GetConnectionString("DefaultConnection");
builder.Services.AddDbContext<ECommerceDbContext>(options =>
    options.UseSqlServer(connectionString)
);

builder.Services.AddMapster();

TypeAdapterConfig<Product, ProductResponse>.NewConfig().IgnoreNullValues(true);
TypeAdapterConfig<Category, CategoryResponse>.NewConfig().IgnoreNullValues(true);
TypeAdapterConfig<User, UserResponse>.NewConfig().IgnoreNullValues(true);
TypeAdapterConfig<UserUpdateRequest, User>.NewConfig().IgnoreNullValues(true);
TypeAdapterConfig<ProductType, ProductTypeResponse>.NewConfig().IgnoreNullValues(true);
TypeAdapterConfig<UnitOfMeasure, UnitOfMeasureResponse>.NewConfig().IgnoreNullValues(true);
TypeAdapterConfig<Asset, AssetResponse>.NewConfig().IgnoreNullValues(true);
TypeAdapterConfig<ProductReview, ProductReviewResponse>.NewConfig()
    .Map(dest => dest.ReviewerDisplayName, src => $"{src.User.FirstName} {src.User.LastName}".Trim());
TypeAdapterConfig<Order, OrderResponse>.NewConfig()
    .Map(dest => dest.Status, src => (int)src.Status);
TypeAdapterConfig<OrderItem, OrderItemResponse>.NewConfig()
    .Map(dest => dest.ProductName, src => src.Product != null ? src.Product.Name : string.Empty);

builder.Services.AddScoped<IProductService, ProductService>();
builder.Services.AddScoped<BaseProductState>();
builder.Services.AddScoped<InitialProductState>();
builder.Services.AddScoped<DraftProductState>();
builder.Services.AddScoped<ActiveProductState>();

builder.Services.AddScoped<ICategoryService, CategoryService>();
builder.Services.AddScoped<IProductTypeService, ProductTypeService>();
builder.Services.AddScoped<IUnitOfMeasureService, UnitOfMeasureService>();
builder.Services.AddScoped<IUserService, UserService>();
builder.Services.AddScoped<IAssetService, AssetService>();
builder.Services.AddScoped<IRefreshTokenService, RefreshTokenService>();
builder.Services.AddScoped<IAccessManager, AccessManager>();
builder.Services.AddScoped<ICryptoService, CryptoService>();
builder.Services.AddScoped<IOrderService, OrderService>();
builder.Services.AddScoped<IProductReviewService, ProductReviewService>();
builder.Services.AddScoped<IPaymentCardIBXXXXXXService, PaymentCardIBXXXXXXService>();

builder.Services.AddScoped<IValidator<ProductTypeInsertRequest>, ProductTypeInsertValidator>();
builder.Services.AddScoped<IValidator<ProductTypeUpdateRequest>, ProductTypeUpdateValidator>();
builder.Services.AddScoped<IValidator<UnitOfMeasureInsertRequest>, UnitOfMeasureInsertValidator>();
builder.Services.AddScoped<IValidator<UnitOfMeasureUpdateRequest>, UnitOfMeasureUpdateValidator>();
builder.Services.AddScoped<IValidator<CategoriesInsertRequest>, CategoryInsertValidator>();
builder.Services.AddScoped<IValidator<CategoriesUpdateRequest>, CategoryUpdateValidator>();
builder.Services.AddScoped<IValidator<UserInsertRequest>, UserInsertValidator>();
builder.Services.AddScoped<IValidator<UserUpdateRequest>, UserUpdateValidator>();
builder.Services.AddScoped<IValidator<AssetInsertRequest>, AssetInsertValidator>();
builder.Services.AddScoped<IValidator<AssetUpdateRequest>, AssetUpdateValidator>();
builder.Services.AddScoped<IValidator<ProductReviewInsertRequest>, ProductReviewInsertValidator>();
builder.Services.AddScoped<IValidator<ProductReviewUpdateRequest>, ProductReviewUpdateValidator>();
builder.Services.AddScoped<IValidator<PaymentCardIBXXXXXXInsertRequest>, PaymentCardIBXXXXXXInsertValidator>();
builder.Services.AddScoped<IValidator<PaymentCardIBXXXXXXUpdateRequest>, PaymentCardIBXXXXXXUpdateValidator>();

builder.Services.AddOpenApi();

builder.Services.AddAuthentication(options =>
{
    options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
    options.DefaultScheme = JwtBearerDefaults.AuthenticationScheme;
}).AddJwtBearer(o =>
{
    o.TokenValidationParameters = new TokenValidationParameters
    {
        ValidIssuer = builder.Configuration["JwtToken:Issuer"],
        ValidAudience = builder.Configuration["JwtToken:Audience"],
        IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(builder.Configuration["JwtToken:SecretKey"] ?? string.Empty)),
        ValidateIssuer = true,
        ValidateAudience = true,
        ValidateLifetime = true,
        ValidateIssuerSigningKey = true,
        ClockSkew = TimeSpan.Zero
    };
});
builder.Services.AddAuthorization();

builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen(
    options =>
    {
        options.SwaggerDoc("v1", new Microsoft.OpenApi.Models.OpenApiInfo
        {
            Version = "v1",
            Title = "eCommerce API",
            Description = "API for managing products and categories in the eCommerce application"
        });

        var xmlFile = $"{System.Reflection.Assembly.GetExecutingAssembly().GetName().Name}.xml";
        options.IncludeXmlComments(Path.Combine(AppContext.BaseDirectory, xmlFile));

        var jwtSecurityScheme = new OpenApiSecurityScheme
        {
            BearerFormat = "JWT",
            Name = "JWT Authentication",
            In = ParameterLocation.Header,
            Type = SecuritySchemeType.Http,
            Scheme = JwtBearerDefaults.AuthenticationScheme,
            Reference = new OpenApiReference
            {
                Id = JwtBearerDefaults.AuthenticationScheme,
                Type = ReferenceType.SecurityScheme
            }
        };

        options.AddSecurityDefinition(jwtSecurityScheme.Reference.Id, jwtSecurityScheme);
        options.AddSecurityRequirement(new OpenApiSecurityRequirement
                {
                    { jwtSecurityScheme, Array.Empty<string>() }
                });
    });

var app = builder.Build();

{
    app.MapOpenApi();
    app.MapScalarApiReference();
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();
app.Run();
```

---

# KORAK 17 – NOVI kontroler

**Fajl:** `eCommerce/eCommerce.WebAPI/Controllers/PaymentCardIBXXXXXXController.cs`

```csharp
using eCommerce.Model.Requests;
using eCommerce.Model.Responses;
using eCommerce.Model.SearchObjects;
using eCommerce.Services;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;

namespace eCommerce.WebAPI.Controllers;

[Authorize]
public class PaymentCardIBXXXXXXController
    : BaseCRUDController<
        PaymentCardIBXXXXXXResponse,
        PaymentCardIBXXXXXXSearch,
        PaymentCardIBXXXXXXInsertRequest,
        PaymentCardIBXXXXXXUpdateRequest,
        IPaymentCardIBXXXXXXService>
{
    public PaymentCardIBXXXXXXController(IPaymentCardIBXXXXXXService service)
        : base(service)
    {
    }
}
```

---

# KORAK 18 – CIJELI `CheckoutRequest.cs`

**Fajl:** `eCommerce/eCommerce.Model/Requests/CheckoutRequest.cs`

```csharp
namespace eCommerce.Model.Requests;

public class CheckoutRequest
{
    public List<CheckoutLineRequest> Items { get; set; } = new();

    public string? ShippingAddress { get; set; }
    public string? ShippingCity { get; set; }
    public string? ShippingState { get; set; }
    public string? ShippingZipCode { get; set; }
    public string? ShippingCountry { get; set; }

    public int PaymentCardIBXXXXXXId { get; set; }
}
```

---

# KORAK 19 – CIJELI `OrderService.cs`

**Fajl:** `eCommerce/eCommerce.Services/OrderService.cs`

```csharp
using eCommerce.Model.Exceptions;
using eCommerce.Model.Requests;
using eCommerce.Model.Responses;
using eCommerce.Model.SearchObjects;
using eCommerce.Services.Database;
using MapsterMapper;
using Microsoft.EntityFrameworkCore;

namespace eCommerce.Services;

public class OrderService : BaseReadService<Order, OrderResponse, OrderSearchObject>, IOrderService
{
    private readonly IAuthenticatedUserAccessor _userAccessor;

    public OrderService(ECommerceDbContext dbContext, IMapper mapper, IAuthenticatedUserAccessor userAccessor)
        : base(mapper, dbContext)
    {
        _userAccessor = userAccessor;
    }

    public override async Task<PageResult<OrderResponse>> GetAllAsync(OrderSearchObject? search = null)
    {
        search ??= new OrderSearchObject();
        if (string.IsNullOrWhiteSpace(search.SortBy))
        {
            search.SortBy = "OrderDate desc";
        }

        return await base.GetAllAsync(search);
    }

    protected override async Task<IQueryable<Order>> IncludeRelatedEntitiesAsync(
        OrderSearchObject? search,
        IQueryable<Order> query = null!)
    {
        return await Task.FromResult(query.Include(o => o.OrderItems).ThenInclude(oi => oi.Product));
    }

    protected override IEnumerable<Order> ApplyFilters(IEnumerable<Order> query, OrderSearchObject? search)
    {
        var userId = _userAccessor.GetUserId();
        if (!userId.HasValue)
        {
            return Enumerable.Empty<Order>();
        }

        query = query.Where(o => o.UserId == userId.Value);

        if (search?.Status.HasValue == true)
        {
            query = query.Where(o => (int)o.Status == search.Status.Value);
        }

        return query;
    }

    public override async Task<OrderResponse> GetByIdAsync(int id)
    {
        var userId = _userAccessor.GetUserId();
        if (!userId.HasValue)
        {
            throw new KeyNotFoundException($"{typeof(Order).Name} with id {id} not found.");
        }

        var order = await _dbContext.Orders
            .AsNoTracking()
            .Include(o => o.OrderItems)
            .ThenInclude(oi => oi.Product)
            .FirstOrDefaultAsync(o => o.Id == id && o.UserId == userId.Value);

        if (order == null)
        {
            throw new KeyNotFoundException($"{typeof(Order).Name} with id {id} not found.");
        }

        return _mapper.Map<OrderResponse>(order);
    }

    public async Task<OrderResponse> CheckoutAsync(CheckoutRequest request)
    {
        var userId = _userAccessor.GetUserId()
                     ?? throw new InvalidOperationException("User id claim is missing.");

        if (request.Items == null || request.Items.Count == 0)
        {
            throw new ClinetException("Cart is empty.");
        }

        if (request.PaymentCardIBXXXXXXId <= 0)
        {
            throw new ClinetException("Please select a payment card.");
        }

        var card = await _dbContext.Set<PaymentCardIBXXXXXX>()
            .FirstOrDefaultAsync(c => c.Id == request.PaymentCardIBXXXXXXId && c.UserId == userId);

        if (card == null)
        {
            throw new ClinetException("Payment card was not found.");
        }

        if (card.ExpiryDate <= DateTime.UtcNow)
        {
            throw new ClinetException("The selected card is expired.");
        }

        await using var tx = await _dbContext.Database.BeginTransactionAsync();

        try
        {
            var merged = request.Items
                .Where(i => i.Quantity > 0)
                .GroupBy(i => i.ProductId)
                .Select(g => new { ProductId = g.Key, Quantity = g.Sum(x => x.Quantity) })
                .ToList();

            decimal total = 0;
            var order = new Order
            {
                UserId = userId,
                OrderDate = DateTime.UtcNow,
                OrderNumber = $"O-{DateTime.UtcNow:yyyyMMddHHmmss}-{userId}",
                Status = OrderStatus.Processing,
                ShippingAddress = OrDash(request.ShippingAddress),
                ShippingCity = OrDash(request.ShippingCity),
                ShippingState = OrDash(request.ShippingState),
                ShippingZipCode = OrDash(request.ShippingZipCode),
                ShippingCountry = OrDash(request.ShippingCountry),
                PaymentCardIBXXXXXXId = card.Id,
                PaymentDate = DateTime.UtcNow,
            };

            foreach (var line in merged)
            {
                var product = await _dbContext.Products.FindAsync(line.ProductId);
                if (product == null)
                {
                    throw new ClinetException($"Product {line.ProductId} was not found.");
                }

                if (!product.IsActive)
                {
                    throw new ClinetException($"Product '{product.Name}' is not available.");
                }

                if (product.StockQuantity < line.Quantity)
                {
                    throw new ClinetException($"Insufficient stock for '{product.Name}'.");
                }

                var unitPrice = product.Price;
                total += unitPrice * line.Quantity;
                product.StockQuantity -= line.Quantity;

                order.OrderItems.Add(new OrderItem
                {
                    ProductId = product.Id,
                    Quantity = line.Quantity,
                    UnitPrice = unitPrice,
                });
            }

            var spent = await _dbContext.Orders
                .Where(o => o.PaymentCardIBXXXXXXId == card.Id && o.Status != OrderStatus.Cancelled)
                .SumAsync(o => (decimal?)o.TotalAmount) ?? 0;

            var available = card.InitialBalance - spent;

            if (available < total)
            {
                throw new ClinetException(
                    $"Insufficient funds. Available: {available:0.00}, order: {total:0.00}.");
            }

            order.TotalAmount = total;
            _dbContext.Orders.Add(order);
            await _dbContext.SaveChangesAsync();
            await tx.CommitAsync();

            return await GetByIdAsync(order.Id);
        }
        catch
        {
            await tx.RollbackAsync();
            throw;
        }
    }

    private static string OrDash(string? value) =>
        string.IsNullOrWhiteSpace(value) ? "—" : value.Trim();
}
```

Build. Pokreni API. Swagger:

1. `POST /Access/Login` → `{ "username": "customer1", "password": "Test123" }`
2. Authorize (lock) → `Bearer TOKEN`
3. `POST /PaymentCardIBXXXXXX`

```json
{
  "cardNumber": "123456789012",
  "cvc": "123",
  "expiryDate": "2027-12-31T00:00:00Z",
  "initialBalance": 200
}
```

Tek sada Flutter.

---

# KORAK 20 – NOVI Flutter model

**Fajl:** `eCommerce/UI/ecommerce_mobile/lib/models/payment_card.dart`

```dart
import 'package:json_annotation/json_annotation.dart';

part 'payment_card.g.dart';

@JsonSerializable()
class PaymentCardTransaction {
  final DateTime? date;
  final double? amount;

  PaymentCardTransaction({this.date, this.amount});

  factory PaymentCardTransaction.fromJson(Map<String, dynamic> json) =>
      _$PaymentCardTransactionFromJson(json);

  Map<String, dynamic> toJson() => _$PaymentCardTransactionToJson(this);
}

@JsonSerializable()
class PaymentCard {
  final int? id;
  final int? userId;
  final String? cardNumber;
  final String? cvc;
  final DateTime? expiryDate;
  final double? initialBalance;
  final double? availableBalance;
  final List<PaymentCardTransaction> transactions;

  PaymentCard({
    this.id,
    this.userId,
    this.cardNumber,
    this.cvc,
    this.expiryDate,
    this.initialBalance,
    this.availableBalance,
    this.transactions = const [],
  });

  factory PaymentCard.fromJson(Map<String, dynamic> json) =>
      _$PaymentCardFromJson(json);

  Map<String, dynamic> toJson() => _$PaymentCardToJson(this);
}
```

---

# KORAK 21 – Generiši `.g.dart`

Terminal, folder `eCommerce/UI/ecommerce_mobile`:

```
dart run build_runner build --delete-conflicting-outputs
```

Mora nastati `lib/models/payment_card.g.dart`.

---

# KORAK 22 – NOVI Flutter provider

**Fajl:** `eCommerce/UI/ecommerce_mobile/lib/providers/payment_card_provider.dart`

```dart
import 'package:ecommerce_mobile/models/payment_card.dart';
import 'package:ecommerce_mobile/providers/base_provider.dart';

class PaymentCardProvider extends BaseProvider<PaymentCard> {
  PaymentCardProvider() : super('PaymentCardIBXXXXXX');

  @override
  PaymentCard fromJson(data) =>
      PaymentCard.fromJson(data as Map<String, dynamic>);
}
```

---

# KORAK 23 – `main.dart` (samo vrh fajla zamijeni)

**Fajl:** `eCommerce/UI/ecommerce_mobile/lib/main.dart`

Zamijeni **samo** import i `void main()` ovim. `MyApp`, `LoginPage` i ostalo ostaju.

```dart
import 'package:ecommerce_mobile/layouts/container_screen.dart';
import 'package:ecommerce_mobile/providers/auth_provider.dart';
import 'package:ecommerce_mobile/providers/cart_provider.dart';
import 'package:ecommerce_mobile/providers/category_provider.dart';
import 'package:ecommerce_mobile/providers/order_provider.dart';
import 'package:ecommerce_mobile/providers/payment_card_provider.dart';
import 'package:ecommerce_mobile/providers/product_provider.dart';
import 'package:ecommerce_mobile/providers/product_review_provider.dart';
import 'package:ecommerce_mobile/providers/user_provider.dart';
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

void main() {
  runApp(
    MultiProvider(
      providers: [
        ChangeNotifierProvider(create: (_) => AuthProvider()),
        ChangeNotifierProvider(create: (_) => ProductProvider()),
        ChangeNotifierProvider(create: (_) => CartProvider()),
        ChangeNotifierProvider(create: (_) => CategoryProvider()),
        ChangeNotifierProvider(create: (_) => UserProvider()),
        ChangeNotifierProvider(create: (_) => OrderProvider()),
        ChangeNotifierProvider(create: (_) => ProductReviewProvider()),
        ChangeNotifierProvider(create: (_) => PaymentCardProvider()),
      ],
      child: const MyApp(),
    ),
  );
}
```

Nova linija je `PaymentCardProvider` + njegov import.

---

# KORAK 24 – CIJELI `profile_screen.dart`

**Fajl:** `eCommerce/UI/ecommerce_mobile/lib/screens/profile_screen.dart`

Zamijeni cijeli fajl:

```dart
import 'package:ecommerce_mobile/providers/auth_provider.dart';
import 'package:ecommerce_mobile/screens/change_password_screen.dart';
import 'package:ecommerce_mobile/screens/orders_list_screen.dart';
import 'package:ecommerce_mobile/screens/payment_card_details.dart';
import 'package:ecommerce_mobile/screens/profile_settings_screen.dart';
import 'package:ecommerce_mobile/screens/user_review_screen.dart';
import 'package:ecommerce_mobile/utils/utils_widgets.dart';
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

import '../main.dart' hide alertBox;
import '../models/payment_card.dart';
import '../models/user.dart';
import '../providers/payment_card_provider.dart';
import '../providers/user_provider.dart';

class ProfileScreen extends StatefulWidget {
  const ProfileScreen({super.key});

  @override
  State<ProfileScreen> createState() => _ProfileScreenState();
}

class _ProfileScreenState extends State<ProfileScreen> {
  late UserProvider _userProvider;
  late PaymentCardProvider _cardProvider;

  late User user;
  List<PaymentCard> _cards = [];

  bool isLoading = true;

  @override
  void initState() {
    super.initState();
    _userProvider = context.read<UserProvider>();
    _cardProvider = context.read<PaymentCardProvider>();
    initData();
  }

  Future<void> initData() async {
    try {
      var result = await _userProvider.getById(
        int.tryParse(AuthProvider.accessTokenDecoded?['Id'] ?? '0') ?? 0,
      );
      var cards = await _cardProvider.get(filter: {'page': 1, 'pageSize': 50});

      setState(() {
        user = result;
        _cards = cards.items ?? [];
        isLoading = false;
      });
    } on Exception catch (e) {
      alertBox(context, 'Error', e.toString());
    }
  }

  @override
  Widget build(BuildContext context) {
    return SafeArea(
      child: SingleChildScrollView(
        child: isLoading
            ? const CircularProgressIndicator()
            : Column(
                children: [
                  const SizedBox(height: 40),
                  _buildProfileInfo(),
                  const SizedBox(height: 24),
                  _buildMyCards(),
                  const SizedBox(height: 24),
                  _buildProfileMenu(),
                ],
              ),
      ),
    );
  }

  Widget _buildMyCards() {
    return Padding(
      padding: const EdgeInsets.fromLTRB(26, 0, 26, 0),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.stretch,
        children: [
          Row(
            mainAxisAlignment: MainAxisAlignment.spaceBetween,
            children: [
              const Text(
                'Moje kartice',
                style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
              ),
              TextButton(
                onPressed: () async {
                  final refresh = await Navigator.push(
                    context,
                    MaterialPageRoute(
                      builder: (_) => const PaymentCardDetailsScreen(),
                    ),
                  );
                  if (refresh == 'reload') {
                    initData();
                  }
                },
                child: const Text('Dodaj karticu'),
              ),
            ],
          ),
          if (_cards.isEmpty)
            const Text('Nemate sačuvanih kartica.')
          else
            ..._cards.map((card) {
              return Card(
                child: ListTile(
                  title: Text(card.cardNumber ?? ''),
                  subtitle: Text(
                    'Istek: ${card.expiryDate ?? ''}   '
                    'Početno: ${card.initialBalance ?? 0}   '
                    'Dostupno: ${card.availableBalance ?? 0}',
                  ),
                  onTap: () async {
                    final refresh = await Navigator.push(
                      context,
                      MaterialPageRoute(
                        builder: (_) => PaymentCardDetailsScreen(card: card),
                      ),
                    );
                    if (refresh == 'reload') {
                      initData();
                    }
                  },
                ),
              );
            }),
        ],
      ),
    );
  }

  Row _buildProfileInfo() {
    return Row(
      mainAxisAlignment: MainAxisAlignment.center,
      children: [
        Column(
          children: [
            CircleAvatar(
              backgroundImage: user.profileImageBase64 != null
                  ? ImageFromBase64StringWithoutDimnesions(
                      user.profileImageBase64!,
                    )
                  : const AssetImage("assets/images/no_profile.png"),
              radius: 70,
            ),
            const SizedBox(height: 20),
            Text(
              "${user.firstName} ${user.lastName}",
              style: const TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
            ),
            Text(
              user.username ?? "No username",
              style: TextStyle(fontSize: 16, color: Colors.grey[600]),
            ),
          ],
        ),
      ],
    );
  }

  Padding _buildProfileMenu() {
    return Padding(
      padding: const EdgeInsets.fromLTRB(26.0, 0, 26.0, 0),
      child: Card(
        elevation: 10,
        child: ListView(
          shrinkWrap: true,
          physics: const NeverScrollableScrollPhysics(),
          children: [
            ListTile(
              leading: const Icon(Icons.shopping_bag),
              title: const Text("Your orders"),
              onTap: () {
                Navigator.push(
                  context,
                  MaterialPageRoute(
                    builder: (context) => const OrdersListScreen(),
                  ),
                );
              },
            ),
            ListTile(
              leading: const Icon(Icons.reviews),
              title: const Text("Your reviews"),
              onTap: () {
                Navigator.push(
                  context,
                  MaterialPageRoute(
                    builder: (context) => const UserReviewScreen(),
                  ),
                );
              },
            ),
            ListTile(
              leading: const Icon(Icons.pending),
              title: const Text("Edit profile"),
              onTap: () async {
                var refresh = await Navigator.push(
                  context,
                  MaterialPageRoute(
                    builder: (context) => ProfileSettingsScreen(user: user),
                  ),
                );

                if (refresh == 'reload') {
                  initData();
                }
              },
            ),
            ListTile(
              leading: const Icon(Icons.lock_outline),
              title: const Text("Change password"),
              onTap: () {
                Navigator.push(
                  context,
                  MaterialPageRoute(
                    builder: (context) => ChangePasswordScreen(),
                  ),
                );
              },
            ),
            ListTile(
              leading: const Icon(Icons.logout),
              title: const Text("Log out"),
              onTap: () async {
                final leave = await showDialog<bool>(
                  context: context,
                  builder: (ctx) => AlertDialog(
                    title: const Text("Log out"),
                    content: const Text("Are you sure you want to log out?"),
                    actions: [
                      TextButton(
                        onPressed: () => Navigator.pop(ctx, false),
                        child: const Text("Cancel"),
                      ),
                      TextButton(
                        onPressed: () => Navigator.pop(ctx, true),
                        child: const Text("Log out"),
                      ),
                    ],
                  ),
                );
                if (leave != true || !mounted) return;
                context.read<AuthProvider>().logout();
                if (!mounted) return;
                Navigator.of(context).pushAndRemoveUntil(
                  MaterialPageRoute(builder: (context) => LoginPage()),
                  (route) => route.isFirst,
                );
              },
            ),
          ],
        ),
      ),
    );
  }
}
```

---

# KORAK 25 – NOVI `payment_card_details.dart` (ime iz zadatka)

**Fajl:** `eCommerce/UI/ecommerce_mobile/lib/screens/payment_card_details.dart`

```dart
import 'package:ecommerce_mobile/models/payment_card.dart';
import 'package:ecommerce_mobile/providers/payment_card_provider.dart';
import 'package:ecommerce_mobile/utils/api_client_exception.dart';
import 'package:ecommerce_mobile/utils/utils_widgets.dart';
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

class PaymentCardDetailsScreen extends StatefulWidget {
  final PaymentCard? card;

  const PaymentCardDetailsScreen({super.key, this.card});

  @override
  State<PaymentCardDetailsScreen> createState() =>
      _PaymentCardDetailsScreenState();
}

class _PaymentCardDetailsScreenState extends State<PaymentCardDetailsScreen> {
  final _number = TextEditingController();
  final _cvc = TextEditingController();
  final _expiry = TextEditingController();
  final _balance = TextEditingController();
  bool _saving = false;
  PaymentCard? _loaded;

  bool get _isEdit => widget.card?.id != null;

  @override
  void initState() {
    super.initState();
    final c = widget.card;
    if (c != null) {
      _number.text = c.cardNumber ?? '';
      _cvc.text = c.cvc ?? '';
      _expiry.text = c.expiryDate?.toIso8601String().split('T').first ?? '';
      _balance.text = c.initialBalance?.toString() ?? '';
    }
    if (_isEdit) {
      _loadDetails();
    }
  }

  Future<void> _loadDetails() async {
    try {
      final full = await context.read<PaymentCardProvider>().getById(
        widget.card!.id!,
      );
      setState(() {
        _loaded = full;
        _number.text = full.cardNumber ?? _number.text;
        _cvc.text = full.cvc ?? _cvc.text;
        _expiry.text =
            full.expiryDate?.toIso8601String().split('T').first ?? _expiry.text;
        _balance.text = full.initialBalance?.toString() ?? _balance.text;
      });
    } on Exception catch (e) {
      if (mounted) alertBox(context, 'Error', e.toString());
    }
  }

  @override
  void dispose() {
    _number.dispose();
    _cvc.dispose();
    _expiry.dispose();
    _balance.dispose();
    super.dispose();
  }

  String? _localError() {
    if (!RegExp(r'^[0-9]{12}$').hasMatch(_number.text)) {
      return 'Broj kartice mora imati tačno 12 cifara.';
    }
    if (!RegExp(r'^[0-9]{3}$').hasMatch(_cvc.text)) {
      return 'CVC mora imati tačno 3 cifre.';
    }
    final expiry = DateTime.tryParse(_expiry.text);
    if (expiry == null) {
      return 'Unesi datum isteka (npr. 2027-12-31).';
    }
    if (!expiry.isAfter(DateTime.now())) {
      return 'Kartica je istekla.';
    }
    final balance = double.tryParse(_balance.text.replaceAll(',', '.'));
    if (balance == null || balance < 0) {
      return 'Početno stanje mora biti broj ≥ 0.';
    }
    return null;
  }

  Map<String, dynamic> _body() {
    final expiry = DateTime.parse(_expiry.text);
    return {
      'cardNumber': _number.text,
      'cvc': _cvc.text,
      'expiryDate': expiry.toUtc().toIso8601String(),
      'initialBalance': double.parse(_balance.text.replaceAll(',', '.')),
    };
  }

  Future<void> _save() async {
    final err = _localError();
    if (err != null) {
      alertBox(context, 'Validacija', err);
      return;
    }

    setState(() => _saving = true);
    try {
      final provider = context.read<PaymentCardProvider>();
      if (_isEdit) {
        await provider.update(widget.card!.id!, _body());
      } else {
        await provider.insert(_body());
      }
      if (mounted) Navigator.pop(context, 'reload');
    } on ApiClientException catch (e) {
      if (mounted) alertBox(context, 'Error', e.message);
    } on Exception catch (e) {
      if (mounted) alertBox(context, 'Error', e.toString());
    } finally {
      if (mounted) setState(() => _saving = false);
    }
  }

  @override
  Widget build(BuildContext context) {
    final txs = _loaded?.transactions ?? widget.card?.transactions ?? [];

    return Scaffold(
      appBar: AppBar(
        title: Text(_isEdit ? 'Uredi karticu' : 'Dodaj karticu'),
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            TextField(
              controller: _number,
              decoration: const InputDecoration(labelText: 'Broj kartice'),
              keyboardType: TextInputType.number,
              maxLength: 12,
            ),
            TextField(
              controller: _cvc,
              decoration: const InputDecoration(labelText: 'CVC'),
              keyboardType: TextInputType.number,
              maxLength: 3,
            ),
            TextField(
              controller: _expiry,
              decoration: const InputDecoration(
                labelText: 'Datum isteka (YYYY-MM-DD)',
              ),
            ),
            TextField(
              controller: _balance,
              decoration: const InputDecoration(labelText: 'Početno stanje'),
              keyboardType: const TextInputType.numberWithOptions(decimal: true),
            ),
            const SizedBox(height: 16),
            SizedBox(
              width: double.infinity,
              height: 48,
              child: FilledButton(
                onPressed: _saving ? null : _save,
                child: Text(_isEdit ? 'Spremi' : 'Dodaj'),
              ),
            ),
            if (_isEdit) ...[
              const SizedBox(height: 24),
              const Text(
                'Transakcije',
                style: TextStyle(fontSize: 16, fontWeight: FontWeight.bold),
              ),
              const SizedBox(height: 8),
              if (txs.isEmpty)
                const Text('Nema transakcija.')
              else
                ...txs.map(
                  (t) => ListTile(
                    contentPadding: EdgeInsets.zero,
                    title: Text('${t.amount ?? 0}'),
                    subtitle: Text('${t.date ?? ''}'),
                  ),
                ),
            ],
          ],
        ),
      ),
    );
  }
}
```

---

# KORAK 26 – CIJELI `order_provider.dart`

**Fajl:** `eCommerce/UI/ecommerce_mobile/lib/providers/order_provider.dart`

```dart
import 'dart:convert';

import 'package:ecommerce_mobile/models/order.dart';
import 'package:ecommerce_mobile/providers/base_provider.dart';
import 'package:http/http.dart' as http;

class OrderProvider extends BaseProvider<Order> {
  OrderProvider() : super('Orders');

  @override
  Order fromJson(data) => Order.fromJson(data as Map<String, dynamic>);

  Future<Order> checkout(
    List<Map<String, dynamic>> items, {
    required int paymentCardId,
  }) async {
    final uri = Uri.parse('${BaseProvider.baseUrl}Orders/Checkout');
    final headers = createHeaders();
    final body = jsonEncode({
      'items': items,
      'paymentCardIBXXXXXXId': paymentCardId,
    });
    final response = await http.post(uri, headers: headers, body: body);
    validateResponse(response);
    return Order.fromJson(jsonDecode(response.body) as Map<String, dynamic>);
  }

  Future<List<Order>> fetchMyOrders() async {
    final result = await get(filter: {'page': 1, 'pageSize': 100});
    return result.items ?? [];
  }
}
```

JSON ključ `paymentCardIBXXXXXXId` mora odgovarati C# `PaymentCardIBXXXXXXId` nakon što zamijeniš indeks.

---

# KORAK 27 – CIJELI `cart_list_screen.dart`

**Fajl:** `eCommerce/UI/ecommerce_mobile/lib/screens/cart_list_screen.dart`

```dart
import 'package:ecommerce_mobile/models/payment_card.dart';
import 'package:ecommerce_mobile/providers/auth_provider.dart';
import 'package:ecommerce_mobile/providers/cart_provider.dart';
import 'package:ecommerce_mobile/providers/order_provider.dart';
import 'package:ecommerce_mobile/providers/payment_card_provider.dart';
import 'package:ecommerce_mobile/screens/order_detail_screen.dart';
import 'package:ecommerce_mobile/utils/api_client_exception.dart';
import 'package:ecommerce_mobile/utils/utils_widgets.dart';
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

class CartListScreen extends StatefulWidget {
  final VoidCallback? onGoToHome;

  const CartListScreen({super.key, this.onGoToHome});

  @override
  State<CartListScreen> createState() => _CartListScreenState();
}

class _CartListScreenState extends State<CartListScreen> {
  late CartProvider _cartProvider;
  bool _checkoutBusy = false;
  List<PaymentCard> _cards = [];
  int? _selectedCardId;

  @override
  void initState() {
    super.initState();
    _cartProvider = context.read<CartProvider>();
    _loadCards();
  }

  Future<void> _loadCards() async {
    try {
      final result = await context.read<PaymentCardProvider>().get(
        filter: {'page': 1, 'pageSize': 50},
      );
      setState(() {
        _cards = result.items ?? [];
        if (_cards.isNotEmpty) {
          _selectedCardId = _cards.first.id;
        }
      });
    } on Exception catch (e) {
      if (mounted) alertBox(context, 'Cards', e.toString());
    }
  }

  double _subtotal(CartProvider cart) {
    return cart.cart.items.fold<double>(
      0,
      (sum, item) => sum + (item.product.price ?? 0) * item.quantity,
    );
  }

  Future<void> _checkout() async {
    if (AuthProvider.accesstoken == null || AuthProvider.accesstoken!.isEmpty) {
      alertBox(context, 'Login required', 'Please log in to place an order.');
      return;
    }

    if (_cartProvider.cart.items.isEmpty) {
      return;
    }

    if (_selectedCardId == null) {
      alertBox(
        context,
        'Payment',
        'Odaberite karticu ili je dodajte u profilu.',
      );
      return;
    }

    final invalid = _cartProvider.cart.items
        .where((e) => e.product.id == null)
        .toList();
    if (invalid.isNotEmpty) {
      alertBox(context, 'Cart error', 'Some items are missing a product id.');
      return;
    }

    setState(() => _checkoutBusy = true);
    try {
      final payload = _cartProvider.cart.items
          .map(
            (e) => <String, dynamic>{
              'productId': e.product.id,
              'quantity': e.quantity,
            },
          )
          .toList();
      final order = await context.read<OrderProvider>().checkout(
        payload,
        paymentCardId: _selectedCardId!,
      );

      _cartProvider.clearCart();

      if (!mounted) return;
      await Navigator.push(
        context,
        MaterialPageRoute(
          builder: (context) => OrderDetailScreen(orderId: order.id),
        ),
      );
    } on ApiClientException catch (e) {
      if (mounted) {
        alertBox(context, 'Order could not be placed', e.message);
      }
    } on Exception catch (e) {
      if (mounted) {
        alertBox(context, 'Checkout', e.toString());
      }
    } finally {
      if (mounted) {
        setState(() => _checkoutBusy = false);
      }
    }
  }

  @override
  Widget build(BuildContext context) {
    return Consumer(
      builder: (context, CartProvider cartProvider, child) {
        return SafeArea(
          child: Column(
            children: [
              _buildHeader(),
              Expanded(child: _buildItemList()),
              if (cartProvider.cart.items.isNotEmpty) _buildFooter(cartProvider),
            ],
          ),
        );
      },
    );
  }

  Widget _buildFooter(CartProvider cartProvider) {
    final total = _subtotal(cartProvider);
    return Padding(
      padding: const EdgeInsets.fromLTRB(16, 8, 16, 16),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.stretch,
        children: [
          Row(
            mainAxisAlignment: MainAxisAlignment.spaceBetween,
            children: [
              const Text('Estimated total', style: TextStyle(fontSize: 16)),
              Text(
                '\$${total.toStringAsFixed(2)}',
                style: const TextStyle(
                  fontSize: 18,
                  fontWeight: FontWeight.bold,
                ),
              ),
            ],
          ),
          const SizedBox(height: 12),
          if (_cards.isEmpty)
            const Padding(
              padding: EdgeInsets.only(bottom: 8),
              child: Text('Dodajte karticu u profilu prije plaćanja.'),
            )
          else
            Padding(
              padding: const EdgeInsets.only(bottom: 8),
              child: DropdownButton<int>(
                isExpanded: true,
                value: _selectedCardId,
                hint: const Text('Odaberi karticu'),
                items: _cards
                    .map(
                      (c) => DropdownMenuItem(
                        value: c.id,
                        child: Text(
                          '${c.cardNumber}  (dostupno: ${c.availableBalance ?? c.initialBalance})',
                        ),
                      ),
                    )
                    .toList(),
                onChanged: (v) => setState(() => _selectedCardId = v),
              ),
            ),
          SizedBox(
            height: 48,
            child: FilledButton(
              onPressed: _checkoutBusy ? null : _checkout,
              child: _checkoutBusy
                  ? const SizedBox(
                      width: 22,
                      height: 22,
                      child: CircularProgressIndicator(
                        strokeWidth: 2,
                        color: Colors.white,
                      ),
                    )
                  : const Text('Place order'),
            ),
          ),
        ],
      ),
    );
  }

  Widget _buildItemList() {
    if (_cartProvider.cart.items.isEmpty) {
      return _buildEmptyCart();
    }
    return ListView.builder(
      itemCount: _cartProvider.cart.items.length,
      itemBuilder: (context, index) {
        var cartItem = _cartProvider.cart.items[index];
        return ListTile(
          title: Text(cartItem.product.name ?? ""),
          subtitle: Text('Quantity: ${cartItem.quantity}'),
          trailing: IconButton(
            icon: const Icon(Icons.remove_shopping_cart),
            onPressed: () {
              _cartProvider.removeFromCart(cartItem.product);
            },
          ),
        );
      },
    );
  }

  Widget _buildEmptyCart() {
    return GestureDetector(
      onTap: widget.onGoToHome,
      child: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Icon(Icons.shopping_cart_outlined, size: 80, color: Colors.grey),
            const SizedBox(height: 16),
            const Text(
              "Your cart is empty",
              style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            const Text(
              "Tap to browse products",
              style: TextStyle(fontSize: 16, color: Colors.grey),
            ),
            const SizedBox(height: 24),
            ElevatedButton.icon(
              onPressed: widget.onGoToHome,
              icon: const Icon(Icons.home),
              label: const Text("Go to Home"),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildHeader() {
    return const Padding(
      padding: EdgeInsets.all(8.0),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Text(
            "Your Cart",
            style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
          ),
          SizedBox(height: 8),
          Text(
            "Review your selected items before checkout.",
            style: TextStyle(fontSize: 16, color: Colors.grey),
          ),
        ],
      ),
    );
  }
}
```

---

# KORAK 28 – Predaja

1. Visual Studio → desni klik na solution → **Clean Solution**
2. U folderu `ecommerce_mobile`:

```
flutter clean
```

3. Zip cijeli projekat, ime = tvoj indeks
4. FTP `Upload/RSII`, user/pass `student_250250`

---

# Brza lista fajlova

| Korak | Fajl | Novo / zamjena |
|-------|------|----------------|
| 1 | `appsettings.Development.json` | zamjena connection stringa |
| 2 | `Database/PaymentCardIBXXXXXX.cs` | NOVO |
| 3 | `Database/User.cs` | cijeli fajl |
| 4 | `Database/Order.cs` | cijeli fajl |
| 5 | `Database/eCommerceConfiguration.cs` | cijeli fajl |
| 6 | `Database/eCommerceDbContext.cs` | cijeli fajl |
| 7 | migracija | komanda |
| 8–11 | Model Request/Response/Search | NOVO |
| 12–13 | Validators | NOVO |
| 14–15 | Interface + Service | NOVO |
| 16 | `Program.cs` | cijeli fajl |
| 17 | Controller | NOVO |
| 18 | `CheckoutRequest.cs` | cijeli fajl |
| 19 | `OrderService.cs` | cijeli fajl |
| 20–22 | Flutter model + provider | NOVO |
| 23 | `main.dart` vrh | zamjena |
| 24 | `profile_screen.dart` | cijeli fajl |
| 25 | `payment_card_details.dart` | NOVO |
| 26 | `order_provider.dart` | cijeli fajl |
| 27 | `cart_list_screen.dart` | cijeli fajl |
