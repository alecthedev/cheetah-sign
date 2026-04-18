# Deployment and operations (Cheetah Sign)

## Overview

Cheetah Sign consists of:


| Tier           | Technology                             | Role                                                                                      |
| -------------- | -------------------------------------- | ----------------------------------------------------------------------------------------- |
| **Web client** | Vue 3 + Vite (`cheetahsign-webclient`) | SPA: admin UI and signer flows. Built output is static files (`npm run build` → `dist/`). |
| **API**        | ASP.NET Core 8 (`Cheetah.Sign.Api`)    | REST API, PDF handling, SMTP email. Listens on **8080** inside the container image.       |
| **Database**   | PostgreSQL                             | Application data; accessed via `ConnectionStrings:DefaultConnection`.                     |


**Local and developer setup** (Docker Compose, tests, ports) are documented in [Development.md](./Development.md). This file focuses on **production-style deployment**, **configuration**, **CI**, and **operations**.

---

## Architecture notes for deployment

- **API image:** `bsu.cheetah.sign.2025/Cheetah.Sign.Api/Dockerfile` builds on `mcr.microsoft.com/dotnet/aspnet:8.0` and installs **LibreOffice** for office-document conversion. The API process must run in an environment where LibreOffice remains available (same container image or equivalent host install).
- **Frontend and API URL:** The webclient uses axios with `baseURL: "/api"` (see `cheetahsign-webclient/src/sdk/service-base/apiAxios.ts`). In development, Vite proxies `/api` to the backend. In production, the **same origin** must expose:
  - static files for `/` (and client routes), and  
  - the API under `**/api`** (reverse proxy path to the API service), **unless** you change the client build to use a full API base URL (not configured in the repo today).
- **HTTPS:** `Program.cs` calls `UseHttpsRedirection()`. Terminate TLS at your load balancer or reverse proxy, or configure Kestrel certificates for the API and static host as your platform requires.
- **Migrations:** On startup (non-`Testing` environment), the API runs EF Core migrations (`MigrateDatabase()` in `Program.cs`). Plan deploy order so the database is reachable before traffic is shifted; first startup after a schema change applies pending migrations automatically.

---

## Configuration (production)

ASP.NET Core loads configuration in the usual order: environment variables, then optional files. **Production secrets must not be committed.** Use your platform’s secret store (e.g. parameter store, Key Vault, Kubernetes secrets) and inject **environment variables** into the API container or process.

### Required for a working system


| Area            | Variable / key                         | Notes                                                                                                          |
| --------------- | -------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| **Database**    | `ConnectionStrings__DefaultConnection` | Npgsql connection string to your PostgreSQL instance (host, port, database, user, password, SSL mode if used). |
| **Environment** | `ASPNETCORE_ENVIRONMENT`               | Set to `Production` (or your chosen non-Development name) for live deployments.                                |


### Required when sending email (signing invites, final documents)

Same keys as in [Development.md](./Development.md#configuration-reference): `Email__SmtpHost`, `Email__SmtpPort`, `Email__FromAddress`, `Email__FromName`, `Email__UserName`, `Email__Password`. If any required value is missing, **sending** mail throws when those code paths run (see `SmtpEmailSender`).

### Email link behavior (operational)

Signing-invite messages are built in `SmtpEmailSender` with a link host that may still reflect **development** assumptions. Before go-live, **verify** invite and notification URLs match your public site (code search: `SendSigningInviteAsync`). Adjust configuration or code in a controlled release if links must point to production hostnames.

---

## Database

- **Migrations:** Shipped in `Cheetah.Sign.Api/Migrations/`. Applied at API startup.
- **Backups:** Use PostgreSQL backups (continuous archiving, snapshots, or managed-database backups) according to your RPO/RTO. Test restore in a non-production environment before relying on it for incidents.
- **Rollback:** Rolling back **application** versions without rolling back **schema** can fail if a migration removed columns. Prefer forward-fix migrations or restore DB from backup to a compatible schema version if you must revert.

---

## Building and publishing images

### API

From the repository root (`bsu.cheetah.sign.2025/`):

- **Docker:** `docker build -f Cheetah.Sign.Api/Dockerfile .` (see Dockerfile for stages).
- **.NET container publish:** The student notes referenced `dotnet publish` with container targets; use [Microsoft’s current guidance](https://learn.microsoft.com/en-us/dotnet/core/docker/publish-as-container) for your SDK version if you publish to a registry without a hand-written Dockerfile.

Tag and push images to your registry (e.g. ECR, ACR, GCR). Restrict pull permissions to deployment principals.

### Web client

From `cheetahsign-webclient/`:

```bash
npm ci
npm run build
```

Deploy the contents of `**dist/**` to your static host or to an image that serves static files (the repo’s `Dockerfile` `prod` stage uses `http-server` for `dist`). Ensure `**/api**` on the same site hostname routes to the API (reverse proxy), or adjust the client to use an explicit API base URL in a future change.

---

## Networking and ports (reference)

Default **development** Compose maps (see [Development.md](./Development.md#ports-full-docker-stack)):


| Service               | Host port (typical dev) |
| --------------------- | ----------------------- |
| Web (Vite dev server) | 8080                    |
| API                   | 6001 → container 8080   |
| PostgreSQL            | 5432                    |


In **production**, map only what you need publicly (usually HTTPS → reverse proxy → API and static content). Do not expose PostgreSQL to the public internet.

---

## CI/CD (Bitbucket Pipelines)

The repository includes `bsu.cheetah.sign.2025/bitbucket-pipelines.yml`. At a high level it defines:

1. **Build and Test (.NET):** Uses `mcr.microsoft.com/dotnet/sdk:8.0`, waits for a **PostgreSQL** service, starts the API (`dotnet run`), waits for `http://localhost:6001/filelist`, then runs `dotnet test`.
2. **Front End tests:** Node image, `npm install` in `cheetahsign-webclient`, then `npm test`.

**Handoff actions for the owning team:**

- Confirm pipeline **database name, port, and credentials** match what unit tests expect (`PgDataFixture` reads `POSTGRES_*` and `TEST_CONNECTION_STRING`; defaults may differ from the service definition—run the pipeline after takeover and fix env alignment if tests fail).
- Add or adjust **deployment steps** (push images, ECS/Kubernetes, etc.) if not present; the checked-in file focuses on **build and test**, not full release automation.

---

## Release checklist (suggested)

Use or copy into your change-management tool:

1. **Code:** Merge to the agreed release branch; tag the release (version scheme is your choice).
2. **Database:** Review new EF migrations; schedule maintenance window if a long migration is expected.
3. **Config:** Secrets and connection strings updated in the target environment.
4. **Images / artifacts:** API and web `dist` (or image) built from the tagged commit.
5. **Deploy:** Apply API + static assets; confirm API reaches PostgreSQL.
6. **Smoke test:** Load the SPA, sign in / upload / one signing path as appropriate; confirm email in a test mailbox if SMTP is enabled.
7. **Monitor:** Check application and infrastructure logs for errors after cutover.

---

## Rollback

- **Application:** Redeploy the previous **known-good** image or static build artifact.
- **Database:** If the new version only added **backward-compatible** migrations, an older API might still run; if not, coordinate with DB restore or a forward fix. Do not assume down-migrations exist.

---

## Observability and troubleshooting

### Logging

- ASP.NET Core logs to stdout/stderr by default—capture via your container runtime or platform (CloudWatch, Azure Monitor, `docker logs`, etc.).
- Increase log verbosity temporarily via configuration if needed (`Logging:LogLevel`).

### No dedicated health endpoint

There is no separate `/health` route in the current API surface. For load balancer checks, use an inexpensive existing GET route your team accepts (the pipeline uses `/filelist`) or add a small health endpoint in a future release.

### Troubleshooting playbook


| Symptom                             | Things to check                                                                                                                        |
| ----------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| API fails on startup                | Database reachable; `ConnectionStrings__DefaultConnection` correct; TLS to Postgres if required; migration errors in logs.             |
| 502 / connection refused from proxy | API container running; correct internal port (**8080**); proxy upstream URL.                                                           |
| SPA loads but API calls fail        | Same-origin `**/api`** routing; CORS is not the default issue if the browser calls `/api` on the same host—verify reverse proxy paths. |
| Email not delivered                 | SMTP env vars; firewall to SMTP host; credentials; spam filters.                                                                     |
| Document conversion fails           | LibreOffice present in API image / host (see Dockerfile).                                                                              |
| **.NET build:** duplicate `Assembly*` attribute errors (e.g. `AssemblyCompanyAttribute` defined multiple times) | Stale `obj`/`bin` after branch switches or pulls. From the repo root `bsu.cheetah.sign.2025/`, run `dotnet clean` on the solution, then build again. If errors persist, remove the unit test project output: delete `Cheetah.Sign.Api/UnitTests/obj` and `Cheetah.Sign.Api/UnitTests/bin`, then rebuild. CI agents should use a clean workspace or the same clean step before `dotnet test` / publish. |


### Docker (operator)

If you deploy with Compose on a VM:

```bash
docker compose ps
docker compose logs -f <service-name>
```

Ensure host firewall allows only required ports.

---

## Known operational considerations (current state)

- **Client acceptance** is recorded in [uat.md](./uat.md). Remaining product or technical work should live in your **issue tracker**, not in internal-only tooling.
- **pgAdmin** in `docker-compose.yml` is a **development** convenience; do not expose it publicly in production without authentication and network controls.

---

## Related documentation

- [Development.md](./Development.md) — local setup, tests, ports, developer configuration.
- [uat.md](./uat.md) — user acceptance and feature status.

