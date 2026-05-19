# Backend Implementation Plan: SMAT-110 Automatic Email Alert Dispatch

## Overview

This ticket implements the **alert dispatch engine** for SMAT: when domain events occur (XML upload processed, batch closed, validation closed, etc.), the system automatically sends email notifications to the relevant users of each project without blocking the main flow.

Architecture principles applied:
- **Domain-Driven Design (DDD)**: Domain events published by command handlers; a dedicated handler (AlertaEventHandler) reacts to them in the Application layer.
- **Clean Architecture**: Domain layer stays framework-agnostic. Infrastructure holds email provider adapters and the background worker. Presentation layer exposes the history read endpoints.
- **Dependency Inversion**: `IEmailSender` and `INotificacionQueue` interfaces are defined in the Application/Domain boundary; implementations live in Infrastructure.
- **Async-first**: The main request/response cycle is never blocked by email I/O. A `Channel<T>` (in-process queue) decouples the Domain Event publication from actual email delivery.

**Prerequisite**: US-022 (TIPO_ALERTA table and its CRUD) must be merged before this ticket is started.

---

## Architecture Context

### Layers involved

| Layer | Responsibility in this ticket |
|---|---|
| **Domain** | `HistorialAlerta` entity, `INotificacionQueue` interface, Domain Event records |
| **Application** | `AlertaDespachoService`, `AlertaEventHandler<T>`, `GetHistorialAlertasQuery`, `ReintentarEnvioCommand` |
| **Infrastructure** | `SmtpEmailSender`, `SendGridEmailSender`, `NotificacionWorker` (IHostedService), EF Core migration, DI registration |
| **Presentation** | `AlertasController` (GET history + POST retry), route guards (Admin role) |

### Key components

```
src/
├── Domain/
│   └── Alertas/
│       ├── Entities/HistorialAlerta.cs
│       ├── Events/                        ← Domain event records (already published by other handlers)
│       │   ├── CargaProcesadaEvent.cs
│       │   ├── CargaErrorEvent.cs
│       │   ├── LoteCerradoEvent.cs
│       │   ├── ValidacionCerradaEvent.cs
│       │   └── ValidacionAutomaticaEvent.cs
│       └── Interfaces/INotificacionQueue.cs
├── Application/
│   └── Alertas/
│       ├── Handlers/AlertaEventHandler.cs
│       ├── Services/AlertaDespachoService.cs
│       ├── Models/NotificacionJob.cs
│       ├── Queries/GetHistorialAlertas/
│       │   ├── GetHistorialAlertasQuery.cs
│       │   └── GetHistorialAlertasHandler.cs
│       └── Commands/ReintentarEnvio/
│           ├── ReintentarEnvioCommand.cs
│           └── ReintentarEnvioHandler.cs
├── Infrastructure/
│   ├── Email/
│   │   ├── IEmailSender.cs
│   │   ├── SmtpEmailSender.cs
│   │   └── SendGridEmailSender.cs
│   ├── Queue/
│   │   └── InMemoryNotificacionQueue.cs
│   ├── Workers/
│   │   └── NotificacionWorker.cs
│   ├── Persistence/
│   │   └── Migrations/XXXX_AddHistorialAlertas.cs
│   └── DependencyInjection/AlertasServiceExtensions.cs
└── Presentation/
    └── Controllers/AlertasController.cs
```

---

## Implementation Steps

### Step 0: Create Feature Branch

- **Action**: Create and switch to a new feature branch.
- **Branch name**: `feature/SMAT-110-backend`
- **Implementation steps**:
  1. Ensure you are on `develop` (or `main`) and it is up to date:
     ```bash
     git checkout develop && git pull origin develop
     ```
  2. Create the feature branch:
     ```bash
     git checkout -b feature/SMAT-110-backend
     ```
  3. Verify:
     ```bash
     git branch
     ```
- **Notes**: All code changes must happen inside this branch. Never commit directly to `develop` or `main`.

---

### Step 1: Define Domain Event Contracts

- **Files**:
  - `src/Domain/Alertas/Events/CargaProcesadaEvent.cs`
  - `src/Domain/Alertas/Events/CargaErrorEvent.cs`
  - `src/Domain/Alertas/Events/LoteCerradoEvent.cs`
  - `src/Domain/Alertas/Events/ValidacionCerradaEvent.cs`
  - `src/Domain/Alertas/Events/ValidacionAutomaticaEvent.cs`
- **Action**: Define each event as a `record` implementing `INotification` (MediatR). Add only the fields strictly required by `AlertaDespachoService` for recipient resolution and template rendering.
- **Function signatures**:
  ```csharp
  public record CargaProcesadaEvent(Guid ObraId, Guid CargaId, int Semana, string AnalistaEmail) : INotification;
  public record CargaErrorEvent(Guid ObraId, Guid CargaId, string ErrorDetalle, string AnalistaEmail) : INotification;
  public record LoteCerradoEvent(Guid ObraId, Guid LoteId, Guid ControladorId, int Semana) : INotification;
  public record ValidacionCerradaEvent(Guid ObraId, Guid ValidacionId, Guid ValidadorId, int Semana, string Tipo) : INotification;
  public record ValidacionAutomaticaEvent(Guid ObraId, Guid ValidacionId, int Semana) : INotification;
  ```
- **Implementation notes**:
  - If these events already exist in the codebase (from US-008, US-012, US-013 work), verify their fields match the contracts above. Add missing fields — do not break existing usages.
  - `Tipo` on `ValidacionCerradaEvent` is `"MANUAL"` or `"AUTOMATICA"`.

---

### Step 2: Define `INotificacionQueue` Interface (Domain boundary)

- **File**: `src/Domain/Alertas/Interfaces/INotificacionQueue.cs`
- **Action**: Define the queue abstraction that decouples `AlertaDespachoService` from the concrete `Channel<T>`.
- **Function signature**:
  ```csharp
  public interface INotificacionQueue
  {
      ValueTask EnqueueAsync(NotificacionJob job, CancellationToken ct = default);
      IAsyncEnumerable<NotificacionJob> ReadAllAsync(CancellationToken ct);
  }
  ```
- **Implementation notes**:
  - Keep this interface in the Domain/Application boundary so it has no Infrastructure dependency.
  - `NotificacionJob` is a simple record (see Step 3).

---

### Step 3: Define `NotificacionJob` Model

- **File**: `src/Application/Alertas/Models/NotificacionJob.cs`
- **Action**: Define the data transfer object enqueued by `AlertaDespachoService` and consumed by `NotificacionWorker`.
- **Function signature**:
  ```csharp
  public record NotificacionJob(
      Guid TipoAlertaId,
      IReadOnlyList<string> Destinatarios,
      string Asunto,
      string Cuerpo
  );
  ```

---

### Step 4: Define `IEmailSender` Interface

- **File**: `src/Infrastructure/Email/IEmailSender.cs`
- **Action**: Define the email sending abstraction.
- **Function signature**:
  ```csharp
  public interface IEmailSender
  {
      Task SendAsync(EmailMessage message, CancellationToken ct = default);
  }

  public record EmailMessage(
      IReadOnlyList<string> To,
      string Subject,
      string HtmlBody
  );
  ```
- **Implementation notes**:
  - Place this interface in Infrastructure (not Domain) since it has no bearing on business rules — it is a pure I/O concern.

---

### Step 5: Implement `AlertaDespachoService`

- **File**: `src/Application/Alertas/Services/AlertaDespachoService.cs`
- **Action**: Core service that, given a domain event context, resolves recipients, renders the template, and enqueues the job.
- **Dependencies**: `IAlertaRepository` (from US-022 — reads `TIPO_ALERTA`), `IUsuarioObraPerfilRepository` (resolves emails by role+obra), `INotificacionQueue`.
- **Function signature**:
  ```csharp
  public class AlertaDespachoService
  {
      public AlertaDespachoService(
          IAlertaRepository alertaRepository,
          IUsuarioObraPerfilRepository usuarioObraPerfilRepository,
          INotificacionQueue queue,
          ILogger<AlertaDespachoService> logger) { }

      public async Task DespacharAsync(
          string codigoEvento,
          Guid obraId,
          Dictionary<string, string> contexto,
          CancellationToken ct = default) { }
  }
  ```
- **Implementation steps**:
  1. Call `alertaRepository.GetActivosPorEventoAsync(codigoEvento)` to get all active `TipoAlerta` records for the event.
  2. If no active types exist, log debug and return (no error).
  3. For each `TipoAlerta`:
     a. Resolve recipients: call `usuarioObraPerfilRepository.GetEmailsByRolAsync(obraId, tipoAlerta.RolDestinatario)`.
     b. Filter out entries where email is null/empty — log a warning per skipped user.
     c. If recipient list is empty after filtering, skip this `TipoAlerta`.
     d. Render the subject and body by replacing template variables (e.g., `{{obra}}`, `{{semana}}`) with values from `contexto` dictionary.
     e. Enqueue a `NotificacionJob(tipoAlerta.Id, destinatarios, asuntoRendered, cuerpoRendered)`.
  4. Wrap in try/catch; log errors but do not rethrow — dispatch failures must never propagate to the caller.
- **Implementation notes**:
  - Template variable replacement: simple `string.Replace("{{key}}", value)` loop over `contexto`. Do not use external template engines for now.
  - `contexto` dictionary keys: `obra`, `semana`, `analista`, `validador`, `controlador`, `errorDetalle`. Callers (AlertaEventHandler) populate only the keys relevant to their event.

---

### Step 6: Implement `AlertaEventHandler<T>`

- **File**: `src/Application/Alertas/Handlers/AlertaEventHandler.cs`
- **Action**: Generic MediatR `INotificationHandler<T>` that translates each domain event into a `DespacharAsync` call.
- **Function signature**:
  ```csharp
  public class AlertaEventHandler<TEvent> : INotificationHandler<TEvent>
      where TEvent : INotification
  {
      public AlertaEventHandler(
          AlertaDespachoService despachoService,
          IEventoContextoResolver<TEvent> contextResolver) { }

      public async Task Handle(TEvent notification, CancellationToken cancellationToken) { }
  }
  ```
  Alternatively (simpler, preferred for this ticket size), implement one handler per event type:
  ```csharp
  public class CargaProcesadaAlertaHandler : INotificationHandler<CargaProcesadaEvent>
  {
      public async Task Handle(CargaProcesadaEvent notification, CancellationToken ct)
      {
          var contexto = new Dictionary<string, string>
          {
              ["obra"] = notification.ObraId.ToString(),
              ["semana"] = notification.Semana.ToString(),
              ["analista"] = notification.AnalistaEmail
          };
          await _despachoService.DespacharAsync("CARGA_PROCESADA", notification.ObraId, contexto, ct);
      }
  }
  ```
  Repeat for: `CargaErrorEvent` → `"CARGA_ERROR"`, `LoteCerradoEvent` → `"LOTE_CERRADO"`, `ValidacionCerradaEvent` → `"VALIDACION_CERRADA"`, `ValidacionAutomaticaEvent` → `"VALIDACION_AUTOMATICA"`.
- **Implementation notes**:
  - One concrete handler class per event keeps the code readable and avoids reflection-heavy generics.
  - Each handler is registered automatically by MediatR's `AddMediatR(assemblies)` scan — no manual DI registration needed.

---

### Step 7: Instrument Existing Command Handlers to Publish Domain Events

- **Files to modify** (already exist — add `IPublisher.Publish` calls):
  - `src/Application/Cargas/Commands/ProcessXmlCommand.cs` (or Handler) → publish `CargaProcesadaEvent` on success, `CargaErrorEvent` on failure.
  - `src/Application/Lotes/Commands/CerrarLoteCommand.cs` → publish `LoteCerradoEvent` after `UPDATE LOTE_AVANCES SET fechaCierre`.
  - `src/Application/Validaciones/Commands/CerrarValidacionCommand.cs` → publish `ValidacionCerradaEvent` after INSERT VALIDACION_SEMANA.
  - `src/Application/Validaciones/Commands/CierreAutomaticoCommand.cs` → publish `ValidacionAutomaticaEvent` after INSERT VALIDACION_SEMANA APROBADO_AUTOMATICO.
- **Implementation steps** (same pattern for each):
  1. Inject `IPublisher _publisher` via constructor.
  2. After the `await _context.SaveChangesAsync(ct)` (or equivalent success point), add:
     ```csharp
     await _publisher.Publish(new CargaProcesadaEvent(obraId, cargaId, semana, analista), ct);
     ```
  3. Publish **after** the transaction commits to avoid sending alerts for rolled-back operations.
  4. Wrap the Publish call in try/catch — event handler failures must not roll back the command transaction.
- **Implementation notes**:
  - Use `IPublisher` (fire-and-forget style with `_ =` is acceptable but using `await` is safer for error logging).
  - Do not add `IPublisher` calls inside try/catch that swallows DB exceptions — the event must only fire on successful persistence.

---

### Step 8: Implement `InMemoryNotificacionQueue`

- **File**: `src/Infrastructure/Queue/InMemoryNotificacionQueue.cs`
- **Action**: Concrete implementation of `INotificacionQueue` using `System.Threading.Channels.Channel<T>`.
- **Function signature**:
  ```csharp
  public class InMemoryNotificacionQueue : INotificacionQueue
  {
      private readonly Channel<NotificacionJob> _channel;

      public InMemoryNotificacionQueue(IOptions<NotificacionesOptions> options)
      {
          _channel = Channel.CreateBounded<NotificacionJob>(
              new BoundedChannelOptions(options.Value.ChannelCapacity)
              {
                  FullMode = BoundedChannelFullMode.DropOldest
              });
      }

      public ValueTask EnqueueAsync(NotificacionJob job, CancellationToken ct = default)
          => _channel.Writer.WriteAsync(job, ct);

      public IAsyncEnumerable<NotificacionJob> ReadAllAsync(CancellationToken ct)
          => _channel.Reader.ReadAllAsync(ct);
  }
  ```
- **Implementation notes**:
  - `DropOldest` mode ensures the main API thread is never blocked if the worker falls behind.
  - Register as `Singleton` — the channel must be shared between the API (writer) and the Worker (reader).
  - `NotificacionesOptions.ChannelCapacity` default: `500`.

---

### Step 9: Implement `SmtpEmailSender` and `SendGridEmailSender`

- **File**: `src/Infrastructure/Email/SmtpEmailSender.cs`
  ```csharp
  public class SmtpEmailSender : IEmailSender
  {
      private readonly SmtpOptions _options;
      public async Task SendAsync(EmailMessage message, CancellationToken ct)
      {
          using var client = new SmtpClient();
          await client.ConnectAsync(_options.Host, _options.Port, SecureSocketOptions.StartTls, ct);
          await client.AuthenticateAsync(_options.User, _options.Password, ct);
          var mime = BuildMimeMessage(message);
          await client.SendAsync(mime, ct);
          await client.DisconnectAsync(true, ct);
      }
  }
  ```
- **File**: `src/Infrastructure/Email/SendGridEmailSender.cs`
  ```csharp
  public class SendGridEmailSender : IEmailSender
  {
      private readonly SendGridClient _client;
      public async Task SendAsync(EmailMessage message, CancellationToken ct)
      {
          var msg = MailHelper.CreateSingleEmailToMultipleRecipients(
              from: new EmailAddress(_options.FromEmail),
              tos: message.To.Select(e => new EmailAddress(e)).ToList(),
              subject: message.Subject,
              plainTextContent: null,
              htmlContent: message.HtmlBody);
          var response = await _client.SendEmailAsync(msg, ct);
          if (!response.IsSuccessStatusCode)
              throw new EmailSendException($"SendGrid returned {response.StatusCode}");
      }
  }
  ```
- **Implementation notes**:
  - NuGet packages required: `MailKit` (SMTP), `SendGrid` (SendGrid SDK).
  - Active provider is selected in `AlertasServiceExtensions` based on `Email:Provider` config key.
  - `EmailSendException` is a custom exception defined in Infrastructure — the Worker catches it to trigger Polly retries.

---

### Step 10: Implement `HistorialAlerta` Entity and EF Core Migration

- **File**: `src/Domain/Alertas/Entities/HistorialAlerta.cs`
  ```csharp
  public class HistorialAlerta
  {
      public Guid Id { get; private set; } = Guid.NewGuid();
      public Guid TipoAlertaId { get; private set; }
      public List<string> Destinatarios { get; private set; } = new();
      public string Asunto { get; private set; } = string.Empty;
      public string Estado { get; private set; } // "ENVIADO" | "ERROR"
      public string? ErrorDetalle { get; private set; }
      public int Intentos { get; private set; } = 1;
      public DateTimeOffset SentAt { get; private set; } = DateTimeOffset.UtcNow;
      public DateTimeOffset CreatedAt { get; private set; } = DateTimeOffset.UtcNow;

      public static HistorialAlerta Enviado(Guid tipoAlertaId, List<string> destinatarios, string asunto, int intentos)
          => new() { TipoAlertaId = tipoAlertaId, Destinatarios = destinatarios, Asunto = asunto, Estado = "ENVIADO", Intentos = intentos };

      public static HistorialAlerta Error(Guid tipoAlertaId, List<string> destinatarios, string asunto, string errorDetalle, int intentos)
          => new() { TipoAlertaId = tipoAlertaId, Destinatarios = destinatarios, Asunto = asunto, Estado = "ERROR", ErrorDetalle = errorDetalle, Intentos = intentos };
  }
  ```
- **Migration**: `dotnet ef migrations add AddHistorialAlertas`
  Key mappings in `OnModelCreating`:
  ```csharp
  builder.Entity<HistorialAlerta>(e =>
  {
      e.HasKey(x => x.Id);
      e.Property(x => x.Destinatarios).HasColumnType("jsonb");
      e.Property(x => x.Estado).HasMaxLength(10);
      e.HasIndex(x => x.TipoAlertaId);
      e.HasIndex(x => x.Estado);
      e.HasIndex(x => x.SentAt);
  });
  ```
- **Retention job**: Add a `IHostedService` (or use the existing `NotificacionWorker`) to run daily:
  ```csharp
  await _context.HistorialAlertas
      .Where(h => h.SentAt < DateTimeOffset.UtcNow.AddDays(-90))
      .ExecuteDeleteAsync(ct);
  ```

---

### Step 11: Implement `NotificacionWorker`

- **File**: `src/Infrastructure/Workers/NotificacionWorker.cs`
- **Action**: `BackgroundService` (IHostedService) that dequeues jobs and sends emails with Polly retry + circuit breaker.
- **Function signature**:
  ```csharp
  public class NotificacionWorker : BackgroundService
  {
      protected override async Task ExecuteAsync(CancellationToken stoppingToken)
      {
          await foreach (var job in _queue.ReadAllAsync(stoppingToken))
              await ProcessJobAsync(job, stoppingToken);
      }

      private async Task ProcessJobAsync(NotificacionJob job, CancellationToken ct)
      {
          int intentos = 0;
          try
          {
              await _retryPolicy.ExecuteAsync(async () =>
              {
                  intentos++;
                  await _emailSender.SendAsync(
                      new EmailMessage(job.Destinatarios, job.Asunto, job.Cuerpo), ct);
              });
              await SaveHistorial(HistorialAlerta.Enviado(job.TipoAlertaId, job.Destinatarios.ToList(), job.Asunto, intentos), ct);
          }
          catch (Exception ex)
          {
              _logger.LogError(ex, "Failed to send notification for TipoAlerta {Id} after {Intentos} attempts", job.TipoAlertaId, intentos);
              await SaveHistorial(HistorialAlerta.Error(job.TipoAlertaId, job.Destinatarios.ToList(), job.Asunto, ex.Message, intentos), ct);
          }
      }
  }
  ```
- **Polly policy** (configured in `AlertasServiceExtensions`):
  ```csharp
  Policy
    .Handle<EmailSendException>()
    .WaitAndRetryAsync(3, attempt => TimeSpan.FromSeconds(Math.Pow(2, attempt) * 5))
    .WrapAsync(
      Policy.Handle<EmailSendException>()
            .CircuitBreakerAsync(5, TimeSpan.FromMinutes(2)));
  ```
- **Implementation notes**:
  - Register `NotificacionWorker` via `services.AddHostedService<NotificacionWorker>()`.
  - `_context` for SaveHistorial must be resolved from a scoped factory (`IServiceScopeFactory`) since workers are singletons.
  - Circuit breaker opens after 5 consecutive failures — worker logs a warning and sleeps briefly before retrying the next item.

---

### Step 12: Implement REST Endpoints — `AlertasController`

- **File**: `src/Presentation/Controllers/AlertasController.cs`
- **Action**: Thin controller exposing two endpoints. Uses MediatR to dispatch queries/commands.
- **Endpoints**:

  **GET /api/alertas/historial**
  ```
  Query params: evento (string?), estado (string?), desde (DateTimeOffset?), hasta (DateTimeOffset?), page (int=1), pageSize (int=20)
  Auth: [Authorize(Roles = "Admin")]
  Response 200: { items: [...], total: int, page: int, pageSize: int }
  Response 403: if not Admin
  ```

  **POST /api/alertas/historial/{historialId}/reintento**
  ```
  Auth: [Authorize(Roles = "Admin")]
  Response 200: { historialId, estado, sentAt }
  Response 404: HistorialAlerta not found
  Response 422: if estado != "ERROR"
  Response 403: if not Admin
  ```
- **Implementation steps**:
  1. `[HttpGet("historial")]` → dispatch `GetHistorialAlertasQuery` → return paginated result.
  2. `[HttpPost("historial/{historialId:guid}/reintento")]` → dispatch `ReintentarEnvioCommand(historialId)` → on success return 200, on `NotFoundException` return 404, on `InvalidStateException` return 422.
- **Implementation notes**:
  - The controller must NOT contain business logic — only map HTTP params to query/command objects and map results to HTTP responses.
  - Add `[ApiController]` and `[Route("api/alertas")]` attributes.

---

### Step 13: Register Dependencies (`AlertasServiceExtensions`)

- **File**: `src/Infrastructure/DependencyInjection/AlertasServiceExtensions.cs`
- **Action**: Single extension method called from `Program.cs` / `Startup.cs` to register all alertas-related services.
- **Implementation**:
  ```csharp
  public static IServiceCollection AddAlertas(this IServiceCollection services, IConfiguration config)
  {
      services.Configure<NotificacionesOptions>(config.GetSection("Notificaciones"));
      services.Configure<EmailOptions>(config.GetSection("Email"));

      // Queue — singleton so worker and API share the same channel
      services.AddSingleton<INotificacionQueue, InMemoryNotificacionQueue>();

      // Email sender — chosen by config
      var provider = config["Email:Provider"] ?? "smtp";
      if (provider == "sendgrid")
          services.AddScoped<IEmailSender, SendGridEmailSender>();
      else
          services.AddScoped<IEmailSender, SmtpEmailSender>();

      // Worker
      services.AddHostedService<NotificacionWorker>();

      // Application services (scoped)
      services.AddScoped<AlertaDespachoService>();

      return services;
  }
  ```

---

### Step 14: Write Unit Tests

- **Files**:
  - `tests/Application.Tests/Alertas/AlertaDespachoServiceTests.cs`
  - `tests/Application.Tests/Alertas/AlertaEventHandlerTests.cs`
  - `tests/Infrastructure.Tests/Workers/NotificacionWorkerTests.cs`

#### `AlertaDespachoServiceTests`
- **Setup**: Mock `IAlertaRepository`, `IUsuarioObraPerfilRepository`, `INotificacionQueue`.
- **Test cases**:
  1. **Active TipoAlerta exists → job enqueued**: given one active `TipoAlerta` for `CARGA_PROCESADA` and one recipient email, `EnqueueAsync` is called once with correct subject/body.
  2. **No active TipoAlerta → nothing enqueued**: given zero active types, `EnqueueAsync` is never called.
  3. **Recipient without email → skipped with warning**: given one recipient with null email, `EnqueueAsync` is not called; logger captures warning.
  4. **Multiple TipoAlerta → multiple jobs enqueued**: given 2 active types, `EnqueueAsync` is called twice.
  5. **Template variables resolved correctly**: subject `"Carga {{obra}} semana {{semana}}"` with context `{obra: "Norte", semana: "18"}` → `"Carga Norte semana 18"`.

#### `NotificacionWorkerTests`
- **Setup**: Mock `INotificacionQueue`, `IEmailSender`, `IServiceScopeFactory` (for `IHistorialAlertaRepository`).
- **Test cases**:
  1. **Successful send → HISTORIAL estado ENVIADO**: `SendAsync` succeeds → `HistorialAlerta.Enviado` saved.
  2. **SMTP fails 3 times → estado ERROR recorded**: `SendAsync` throws `EmailSendException` three times → Polly exhausts retries → `HistorialAlerta.Error` saved with `Intentos = 3`.
  3. **First attempt fails, second succeeds → ENVIADO with Intentos = 2**.

#### `AlertaEventHandlerTests`
- **Test cases**:
  1. **CargaProcesadaEvent → DespacharAsync called with correct codigoEvento and contexto**.
  2. **Handler does not throw when DespacharAsync throws** (resilience check).

---

### Step 15: Update Technical Documentation

- **Action**: Review all changes and update the affected documentation files.
- **Implementation steps**:
  1. **`docs/data-model.md`**: Add `HISTORIAL_ALERTAS` entity description (fields, validation rules, relationships with `TIPO_ALERTA`). Add retention policy note (90 days).
  2. **`docs/api-spec.yml`**: Add `GET /api/alertas/historial` and `POST /api/alertas/historial/{id}/reintento` endpoint definitions with request parameters, response schemas, and error codes (403, 404, 422).
  3. **`docs/backend-standards.mdc`**: If a new architectural pattern was used (e.g., `Channel<T>` bounded queue, `DropOldest` strategy), add a note in the "Architecture Patterns" or "Background Services" section.
  4. All documentation must be written in **English** per `base-standards.mdc`.
- **Notes**: This step is **mandatory** before the PR is opened.

---

## Implementation Order

1. Step 0 — Create feature branch `feature/SMAT-110-backend`
2. Step 1 — Define / verify Domain Event contracts
3. Step 2 — `INotificacionQueue` interface
4. Step 3 — `NotificacionJob` model
5. Step 4 — `IEmailSender` interface + `EmailMessage` record
6. Step 5 — `AlertaDespachoService`
7. Step 6 — `AlertaEventHandler` (one per event)
8. Step 7 — Instrument existing command handlers to publish events
9. Step 8 — `InMemoryNotificacionQueue`
10. Step 9 — `SmtpEmailSender` + `SendGridEmailSender`
11. Step 10 — `HistorialAlerta` entity + EF Core migration
12. Step 11 — `NotificacionWorker`
13. Step 12 — `AlertasController` (REST endpoints)
14. Step 13 — DI registration (`AlertasServiceExtensions`)
15. Step 14 — Unit tests (90% coverage threshold)
16. Step 15 — Documentation update (mandatory before PR)

---

## Testing Checklist

- [ ] `AlertaDespachoServiceTests`: 5 test cases pass (active types, no types, missing email, multiple types, template resolution).
- [ ] `NotificacionWorkerTests`: 3 test cases pass (success, 3-retry failure, partial retry success).
- [ ] `AlertaEventHandlerTests`: 2 test cases pass.
- [ ] `GET /api/alertas/historial` returns 200 with paginated list when Admin; 403 when not Admin.
- [ ] `POST /api/alertas/historial/{id}/reintento` returns 200 on success; 404 when ID not found; 422 when estado != ERROR.
- [ ] End-to-end: trigger `CargaProcesadaEvent` → verify email sent to recipient → verify `HISTORIAL_ALERTAS` row with estado `ENVIADO`.
- [ ] SMTP unavailable scenario: 3 retries → `HISTORIAL_ALERTAS` row with estado `ERROR`, `Intentos = 3`.
- [ ] Main request thread completes without blocking on email I/O (verify with integration test timing).

---

## Error Response Format

```json
// 404 Not Found
{ "error": "HistorialAlerta not found", "code": "HISTORIAL_NOT_FOUND" }

// 422 Unprocessable Entity
{ "error": "Retry is only allowed for failed alerts (estado = ERROR)", "code": "INVALID_ALERT_STATE" }

// 403 Forbidden
{ "error": "Access denied", "code": "FORBIDDEN" }

// 500 Internal Server Error
{ "error": "An unexpected error occurred", "code": "INTERNAL_ERROR" }
```

HTTP status code mapping:
| Scenario | Status |
|---|---|
| Successful list | 200 |
| Successful retry | 200 |
| HistorialAlerta not found | 404 |
| Estado is not ERROR | 422 |
| Not Admin | 403 |
| Unexpected exception | 500 |

---

## Dependencies

| Package | Purpose | Registration |
|---|---|---|
| `MediatR` | Domain event publication + query/command dispatch | Already registered |
| `MailKit` | SMTP email sending | `SmtpEmailSender` |
| `SendGrid` | SendGrid email SDK | `SendGridEmailSender` |
| `Polly` | Retry + circuit breaker for email delivery | `NotificacionWorker` |
| `Microsoft.Extensions.Hosting` | `BackgroundService` base class | Already present |
| `Npgsql.EntityFrameworkCore.PostgreSQL` | JSONB support for `Destinatarios` column | Already present |

---

## Notes

- **Secrets management**: `Email:Smtp:Password` and `Email:SendGrid:ApiKey` must **never** be committed to `appsettings.json`. Use `dotnet user-secrets` locally and environment variables / Azure Key Vault in production.
- **Event ordering**: Always publish domain events **after** `SaveChangesAsync` to guarantee the event only fires on committed data.
- **Channel capacity**: Default 500. If load tests reveal the channel fills frequently, consider migrating to Redis Streams or Azure Service Bus for durability across worker restarts.
- **Concurrent workers**: `NotificacionWorker` is a single-instance background service. Do not register it multiple times.
- **Retries and circuit breaker**: Circuit breaker opens after 5 consecutive `EmailSendException` failures within a 2-minute window. During open state, jobs are still dequeued but immediately saved as ERROR without attempting delivery — prevents queue backup.
- **US-022 dependency**: `IAlertaRepository.GetActivosPorEventoAsync` must be available (implemented in US-022). Confirm the method signature before starting Step 5.
- **Language**: All code, tests, comments, log messages, and documentation in **English** per `base-standards.mdc`.

---

## Next Steps After Implementation

1. Open PR `feature/SMAT-110-backend → develop` with the template and link to this plan.
2. Request review from at least one backend peer (architectural compliance + Polly usage).
3. After merge, notify frontend team to proceed with SMAT-110 frontend work (`plan-frontend-ticket`).
4. After frontend is merged, run the integration test suite end-to-end in staging.
5. Configure `Email:Provider`, SMTP/SendGrid credentials in the staging environment before testing.

---

## Implementation Verification

### Code Quality
- [ ] All new code is fully typed (no `any`, no implicit `object`)
- [ ] No business logic in controllers
- [ ] `AlertaDespachoService` does not reference `IEmailSender` directly (only the Worker does)
- [ ] Domain Events are published post-`SaveChangesAsync`
- [ ] No secrets in committed files

### Functionality
- [ ] Email sent correctly for all 5 domain event types
- [ ] No-alert-type scenario handled silently
- [ ] Missing recipient email skipped with warning log
- [ ] Retry policy: 3 attempts with exponential backoff
- [ ] Circuit breaker activates after 5 consecutive failures
- [ ] History persisted for both ENVIADO and ERROR states
- [ ] Retention job deletes records older than 90 days

### Testing
- [ ] 90% coverage threshold met across branches, functions, lines, statements
- [ ] FakeEmailSender used in all unit tests (no real SMTP calls)
- [ ] AAA pattern followed in all test methods

### Integration
- [ ] `AddAlertas()` called in `Program.cs`
- [ ] Migration applied: `HISTORIAL_ALERTAS` table created with indexes
- [ ] `Channel<T>` registered as Singleton — shared between API and Worker

### Documentation
- [ ] `docs/data-model.md` updated with `HISTORIAL_ALERTAS` entity
- [ ] `docs/api-spec.yml` updated with new endpoints
- [ ] `docs/backend-standards.mdc` updated if new patterns introduced
