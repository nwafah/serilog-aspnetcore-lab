# Serilog in ASP.NET Core — Hands-on Lab

A practical ASP.NET Core 10 project for learning structured logging
with Serilog through small, focused experiments.

Based on [Structured Logging with Serilog in ASP.NET Core](https://codewithmukesh.com/blog/structured-logging-with-serilog-in-aspnet-core/)
by Mukesh Murugan.

## What This Lab Covers

- Two-stage initialization: bootstrap logger and application logger.
- Logging startup failures and flushing logs on shutdown.
- Log levels and source-specific overrides.
- `ILogger<T>` categories and the `SourceContext` property.
- Message templates versus string interpolation.
- Compact JSON output.
- Console, rolling File, and Seq sinks.
- Machine name, process ID, and managed thread ID enrichment.
- Structured exception details with `Serilog.Exceptions`.
- Scoped properties using `LogContext`.
- Correlation ID middleware.
- HTTP request completion logging.
- Adding business properties using `IDiagnosticContext`.

## Requirements

- .NET 10 SDK.
- Docker Desktop running with Linux containers.

## Run the Lab

Run these commands from the project directory containing
`SerilogLab.Api.csproj` and `compose.yaml`:

```powershell
docker compose up -d
dotnet restore
dotnet run --urls http://localhost:5077
```

Open Seq at:

```text
http://localhost:8081
```

The application sends logs to Seq at `http://localhost:5341`.
JSON log files are written to the `logs` directory.

## Try the Endpoints

### Order Preview

```powershell
$response = Invoke-WebRequest `
  -Uri "http://localhost:5077/lab/orders/123" `
  -Headers @{ "X-Correlation-Id" = "lab-abc-123" }

$response.Content
$response.Headers["X-Correlation-Id"]
```

This demonstrates structured properties, scoped enrichment,
and HTTP request completion logging.

The middleware uses the supplied `X-Correlation-Id`, or generates
one when the header is missing. It includes the ID in the response
and in logs produced within its request scope.

### Simulated Error

```powershell
curl.exe -i http://localhost:5077/lab/error
```

This endpoint intentionally returns HTTP 500 and logs an exception
with additional data:

```text
ExceptionDetail.Data.LabCode = PREVIEW_FAILED
```

In Seq, compare the application error log with the HTTP request
completion log. They describe different aspects of the same request.

## Explore Logs in Seq

Try these filters:

```text
OrderId = 123
```

```text
CorrelationId = 'lab-abc-123'
```

```text
StatusCode = 500
```

Inspect properties such as:

- `SourceContext`
- `MachineName`
- `ProcessId`
- `ThreadId`
- `CorrelationId`
- `OrderId`
- `Elapsed`
- `ExceptionDetail`

## Key Lessons

| Concept | Purpose |
|---|---|
| Message template | Preserves values as named, queryable properties |
| `SourceContext` | Identifies the logging source |
| Sink | Selects the destination for logs |
| Formatter | Controls the output representation |
| Enricher | Adds contextual properties to log events |
| `LogContext` | Adds properties within an execution scope |
| `IDiagnosticContext` | Adds properties to the request completion event |

A log event is the structured record produced by a logging call,
including its timestamp, level, message template, and properties.

## Next Experiments

- Mark slow requests as warnings.
- Reduce successful health-check request logs.
- Mask sensitive properties when destructuring objects.
- Export logs through the OpenTelemetry sink.
- Review logging configuration for production use.

These topics have been studied conceptually and are pending
implementation in this lab.

## Stop Seq

```powershell
docker compose down
```

This stops the containers while retaining the named volume
used for Seq data.
