# Tasks

## 1. Application foundation

- [ ] 1.1 Scaffold a .NET 10 ASP.NET Core Blazor Web App with Interactive Server, a solution, and an xUnit test project; verify the solution builds and the application serves its initial page.
- [ ] 1.2 Add compatible stable linq2db and Microsoft.Data.Sqlite dependencies and register the backend catalog/persistence services; verify restore succeeds and application service resolution works without EF Core.
- [ ] 1.3 Document prerequisites and local startup in README.md; verify the documented commands start the app from a fresh checkout.

## 2. SQLite persistence

- [ ] 2.1 Map the Books table with an identity key, required title/author/copy count, optional ISBN/category/publication year, and numeric constraints; add real-SQLite integration coverage verifying the schema accepts valid records and rejects invalid database values.
- [ ] 2.2 Add configurable persistent file storage and idempotent startup initialization, keeping data outside static assets and out of Git; verify integration tests initialize a new database and reopen an existing database without losing records.
- [ ] 2.3 Implement asynchronous linq2db create, read, update, and delete operations with per-operation connections and affected-row checks; verify real-SQLite integration tests cover successful operations, stable identifiers, missing records, and preservation of unrelated records.
- [ ] 2.4 Document the database-path setting, writable-directory requirement, persistence, and backup procedure in README.md; verify the documented override selects a different SQLite file and restarting retains its data.

## 3. Backend catalog behavior

- [ ] 3.1 Implement shared backend validation and text normalization for add/edit operations; verify integration tests reject invalid writes without changing stored data and accept zero copies and omitted optional metadata.
- [ ] 3.2 Implement stable catalog ordering and trimmed case-insensitive substring search across title, author, and ISBN; verify integration tests cover matching fields, blank search, no matches, and deterministic results using real SQLite records.
- [ ] 3.3 Expose clear validation, missing-record, and storage-failure outcomes to the UI without falsely reporting success; verify database-boundary failure tests exercise a real unavailable or locked SQLite database and confirm failed edits do not insert replacements.

## 4. Librarian catalog UI

- [ ] 4.1 Build the catalog page with metadata columns, search, add/edit/delete actions, and distinct empty/no-results states; verify UI checks cover saved records, cleared searches, and both empty states.
- [ ] 4.2 Build a reusable add/edit form with field feedback, cancellation, a busy state, and success/failure messages that retain unsaved values after a failed save; verify UI tests cover valid submission, invalid fields, unchanged data after cancellation, and retryable failure feedback.
- [ ] 4.3 Add deletion confirmation identifying the selected title and refresh the catalog after successful changes; verify UI tests cover confirmed deletion, cancelled deletion, and failure without a success message.
- [ ] 4.4 Document the librarian catalog workflow, supported fields, and current single-instance/last-save-wins behavior in README.md; verify the documented add, search, edit, and delete sequence matches the UI.

## 5. Application integration verification

- [ ] 5.1 Run the complete build and automated test suite; verify all required catalog scenarios pass and temporary SQLite test directories are cleaned up.
- [ ] 5.2 Run a browser smoke check through the hosted Blazor app using a temporary database: add a title with multiple copies, search, edit, cancel and confirm deletion, and restart the host; verify UI behavior and persisted additions/edits/deletions match the spec without modifying a user's database.
