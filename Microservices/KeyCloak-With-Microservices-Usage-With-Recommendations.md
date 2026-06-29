I'll help you understand how to use Keycloak with microservices, including the preferred stack and practical examples.

## Keycloak Overview for Microservices

Keycloak is an open-source Identity and Access Management solution that's particularly well-suited for microservice architectures because it:
- Provides centralized authentication and authorization
- Supports OAuth 2.0, OpenID Connect, and SAML
- Enables single sign-on (SSO) across multiple services
- Handles token-based security efficiently

## Preferred Stack with Keycloak

### **Core Components:**
1. **Keycloak Server** - Central identity provider
2. **API Gateway** (Kong, NGINX, or Ocelot) - Optional but recommended for routing
3. **Microservices** - Your application services
4. **Frontend Application** - Web/mobile apps

### **Technology Stack Options:**

**For .NET Microservices:**
- ASP.NET Core with `Microsoft.AspNetCore.Authentication.JwtBearer`
- Ocelot as API Gateway (optional)
- Entity Framework Core for data

**For Python/FastAPI Microservices:**
- FastAPI with `python-jose` or `pyjwt`
- `fastapi-keycloak` or custom middleware
- SQLAlchemy or Tortoise ORM

**For Node.js/NestJS:**
- NestJS with `@nestjs/passport` and `passport-jwt`
- Keycloak-connect adapter

## Architecture Pattern

```
┌─────────────┐     ┌──────────────┐     ┌─────────────────┐
│   Client    │────▶│  API Gateway  │────▶│  Microservice 1 │
│ (Web/Mobile)│     │  (Optional)   │     │                 │
└─────────────┘     └──────────────┘     └─────────────────┘
       │                                        │
       │                                        │
       ▼                                        ▼
┌─────────────┐                        ┌─────────────────┐
│  Keycloak   │◀──────────────────────▶│  Microservice 2 │
│  Server     │   Token Validation     │                 │
└─────────────┘                        └─────────────────┘
```

## Practical Example: ASP.NET Core Microservice with Keycloak

### 1. **Keycloak Setup**

First, create a realm and client in Keycloak:
- Realm: `ecommerce`
- Client: `order-service` (Confidential access type)
- Enable "Service Accounts Enabled" for service-to-service communication

### 2. **ASP.NET Core Configuration**

```csharp
// Program.cs
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.IdentityModel.Tokens;

var builder = WebApplication.CreateBuilder(args);

// Add JWT Authentication with Keycloak
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.Authority = "http://localhost:8080/realms/ecommerce";
        options.RequireHttpsMetadata = false; // For development only
        
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidIssuer = "http://localhost:8080/realms/ecommerce",
            ValidateAudience = true,
            ValidAudiences = new[] { "account", "order-service" },
            ValidateLifetime = true,
            NameClaimType = "preferred_username",
            RoleClaimType = "realm_access.roles"
        };
        
        // Optional: Include events for debugging
        options.Events = new JwtBearerEvents
        {
            OnAuthenticationFailed = context =>
            {
                Console.WriteLine($"Authentication failed: {context.Exception.Message}");
                return Task.CompletedTask;
            }
        };
    });

builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("AdminOnly", policy => 
        policy.RequireRole("admin"));
    
    options.AddPolicy("CanCreateOrder", policy => 
        policy.RequireClaim("permissions", "order:create"));
});

builder.Services.AddControllers();

var app = builder.Build();

app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();
app.Run();
```

### 3. **Controller with Role-Based Authorization**

```csharp
[ApiController]
[Route("api/[controller]")]
[Authorize] // Requires valid JWT token
public class OrdersController : ControllerBase
{
    [HttpGet]
    [Authorize(Policy = "CanViewOrders")]
    public IActionResult GetOrders()
    {
        var userId = User.FindFirst("sub")?.Value;
        var username = User.Identity.Name;
        var roles = User.Claims
            .Where(c => c.Type == "realm_access.roles")
            .Select(c => c.Value);
            
        return Ok(new { userId, username, roles });
    }
    
    [HttpPost]
    [Authorize(Policy = "CanCreateOrder")]
    public IActionResult CreateOrder([FromBody] OrderDto order)
    {
        // Business logic here
        return CreatedAtAction(nameof(GetOrders), new { id = 1 }, order);
    }
}
```

### 4. **Service-to-Service Communication**

For microservices communicating with each other:

```csharp
public class KeycloakTokenService
{
    private readonly HttpClient _httpClient;
    private readonly string _tokenEndpoint;
    private readonly string _clientId;
    private readonly string _clientSecret;
    
    public KeycloakTokenService(HttpClient httpClient, IConfiguration config)
    {
        _httpClient = httpClient;
        _tokenEndpoint = $"{config["Keycloak:Authority"]}/protocol/openid-connect/token";
        _clientId = config["Keycloak:ClientId"];
        _clientSecret = config["Keycloak:ClientSecret"];
    }
    
    public async Task<string> GetAccessTokenAsync()
    {
        var content = new FormUrlEncodedContent(new[]
        {
            new KeyValuePair<string, string>("grant_type", "client_credentials"),
            new KeyValuePair<string, string>("client_id", _clientId),
            new KeyValuePair<string, string>("client_secret", _clientSecret)
        });
        
        var response = await _httpClient.PostAsync(_tokenEndpoint, content);
        response.EnsureSuccessStatusCode();
        
        var tokenResponse = await response.Content.ReadFromJsonAsync<TokenResponse>();
        return tokenResponse.AccessToken;
    }
}

public class OrderServiceClient
{
    private readonly HttpClient _httpClient;
    private readonly KeycloakTokenService _tokenService;
    
    public OrderServiceClient(HttpClient httpClient, KeycloakTokenService tokenService)
    {
        _httpClient = httpClient;
        _tokenService = tokenService;
    }
    
    public async Task<Order> CreateOrderAsync(Order order)
    {
        var token = await _tokenService.GetAccessTokenAsync();
        _httpClient.DefaultRequestHeaders.Authorization = 
            new AuthenticationHeaderValue("Bearer", token);
        
        var response = await _httpClient.PostAsJsonAsync("/api/orders", order);
        response.EnsureSuccessStatusCode();
        
        return await response.Content.ReadFromJsonAsync<Order>();
    }
}
```

### 5. **appsettings.json Configuration**

```json
{
  "Keycloak": {
    "Authority": "http://localhost:8080/realms/ecommerce",
    "ClientId": "order-service",
    "ClientSecret": "your-client-secret-here"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore.Authentication": "Debug"
    }
  }
}
```

## FastAPI Example with Keycloak

```python
# main.py
from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer
import jwt
from jwt import PyJWKClient

app = FastAPI()

KEYCLOAK_URL = "http://localhost:8080"
REALM = "ecommerce"
JWKS_URL = f"{KEYCLOAK_URL}/realms/{REALM}/protocol/openid-connect/certs"

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")
jwks_client = PyJWKClient(JWKS_URL)

def decode_token(token: str):
    try:
        signing_key = jwks_client.get_signing_key_from_jwt(token)
        payload = jwt.decode(
            token,
            signing_key.key,
            algorithms=["RS256"],
            audience="account",
            issuer=f"{KEYCLOAK_URL}/realms/{REALM}"
        )
        return payload
    except jwt.ExpiredSignatureError:
        raise HTTPException(status_code=401, detail="Token expired")
    except jwt.InvalidTokenError:
        raise HTTPException(status_code=401, detail="Invalid token")

async def get_current_user(token: str = Depends(oauth2_scheme)):
    payload = decode_token(token)
    return payload

@app.get("/api/orders")
async def get_orders(current_user: dict = Depends(get_current_user)):
    # Check roles from token
    realm_access = current_user.get("realm_access", {})
    roles = realm_access.get("roles", [])
    
    if "admin" not in roles:
        raise HTTPException(status_code=403, detail="Not authorized")
    
    return {"orders": [], "user": current_user.get("preferred_username")}

@app.post("/api/orders")
async def create_order(order: dict, current_user: dict = Depends(get_current_user)):
    return {"status": "created", "order": order}
```

## Best Practices

1. **Use HTTPS in Production**: Always enable HTTPS for Keycloak and your services
2. **Token Caching**: Cache JWKS keys to avoid frequent network calls
3. **Short-Lived Tokens**: Configure access tokens to expire quickly (5-15 minutes)
4. **Refresh Tokens**: Use refresh tokens for better UX without re-authentication
5. **Centralized Logout**: Implement proper session invalidation across services
6. **Rate Limiting**: Protect Keycloak endpoints from brute force attacks
7. **Monitoring**: Track authentication failures and token validation metrics

## Docker Compose for Quick Setup

```yaml
version: '3.8'
services:
  keycloak:
    image: quay.io/keycloak/keycloak:latest
    environment:
      KEYCLOAK_ADMIN: admin
      KEYCLOAK_ADMIN_PASSWORD: admin
    ports:
      - "8080:8080"
    command: start-dev
  
  order-service:
    build: ./order-service
    ports:
      - "5001:80"
    environment:
      - Keycloak__Authority=http://keycloak:8080/realms/ecommerce
    depends_on:
      - keycloak
```

This setup provides a robust, scalable authentication system for your microservices architecture. Would you like me to elaborate on any specific aspect, such as implementing fine-grained permissions, handling multi-tenancy, or integrating with an API gateway?