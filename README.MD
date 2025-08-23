# Documentación Completa del Backend - Tablero API

Este documento describe de manera detallada el proyecto **Tablero API**, desarrollado en **.NET 8** con **ASP.NET Core Web API**. Su finalidad es proporcionar un backend para la gestión de información deportiva relacionada con equipos, localidades, partidos, cuartos e imágenes. A lo largo de esta documentación se presentan los objetivos, la arquitectura del sistema, las tecnologías utilizadas, los pasos de instalación y configuración, la explicación de los modelos/DTOs y **todos los endpoints**, además de la guía de **pruebas**, **solución de errores** y **despliegue**.

---

## Introducción

El proyecto Tablero API nace con el objetivo de centralizar la información de un tablero deportivo. Muchas aplicaciones necesitan manejar datos como equipos, sedes (localidades), partidos y estadísticas de juego. Este backend ofrece servicios REST para que un frontend pueda consultar y modificar dicha información de forma sencilla.

La API se apoya en **Entity Framework Core** para manejar la persistencia de datos en **SQL Server**. Además, incluye **Swagger**, lo que permite visualizar y probar todos los endpoints desde un navegador. Esto convierte al proyecto en una base sólida para aplicaciones deportivas que requieran gestionar información en tiempo real.

### Objetivos principales

* Proporcionar un sistema robusto para almacenar y consultar información deportiva.
* Permitir la interacción con entidades como Localidades, Equipos, Partidos, Cuartos e Imágenes.
* Garantizar la integridad de los datos mediante relaciones bien definidas.
* Facilitar la conexión con aplicaciones frontend a través de una API REST clara y documentada.
* Permitir el despliegue tanto local como en entornos con Docker o IIS.

---

## Arquitectura del Proyecto

El proyecto sigue una arquitectura **MVC simplificada** donde cada componente cumple un rol específico:

* **Controllers/** → Contienen la lógica para manejar las peticiones HTTP (GET, POST, PUT, DELETE). Cada controlador representa una entidad principal.
* **Models/** → Definen las clases que representan las tablas de la base de datos, así como los **DTOs** (Data Transfer Objects) que son las versiones simplificadas para enviar y recibir datos.
* **Data/** → Contiene el archivo `AppDbContext.cs`, el cual se encarga de configurar Entity Framework Core y de conectar el proyecto con la base de datos.
* **Migrations/** → Aquí se guardan las migraciones creadas por EF Core. Estas definen la estructura de la base de datos y sus cambios en el tiempo.
* **Program.cs** → Punto de entrada de la aplicación. Aquí se configuran servicios como EF Core, Swagger, CORS e inyección de dependencias.
* **appsettings.json** → Archivo de configuración donde se definen cadenas de conexión y parámetros básicos.

### Flujo de ejecución

1. El cliente (por ejemplo, un frontend en Angular) realiza una petición HTTP a la API.
2. El **Controller** correspondiente recibe la petición y, si es necesario, transforma los datos a través de un DTO.
3. El **AppDbContext** se encarga de comunicarse con la base de datos mediante EF Core.
4. La respuesta se devuelve al cliente en formato JSON.

Este esquema mantiene una separación clara de responsabilidades, lo que facilita el mantenimiento y escalabilidad del proyecto.

---

## Tecnologías utilizadas

* **.NET 8 / ASP.NET Core Web API**: Framework principal para construir la API REST.
* **Entity Framework Core**: ORM para trabajar con la base de datos utilizando clases en C#.
* **SQL Server**: Motor de base de datos relacional utilizado para almacenar la información.
* **Swagger / Swashbuckle**: Genera documentación y una UI de pruebas.
* **Docker** (opcional): Conteneriza la aplicación y la base de datos.
* **NuGet Packages**: Librerías administradas por NuGet (Microsoft.EntityFrameworkCore, Microsoft.AspNetCore, etc.).

---

## Requisitos previos

* Visual Studio 2022 o Visual Studio Code
* .NET 8 SDK instalado
* SQL Server (LocalDB, Express o Docker)
* Opcional: Postman para realizar pruebas adicionales

Comprueba .NET:

```bash
dotnet --info
```

---

## Instalación paso a paso

### 1. Descargar el proyecto

1. Clona el repositorio o descomprime la carpeta `Back-Tablero-main`.
2. Abre la solución `tablero_api.sln` en Visual Studio 2022.

### 2. Restaurar dependencias

Visual Studio lo hace automáticamente. Manual (opcional):

```bash
dotnet restore
```

### 3. Configurar la cadena de conexión

Crea/edita `tablero_api/appsettings.Development.json`:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\MSSQLLocalDB;Database=Tablero_DB;Integrated Security=True;TrustServerCertificate=True;"
  }
}
```

Para SQL Express o Docker, cambia `Server` y credenciales.

### 4. Aplicar migraciones

**Consola del Administrador de Paquetes** en Visual Studio:

```powershell
Update-Database
```

Crea la BD `Tablero_DB` y sus tablas.

### 5. Ejecutar la API

Inicia con **F5** / **Ctrl + F5**. URLs típicas:

```
https://localhost:7146
http://localhost:5060
```

Swagger:

```
https://localhost:7146/swagger
```

---

## Configuración detallada

### appsettings.json

Parámetros generales:

* **Logging**: Niveles de log.
* **ConnectionStrings**: Cadenas de conexión.
* **AllowedOrigins**: Orígenes permitidos para CORS.

### launchSettings.json

Perfiles de ejecución y variables de entorno (en `Properties/launchSettings.json`):

```json
{
  "profiles": {
    "tablero_api": {
      "commandName": "Project",
      "dotnetRunMessages": true,
      "applicationUrl": "https://localhost:7146;http://localhost:5060",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    }
  }
}
```

### Configuración de CORS

En `Program.cs`:

```csharp
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowFrontend", policy =>
    {
        policy.WithOrigins("http://localhost:4200")
              .AllowAnyHeader()
              .AllowAnyMethod();
    });
});
...
app.UseCors("AllowFrontend");
```

Agrega aquí las URLs del frontend que vayas a usar.

---

## Modelos y DTOs

### Localidad

**Entidad**

```csharp
public class Localidad {
    public int id_Localidad { get; set; }
    public string Nombre { get; set; }
}
```

**DTO**

```json
{
  "id_Localidad": 1,
  "nombre": "Ciudad X"
}
```

### Equipo

**Entidad**

```csharp
public class Equipo {
    public int id_Equipo { get; set; }
    public string Nombre { get; set; }
    public int id_Localidad { get; set; }
    public Localidad Localidad { get; set; }
}
```

**DTO**

```json
{
  "id_Equipo": 1,
  "nombre": "Equipo A",
  "id_Localidad": 1
}
```

###  Partido

**Entidad**

```csharp
public class Partido {
    public int id_Partido { get; set; }
    public DateTime FechaHora { get; set; }
    public int id_Localidad { get; set; }
    public int id_Local { get; set; }
    public int id_Visitante { get; set; }
}
```

**DTO**

```json
{
  "id_Partido": 1,
  "fechaHora": "2025-08-22T15:00:00",
  "id_Localidad": 1,
  "id_Local": 2,
  "id_Visitante": 3
}
```

### Cuarto

**Entidad**

```csharp
public class Cuarto {
    public int id_Cuarto { get; set; }
    public int Numero { get; set; }
    public int id_Partido { get; set; }
    public Partido Partido { get; set; }
}
```

**DTO**

```json
{
  "id_Cuarto": 1,
  "numero": 1,
  "id_Partido": 1
}
```

### Imagen

**Entidad**

```csharp
public class Imagen {
    public int id_Imagen { get; set; }
    public string Url { get; set; }
}
```

**DTO**

```json
{
  "id_Imagen": 10,
  "url": "https://miservidor.com/foto.jpg"
}
```

---

## Endpoints de la API

### Localidad

* **GET /api/Localidad** → Devuelve todas las localidades
* **GET /api/Localidad/{id}** → Detalle por ID
* **POST /api/Localidad**

```json
{ "nombre": "Ciudad X" }
```

* **PUT /api/Localidad/{id}**

```json
{ "nombre": "Ciudad Y" }
```

* **DELETE /api/Localidad/{id}**

### Equipo

* **GET /api/Equipo**
* **GET /api/Equipo/{id}**
* **POST /api/Equipo**

```json
{ "nombre": "Equipo A", "id_Localidad": 1 }
```

* **PUT /api/Equipo/{id}**

```json
{ "nombre": "Equipo A (actualizado)", "id_Localidad": 1 }
```

* **DELETE /api/Equipo/{id}**

### Partido

* **GET /api/Partido**
* **GET /api/Partido/{id}**
* **POST /api/Partido**

```json
{
  "fechaHora": "2025-08-22T15:00:00",
  "id_Localidad": 1,
  "id_Local": 2,
  "id_Visitante": 3
}
```

* **PUT /api/Partido/{id}**
* **DELETE /api/Partido/{id}**

### Cuarto

* **GET /api/Cuarto**
* **GET /api/Cuarto/{id}**
* **POST /api/Cuarto**

```json
{ "numero": 1, "id_Partido": 1 }
```

* **PUT /api/Cuarto/{id}**
* **DELETE /api/Cuarto/{id}**

### Imagen

* **GET /api/Imagen**
* **GET /api/Imagen/{id}**
* **POST /api/Imagen**

```json
{ "url": "https://..." }
```

* **DELETE /api/Imagen/{id}**

### Tablero

* **GET /api/Tablero** → Devuelve información consolidada del tablero (según implementación)

---

## Pruebas con Swagger y Postman

### Probar con Swagger (rápido para empezar)

1. Inicia la API y abre `http(s)://localhost:<puerto>/swagger`.
2. Despliega un endpoint, pulsa **Try it out** y completa el **Request body** si corresponde.
3. Pulsa **Execute** y revisa el **Response code** y **Response body**.

### Exportar OpenAPI desde Swagger

* URL: `http(s)://localhost:<puerto>/swagger/v1/swagger.json` (guárdalo para Postman).

### Probar con Postman

1. **Import** el archivo OpenAPI.
2. Crea una **Environment** con `baseUrl`.
3. Ejecuta requests de ejemplo (ver secciones anteriores). Guarda ejemplos de respuesta para validar.

---

## Errores comunes y soluciones

### Certificado HTTPS en localhost (NET::ERR\_CERT\_INVALID)

* Rápido: usa HTTP.
* Correcto:

  ```bash
  dotnet dev-certs https --clean
  dotnet dev-certs https --trust
  ```

  Limpia HSTS en `chrome://net-internals/#hsts`.

### Error 500 en POST por claves foráneas

* Sucede si envías IDs que **no existen** en tablas relacionadas (ej. `id_Localidad`). Crea primero la entidad padre.

### “Cannot open database / Login failed / Timeout”

* Revisa **ConnectionString**, que SQL esté corriendo y credenciales correctas.
* En local: `Trusted_Connection=True` suele ser lo más fácil.
* En Docker: host `sqlserver`, puerto `1433`.

### Migraciones no aplicadas

```bash
dotnet ef migrations add NombreCambio
dotnet ef database update
```

### CORS bloquea el frontend

* Agrega la URL del frontend a `WithOrigins(...)` y reinicia la API.

### 404 Not Found

* Verifica ruta correcta y parámetros.

### 400 Bad Request

* Asegúrate que el JSON coincida con el DTO (nombres y tipos).

---

##  Despliegue

### IIS (Windows)

1. `dotnet publish -c Release -o ./publish`
2. Sitio en IIS apuntando a `./publish`.
3. **App Pool**: *No Managed Code*.
4. Env vars (si aplica): `ASPNETCORE_ENVIRONMENT`, `ConnectionStrings__DefaultConnection`.

### Docker / Docker Compose

```bash
docker compose up -d
docker compose logs -f api   # si el servicio se llama api
```

Ajusta puertos y cadena de conexión para apuntar a `sqlserver,1433`.

### Linux (Kestrel + Nginx)

Publica y ejecuta `dotnet tablero_api.dll`. Configura Nginx como reverse proxy:

```nginx
server {
  listen 80;
  server_name tu-dominio.com;
  location / {
    proxy_pass         http://127.0.0.1:5060;
    proxy_http_version 1.1;
    proxy_set_header   Upgrade $http_upgrade;
    proxy_set_header   Connection keep-alive;
    proxy_set_header   Host $host;
    proxy_cache_bypass $http_upgrade;
    proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header   X-Forwarded-Proto $scheme;
  }
}
```

### Variables de entorno

* `ASPNETCORE_ENVIRONMENT=Production`
* `ConnectionStrings__DefaultConnection=Server=...;Database=...;...`

---

## Buenas prácticas y seguridad básica

* Valida datos en los DTOs (atributos `[Required]`, rangos, etc.).
* Maneja errores con respuestas claras (400/404/409/500) y logs útiles.
* Si agregas autenticación, usa **JWT** y protege endpoints sensibles.
* Usa **DTOs** para no exponer campos internos.
* Agrega **índices** en columnas más consultadas.

---

## Anexos de ejemplos (JSON rápidos)

### Localidad

```json
{ "nombre": "Ciudad X" }
```

### Equipo

```json
{ "nombre": "Equipo A", "id_Localidad": 1 }
```

### Partido

```json
{
  "fechaHora": "2025-08-22T15:00:00",
  "id_Localidad": 1,
  "id_Local": 2,
  "id_Visitante": 3
}
```

### Cuarto

```json
{ "numero": 1, "id_Partido": 1 }
```

### Imagen

```json
{ "url": "https://misitio.com/imagen.jpg" }
```

---

## Conclusiones

Este backend en **.NET 8** proporciona una base sólida para aplicaciones deportivas que necesiten administrar equipos, localidades, partidos, cuartos e imágenes. La estructura del proyecto es clara, las migraciones facilitan la creación de la base de datos y Swagger permite probar la API rápidamente. Además, el soporte para Docker y la posibilidad de desplegar en IIS o Linux lo hacen flexible para distintos entornos.

Si se integra con un frontend (por ejemplo, Angular), se puede construir un tablero visual y dinámico para consultas y administración de datos en tiempo real.
