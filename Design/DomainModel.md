# Domain Model

```mermaid
classDiagram
    direction TB


    class Documents {
        +Id: int
        +Name: string
        +File: byte[]
        +Coordinates: string[]
        +Types: string[]
        +Date: string
    }

    class Jobs {
        +Id: int
        +DocId: int
        +FileName: string
        +ClientName: string
        +Status: string
        +JobPass: string
        +Timestamp: DateTime[]
        +Action: string[]
        +File: byte[]
        +Coordinates: string[]
        +Types: string[]
        +Date: string
    }

    class Client {
        +Id: int
        +Name: string
        +Email: string
        +Phone: string
        +Address: string
    }


    class Administrator {
        +Id: int
        +FirstName: string
        +LastName: string
        +Email: string
        +Role: string
        +CreatedDate: DateTime
        +LastLogin: DateTime
    }

    class DocumentBlueprint {
        +Id: int
        +Name: string
        +TemplateFile: byte[]
        +CreatedBy: int
        +CreatedDate: DateTime
        +IsActive: bool
    }

    class SigningArea {
        +Id: int
        +BlueprintId: int
        +PageNumber: int
        +XCoordinate: double
        +YCoordinate: double
        +Width: int
        +Height: int
        +FieldType: string
        +IsRequired: bool
    }

    class AuditTrail {
        +Id: int
        +JobId: int
        +Action: string
        +PerformedBy: int
        +Timestamp: DateTime
        +Details: string
        +IPAddress: string
    }

    class ClientGroup {
        +Id: int
        +Name: string
        +Description: string
        +CreatedBy: int
        +CreatedDate: DateTime
    }

    class DocumentRepository {
        +Id: int
        +Name: string
        +Description: string
        +IsPublic: bool
        +CreatedBy: int
        +CreatedDate: DateTime
    }


    class DocumentStatus {
        <<enumeration>>
        DRAFT
        READY
        SENT
        IN_PROGRESS
        COMPLETED
        REJECTED
        EXPIRED
    }

    class AuditType {
        <<enumeration>>
        CREATED
        VIEWED
        SENT
        SIGNED
        COMPLETED
        REJECTED
        EXPIRED
        DOWNLOADED
    }


    class ClientProfile {
        +id: string
        +name: string
        +email: string
        +phone: string
        +address: string
    }

    class GrabbedJob {
        +id: number
        +fileName: string
        +clientName: string
        +status: string
        +timestamp: Date[]
        +action: string[]
    }

    class SignedJob {
        +jobId: number
        +docName: string
        +clientName: string
        +signingInputs: string[]
    }


    class Job {
        +docId: int
        +fileName: string
        +clientName: string
        +clientEmail: string
        +status: string
    }

    class DocumentCoordinates {
        +File: int
        +Coordinates: CoordinateObject[]
        +Types: string[]
    }

    class CoordinateObject {
        +myPage: string
        +inputType: string
        +xCoord: string
        +yCoord: string
        +width: string
        +height: string
    }

    %% CORE RELATIONSHIPS
    Documents "1" o-- "0..*" Jobs
    Client "1" o-- "0..*" Jobs
    Administrator "1" o-- "0..*" Client
    Administrator "1" o-- "0..*" DocumentBlueprint
    DocumentBlueprint "1" o-- "0..*" SigningArea
    DocumentBlueprint "1" o-- "0..*" Jobs
    Jobs "1" o-- "0..*" AuditTrail
    Client "1" o-- "0..*" ClientGroup
    DocumentRepository "1" o-- "0..*" DocumentBlueprint
    DocumentRepository "1" o-- "0..*" Jobs

    %% STATUS RELATIONSHIPS
    Jobs "1" -- "1" DocumentStatus
    AuditTrail "1" -- "1" AuditType

    %% FRONTEND RELATIONSHIPS
    Client "1" -- "1" ClientProfile
    Jobs "1" -- "1" GrabbedJob
    Jobs "1" -- "1" SignedJob
    Job "1" -- "1" Jobs
    DocumentCoordinates "1" -- "1" Documents
```

## Classes

The main users of the system who manage document signing workflows. Admins can upload documents, manage clients, and monitor the signing process. Each admin will have their own login and can customize their experience with favorited documents.

### DocumentBlueprint

Template documents that admins create and reuse. Admins upload a document, set signing areas, and save it as a blueprint. These blueprints can then be used to create multiple signing jobs.

### SigningArea

Defines exactly where clients need to sign on a document. Contains coordinates (X, Y), width, height, and field type (signature, initial, etc.) for each signing location.

### Documents (Current)

Basic PDF storage in the current system. Contains the file data and simple coordinate arrays for signing fields.

### Jobs (Current)

Individual signing tasks created from documents. Tracks the signing process, client information, and basic audit trail using arrays.

### Client

People who need to sign documents. Contains contact information like name, email, phone, and address.

### ClientGroup

Organizes clients into groups for easier management. Allows admins to send documents to multiple clients at once.

### DocumentRepository

Organized storage for documents and templates. Helps admins categorize and manage their document library.
Audit and Status

### AuditTrail

Logs all actions performed on documents. Records who did what, when, and what type of action it was.

### AuditType

Types of actions that can be tracked:
CREATED: New document job created
SENT: Document sent to client
VIEWED: Client opened document
SIGNED: Client signed document
COMPLETED: All actions finished
REJECTED: Document rejected
EXPIRED: Document expired

### DocumentStatus

Current state of a document:
DRAFT: Still being prepared
READY: Ready to send
SENT: Sent to client
IN_PROGRESS: Client is signing
COMPLETED: Fully processed
REJECTED: Rejected by client
EXPIRED: Past due date
Frontend Models

### ClientProfile

Frontend version of Client data for displaying in the UI.

### GrabbedJob

Job information for display purposes, without the full document data.

### SignedJob

Completed signing data with client signatures.
Job (API)
Request data for creating new signing jobs.
DocumentCoordinates
Current system for managing signing field positions.
CoordinateObject
Specific coordinates and dimensions for each signing field.
