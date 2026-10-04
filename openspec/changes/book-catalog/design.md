# Design

## Context

The repository contains a README and OpenSpec configuration but no application, database schema, or existing capabilities. The installed SDK is .NET 10.0.400. ASP.NET Core, Blazor, linq2db, SQLite, and one record per title are agreed choices; see proposal.md for motivation and specs/book-catalog/spec.md for behavior.

This design is needed because the change establishes the application architecture and introduces the first persistent data model and external dependencies.

## Goals / Non-Goals

**Goals:**
- Keep the UI and backend in one deployable application.
- Separate interactive components, backend catalog operations, and database access sufficiently to test each boundary.
- Make database setup automatic and repeatable for a new installation.

**Non-Goals:**
- A separately deployed frontend, public REST API, or browser-side database access.
- A general-purpose repository framework, distributed database, or migration platform for this initial single-table schema.
- Account management or authentication flows in this version.

## Decisions

### One .NET 10 Blazor Web App using Interactive Server

Use an ASP.NET Core Blazor Web App with Interactive Server rendering, one web project, and a separate test project. .NET 10 matches the installed SDK. Interactive Server keeps the catalog operations and database credentials on the backend without an additional API client or WebAssembly deployment.

Components call a backend catalog service through dependency injection. That service owns validation and delegates database access to a small linq2db persistence component; components never issue database queries. A separate REST API can be added later if a second client needs it. WebAssembly with API endpoints was considered but adds transport and deployment concerns unnecessary for this first UI.

```text
Librarian browser
       |
       v
ASP.NET Core / Blazor Interactive Server
       |
       v
Backend catalog service
       |
       v
linq2db persistence component
       |
       v
SQLite file
```

### linq2db with a compatible SQLite provider

Use linq2db with Microsoft.Data.Sqlite and compatible stable package versions selected during implementation. EF Core is not used. Map a single Books table with an integer identity primary key, required title and author, nullable ISBN/category/publication year, and a required integer copy count. Do not require unique titles or ISBNs: separate editions can share title and author, and ISBN is optional.

Open and dispose a database connection per operation. Avoid keeping a connection alive for a Blazor circuit, which can outlast an individual interaction. Use asynchronous operations and parameterized linq2db queries for writes and identifier lookups. Check affected-row counts when updating or deleting so missing records do not become silent successes. Whole-row updates are atomic; simultaneous edits use last successful save wins for this version rather than adding conflict resolution.

Use backend validation for every write, independent of form validation. Normalize text by trimming and convert empty optional text to null. Require title and author, a non-negative integer copy count, and an optional integer publication year between 1 and 9999. ISBN is optional descriptive metadata without checksum validation in this version. SQLite constraints should also enforce required values and numeric bounds.

### Persistent file and repeatable initialization

Configure the database file path through application configuration with an environment-variable override. Resolve the default to a writable application data directory, outside static web assets, and ignore the file in Git. Create the containing directory and missing table at startup, without dropping or replacing an existing table or seeding records.

Initialization must be idempotent and fail visibly if the file cannot be opened or the schema cannot be initialized. Existing databases must remain untouched by normal restarts. Future schema changes will require a deliberate migration strategy rather than relying on initial table creation.

A disk file is preferred over an in-memory database so data survives process restarts. A separate database server would add operational work without benefiting this small catalog.

### Small catalog UI and deterministic search

Provide a catalog page and a shared form for add/edit interactions. Display all metadata, with an add action and edit/delete actions per record. Use clear empty-catalog and no-search-results messages, field-specific validation, a busy state preventing repeated submissions, and visible success/failure feedback. Deletion requires a confirmation naming the selected book. Preserve entered values after a failed save.

For the first small catalog, load catalog records using linq2db and apply trimmed, ordinal case-insensitive substring matching to title, author, and ISBN in the backend service. Sort results by title and then identifier for stable presentation. This gives predictable case-insensitive behavior without depending on SQLite's limited built-in case folding. Server-side search and pagination can replace this approach if the catalog outgrows a full-record read; no pagination is introduced now.

### Verification at the database and UI boundaries

Use xUnit integration tests against unique temporary SQLite files and the actual backend/persistence implementation. Exercise initialization on a new and existing database; create, edit, delete, search, validation rejection, missing-record handling, and persistence after closing and reopening connections. Verify rejected writes leave stored values unchanged. Include storage-failure handling without replacing the successful database scenarios with mocks.

Verify browser interactions for add/edit/cancel, search, field errors, delete confirmation/cancellation, and empty states. Use focused component tests for deterministic form/error states when suitable and a browser smoke check for the complete path through the running ASP.NET Core app to SQLite. Each automated database test owns and cleans up its own temporary directory.

## Risks / Trade-offs

- SQLite permits limited concurrent writing -> keep connections and writes short; this design targets a small librarian catalog and one application instance.
- Full-catalog reads grow with catalog size -> keep the initial implementation simple, and introduce database-side filtering and pagination when measured usage requires them.
- Concurrent edits can overwrite earlier values -> document last successful save wins; optimistic concurrency is deferred.
- A Blazor server interaction needs a live connection -> preserve ordinary form state during recoverable errors and provide actionable failure feedback.
- The SQLite file is the durable data store -> keep it on persistent writable storage during deployment and back it up before replacing or upgrading an installation.
- No login is included -> the initial deployment is intended for access controlled by the hosting environment; wider access would require a separate authentication decision.

## Migration Plan

1. Build the new application and configure a writable SQLite file location.
2. On first startup, create the missing database and Books table; show an empty catalog.
3. Verify adding a record and retaining it after a restart.
4. On subsequent starts, reuse the configured file without resetting its data.
5. Before upgrades, stop writes and back up the SQLite file. Roll back application binaries without deleting the database; this initial change has no legacy schema or data to migrate.
