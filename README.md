# YARP.Example

Dieses Projekt demonstriert die Verwendung von **YARP (Yet Another Reverse Proxy)** als API Gateway für .NET 10 Anwendungen.

## 📋 Projektübersicht

Das Beispiel besteht aus drei ASP.NET Core Projekten:

- **ApiGateway**: YARP Reverse Proxy, der als zentraler Einstiegspunkt fungiert
- **WebApi1**: Backend-API auf Port 7001 (HTTPS) / 5001 (HTTP)
- **WebApi2**: Backend-API auf Port 7106 (HTTPS) / 5106 (HTTP)

## 🚀 Was macht YARP?

**YARP (Yet Another Reverse Proxy)** ist ein hochleistungsfähiger, konfigurierbarer Reverse Proxy von Microsoft für .NET. In diesem Beispiel:

### Routing-Konfiguration

Das ApiGateway leitet Anfragen basierend auf URL-Pfaden an die entsprechenden Backend-Services weiter:

- **`/WebApi1/**`** → wird zu WebApi1 (`https://localhost:7001`) weitergeleitet
- **`/WebApi2/**`** → wird zu WebApi2 (`https://localhost:7106`) weitergeleitet

### Path Transformation

YARP entfernt automatisch den Pfad-Präfix, bevor die Anfrage an den Backend-Service weitergeleitet wird:

```
Anfrage: https://localhost:7170/WebApi1/api/values
   ↓
Weitergeleitet an: https://localhost:7001/api/values
```

### Vorteile

- ✅ **Zentraler Einstiegspunkt**: Ein Gateway für alle Backend-APIs
- ✅ **Flexible Konfiguration**: Routing über `appsettings.json`
- ✅ **Load Balancing**: Unterstützung für mehrere Destinations pro Cluster
- ✅ **Transformationen**: URL-Rewriting, Header-Manipulation, etc.
- ✅ **Hot Reload**: Änderungen in der Konfiguration werden ohne Neustart übernommen

## 🛠️ Starten des Projekts

### Alle Services starten

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

### Zugriff auf die APIs

**Über das API Gateway:**
- WebApi1 Swagger: https://localhost:7170/WebApi1/swagger
- WebApi2 Swagger: https://localhost:7170/WebApi2/swagger

**Direkter Zugriff:**
- WebApi1 Swagger: https://localhost:7001/swagger
- WebApi2 Swagger: https://localhost:7106/swagger

## ⚠️ Wichtig: SwaggerUI mit YARP

### Problem mit absoluten Pfaden

Wenn SwaggerUI hinter einem Reverse Proxy wie YARP läuft, muss die Swagger-Endpunkt-Konfiguration **relative Pfade** verwenden, sonst funktioniert die UI nicht korrekt.

### ❌ Falsch (Standard-Konfiguration)

```csharp
app.UseSwaggerUI(); // Verwendet absoluten Pfad: /swagger/v1/swagger.json
```

Dies funktioniert nicht, wenn die API über das Gateway aufgerufen wird, da YARP die Anfrage an `/swagger/v1/swagger.json` nicht korrekt weiterleitet.

### ✅ Richtig (Relative Pfade)

```csharp
app.UseSwaggerUI(options =>
{
    // Verwende relative Pfade ohne führenden Slash
    options.SwaggerEndpoint("../swagger/v1/swagger.json", "API V1");
});
```

**Warum ist das wichtig?**
- Der Browser löst relative Pfade basierend auf der aktuellen URL auf
- Bei `/WebApi1/swagger` wird `../swagger/v1/swagger.json` zu `/WebApi1/swagger/v1/swagger.json`
- YARP kann diese Anfrage korrekt an den Backend-Service weiterleiten

### Konfiguration in diesem Projekt

Beide Backend-APIs (WebApi1 und WebApi2) sind bereits mit relativen Pfaden konfiguriert:

**WebApi1/Program.cs** und **WebApi2/Program.cs**:
```csharp
app.UseSwaggerUI(options =>
{
    options.SwaggerEndpoint("../swagger/v1/swagger.json", "API V1");
});
```

## 📦 Verwendete Packages

- **Microsoft.AspNetCore.OpenApi** (10.0.5): OpenAPI-Unterstützung
- **Swashbuckle.AspNetCore**: Swagger/OpenAPI UI
- **Yarp.ReverseProxy**: YARP Reverse Proxy (im ApiGateway)

## 🔧 YARP Konfiguration

Die YARP-Konfiguration erfolgt in `ApiGateway/appsettings.json`:

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

## 📚 Weitere Informationen

- [YARP Documentation](https://microsoft.github.io/reverse-proxy/)
- [ASP.NET Core OpenAPI](https://learn.microsoft.com/aspnet/core/fundamentals/openapi/)
- [Swashbuckle Documentation](https://github.com/domaindrivendev/Swashbuckle.AspNetCore)

## 📝 Technologie-Stack

- **.NET 10.0**
- **C# 14.0**
- **ASP.NET Core**
- **YARP (Yet Another Reverse Proxy)**
- **Swagger/OpenAPI**
