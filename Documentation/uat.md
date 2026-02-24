# User Acceptance Testing (UAT)

## Last Updated:

---

# 1. Feature Overview

| Feature | Priority | Iteration | Client Facing (Y/N) |
|---------|----------|-----------|---------------------|
|UploadDocument|Medium|0|Y|
|DocumentSigning|High|0|Y|

---

# 2. Acceptance Scenarios

## Feature: Upload Document

### Scenario 1: Happy Path
**Given:** Given a admin user uploads a document that is system accepted

**When:** When they want to upload a document for signing

**Then:** Then the file is uploaded (and converted if needed) to the database

**Status:** Client Accepted

**Evidence:** UploadDocumentTest.cs, DocumentConverterTest.cs

### Scenario 2: Error Handling
**Given:**

**When:**

**Then:**

**Status:**

**Evidence:**

### Scenario 3: Edge Case
...

---

## Feature: Document Signing

### Scenario 1: Happy Path
**Given:** Given a signing user and a document with fields to sign

**When:** When they recieve a valid job

**Then:** Then the user is able to fill the fields with valid data

**Status:** Client Accepted

**Evidence:** document-signing.test.ts

### Scenario 2: Error Handling
**Given:**

**When:**

**Then:**

**Status:**

**Evidence:**

### Scenario 3: Edge Case
...

---

## Feature: Upload Document

### Scenario 1: Happy Path
**Given:** Given a admin user uploads a document that is system accepted

**When:** When they want to upload a document for signing

**Then:** Then the file is uploaded (and converted if needed) to the database

**Status:** Client Accepted

**Evidence:** UploadDocumentTest.cs, DocumentConverterTest.cs

### Scenario 2: Error Handling
**Given:**

**When:**

**Then:**

**Status:**

**Evidence:**

### Scenario 3: Edge Case
...

---

# 3. Client UAT Log

These tests are validated by the client.

| Date | Feature | Client Feedback | Action Required | Resolved (Y/N) |
|------|---------|-----------------|-----------------|----------------|
||Feature1||||

---

# 4. Open Acceptance Risks

## Risk1
- Risk:
- Mitigation Plan:

## Risk2
