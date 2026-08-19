# Repository Instructions for Coding Agents

This file tells coding agents how to work safely and consistently in this repository. Read it before generating or modifying any code.

## Project Overview

**Smart Parking Navigator** is a .NET 10 Aspire solution that combines HDB car park static information with live availability from the data.gov.sg Car Park Availability API and displays results on a Blazor frontend.

The repository is a **workshop starter**. Application features, tests, and internal API contracts are not yet implemented. Do not invent endpoints, component interfaces, or data models beyond what exists in the current code.

## Solution Structure

```text
/
├── data/                                      # Reference data and API contract
│   ├── HDBCarparkInformation.csv              # Static HDB car park data (SVY21 coordinates)
│   ├── CarparkAvailability.json               # Published data.gov.sg OpenAPI specification
│   ├── carpark-availability-sample.json       # Representative live API response
│   └── carpark-availability.http              # HTTP request for manual API testing
├── docs/                                      # Workshop guides
├── src/
│   ├── CarparkAvailability.ApiApp/            # ASP.NET Core minimal-API backend
│   ├── CarparkAvailability.AppHost/           # .NET Aspire orchestration host
│   ├── CarparkAvailability.ServiceDefaults/   # Shared Aspire service defaults
│   └── CarparkAvailability.WebApp/            # Blazor Web App frontend
├── CarparkAvailability.slnx                   # Solution file
├── Directory.Build.props                      # Shared MSBuild properties (net10.0)
└── Directory.Packages.props                   # Central package version management
```

### Project Responsibilities

| Project | Role |
|---|---|
| **ApiApp** | ASP.NET Core minimal-API service. Fetches live availability from data.gov.sg (server-side only) and joins it with HDB static data. Receives `DataGovSg__ApiKey` from AppHost. |
| **WebApp** | Blazor Web App (interactive). Calls ApiApp via service discovery. Renders maps using Google Maps. Receives `GoogleMaps__ApiKey` from AppHost. |
| **AppHost** | .NET Aspire `DistributedApplication` host. Wires up ApiApp and WebApp, injects secrets, and exposes WebApp external HTTP endpoints. |
| **ServiceDefaults** | Extension methods for OpenTelemetry, resilience, health checks, and service discovery shared by ApiApp and WebApp. |

## Technology Stack

- **.NET 10** (SDK `10.0.100`, `rollForward: latestFeature`)
- **.NET Aspire** orchestration and service defaults
- **ASP.NET Core** minimal API (ApiApp)
- **Blazor Web App** (WebApp)
- **OpenTelemetry** for traces, metrics, and logs (via ServiceDefaults)
- **Microsoft.Extensions.ServiceDiscovery** for inter-service communication
- **Microsoft.Extensions.Http.Resilience** for HTTP client resilience
- **Central Package Management** via `Directory.Packages.props`

## Build, Run, and Validation Commands

All commands run from the repository root.

```bash
# Restore and build the full solution
dotnet build CarparkAvailability.slnx

# Run the solution (requires configured secrets — see docs/data-gov-sg-api-key.md and docs/google-maps-api-key.md)
dotnet run --project src/CarparkAvailability.AppHost

# Run only the API (no Aspire orchestration)
dotnet run --project src/CarparkAvailability.ApiApp

# Run only the frontend
dotnet run --project src/CarparkAvailability.WebApp
```

There are no automated tests yet. When tests are added, they must be runnable with:

```bash
dotnet test CarparkAvailability.slnx
```

## Data and API Contract

- The **authoritative API contract** is `data/CarparkAvailability.json` (data.gov.sg OpenAPI specification).
- `data/carpark-availability-sample.json` is a representative live response for offline development and contract tests.
- `data/HDBCarparkInformation.csv` contains static car park details. Coordinates are in **SVY21** and must be converted to **WGS84** before use on a map.
- Use `carpark_number` (availability) and `car_park_no` (HDB information) as the join key.
- Do not hard-code carpark IDs or coordinate values in production code.

## Secret Handling

- **Never commit API keys, tokens, or credentials** to the repository.
- Secrets are injected by AppHost at runtime using `AddParameterFromConfiguration`:
  - `DataGovSg__ApiKey` → ApiApp environment variable
  - `GoogleMaps__ApiKey` → WebApp environment variable
- Store secrets locally using `dotnet user-secrets` or Aspire's `aspire.config.json` mechanism. See `docs/data-gov-sg-api-key.md` and `docs/google-maps-api-key.md`.

## Service Boundary Guardrails

- **data.gov.sg must only be called from ApiApp (server-side).** Never call data.gov.sg directly from Blazor components, JavaScript, or the browser.
- **Google Maps runs in the browser via WebApp only.** The Google Maps API key must not be exposed to ApiApp or committed to source.
- WebApp communicates with ApiApp through .NET Aspire service discovery (`WithReference(api)`).

## Testing

Tests do not exist yet. When they are added:

- Follow the test patterns and naming conventions established by the first test project that is created.
- Tests must align with the PRD and TRD once those documents exist (`PRD.md`, `TRD.md`).
- Contract tests should validate live `CarparkAvailability` API responses against `data/CarparkAvailability.json`.
- Do not invent test commands or test project names until the test projects are present.

## Documentation

- `IDEATION.md` — initial product concept and feature list.
- `docs/` — workshop step guides.
- `PRD.md` and `TRD.md` will be added in a later workshop step; generated code must be consistent with them once they exist.
- Do not modify `README.md` workshop content or `docs/` guides unless explicitly asked.

## Commit and Pull-Request Guidelines

- Use the **conventional commits** format: `type(scope): short description`
  - Common types: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`
  - Examples: `feat(api): add carpark availability endpoint`, `docs: create AGENTS.md`
- Keep commits focused and atomic — one logical change per commit.
- Pull request titles must match the conventional commit format of the primary commit.
- Reference the related issue in the PR description (`Closes #N`).
- Do not merge a PR that fails `dotnet build CarparkAvailability.slnx`.
- Do not commit generated build artifacts, `bin/`, `obj/`, or `.user` files — these are covered by `.gitignore`.
