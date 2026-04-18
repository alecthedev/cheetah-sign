# Setup of the Development Environment

## Overview

This project is built with Vue.js + TypeScript, .NET (C#), and PostgreSQL. The stack is containerized with Docker; the usual path is `docker compose up` from the API repo root (see [Run modes](#run-modes)). You can also run the API and webclient on the host for debugging (PostgreSQL still required).

Canonical developer setup for this product lives in this file. Deployment and operations are covered separately in [Deployment.md](./Deployment.md).

**Support contacts:** The addresses below are from the student capstone team; replace or extend them when the product is owned by another organization. For access to the Bitbucket repository, use your Accutech or successor process.

- gabe.chandler@trustasc.com  
- kade.dentel@trustasc.com  

---

## Current project state (handoff)

- **Product status:** User acceptance scenarios in [uat.md](./uat.md) are marked **Client Accepted** for the listed features (upload, library/favorites, send flows, packets, builder, signing, jobs/audit, downloads, etc.). Treat that file as the formal acceptance record.
- **Engineering backlog:** Non-blocking improvements (for example packet/job schema cleanup, signing UX polish, or splitting endpoint tests into a dedicated integration test project) should be captured in your team’s issue tracker or backlog. [uat.md](./uat.md) records what the client accepted; treat anything beyond that as normal product maintenance unless your organization agrees otherwise.

---

## Environment and tooling

| Tool | Expected | Notes |
|------|----------|--------|
| **Docker** | Docker Desktop or Docker Engine + Compose v2 | Required for the recommended full-stack and integration-test flows. |
| **Node.js** | 20.x | Matches `cheetahsign-webclient` tooling; use `node -v` to verify. |
| **npm** | Comes with Node | Used for the webclient; lockfile is `package-lock.json`. |
| **.NET SDK** | 8.x | `dotnet --info` should show SDK 8. |
| **Vite** | 6.x (see `cheetahsign-webclient/package.json`) | Dev server defaults to port **8080**. |

**OS:** Developers use Linux, macOS, and Windows. Paths below use forward slashes; on Windows use your shell (Git Bash, WSL, or PowerShell) accordingly.

---

## High-level system architecture

Cheetah Sign is split into four main parts:

| Module | Role | How it connects |
|--------|------|-----------------|
| **Frontend (webclient)** | Vue 3 + Vite app; admin UI (upload, document builder, send jobs, client list) and signer-facing signing page. | Browser → HTTP to the API (SDK under `cheetahsign-webclient/src/sdk/`; axios `baseURL` is `/api`, proxied in dev). |
| **API (Cheetah.Sign.Api)** | .NET 8 ASP.NET Core app; REST endpoints, business logic, PDF handling, email. | Listens on **8080 inside the API container**; on the host, Compose maps that to **6001** (see [Ports](#ports)). Uses PostgreSQL; sends mail via SMTP when configured. |
| **Database (PostgreSQL)** | Stores documents, packets, jobs, signers, clients, audit. | API uses `ConnectionStrings:DefaultConnection` (and compose overrides via env). |
| **Docker** | Orchestrates Postgres, optional pgAdmin, API, webclient. | `docker compose up` from `bsu.cheetah.sign.2025/`; `docker-compose-tests.yml` is a separate stack for frontend integration tests. |

**Major components for onboarding:** Webclient (`src/components`, `src/sdk`, router); API (`Endpoints/`, `Services/`, `Interfaces/`, `Program.cs`); Postgres schema via `Contexts/AppDbContext.cs` and EF migrations.

---

## Ports (full Docker stack)

When you run the main `docker-compose.yml`:

| Service | Host port | Purpose |
|---------|-----------|---------|
| Webclient (Vite) | **8080** | Admin and signer UI: http://localhost:8080/ |
| API | **6001** | HTTP API (container port 8080 → host 6001) |
| PostgreSQL | **5432** | Database (matches `DefaultConnection` in appsettings for localhost) |
| pgAdmin | **8181** | DB UI (credentials in compose file) |

---

## Cloning the repository

Source repository (Bitbucket):

https://bitbucket.org/accutechcapstone/bsu.cheetah.sign.2025

Clone with SSH or HTTPS, then open the **`bsu.cheetah.sign.2025`** folder. You should see **Cheetah.Sign.Api** (backend), **cheetahsign-webclient** (frontend), and Compose files **`docker-compose.yml`** and **`docker-compose-tests.yml`**.

*(Optional screenshots for onboarding can live under `Documentation/images/` next to this file if your team adds them.)*

---

## Prerequisites

### Installation

- [Docker](https://docs.docker.com/get-docker/) with Compose v2 (`docker compose version`).
- IDE (e.g. [VS Code](https://code.visualstudio.com/) with the [Docker extension](https://code.visualstudio.com/docs/containers/overview)).
- For local webclient development: **Node.js 20** and **npm**.
- For local API development: **.NET 8 SDK**.

### Docker login (if Compose pull/build fails with auth errors)

In a terminal:

```bash
docker login -u <username>
```

---

## Configuration reference

Configuration uses the usual ASP.NET Core order: **environment variables** (including those set by Docker Compose), then **user secrets** (local Development only), then **appsettings.json**.

### Database

| Config key | Purpose |
|------------|---------|
| `ConnectionStrings:DefaultConnection` | Npgsql connection string used in non-Testing environments (`Program.cs`). |

In **Docker**, Compose sets `ConnectionStrings__DefaultConnection` for the API container so the API reaches the `sign-pgdata` host (see `docker-compose.yml`).

For **`dotnet run`** on the host, `appsettings.json` defaults to `Server=localhost;Port=5432;...` — run PostgreSQL on 5432 or change the connection string / user secrets to match your instance.

### Email (SMTP)

| Config key | Env var (Docker / shell) | Notes |
|------------|---------------------------|--------|
| `Email:SmtpHost` | `Email__SmtpHost` | Required when sending mail. |
| `Email:SmtpPort` | `Email__SmtpPort` | Default 587 if omitted. |
| `Email:FromAddress` | `Email__FromAddress` | Required when sending. |
| `Email:FromName` | `Email__FromName` | Optional display name. |
| `Email:UserName` | `Email__UserName` | SMTP auth. |
| `Email:Password` | `Email__Password` | SMTP auth. |

Copy **`bsu.cheetah.sign.2025/.env.example`** to **`.env`** beside `docker-compose.yml`, fill in values, and do not commit `.env`. Containers do not read the host’s `dotnet user-secrets` store.

---

## Run modes

### 1. Full stack with Docker (recommended)

From **`bsu.cheetah.sign.2025/`**:

```bash
docker compose up
```

First build can take several minutes. When healthy, open http://localhost:8080/ for the UI. The API is available at http://localhost:6001/ (e.g. Swagger in Development).

**Smoke test:** Upload a document; if it appears in the library, the stack is wired correctly.

Inside Compose, the Vite dev server proxies `/api` to the `sign-api` service (`cheetahsign-webclient/vite.config.ts`).

### 2. API on the host + webclient on the host

Use this when you want breakpoints in both tiers without rebuilding images.

1. Start **PostgreSQL** with credentials matching `ConnectionStrings:DefaultConnection` (or override via user secrets / env).
2. From **`Cheetah.Sign.Api/`**, run `dotnet run` (HTTP profile uses **http://localhost:6001** per `Properties/launchSettings.json`).
3. Configure **SMTP** via user secrets or environment variables if you will exercise email.
4. From **`cheetahsign-webclient/`**, the Vite config proxies `/api` to **`http://sign-api:8080`**, which only resolves **inside Docker**. For a purely local loop, point the proxy at your local API, e.g. in `vite.config.ts` set `server.proxy["/api"].target` to `http://localhost:6001`, run `npm install` once, then `npm run dev` (port **8080**).

### 3. Integration test stack (webclient integration tests)

Frontend integration tests expect the **test** Compose file and a cleared test database workflow (see [Running integration tests](#running-integration-tests)).

From **`bsu.cheetah.sign.2025/`**:

```bash
docker compose -f docker-compose-tests.yml up
```

Then in **`cheetahsign-webclient/`**:

```bash
npm run integration
```

Use **`docker-compose-tests.yml`** (with an “s”); there is no `docker-compose-test.yml`.

---

## Docker operations (developer)

- **Rebuild after dependency changes:** `docker compose build` or `docker compose up --build`.
- **Logs:** `docker compose logs -f` (optionally pass a service name).
- **Database volume:** The main compose file uses a named volume for Postgres data; removing volumes resets the DB (destructive).
- **One stack at a time:** The main `docker-compose.yml` and `docker-compose-tests.yml` both map common host ports (e.g. **5432**, **6001**). Stop one compose stack before starting the other to avoid port conflicts.

---

## Email (required to send)

The API registers **`SmtpEmailSender`** as `IEmailSender`. There is no no-op sender in production code (tests may use substitutes).

Email is read from **appsettings**, **user secrets** (local `dotnet run`), or **environment variables** / **`.env`** (Docker).

**Docker:** Put `.env` next to `docker-compose.yml`. Map keys with double underscores, e.g. `Email__SmtpHost` → `Email:SmtpHost`.

**Local API:** Example: `dotnet user-secrets set "Email:SmtpHost" "smtp.example.com"` (and the other keys under `Email:*`).

**Behavior:** The app can start without SMTP. When a flow **calls** `IEmailSender` (e.g. signing invite, final document), missing required keys cause `InvalidOperationException` from `SmtpEmailSender` (“Email configuration missing: …”). To develop without email, avoid flows that send mail or supply full SMTP settings.

---

## Database lifecycle

- **Migrations:** EF Core migrations live under **`Cheetah.Sign.Api/Migrations/`**. On startup (non-Testing environment), the API runs **`MigrateDatabase()`** (`Program.cs`) so the database schema is brought up to date automatically.
- **Changing the model:** After editing `AppDbContext` / entities, add a migration with the EF tools from the API project (see Microsoft docs for `dotnet ef migrations add`). Commit migration files with the code change.
- **Testing profile:** When `ASPNETCORE_ENVIRONMENT` is **Testing** (integration test compose), the app uses an in-memory fixture database path instead of applying migrations the same way — see `Program.cs`.

---

## Unit and integration testing

Cheetah Sign uses **Vitest** (frontend), **MSW** where applicable, and **xUnit** (backend). Backend tests use **`PgDataFixture`** / **`PgDataFixtureIntegration`** under `Contexts/` for isolated databases.

### Quality expectations (development)

- Prefer running **frontend unit tests** and **backend unit tests** before merging substantial changes.
- **Frontend integration tests** require the test Docker stack; treat failures there as regressions in API + client contract.

### Commands

| Context | Command | Purpose |
|---------|---------|---------|
| Webclient | `npm test` | Frontend unit tests |
| Webclient | `npm run coverage` | Frontend coverage |
| Webclient | `npm run integration` | Integration tests (**requires** `docker compose -f docker-compose-tests.yml up`) |
| API | `./run-backend-tests.sh` | Backend unit tests (no coverage) |
| API | `./run-backend-coverage.sh` or `run-backend-coverage.ps1` | Backend coverage + HTML report under `coveragereport/` |

**Backend coverage scripts** expect [Coverlet](https://github.com/coverlet-coverage/coverlet) and optionally [ReportGenerator](https://github.com/danielpalme/ReportGenerator) (`dotnet tool install -g dotnet-reportgenerator-globaltool`). The scripts print a summary and emit `coveragereport/index.html`.

### Running integration tests

The **`docker-compose-tests.yml`** stack must be running before `npm run integration`. That compose uses environment **Testing** so the API uses the integration fixture database and exposes mock endpoints used to reset data between tests.

Vitest integration tests call backend endpoints to prepare state; see `cheetahsign-webclient/src/integration-tests/` for details.

**Note:** A separate **`IntegrationTests/`** .NET project (WebApplicationFactory, etc.) is **not** present in the repository; backend endpoint coverage today includes **`UnitTests/`** hitting localhost where noted in those tests.

### Duplicate assembly attribute errors (.NET build)

If the build fails with **duplicate** `AssemblyCompanyAttribute`, `AssemblyVersionAttribute`, or similar **assembly info** errors, stale output is usually the cause (common after switching branches or pulling).

1. From **`bsu.cheetah.sign.2025/`**, run **`dotnet clean`** on the solution, then build again.
2. If it still fails, delete the unit test build folders: **`Cheetah.Sign.Api/UnitTests/obj`** and **`Cheetah.Sign.Api/UnitTests/bin`**, then rebuild.

See also the troubleshooting table in [Deployment.md](./Deployment.md#troubleshooting-playbook).

---

## Project and folder structure

### Frontend (`cheetahsign-webclient`)

- **`src/components/`** — Vue components (upload, tables, Document Builder, job views, etc.).
- **`src/sdk/`** — API clients (async methods per feature area); shared HTTP setup under `sdk/service-base/` (axios instance with `baseURL: "/api"`).
- **`src/unit-tests/`** / **`src/integration-tests/`** — Vitest tests.

### Backend (`Cheetah.Sign.Api`)

| Layer | Location | Description |
|-------|----------|-------------|
| **Interfaces** | `Interfaces/*.cs` | Service contracts (`IJobService`, `IPacketService`, etc.). |
| **Services** | `Services/*.cs` | Business logic and persistence via `AppDbContext`. |
| **Endpoints** | `Endpoints/*.cs` | Minimal route wiring; call injected services. |
| **Data** | `Contexts/AppDbContext.cs` | EF Core model. |

**PDF and coordinates:** Implemented in `Services/` (e.g. `DocumentEditor`, `CoordinateConverter`, `CoordinateService`) using [iText Core](https://itextpdf.com/products/itext-core).

**Interfaces registered in `Program.cs` (scoped):**

| Interface | Implementation | Role |
|-----------|----------------|------|
| `IAuditTrailService` | `ScopedAuditTrailService` | Job audit events. |
| `IEmailSender` | `SmtpEmailSender` | SMTP email. |
| `IJobService` | `JobService` | Jobs lifecycle, signing hooks. |
| `IPacketService` | `PacketService` | Packets and documents in packets. |
| `IClientProfileService` | `ClientProfileService` | Client profiles. |
| `IDocumentQueryService` | `DocumentQueryService` | Document queries and file bytes. |
| `IDocumentConverter` | `LibreOfficeDocumentConverter` | Office → PDF conversion where applicable. |
| `IUploadDocumentService` | `UploadDocumentService` | Upload pipeline. |
| `ICoordinateService` | `CoordinateService` | Coordinates and PDF rendering for signing. |
| `IFieldValidationService` | `FieldValidationService` | Field validation presets. |
| `IJobSummaryService` | `JobSummaryService` | Job summary/listing helpers. |
| `IPacketZipBuilder` | `PacketZipBuilder` | Packet zip download support. |

New behavior should extend the appropriate service interface and keep endpoints thin.

### Important files

- **`docker-compose.yml`** / **`docker-compose-tests.yml`** — Main app vs integration-test stack (separate DB and API configuration for tests).
- **`Program.cs`** — DbContext, SMTP-backed email, scoped services, endpoint registration, `MigrateDatabase()` (except in Testing).
- **`Contexts/PgDataFixture.cs`**, **`PgDataFixtureIntegration.cs`** — Test databases.
- **`cheetahsign-webclient/src/main.ts`**, **`App.vue`**, **`DefaultLayout.vue`** — App bootstrap and default chrome (signing and Document Builder may bypass the default layout).

Endpoint modules are registered in `Program.cs`, including (among others): file delete, document upload/query, file jobber, packets, job signer, edit document, saved field templates, client profile, mock endpoints (for integration tests), favorites, job view.

---

## Editing the frontend

Admin UI and the signing experience live under **`cheetahsign-webclient`**. `DefaultLayout.vue` wraps most routes; Document Builder and signing use their own layouts. Shared API access goes through **`src/sdk/`** (axios-based clients).

---

## Editing the backend (endpoints and services)

Flow: **HTTP → Endpoint → injected interface → service → `AppDbContext` / external systems → response.** Register new route groups in `Program.cs` after implementing the service behavior.

---

## Editing the database (pgAdmin / EF)

Optional: open pgAdmin at http://localhost:8181/ using credentials from `docker-compose.yml`. Prefer schema changes via **EF migrations** checked into `Migrations/` rather than manual edits in production-bound databases.

---
