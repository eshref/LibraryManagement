# Proposal

## Why

Librarians need a small web application to maintain and search a book catalog. This first version provides persistent catalog management without the complexity of lending workflows or individual-copy tracking.

## What Changes

- Introduce a librarian-facing Blazor UI backed by ASP.NET Core.
- Store catalog data in SQLite using linq2db for database access.
- Represent each book title with a single catalog record containing title, author, optional ISBN, category, publication year, and a non-negative copy count.
- Support listing, searching, adding, editing, and deleting catalog records, with validation and confirmation before deletion.
- Limit this version to catalog management; member records, loans, reservations, fines, and individual physical-copy identifiers are outside its scope.

## Capabilities

### New Capabilities

- `book-catalog`: Persistent management and search of book titles and their copy counts through a librarian-facing web interface.

### Modified Capabilities

None.

## Impact

- Establish the initial ASP.NET Core and Blazor application in this otherwise empty repository.
- Add linq2db and a compatible SQLite provider, with no EF Core dependency.
- Add database initialization and configuration for a persistent SQLite file.
- Add integration coverage against real SQLite for catalog operations and UI verification for the librarian workflows.
- Use one application deployment hosting the UI and backend. Separate user accounts and authentication are not part of this initial proposal.
