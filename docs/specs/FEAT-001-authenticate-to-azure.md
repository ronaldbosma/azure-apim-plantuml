# Feature Specification: Authenticate to Azure

<!--
  TEMPLATE INSTRUCTIONS
  =====================
  This template follows a spec-driven development approach for AI-assisted coding.
  Fill in each section focusing on WHAT the feature does and WHY — not HOW it should
  be implemented. Implementation details belong in the technical plan, not here.

  Usage:
  - One spec per feature or functional slice
  - Store in docs/specs/ and version-control alongside your code
  - Use [NEEDS CLARIFICATION: question] markers for unresolved decisions (max 3)
  - Remove optional sections that don't apply — don't leave them as N/A
  - Reference your arc42 architecture docs where relevant rather than duplicating them

  Workflow: Specify → Plan → Tasks → Implement
  This template covers the "Specify" phase.
-->

## 1. Overview

| Field           | Value                                      |
| --------------- | ------------------------------------------ |
| Feature ID      | FEAT-001                                   |
| Status          | Draft                                      |
| Author          | Ronald Bosma                               |
| Created         | 2026-05-06                                 |
| Last updated    | 2026-05-06                                 |
| Epic / Parent   | Epic 1: APIM Data Retrieval (E1.F1)        |
| Arc42 reference | 8. Cross-cutting Concepts — Security       |

### 1.1 Problem Statement

The CLI tool needs to authenticate to Azure before it can call the APIM REST API. Developers should not have to configure credentials separately — the tool must reuse whatever Azure credentials are already present on their workstation (from `az login`, VS Code, environment variables, etc.).

### 1.2 Goal

The tool acquires a valid Azure access token scoped to the Azure Resource Management API using `DefaultAzureCredential` from the `Azure.Identity` library, and surfaces that credential to downstream features that call the APIM REST API.

### 1.3 Non-Goals

- Explicit service principal configuration (CLI flags for client ID / secret / certificate) — that is E4.F1, targeting the 1.0 release
- Managed identity authentication in CI/CD pipelines — also E4.F1
- Triggering a browser-based interactive login flow — users are expected to have already authenticated via `az login` or equivalent
- Credential storage, rotation, or management

---

## 2. User Stories

### US-001: Transparent authentication

**As a** Developer or Architect,
**I want** the tool to automatically pick up my existing Azure credentials,
**so that** I can generate diagrams without configuring authentication separately.

### US-002: Tenant override

**As a** Developer or Architect who is logged into multiple Azure tenants,
**I want** to specify which tenant the tool should use,
**so that** the tool targets the correct APIM instance without me having to change my default tenant.

### US-003: Clear failure feedback

**As a** Developer or Architect,
**I want** a clear, actionable error message when authentication fails,
**so that** I know exactly what to fix (e.g., run `az login`) rather than seeing a cryptic stack trace.

---

## 3. Functional Requirements

| ID     | Requirement                                                                                                                       | Priority | User Story |
| ------ | --------------------------------------------------------------------------------------------------------------------------------- | -------- | ---------- |
| FR-001 | The system shall authenticate to Azure using `DefaultAzureCredential` from the `Azure.Identity` library                          | Must     | US-001     |
| FR-002 | The system shall acquire a token scoped to `https://management.azure.com/.default`                                                | Must     | US-001     |
| FR-003 | The system shall not require any authentication-related CLI parameters for local development scenarios                            | Must     | US-001     |
| FR-004 | The system shall expose an authenticated credential object to all downstream features (APIM retrieval, export)                    | Must     | US-001     |
| FR-005 | The system shall accept an optional `--tenant-id` parameter; when provided, authentication is scoped to that tenant              | Must     | US-002     |
| FR-006 | The system shall surface a clear, actionable error message when no credential in the `DefaultAzureCredential` chain succeeds      | Must     | US-003     |
| FR-007 | The system shall not log, print, or write access tokens or credential secrets to any output                                       | Must     | US-001     |

---

## 4. Acceptance Scenarios

### SC-001: Successful authentication via az CLI (FR-001, FR-002, FR-004)

```gherkin
Given the user has previously run "az login" on their workstation
  And the Azure CLI session is still valid
When the tool starts and initializes authentication
Then a valid access token scoped to "https://management.azure.com/.default" is acquired
  And the credential is available to downstream APIM API calls
```

### SC-002: No credentials available — actionable error (FR-006)

```gherkin
Given the user has no active Azure credentials on their workstation
  And no environment variables for a service principal are set
When the tool starts and initializes authentication
Then the tool exits with a non-zero exit code
  And the error message instructs the user to authenticate (e.g., "Run 'az login' to authenticate")
  And no stack trace is shown by default
```

### SC-003: Credential chain partially fails, succeeds on later provider (FR-001)

```gherkin
Given the user has no environment credential configured
  And the user has an active Azure CLI session
When the tool starts and initializes authentication
Then the credential chain falls through to the Azure CLI provider
  And a valid token is acquired without any error shown to the user
```

### SC-004: Tenant ID overrides default tenant (FR-005)

```gherkin
Given the user is logged into multiple Azure tenants via "az login"
  And the user passes "--tenant-id <specific-tenant-id>" on the command line
When the tool starts and initializes authentication
Then authentication is scoped to the specified tenant
  And the acquired token grants access to resources in that tenant
```

### SC-005: Access token is not leaked to output (FR-007)

```gherkin
Given the user runs the tool with verbose/debug output enabled
When authentication succeeds
Then the access token value does not appear in any console output or log file
```

---

## 5. Domain Model

Authentication in this feature is a cross-cutting infrastructure concern, not a rich domain object. The key abstraction is a **Credential** that downstream features depend on.

### 5.1 Entities

#### Credential

Represents the authenticated identity used to call Azure APIs. Wraps `DefaultAzureCredential` internally; exposed to downstream features through the `TokenCredential` abstraction from `Azure.Core`.

| Attribute   | Type              | Constraints | Description                                     |
| ----------- | ----------------- | ----------- | ----------------------------------------------- |
| (opaque)    | `TokenCredential` | non-null    | The underlying Azure.Identity credential object |
| tenant-id   | string (GUID)     | optional    | Tenant to scope authentication to, if provided  |

### 5.2 Relationships

- A **Credential** is produced once at tool startup and injected into all commands that call the APIM REST API.
- There is exactly one **Credential** instance per tool invocation.

### 5.3 Domain Rules and Invariants

- **Single initialization**: The credential is created once at startup and reused across all API calls. The tool does not create multiple credential instances per invocation.
- **No token exposure**: The raw token value must never be materialized outside of the `Azure.Identity` / `Azure.Core` libraries.

---

## 6. Non-Functional Requirements

| ID      | Category    | Requirement                                                                                      |
| ------- | ----------- | ------------------------------------------------------------------------------------------------ |
| NFR-001 | Performance | Token acquisition must complete within 10 seconds; if it takes longer, the tool times out and exits with an error |
| NFR-002 | Security    | The tool must not log, store, or transmit access tokens or credential secrets                    |
| NFR-003 | Reliability | Token refresh on expiry is handled transparently by `Azure.Core` — the tool does not implement its own refresh logic |

---

## 7. Edge Cases and Error Scenarios

| ID   | Scenario                                                              | Expected Behavior                                                                                              |
| ---- | --------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| EC-1 | All credential providers in the chain fail                            | Exit with non-zero code; show message directing user to `az login`                                            |
| EC-2 | Network is unavailable when acquiring a token                         | Exit with non-zero code; show message that Azure could not be reached                                         |
| EC-3 | Token expires mid-operation (e.g., long-running export)               | `Azure.Core` refreshes transparently; if refresh also fails, the operation fails with a clear error           |
| EC-4 | User is logged into multiple tenants and `--tenant-id` is not passed  | `DefaultAzureCredential` uses the default tenant from the credential source; no error is shown                |
| EC-5 | `--tenant-id` is passed but the tenant ID is invalid or inaccessible  | Authentication fails; tool exits with non-zero code and shows the tenant ID that was rejected                 |
| EC-6 | `Azure.Identity` throws an `AuthenticationFailedException`            | Caught at the top-level error handler; inner exception detail shown only with `--verbose`                     |

---

## 8. Success Criteria

| ID     | Criterion                                                                                         |
| ------ | ------------------------------------------------------------------------------------------------- |
| SC-001 | All acceptance scenarios pass in CI                                                               |
| SC-002 | A developer with an active `az login` session can run any diagram command without additional setup |
| SC-003 | A developer with no Azure credentials receives an error message containing a concrete remediation step |
| SC-004 | A developer with multiple tenants can target a specific tenant via `--tenant-id`                  |
| SC-005 | No access token value appears in any tool output under any verbosity setting                      |

---

## 9. Dependencies and Constraints

### 9.1 Dependencies

- `Azure.Identity` NuGet package — provides `DefaultAzureCredential`
- `Azure.Core` NuGet package — provides the `TokenCredential` abstraction
- E1.F2 (Retrieve APIM configuration) and E1.F3 (Export) depend on this feature being implemented first

### 9.2 Constraints

- MVP scope excludes explicit service principal configuration (reserved for E4.F1)
- The tool must work on Windows, macOS, and Linux (all platforms supported by `DefaultAzureCredential`)

### 9.3 Architecture References

| Arc42 Section             | Relevance                                                         |
| ------------------------- | ----------------------------------------------------------------- |
| 8. Cross-cutting Concepts | Authentication pattern — `DefaultAzureCredential` is the standard |

---

## 10. Open Questions

No open questions.

---

<!--
  CHECKLIST — Complete before moving to the Plan phase
  ====================================================
  - [x] Problem statement is clear and concise
  - [x] All user stories have acceptance scenarios
  - [x] Each functional requirement traces to a user story
  - [x] Domain model covers all entities mentioned in the requirements
  - [x] Domain rules and invariants are listed
  - [x] Edge cases cover failure modes, not just happy paths
  - [x] Non-functional requirements are specific and measurable
  - [x] Arc42 references point to the right sections
  - [x] No more than 3 [NEEDS CLARIFICATION] markers remain
  - [x] Open questions are assigned and have a resolution path
-->
