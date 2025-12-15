## Usage Prompt

Antes de generar o extender código, analiza primero el proyecto base ya creado para entender su arquitectura y convenciones. Verifica si los paquetes necesarios ya están instalados; si falta alguno, pide instalarlo indicando cuáles son, recurda leer y seguir las intrucciones del Enuncido que se te dara en la parte final de este mensaje, avisa si existe alguna contradiccion entre lo que sigue y el enunciodo brindado

Usa este prompt base para crear/extender un backend C#/.NET 9 siguiendo la arquitectura y convenciones aplicadas en este proyecto. Ajusta nombres según el nuevo contexto, pero conserva estructura, paquetes y patrones. Autor en `<remarks>`: Pietro Osores.

### Tecnología y paquetes
- .NET 9, ASP.NET Core Web API, EF Core 9.
- MySQL con `MySql.EntityFrameworkCore` (no Pomelo).
- Swagger: solo `Swashbuckle.AspNetCore` (no agregar `Microsoft.AspNetCore.OpenApi` para evitar conflicto con `Microsoft.OpenApi`).
- `Humanizer` para snake_case.
- Referencias: `Microsoft.EntityFrameworkCore`, `MySql.EntityFrameworkCore`, `Swashbuckle.AspNetCore`, `Humanizer`.

### Estructura de carpetas
```
<ContextName>/
  Domain/Model/{Aggregates,Commands,Queries}
  Domain/{Repositories,Services}
  Application/Internal/{CommandServices,QueryServices}
  Infrastructure/Persistence/EFC/Repositories
  Interfaces/REST/{Resources,Transform,Controller}
Shared/
  Domain/{Model,Exceptions,Repositories}
  Infrastructure/{Interfaces/ASP,Persistence/EFC/{Configuration,Repositories}}
```
- Slugify/kebab-case transformer en `Shared/Infrastructure/Interfaces/ASP/SlugifyParameterTransformer.cs`.

### Patrones y convenciones
- Agregados como clases parciales con ctor que recibe el comando.
- Comandos/queries como `record`.
- Contratos: `I<Entity>Repository`, `I<Entity>CommandService`, `I<Entity>QueryService`.
- Implementaciones: `<Entity>Repository` (hereda `BaseRepository<TEntity>`), `<Entity>CommandService`, `<Entity>QueryService`.
- DTOs como `*Resource` (records).
- Ensambladores: `*ResourceFromEntityAssembler` y `Create*CommandFromResourceAssembler`.
- Controllers: `[ApiController]`, `[Route("api/v1/[controller]")]`, usan DI, GET/POST separados, manejo de conflicto/validación estilo `FavoriteSourcesController`.
- Routing: `LowercaseUrls` + `RouteTokenTransformerConvention` con transformer a kebab-case.
- Respuestas no incluyen campos de auditoría.

### Shared
- `AuditableEntity` con `CreatedAt`, `UpdatedAt`.
- Excepciones: `BusinessRuleValidationException`, `EntityConflictException`, `NotFoundException`.
- `IBaseRepository<TEntity>` con `AddAsync`, `FindByIdAsync`, `ListAsync`, `Update`, `Remove`.
- `IUnitOfWork.CompleteAsync()`.
- `BaseRepository<TEntity>` implementa `IBaseRepository`, usa `AppDbContext`.
- `UnitOfWork` envuelve `SaveChangesAsync`.
- `AppDbContext`: central, snake_case en tablas/columnas (definido explícitamente en Fluent API), usa `UseMySQL`, incluye seeding necesario, override de `SaveChangesAsync` para setear auditoría; check constraints según reglas de negocio.
- `UseSnakeCaseNamingConvention()` puede ser placeholder si ya se configuran nombres explícitos.

### Infraestructura y DI
- En `Program.cs`: agrega controllers con transformer kebab-case, `AddSwaggerGen`, `AddDbContext<AppDbContext>` con `UseMySQL(<connectionString>)` y `UseSnakeCaseNamingConvention()`.
- Registra repositorios y servicios de cada bounded context, y `IUnitOfWork`.
- En Development: `UseSwagger`/`UseSwaggerUI`.
- Antes del pipeline: `EnsureCreated()` para crear DB y tablas si no existen.
- `launchSettings.json`: `launchBrowser: true`, `launchUrl: "swagger"` para abrir Swagger al arrancar.

### Reglas de negocio (ejemplo FIRSTstudent)
- Bus: vehiclePlate único, formato `AAA-0000` uppercase, fuelTankType ∈ {A,B,C,D}, districtId ∈ {1,2,3}, totalSeats 20–40.
- Assignment: un studentId solo una vez; mismo parentId no puede tener hijos en buses distintos; conteo de assignments no debe exceder `TotalSeats`; `AssignedAt` se establece en el ctor (UTC); validar existencia de Bus/Student (usar ACL a Bus Query Service).
- Students se seed con los datos indicados (1..10).

### EF Fluent API
- Tablas snake_case plural (`buses`, `assignments`, `students`); columnas snake_case (`vehicle_plate`, etc.).
- Constraints/checks para reglas de negocio; índices únicos según necesidad (p.ej. vehicle_plate, student_id).
- Relaciones con `OnDelete(DeleteBehavior.Restrict)`.
- Sin defaults SQL en datetime si el servidor no los soporta; se asigna en código.

### Swagger y rutas
- Swagger habilitado en Development; URLs típicas: `https://localhost:7169/swagger`, JSON en `.../swagger/v1/swagger.json`.
- Endpoints en minúsculas con guiones.

### Conexion de a la base de datos
- al finlizar el code solicita la informacion para crear la base de dtos automaticamnte al ejecutar el backend si es que no existe,debes solicitar user y password
  
### Autoría y documentación
- XML docs en inglés, `<remarks>Pietro Osores.</remarks>` en clases/métodos relevantes.
- Comentarios concisos solo cuando agregan claridad.

### SDK
- `global.json` apunta a SDK 9.0.0; asegurarse de tenerlo instalado o ajustar `global.json` a un SDK disponible.

### Alcance
- Sin CORS ni seguridad avanzada (fuera de alcance).
- Sin tests automatizados (fuera de alcance, salvo que se pidan).
