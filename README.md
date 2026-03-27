# YARP.Example

This project demonstrates the use of **YARP (Yet Another Reverse Proxy)** as an API Gateway for .NET 10 applications.

## 📋 Project Overview

The example consists of three ASP.NET Core projects:

- **ApiGateway**: YARP Reverse Proxy acting as the central entry point
- **WebApi1**: Backend API on port 7001 (HTTPS) / 5001 (HTTP)
- **WebApi2**: Backend API on port 7106 (HTTPS) / 5106 (HTTP)

## 🚀 What Does YARP Do?

**YARP (Yet Another Reverse Proxy)** is a high-performance, configurable reverse proxy from Microsoft for .NET. In this example:

### Routing Configuration

The ApiGateway forwards requests based on URL paths to the corresponding backend services:

- **`/WebApi1/**`** → routed to WebApi1 (`https://localhost:7001`)
- **`/WebApi2/**`** → routed to WebApi2 (`https://localhost:7106`)

### Path Transformation

YARP automatically removes the path prefix before forwarding the request to the backend service:

```
Request: https://localhost:7170/WebApi1/api/values
   ↓
Forwarded to: https://localhost:7001/api/values
```

### Benefits

- ✅ **Single Entry Point**: One gateway for all backend APIs
- ✅ **Flexible Configuration**: Routing via `appsettings.json`
- ✅ **Load Balancing**: Support for multiple destinations per cluster
- ✅ **Transformations**: URL rewriting, header manipulation, etc.
- ✅ **Hot Reload**: Configuration changes apply without restart

## 🛠️ Starting the Project

### Start All Services

```bash
# Terminal 1: ApiGateway
cd ApiGateway
dotnet run

# Terminal 2: WebApi1
cd WebApi1
dotnet run

# Terminal 3: WebApi2
cd WebApi2
dotnet run
```

### Accessing the APIs

**Via API Gateway:**
- WebApi1 Swagger: https://localhost:7170/WebApi1/swagger
- WebApi2 Swagger: https://localhost:7170/WebApi2/swagger

**Direct Access:**
- WebApi1 Swagger: https://localhost:7001/swagger
- WebApi2 Swagger: https://localhost:7106/swagger

## ⚠️ Important: SwaggerUI with YARP

### Problem with Absolute Paths

When SwaggerUI runs behind a reverse proxy like YARP, the Swagger endpoint configuration must use **relative paths**, otherwise the UI will not work correctly.

### ❌ Wrong (Default Configuration)

```csharp
app.UseSwaggerUI(); // Uses absolute path: /swagger/v1/swagger.json
```

This doesn't work when the API is accessed via the gateway because YARP cannot correctly forward the request to `/swagger/v1/swagger.json`.

### ✅ Correct (Relative Paths)

```csharp
app.UseSwaggerUI(options =>
{
    // Use relative paths without leading slash
    options.SwaggerEndpoint("../swagger/v1/swagger.json", "API V1");
});
```

**Why is this important?**
- The browser resolves relative paths based on the current URL
- At `/WebApi1/swagger`, `../swagger/v1/swagger.json` becomes `/WebApi1/swagger/v1/swagger.json`
- YARP can correctly forward this request to the backend service

### Configuration in This Project

Both backend APIs (WebApi1 and WebApi2) are already configured with relative paths:

**WebApi1/Program.cs** and **WebApi2/Program.cs**:
```csharp
app.UseSwaggerUI(options =>
{
    options.SwaggerEndpoint("../swagger/v1/swagger.json", "API V1");
});
```

## 📦 Used Packages

- **Microsoft.AspNetCore.OpenApi** (10.0.5): OpenAPI support
- **Swashbuckle.AspNetCore**: Swagger/OpenAPI UI
- **Yarp.ReverseProxy**: YARP Reverse Proxy (in ApiGateway)

## 🔧 YARP Configuration

The YARP configuration is in `ApiGateway/appsettings.json`:

```json
{
  "ReverseProxy": {
    "Routes": {
      "webapi1-route": {
        "ClusterId": "webapi1-cluster",
        "Match": {
          "Path": "/WebApi1/{**catch-all}"
        },
        "Transforms": [
          { "PathRemovePrefix": "/WebApi1" }
        ]
      }
    },
    "Clusters": {
      "webapi1-cluster": {
        "Destinations": {
          "destination1": {
            "Address": "https://localhost:7001"
          }
        }
      }
    }
  }
}
```

## 📚 Further Information

- [YARP Documentation](https://microsoft.github.io/reverse-proxy/)
- [ASP.NET Core OpenAPI](https://learn.microsoft.com/aspnet/core/fundamentals/openapi/)
- [Swashbuckle Documentation](https://github.com/domaindrivendev/Swashbuckle.AspNetCore)

## 📝 Technology Stack

- **.NET 10.0**
- **C# 14.0**
- **ASP.NET Core**
- **YARP (Yet Another Reverse Proxy)**
- **Swagger/OpenAPI**
