# Student Management Application

A sample, production-like full-stack application demonstrating modern .NET best practices.
It contains a backend **ASP.NET Core Web API** (Domain-Driven Design, CQRS + Mediator) and a frontend **Blazor WebAssembly** client, all dockerized and ready to run.

![Student Management Screenshot](https://github.com/SaraRasoulian/DotNet-WebAPI-Blazor-Sample/assets/51083712/6a1cb0b0-6de0-4fd8-91fc-02a3ce6bba04)

---

## Table of contents

* [What this project demonstrates](#what-this-project-demonstrates)
* [Features](#features)
* [Technology stack](#technology-stack)
* [Repository structure](#repository-structure)
* [Prerequisites](#prerequisites)
* [Quick start — run with Docker (recommended)](#quick-start--run-with-docker-recommended)
* [Run locally without Docker](#run-locally-without-docker)
* [Database migrations & seeding](#database-migrations--seeding)
* [API documentation (Swagger)](#api-documentation-swagger)
* [Testing (Unit / Integration / Acceptance)](#testing-unit--integration--acceptance)
* [Configuration & environment variables](#configuration--environment-variables)
* [Troubleshooting & common issues](#troubleshooting--common-issues)


---

## What this project demonstrates

This repository is focused on presenting modern, maintainable patterns for building a web application with .NET:

* Clean Architecture / DDD separation (Domain / Application / Infrastructure / WebApi)
* CQRS + Mediator (MediatR)
* Repository pattern + EF Core for persistence
* Validation and mapping layers (FluentValidation / Mapster)
* Tests across levels: unit, integration and acceptance (xUnit / Testcontainers / SpecFlow)
* Dockerized multi-container setup (API + Blazor client + Postgres + pgAdmin)
* Production-ready concerns: configuration, error handling, health checks

> Note: this repo is educational. In production you should adapt patterns to fit real constraints, security, hosting, secrets management and scale requirements.

---

## Features

* CRUD for Students (Create, Read, Update, Delete)
* Server-side validation with descriptive error responses
* Clean separation for controllers / handlers / repositories
* Docker-compose orchestration for full stack locally
* Swagger/OpenAPI for interactive API exploration
* Example tests (unit, integration, acceptance)
* Postgres database and pgAdmin UI included for convenience

---

## Technology stack

* **Language**: C# (.NET 8)
* **Backend**: ASP.NET Core Web API (8.0)
* **Frontend**: Blazor WebAssembly (8.0)
* **ORM**: Entity Framework Core (8.0)
* **Patterns**: DDD, Clean Architecture, CQRS, Mediator
* **Database**: PostgreSQL
* **Testing**: xUnit, Testcontainers, SpecFlow, NSubstitute, Bogus
* **Mapping**: Mapster
* **Validation**: FluentValidation
* **UI**: Bootstrap 5 + Blazor.Bootstrap
* **Containerization**: Docker, Docker Compose

NuGet packages of note:
`MediatR`, `FluentValidation`, `Mapster`, `xUnit`, `SpecFlow`, `Testcontainers`, `NSubstitute`, `Bogus`, `Blazor.Bootstrap`

---

## Repository structure (high level)

```
.
├─ client/                # Blazor WebUI (WebAssembly)
│  └─ src/WebUI
│     ├─ Pages/Students   # Index, Add, Edit, Delete, View
│     ├─ Services         # StudentService, ApiUrls
│     └─ wwwroot          # static assets, index.html
├─ server/                # Server solution (API + Domain + Infrastructure + Tests)
│  ├─ src/Domain
│  ├─ src/Application
│  ├─ src/Infrastructure
│  └─ src/WebAPI
├─ docker-compose.yml
├─ README.md
└─ LICENSE
```

---

## Prerequisites

* Docker & Docker Compose installed and running (Engine must be started)
* Git (for cloning / pushing)
* .NET SDK 8 (only required for local runs / building outside Docker)
* (Optional) Visual Studio / VS Code for IDE support

---

## Quick start — run with Docker (recommended)

This project ships with a `docker-compose.yml` that brings up 4 services:

* **API** — `.NET Web API` (exposes `http://localhost:5000`)
* **Client** — `Blazor WebAssembly` served via Nginx (exposes `http://localhost:8080`)
* **Postgres** — DB (container port `5432` mapped to host `5433`)
* **PgAdmin** — DB admin UI (exposes `http://localhost:8000`)

1. From the repository root:

```bash
docker compose up --build -d
```

2. Check containers:

```bash
docker ps
```

3. Logs (follow):

```bash
docker compose logs -f
# or for a specific service
docker compose logs -f api
```

4. Browse:

* Blazor client: `http://localhost:8080`
* Web API Swagger: `http://localhost:5000/swagger`
* pgAdmin: `http://localhost:8000` (check docker-compose environment for credentials)

---

## Database migrations & seeding

### Using Package Manager Console in Visual Studio

Open `StudentManagement.Server.sln` in Visual Studio, set `WebAPI` as startup project, open *Package Manager Console* and run:

```powershell
update-database
```

This will apply EF Core migrations and create the database schema in the running Postgres container.

### Using dotnet-ef (CLI)

If you prefer CLI and have `dotnet-ef` installed:

```bash
# from repo root
dotnet tool install --global dotnet-ef    # if not installed
dotnet ef database update \
  --project server/src/Infrastructure \
  --startup-project server/src/WebAPI
```

> If your DB connection string is pointing to the Postgres container ensure host and port are correct (see next section).

---

## API documentation (Swagger)

When the API is running you can explore endpoints & example requests in Swagger:

```
http://localhost:5000/swagger
```

If you see a 404 on `/swagger`, ensure:

* the `api_container` is running (`docker ps`)
* check API logs (`docker compose logs api`)
* verify `ASPNETCORE_ENVIRONMENT` and launch settings in `server/src/WebAPI/appsettings.json`

---

## Run locally without Docker

1. Ensure Postgres is available (local or remote) and update the connection string in:

   * `server/src/WebAPI/appsettings.json` or `server/src/WebAPI/appsettings.Development.json`
2. Apply migrations (see previous section).
3. Run the API:

```bash
cd server/src/WebAPI
dotnet run
# API runs at http://localhost:5000 by default (check launch settings)
```

4. Run the Blazor client (WebUI):

```bash
cd client/src/WebUI
dotnet run
# or publish and serve via a static server or nginx
```

If running client and API separately, ensure the client `Program.cs` `HttpClient` `BaseAddress` points to `http://localhost:5000/` (API URL). Example:

```csharp
builder.Services.AddScoped(sp => new HttpClient {
    BaseAddress = new Uri("http://localhost:5000/")
});
```

---

## Testing (Unit / Integration / Acceptance)

Test projects are under `server/tests/*`. Run all tests with:

```bash
dotnet test server/tests
```

Integration/acceptance tests may expect Docker DB container (or Testcontainers will spin one). Ensure Docker is running before executing integration/acceptance suites.

---

## Configuration & environment variables

* API configuration files: `server/src/WebAPI/appsettings.json` and environment specific files.
* DB connection string is defined in `appsettings.json` (or in docker-compose as environment variables). Typical connection string:

```
Host=postgres_db_container;Port=5432;Database=studentdb;Username=postgres;Password=postgres
```

*(When running Docker compose, the service name `postgres_db_container` or `postgres_db` is commonly used as host within the Docker network.)*

Check `docker-compose.yml` for exact service names and environment variables used by the project.

---

## Troubleshooting & common issues

### 1. Browser console: `TypeError: Failed to fetch` (Blazor client cannot call API)

Symptoms: client loads but shows `Failed to fetch` in browser console when calling API.

Likely causes:

* API is not running (check `docker ps` or `docker compose ps`)
* CORS policy blocking calls (ensure WebAPI allows calls from client origin)
* Wrong `BaseAddress` in Blazor client (open `client/src/WebUI/Program.cs` and set `http://localhost:5000/`)
* Network mismatch: if using Docker compose, both client and api run in same docker network. Blazor running in the browser must call the host-exposed API address `http://localhost:5000/`.

Fixes:

```bash
# View API logs for errors
docker compose logs -f api

# Restart containers
docker compose down
docker compose up --build -d

# Verify API health (example)
curl http://localhost:5000/health   # or /swagger
```

### 2. Swagger 404

* Ensure API container is up and listening (`docker ps`)
* Check logs: `docker compose logs api`
* Confirm port mapping: `5000:5000` in `docker-compose.yml`

### 3. Migrations failing / DB connection refused

* Confirm Postgres container status and mapped port `5433` (host) → `5432` (container).
* Check the connection string in `appsettings.json`.
* Use `docker compose logs postgres` to inspect DB startup.
* Use pgAdmin ([http://localhost:8000](http://localhost:8000)) to connect and inspect DB.

### 4. Git warning: `LF will be replaced by CRLF`

This is a Git line ending warning on Windows. Not an error; to avoid:

```bash
git config --global core.autocrlf true
```

### 5. Docker image builds slow / caching / rebuild

Use:

```bash
docker compose build --no-cache
```

---

## Contributing

Contributions are welcome — and appreciated! A suggested workflow:

1. Fork the repository
2. Create a feature branch: `git checkout -b feat/awesome-feature`
3. Make changes, add tests
4. Run tests locally: `dotnet test`
5. Commit & push, open a PR describing changes and rationale

Please follow the repository's code style and open an issue before major changes.

---


---

## Appendix — useful terminal commands

Start everything:

```bash
docker compose up --build -d
```

Stop and remove:

```bash
docker compose down
```

Rebuild without cache:

```bash
docker compose build --no-cache
docker compose up -d
```

Follow API logs:

```bash
docker compose logs -f api
```

Apply EF Core migrations (CLI):

```bash
dotnet ef database update \
  --project server/src/Infrastructure \
  --startup-project server/src/WebAPI
```

Run unit tests:

```bash
dotnet test server/tests
```

---
