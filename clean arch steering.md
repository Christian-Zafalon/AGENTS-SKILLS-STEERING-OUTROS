---
description: Adapter rules for C#/.NET APIs and transport boundaries, including ASP.NET Core controllers/endpoints, presenters, request validation, and error mapping.
alwaysApply: false
---

# Adapters — C#/.NET

## ASP.NET Core API

- Controllers or Minimal API endpoints are transport adapters, not business services.
- Handle model binding, transport validation, authentication/authorization integration, response shaping, and HTTP status mapping at the API boundary.
- Never pass `HttpRequest`, `HttpContext`, route dictionaries, headers, or raw message envelopes into Application use cases.
- Map validated transport requests into application input models before calling the use case.
- Map domain/application exceptions to HTTP responses at the API boundary. Domain code must not know HTTP status codes.
- Keep endpoint/controller methods thin: extract -> validate -> map -> execute -> map response.
- Use ASP.NET Core's built-in model validation, FluentValidation, or the project's established validation mechanism consistently. Do not duplicate validators across layers.
- Keep OpenAPI/Swagger metadata in the API layer and follow the project's existing generation approach; do not hand-maintain a second contract source.
- Preserve correlation/trace information through logging and diagnostics, but do not pollute business DTOs with transport-only metadata.

## Presentation

- Presenters/response mappers should format output only: DTO shape, serialization concerns, localized formatting, and sensitive-field removal when required.
- Never expose domain entities or EF Core models directly from API responses.
ACABA AQUI------------------------
---
description: Application-layer rules for C#/.NET use cases, input/output ports, DTOs, orchestration, transactions, and dependency boundaries.
alwaysApply: false
---

# Application — C#/.NET

- Organize application services/use cases around business actions, e.g. `CriarOrdem`, `CancelarOrdem`, `EstornarPagamento`.
- Keep each use case focused on one workflow. A typical public entry point is `ExecuteAsync(...)` returning an application DTO/result.
- Inject dependencies through constructors. Application code must not instantiate repositories, gateways, `HttpClient`, `DbContext`, configuration, or infrastructure services directly.
- Application orchestrates: load required state -> invoke domain behavior -> persist -> publish/queue required events -> return an output model.
- Business invariants belong in Domain. Do not duplicate domain rules in application services merely for convenience.
- Input/output DTOs are plain application contracts. Do not expose domain entities, EF Core entities, or transport-specific models as API contracts.
- Keep transport metadata out of use-case inputs unless it is a real business requirement. Do not pass `HttpContext`, `HttpRequest`, `ClaimsPrincipal`, or message envelopes into use cases.
- Use interfaces for output ports when the application depends on persistence, external services, messaging, clock/time, or other replaceable boundaries.
- Prefer explicit transaction boundaries. For reliable integration-event delivery, favor a transactional outbox over fire-and-forget publication after a database commit.
- Return meaningful failures or throw semantic domain/application exceptions consistently with the existing solution convention; never hide invalid business states behind silent `null` results.
- Keep application services independently testable with unit-test doubles; integration behavior belongs in adapter/infrastructure tests.
ACABA AQUI------------------------------
---
globs: **/*.cs
alwaysApply: false
---

# Clean Architecture — C#/.NET Core

- Preserve dependency direction: Domain <- Application <- Adapters/Infrastructure.
- Keep Domain independent from ASP.NET Core, EF Core, Dapper, messaging SDKs, HTTP clients, configuration, and other infrastructure libraries.
- Keep business rules in Domain types; Application orchestrates; adapters/infrastructure translate external protocols and technologies.
- Prefer the existing project structure and established patterns over introducing new abstractions.
- Depend on abstractions only at architectural boundaries where they provide real decoupling, substitution, or testability. Avoid interface-for-every-class.
- Do not leak EF Core entities, `DbContext`, `IQueryable`, HTTP models, provider SDK types, or transport contracts across layer boundaries.
- Use async I/O end-to-end for database, HTTP, messaging, and external calls. Propagate `CancellationToken`.
- Register dependencies in composition-root code (`Program.cs` or dedicated DI extension methods); do not instantiate infrastructure services inside business/application code.
- Read the existing solution structure and representative implementations before creating new architecture or abstractions.
ACABA AQUI----------------------------
---
description: Domain-layer rules for C#/.NET Clean Architecture, including entities, value objects, domain exceptions, aggregates, and domain events.
alwaysApply: false
---

# Domain — C#/.NET

- Keep domain types framework-agnostic. No ASP.NET Core, EF Core, Dapper, Newtonsoft/System.Text.Json transport concerns, or provider SDK dependencies.
- Entities own identity and behavior. Protect invariants through constructors, factories, and behavior methods; avoid anemic setter-only models.
- Aggregate roots are the only public entry point to mutate their aggregate. Do not manipulate child entities directly from application or infrastructure layers.
- Keep aggregate boundaries explicit. Do not hold direct references between aggregate roots; coordinate cross-aggregate workflows in Application and use domain/integration events when appropriate.
- Value objects should be immutable, validated at creation, and compared by value. Prefer domain types over repeated primitive representations when the concept has rules.
- Domain exceptions describe business meaning, not transport details. Never embed HTTP status codes, `ProblemDetails`, EF Core exceptions, or provider exceptions in Domain.
- Domain events represent facts and should be immutable. Keep publication concerns outside Domain.
- Prefer `DateTimeOffset` for business timestamps when the domain requires an absolute point in time; make time behavior explicit and testable.
- Do not add domain abstractions merely to mirror infrastructure APIs. Model the business language first.
ACABA AQUI--------------
---
description: Infrastructure rules for C#/.NET persistence, EF Core/Dapper, external gateways, HTTP clients, messaging, caching, configuration, DI, startup, and graceful shutdown.
alwaysApply: false
---

# Infrastructure — C#/.NET

## Persistence

- Keep EF Core entities/configurations, `DbContext`, migrations, Dapper code, SQL, connection settings, and persistence-specific mapping inside Infrastructure.
- Never expose `DbContext`, EF entities, `IQueryable`, `Expression<>` built for persistence, or provider-specific types to Application or Domain.
- Repositories implement application output ports and return domain/application concepts, not persistence models.
- For EF Core, configure mappings explicitly and keep `OnModelCreating`/entity configurations free of business behavior.
- Keep migrations sequential and reversible where rollback is supported by the project. Never rewrite an already-deployed migration; create a new migration for schema changes.
- Use environment-based configuration and secret stores. Never hardcode credentials or connection strings.
- Avoid `SaveChangesAsync` scattered across a workflow when an explicit transaction/application boundary is required.

## External gateways

- Encapsulate third-party SDKs and HTTP integrations behind application output ports.
- Use `IHttpClientFactory`/typed clients or the project's established HTTP abstraction. Keep provider request/response DTOs inside Infrastructure.
- Translate provider errors into meaningful application/domain failures when callers need provider-independent semantics.
- Configure timeouts, retries, circuit breaking, and rate-limit behavior for transient external failures using the project's approved .NET resilience approach.

## Messaging

- Keep broker clients, exchanges/topics/queues, consumers, serializers, retry policy, and dead-letter handling in Infrastructure.
- Consumers should deserialize/validate the message, map it to an application input, invoke the use case, and acknowledge/fail according to the broker contract. No business rules in consumers.
- Use idempotent consumers for messages that may be delivered more than once.
- Preserve message metadata required for tracing and operational diagnostics without leaking broker envelopes into Domain/Application.

## Cache

- Implement cache ports in Infrastructure. Keep cache-specific serialization, TTLs, keys, and invalidation mechanics outside Application/Domain.
- Define TTL and invalidation strategy per data type/use case; do not scatter arbitrary cache invalidation through business code.

## Composition root / startup

- Centralize concrete dependency registration in `Program.cs` or dedicated DI extension methods called from it.
- Keep startup responsibilities explicit: configuration -> dependency registration -> infrastructure initialization -> application hosting.
- Use .NET DI lifetimes intentionally (`Singleton`, `Scoped`, `Transient`). Match lifetime to state ownership and thread-safety.
- Support graceful shutdown with `IHostApplicationLifetime`, cancellation tokens, and proper disposal of resources.
- Expose health checks for critical dependencies using ASP.NET Core health-check conventions.
ACABA AQUI---------------
---
description: Mapping and testing rules for C#/.NET Clean Architecture, including domain/persistence/DTO mapping and unit/integration test boundaries.
alwaysApply: false
---

# Mapping & Tests — C#/.NET

## Mapping

- Centralize structural mappings between Domain, persistence models, and API/application DTOs.
- Keep mapping deterministic and free of business decisions. If a transformation enforces a business rule, that rule belongs in Domain/Application instead.
- Rebuild value objects and aggregates through constructors/factories or approved domain methods; do not bypass invariants by setting private state through reflection or persistence-only shortcuts unless the existing ORM mapping requires it.
- Prefer explicit mapping for important boundaries. Avoid magic reflection-based mapping when it obscures contract changes or domain behavior.

## Tests

- Unit-test Domain invariants as pure C# tests with no database or HTTP server.
- Unit-test Application workflows using ports/test doubles; verify orchestration and failure behavior.
- Integration-test Infrastructure implementations against the real technology where behavior depends on EF Core, SQL, brokers, Redis, or external adapters.
- API tests should verify transport concerns: routing, model binding/validation, authorization behavior, error/status mapping, and response contracts.
- Test observable behavior and architecture boundaries, not implementation details such as private methods.
