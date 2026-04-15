# User Acceptance Testing (UAT)

## Last Updated:

2026-04-15

---

# 1. Feature Overview

| Feature                                                         | Priority | Iteration | Client Facing (Y/N) |
| --------------------------------------------------------------- | -------- | --------- | ------------------- |
| Document Upload                                                 | High     | 0/1       | Y                   |
| Document Library (list/search/filter/favorites)                 | High     | 0/1/2/3/4 | Y                   |
| Send Document (manual + client dropdown)                        | High     | 0/1/2/3/4 | Y                   |
| Multi-signer Send (sequential + parallel)                       | High     | 1/2/3/4   | Y                   |
| Packet Management (create/edit/reorder/delete)                  | High     | 1/2/3/4   | Y                   |
| Packet Preview                                                  | Medium   | 2/3/4     | Y                   |
| Document Builder (drag/drop fields + signer assignment)         | High     | 0/1/2/3/4 | Y                   |
| Custom Field Templates                                          | Medium   | 3/4       | Y                   |
| Client Management (create/edit/list/validation)                 | High     | 0/1/2/3/4 | Y                   |
| Job Dashboard (status, search, recipient/status filters, audit) | High     | 0/1/2/3/4 | Y                   |
| Document Signing (single + packet docs, validation/autofill)    | High     | 0/1/2/3/4 | Y                   |
| Download Completed Job/Packet + Certificate                     | High     | 2/3/4     | Y                   |

---

# 2. Acceptance Scenarios

## Feature: Document Upload

### Scenario 1: Happy Path

**Given:** Given an admin user uploads a `.pdf`, `.doc`, `.docx`, or `.xlsx` document

**When:** When they submit the file for upload

**Then:** Then the document is accepted and stored (with conversion to PDF where required)

**Status:** Client Accepted

**Evidence:** `UploadDocumentTests.cs`, `DocumentConverterTests.cs`, `upload-form.test.ts`

### Scenario 2: Error Handling

**Given:** Given an admin selects an unsupported file type (example: `.txt`)

**When:** When they submit the upload request

**Then:** Then the request is rejected with invalid file type feedback and nothing is stored

**Status:** Client Accepted

**Evidence:** `UploadDocumentTests.cs` (`testUploadDocument_InvalidFileType`)

### Scenario 3: Edge Case

**Given:** Given an admin uploads a file larger than allowed size

**When:** When upload processing starts

**Then:** Then upload is rejected with size-specific feedback (`file_too_large`)

**Status:** Client Accepted

**Evidence:** `UploadDocumentTests.cs` (`testUploadDocument_FileTooLarge`)

---

## Feature: Document Library (list/search/filter/favorites)

### Scenario 1: Happy Path

**Given:** Given documents exist in the system

**When:** When a user opens the Documents screen

**Then:** Then documents and packet summaries are shown and searchable

**Status:** Client Accepted

**Evidence:** `display-docs.test.ts` (mount fetch, search filtering, packet summaries)

### Scenario 2: Error Handling

**Given:** Given a user attempts to delete a protected/in-use document

**When:** When delete is requested

**Then:** Then the UI shows backend message details and keeps data consistent

**Status:** Client Accepted

**Evidence:** `display-docs.test.ts` (delete error detail paths), `FileDeleterAndPacketDownloadTests.cs` (`DeleteDocument_WhenDocumentInPacket_ReturnsConflict`)

### Scenario 3: Edge Case

**Given:** Given the user uses Favorites-only filter before and after starring docs

**When:** When favorites are toggled and filters are applied

**Then:** Then Favorites filter enablement and resulting rows update correctly

**Status:** Client Accepted

**Evidence:** `display-docs.test.ts` (favorites toggle/filter cases)

---

## Feature: Send Document (manual + client dropdown)

### Scenario 1: Happy Path

**Given:** Given a selected document and a valid recipient (manual entry or selected client)

**When:** When user sends the document

**Then:** Then a sent job is created and visible in job tracking

**Status:** Client Accepted

**Evidence:** `display-docs.test.ts` (`handleSend`, `sendDocumentViaDropdown`, `sendPrepare`)

### Scenario 2: Error Handling

**Given:** Given send payload is incomplete or backend rejects request

**When:** When send operation is attempted

**Then:** Then send is blocked or error details are surfaced without crashing

**Status:** Client Accepted

**Evidence:** `display-docs.test.ts` send failure/guard path coverage

### Scenario 3: Edge Case

**Given:** Given a packet context is active in the unified send modal

**When:** When handle send executes

**Then:** Then payload includes `packetId` and uses packet flow instead of single-doc flow

**Status:** Client Accepted

**Evidence:** `display-docs.test.ts` (`handleSend for packet context includes packetId`)

---

## Feature: Multi-signer Send (sequential + parallel)

### Scenario 1: Happy Path

**Given:** Given multiple signers are assigned to a document or packet

**When:** When send is submitted

**Then:** Then jobs are created with signer assignments and order metadata

**Status:** Client Accepted

**Evidence:** `display-docs.test.ts` (multi-signer send paths)

### Scenario 2: Error Handling

**Given:** Given a signer has no assigned fields

**When:** When send is attempted

**Then:** Then send is blocked and user is warned to complete signer mapping

**Status:** Client Accepted

**Evidence:** `display-docs.test.ts` (`sendPacketToClient alerts when signer has no assigned fields`, `handleSend alerts when signer has no fields`)

### Scenario 3: Edge Case

**Given:** Given parallel signing toggle is enabled

**When:** When send payload is generated

**Then:** Then each signer receives parallel order semantics (same signing order)

**Status:** Client Accepted

**Evidence:** `display-docs.test.ts` (`uses parallel signingOrder when toggle is on`)

---

## Feature: Packet Management (create/edit/reorder/delete)

### Scenario 1: Happy Path

**Given:** Given user creates a packet and adds documents

**When:** When packet is saved or updated

**Then:** Then packet metadata and ordering persist and are reloadable

**Status:** Client Accepted

**Evidence:** `document-builder.test.ts` (`initializePacket`, `savePacket`, reorder tests)

### Scenario 2: Error Handling

**Given:** Given packet API calls fail

**When:** When creating/saving/deleting packet

**Then:** Then user receives error toasts and state is preserved safely

**Status:** Client Accepted

**Evidence:** `document-builder.test.ts`, `display-docs.test.ts` (packet rename/delete failure paths)

### Scenario 3: Edge Case

**Given:** Given packet has only one document

**When:** When user attempts to remove that final document

**Then:** Then remove is prevented with explicit warning

**Status:** Client Accepted

**Evidence:** `document-builder.test.ts` (`removeDocumentFromPacket` warning case)

---

## Feature: Packet Preview

### Scenario 1: Happy Path

**Given:** Given packet contains ordered documents

**When:** When user opens packet preview

**Then:** Then modal shows packet docs sorted by display order and loadable preview content

**Status:** Client Accepted

**Evidence:** `display-docs.test.ts`, `job-display.test.ts` packet preview flows

### Scenario 2: Error Handling

**Given:** Given packet details fail or contain no documents

**When:** When preview is opened

**Then:** Then preview opens with readable error messaging and no crash

**Status:** Client Accepted

**Evidence:** `display-docs.test.ts` (`shows error when packet has no documents`), `job-display.test.ts` empty packet scenarios

### Scenario 3: Edge Case

**Given:** Given packet docs come unsorted from API

**When:** When preview renders

**Then:** Then docs are sorted client-side by display order

**Status:** Client Accepted

**Evidence:** `display-docs.test.ts` (`sorts documents by displayOrder`)

---

## Feature: Document Builder (drag/drop fields + signer assignment)

### Scenario 1: Happy Path

**Given:** Given a user opens a document in the builder

**When:** When they drag fields, assign signers, and build

**Then:** Then coordinates/types are saved and can be restored for editing

**Status:** Client Accepted

**Evidence:** `document-builder.test.ts` (drag/drop/save coordinates/restore)

### Scenario 2: Error Handling

**Given:** Given backend fails during render or save operations

**When:** When build actions execute

**Then:** Then error toasts display and user workflow remains stable

**Status:** Client Accepted

**Evidence:** `document-builder.test.ts` (render failure + save failure paths)

### Scenario 3: Edge Case

**Given:** Given users frequently undo/redo and reorder fields

**When:** When history and ordering actions are used repeatedly

**Then:** Then field state remains consistent and recoverable

**Status:** Client Accepted

**Evidence:** `document-builder.test.ts` (history capture, undo/redo, ordering helpers)

---

## Feature: Custom Field Templates

### Scenario 1: Happy Path

**Given:** Given a user creates a custom field template

**When:** When they submit from custom field modal

**Then:** Then template persists and appears in builder palette

**Status:** Client Accepted

**Evidence:** `document-builder.test.ts` (`Custom field modal` tests), `custom-field-modal.test.ts`

### Scenario 2: Error Handling

**Given:** Given template API operation fails

**When:** When template create/load executes

**Then:** Then failure is shown and existing palette remains usable

**Status:** Client Accepted

**Evidence:** `document-builder.test.ts` failure paths for template/palette load

### Scenario 3: Edge Case

**Given:** Given a template exists but is already used in current document

**When:** When delete eligibility is evaluated

**Then:** Then protected-on-current-doc templates are not deletable

**Status:** Client Accepted

**Evidence:** `document-builder.test.ts` (`canDeletePaletteItem`, `currentDocFieldTriples`)

---

## Feature: Client Management (create/edit/list/validation)

### Scenario 1: Happy Path

**Given:** Given user enters valid client details

**When:** When form is submitted

**Then:** Then client is created/updated and available in recipient lists

**Status:** Client Accepted

**Evidence:** `client-display.test.ts` (`handleForm`, `clientToModel`, edit flow)

### Scenario 2: Error Handling

**Given:** Given invalid name/email/phone/address inputs

**When:** When validation runs

**Then:** Then field-level errors block submission

**Status:** Client Accepted

**Evidence:** `client-display.test.ts` validation suites

### Scenario 3: Edge Case

**Given:** Given expanded address input mode is enabled

**When:** When address is validated/collapsed

**Then:** Then line-level address fields normalize to final stored address string

**Status:** Client Accepted

**Evidence:** `client-display.test.ts` address collapse and validation tests

---

## Feature: Job Dashboard (status, search, filters, audit)

### Scenario 1: Happy Path

**Given:** Given sent/signed jobs exist

**When:** When user opens jobs dashboard

**Then:** Then jobs display with status and audit/timeline details

**Status:** Client Accepted

**Evidence:** `job-display.test.ts` (fetch rows, severity, modal audit data)

### Scenario 2: Error Handling

**Given:** Given delete/download backend fails

**When:** When user performs action

**Then:** Then error toast or message appears and table remains stable

**Status:** Client Accepted

**Evidence:** `job-display.test.ts` delete error handling coverage

### Scenario 3: Edge Case

**Given:** Given combined search + recipient + status filters are active

**When:** When filtering dataset

**Then:** Then result rows exactly match all active criteria

**Status:** Client Accepted

**Evidence:** `job-display.test.ts` search/filter combination tests

---

## Feature: Document Signing (single + packet docs, validation/autofill)

### Scenario 1: Happy Path

**Given:** Given a signer receives a valid signing link/job

**When:** When they complete all required fields and sign

**Then:** Then signature input submits successfully and document status advances

**Status:** Client Accepted

**Evidence:** `document-signing.test.ts` (single-document happy path and signing execution)

### Scenario 2: Error Handling

**Given:** Given backend validation returns field errors (HTTP 400) or signing API fails

**When:** When signer submits invalid/failed sign attempt

**Then:** Then field-level errors are surfaced and no client crash occurs

**Status:** Client Accepted

**Evidence:** `document-signing.test.ts` (400 error mapping + generic failure)

### Scenario 3: Edge Case

**Given:** Given multi-document job with mixed completion status

**When:** When signer opens a completed document

**Then:** Then completed documents are locked while pending docs remain actionable

**Status:** Client Accepted

**Evidence:** `document-signing.test.ts` (document status + lock behavior)

---

## Feature: Download Completed Job/Packet + Certificate

### Scenario 1: Happy Path

**Given:** Given a completed job

**When:** When user downloads output

**Then:** Then system returns correct PDF/ZIP file and expected filename(s)

**Status:** Client Accepted

**Evidence:** `FileDeleterAndPacketDownloadTests.cs`, `job-display.test.ts` (`downloadJobFile`)

### Scenario 2: Error Handling

**Given:** Given unknown/invalid job id or missing files

**When:** When packet download endpoint is called

**Then:** Then endpoint returns NotFound instead of invalid content

**Status:** Client Accepted

**Evidence:** `FileDeleterAndPacketDownloadTests.cs` NotFound coverage

### Scenario 3: Edge Case

**Given:** Given signed jobs requiring certificate of completion

**When:** When download is requested

**Then:** Then ZIP includes signed docs and `certificate_of_completion.pdf`

**Status:** Client Accepted

**Evidence:** `FileDeleterAndPacketDownloadTests.cs`, `job-display.test.ts` certificate download button/flow

---
