# Spec Delta

## Purpose

Enable librarians to maintain a persistent, searchable catalog of book titles and copy counts through a web interface.

## ADDED Requirements

### Requirement: Catalog record
The system SHALL represent each catalog entry as one book title with a stable identifier, title, author, optional ISBN, optional category, optional publication year, and a copy count. Copies SHALL be represented by the count rather than separate physical-copy records.

#### Scenario: Multiple copies of one title
- **WHEN** a librarian saves a book with a copy count of three
- **THEN** the catalog contains one record for that title with a copy count of three
- **AND** it does not create three physical-copy records

#### Scenario: Optional metadata omitted
- **WHEN** a librarian saves a valid title, author, and copy count without ISBN, category, or publication year
- **THEN** the record is saved successfully with the omitted metadata empty

### Requirement: Browse catalog
The system SHALL provide a catalog page listing each book's title, author, ISBN, category, publication year, and copy count, with actions to add, edit, and delete records. An empty catalog SHALL show a clear empty state and an add action.

#### Scenario: Existing catalog
- **WHEN** a librarian opens the catalog page containing saved records
- **THEN** those records and their metadata are visible with edit and delete actions
- **AND** an add action is available

#### Scenario: Empty catalog
- **WHEN** a librarian opens a catalog with no records
- **THEN** the page explains that no books have been added and offers an add action

### Requirement: Search catalog
The system SHALL support case-insensitive substring search across title, author, and ISBN. It SHALL trim the search text, treat blank searches as an unfiltered catalog, and indicate when a nonblank search has no matches.

#### Scenario: Search by metadata
- **WHEN** a librarian enters text matching part of a saved title, author, or ISBN
- **THEN** the catalog displays matching records regardless of the search text's letter case
- **AND** records matching none of those fields are excluded

#### Scenario: Clear search
- **WHEN** a librarian clears the search or enters only whitespace
- **THEN** the catalog displays all records

#### Scenario: No matches
- **WHEN** a librarian searches for text matching no catalog records
- **THEN** the page shows a no-results message without changing saved records

### Requirement: Add catalog record
The system SHALL allow a librarian to add a valid book record and display it in the catalog after successful saving. Cancelling an add operation SHALL leave the catalog unchanged.

#### Scenario: Save a new book
- **WHEN** a librarian submits valid book details
- **THEN** the system creates one persistent record with a new identifier
- **AND** the catalog displays the saved details

#### Scenario: Cancel adding
- **WHEN** a librarian cancels an add form
- **THEN** no new record is saved

### Requirement: Validate catalog inputs
The system SHALL require nonblank title and author and a whole-number copy count of zero or greater. A supplied publication year SHALL be a whole number from 1 through 9999. It SHALL trim text fields and reject invalid submissions with field-specific feedback without changing stored records.

#### Scenario: Invalid submission
- **WHEN** a librarian submits a blank title or author, a missing or invalid copy count, or an invalid supplied publication year
- **THEN** the form identifies the invalid fields
- **AND** no record is created or updated

#### Scenario: Zero copies
- **WHEN** a librarian submits otherwise valid book details with zero copies
- **THEN** the system saves the record with a copy count of zero

#### Scenario: Trim text
- **WHEN** a librarian submits valid text fields with leading or trailing whitespace
- **THEN** the stored text has that whitespace removed

### Requirement: Edit catalog record
The system SHALL allow a librarian to edit an existing record using its current values and save changes while retaining its identifier. Cancelling SHALL retain the saved values. Editing a record that no longer exists SHALL report that condition without creating a replacement record.

#### Scenario: Save edits
- **WHEN** a librarian changes an existing record and submits valid details
- **THEN** the updated metadata and copy count replace the saved values
- **AND** the record retains its identifier

#### Scenario: Cancel edits
- **WHEN** a librarian changes form values and then cancels
- **THEN** the persisted record retains its previous values

#### Scenario: Record removed before saving
- **WHEN** a librarian tries to save edits for a record that has already been deleted
- **THEN** the interface reports that the record no longer exists
- **AND** no replacement record is created

### Requirement: Confirm deletion
The system SHALL request confirmation identifying the selected book before deleting its catalog record. Confirming SHALL remove only that record; cancelling SHALL retain it.

#### Scenario: Confirm deletion
- **WHEN** a librarian confirms deletion of a selected book
- **THEN** that record is removed from persistent storage and the catalog view
- **AND** other records are unchanged

#### Scenario: Cancel deletion
- **WHEN** a librarian cancels the deletion confirmation
- **THEN** the selected record remains unchanged

### Requirement: Persist catalog changes
The system SHALL retain successful additions, edits, and deletions across application restarts. A failed storage operation SHALL produce understandable feedback and SHALL NOT be shown as a successful save or deletion.

#### Scenario: Restart application
- **WHEN** the application restarts after successful catalog changes
- **THEN** added records and edited values remain present and deleted records remain absent

#### Scenario: Storage failure
- **WHEN** saving or deleting a record fails because storage is unavailable
- **THEN** the interface reports that the operation failed without displaying a success message
- **AND** unsaved form values remain available for retry
