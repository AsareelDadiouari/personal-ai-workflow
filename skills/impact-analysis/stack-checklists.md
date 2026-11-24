# Stack checklists for impact analysis

Use only the sections that match the project's detected stack. These are places to look, not assumptions about how the project is built.

## Any stack

- Search by symbol **and** by string: route paths, table and column names, config keys, feature flags, event and queue names. Dynamic access (reflection, string-based lookups, serialization) is invisible to symbol search.
- Configuration per environment: `appsettings.*.json`, `.env.example`, Helm or Compose files, CI variables (names only, never values).
- Generated code and clients: OpenAPI or GraphQL clients, ORM models, protobuf. Check what regenerates them.
- Other consumers outside this repository (mobile apps, partner integrations, reports, data pipelines). These are usually **Unknown**. Say so.

## C# / .NET / ASP.NET Core

- Project references (`*.csproj`, `*.sln`, `Directory.Build.props`) to see which projects depend on the changed one.
- Dependency injection registrations (`Program.cs`, `Startup.cs`, `IServiceCollection` extensions): interface → implementation, lifetimes.
- Controllers and minimal API endpoints, route attributes, filters, middleware order, authorization policies.
- DTOs and serialization settings (property names, nullability, enums as strings or numbers): shape changes break clients.
- EF Core: `DbContext`, entity configurations, migrations folder, raw SQL, `Include` chains, global query filters.
- Background services (`IHostedService`), message handlers, scheduled jobs.
- Test projects (`*.Tests.csproj`): `dotnet test` with a filter on the affected area.

## Angular / TypeScript / JavaScript

- `angular.json`, `package.json`, `tsconfig*.json` for project structure, path aliases, and build targets.
- Services and their injection scope (`providedIn`), standalone components vs. NgModules, route definitions and guards, resolvers, interceptors.
- Shared components, pipes, and directives: search the selector and the class name across templates.
- HTTP calls: the service methods and the API models they map to; check interface changes against backend DTOs.
- State management (signals, RxJS stores, NgRx) for selectors and effects consuming the changed data.
- Unit tests (`*.spec.ts`) and end-to-end tests; check the test runner configured in `package.json` or `angular.json`.

## PostgreSQL / SQL Server

- Foreign keys, unique constraints, check constraints, defaults, and indexes involving the changed tables or columns.
- Views, materialized views, functions, stored procedures, and triggers that reference them (search migration and SQL files; if a database catalog is available and authorized, query it read-only).
- ORM mappings and raw SQL in the application code.
- Data already stored: does existing data satisfy the new constraint? What happens to rows during the migration?
- Locking and duration of the migration on large tables. Behavior differs between PostgreSQL and SQL Server (for example, adding a column with a default, creating indexes concurrently or online).
- Reporting, ETL, or replication consumers: usually **Unknown** from code alone.

## REST APIs

- Contract: paths, methods, request and response shapes, status codes, error formats, pagination, versioning.
- OpenAPI or Swagger documents and generated clients.
- Backward compatibility: removed or renamed fields, newly required fields, changed types or enum values.
- Authentication and authorization on the endpoint; rate limiting; caching headers.

## Docker / CI/CD

- `Dockerfile`, Compose files, base image versions, build arguments, exposed ports, health checks.
- Pipeline definitions (`.github/workflows/`, `azure-pipelines.yml`, `.gitlab-ci.yml`, and similar): which jobs build, test, and deploy the changed parts; required checks.
- Environment-specific deployment steps and migrations run by the pipeline.
