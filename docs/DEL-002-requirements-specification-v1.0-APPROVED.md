# Requirements Specification

## Public Procurement Monitoring and Compliance Management System (PPMCMS)

**Document ID:** PPMCMS-DEL-002
**Version:** 1.0
**Status:** Approved — Project Owner approval recorded 24 November 2026
**Project:** PPMCMS — Public Procurement Monitoring and Compliance Management System
**Owner:** Eugen Walubengo Nyongesa
**Date:** 24 November 2026
**Parent document:** PPMCMS-DEL-001 — Project Charter, Version 1.0 (approved 24 November 2026)
**Repository path:** `docs/DEL-002-requirements-specification.md`

---

## Document control

This document translates the scope and domain baseline in DEL-001 into requirements that can be designed, implemented and verified. It is a working specification for an independent educational portfolio prototype.

The system is not an official PPRA application, does not integrate with government procurement platforms, and must use fictional data only. Requirements in this document do not establish legal compliance or PPRA endorsement.

The Project Owner has approved this document as the requirements baseline subordinate to DEL-001 Version 1.0. The alignment items in Annex E are retained as explicit change-control actions for a future coordinated revision; they do not silently amend the approved parent charter. Any resulting changes shall be assessed, versioned and approved through project change control. Git history is the authoritative version record; separate revision tables are not maintained.

This Version 1.0 baselines the requirements developed through the Version 0.2 revision of the Project Owner's technical review of Version 0.1. It clarifies domain ownership, state transitions, rule execution, seed-data feasibility, test traceability, security controls and operational requirements. It is approved against DEL-001 Version 1.0; it does not amend the parent charter.

### Status terminology

- **Draft:** prepared but not approved.
- **Self-reviewed:** checked by the Project Owner.
- **Externally reviewed:** received genuine review from an identified external reviewer.
- **Approved:** explicitly approved by the Project Owner.
- **Superseded:** replaced by a later version.

No simulated persona is a real stakeholder or reviewer. Persona-derived needs are design inputs, not evidence of interviews or institutional approval.

## 1. Purpose

The purpose of this specification is to define what PPMCMS must do, the quality and security constraints it must satisfy, and how each requirement will be verified.

The requirements cover:

1. User and access management.
2. Procuring entities, suppliers, tenders and contracts.
3. Tender lifecycle controls.
4. Procurement compliance rules and findings.
5. Dashboard and data export.
6. Auditability.
7. REST API behaviour.
8. Infrastructure, security, recovery and usability constraints.

## 2. Product scope and boundaries

### 2.1 In scope

- A web-based procurement monitoring prototype.
- Four roles: Administrator, Procurement Officer, Compliance Officer and Auditor.
- Procuring-entity-level access scoping.
- Fictional procurement records and reproducible seed data.
- Tender lifecycle and contract management.
- Configurable compliance rules where specified.
- Reviewable compliance findings.
- Append-only, hash-chained audit records.
- Dashboard and CSV export if delivered within the agreed priority.
- A same-origin REST API sharing the application service layer with the web interface.
- Docker Compose deployment with Nginx, Django/Gunicorn and PostgreSQL.
- Security, testing, CI, backup and restore evidence.

### 2.2 Out of scope

- Official PPRA or government deployment, endorsement or integration.
- PPIP, e-GP or other external-system integration.
- Real procurement transactions or confidential records.
- File uploads; tender documents are metadata only.
- MFA, SSO, email notifications and a React single-page application.
- PDF report generation, high availability, failover, monitoring and alerting in the MVP.
- Legal determinations or automated findings that assert fraud or misconduct.

## 3. Stakeholder and user analysis

The five simulated stakeholder personas below match DEL-001 §9.2. They are project-analysis personas only; they are not real interviewees, reviewers or PPRA representatives.

| Persona ID | Simulated stakeholder persona | Needs                                                                                      | Linked requirement areas                            |
| ---------- | ----------------------------- | ------------------------------------------------------------------------------------------ | --------------------------------------------------- |
| PER-001    | Procurement Officer           | Maintain tender and contract records, use the workflow, and review entity-scoped findings  | FR-018–FR-050, FR-054–FR-058                      |
| PER-002    | ICT Officer                   | Understand application functions, data flows, deployment and support behaviour             | FR-001–FR-070, NFR-013–NFR-024                    |
| PER-003    | ICT Security Officer          | Review access controls, security configuration, audit integrity and security evidence      | FR-001–FR-012, FR-059–FR-084, NFR-001–NFR-012    |
| PER-004    | System Administrator          | Operate the application and infrastructure, manage backups and restore the service         | FR-001–FR-012, NFR-006–NFR-016 and NFR-025 onward |
| PER-005    | End User                      | Navigate the application, understand validation and findings, and complete permitted tasks | FR-001–FR-084, NFR-017–NFR-024                    |

### 3.1 Application roles are not stakeholder personas

The application has four authorisation roles: Administrator, Procurement Officer, Compliance Officer and Auditor. These are system account roles, not substitutes for the five stakeholder personas above. Their permissions are defined in Section 10.

### 3.2 User stories

The following user stories are simulated and link stakeholder needs to individual requirement IDs. The requirement-level acceptance criteria in Annex A are the testable acceptance baseline.

| Story ID | As a...              | I want...                                                                  | So that...                                                  | Linked requirements                                                   |
| -------- | -------------------- | -------------------------------------------------------------------------- | ----------------------------------------------------------- | --------------------------------------------------------------------- |
| US-001   | Procurement Officer  | to authenticate and manage my assigned entity's procurement records        | I can perform authorised procurement work                   | FR-001–FR-009, FR-013–FR-050                                        |
| US-002   | ICT Officer          | to understand and operate a documented web application, API and deployment | the prototype can be installed, maintained and demonstrated | FR-065–FR-070, NFR-013–NFR-024                                      |
| US-003   | ICT Security Officer | to enforce least privilege, secure sessions and tamper-evident audit       | access and changes can be reviewed and tested               | FR-010–FR-012, FR-059–FR-068, NFR-001–NFR-012                      |
| US-004   | System Administrator | to manage accounts and recover the service safely                          | users and the demonstration environment can be administered | FR-010–FR-012, FR-071–FR-078, NFR-006–NFR-016 and NFR-025–NFR-035 |
| US-005   | End User             | to view understandable dashboards, validation and findings                 | I can interpret the prototype's results                     | FR-042–FR-058, NFR-017–NFR-024                                      |

## 4. Functional requirements

Requirements use unique identifiers. Priority is based on the charter's MoSCoW baseline.

### 4.1 Authentication and account management

| ID     | Requirement                                                                                                                                                                                       | Priority | Verification                                      |
| ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | ------------------------------------------------- |
| FR-001 | The system shall require an authenticated user to access protected application pages and API resources.                                                                                           | Must     | Authentication and API tests                      |
| FR-002 | The system shall allow a user to log in and log out using Django session authentication.                                                                                                          | Must     | Functional tests                                  |
| FR-003 | The system shall store passwords using Argon2 password hashing and shall not store plaintext passwords.                                                                                           | Must     | Configuration and database inspection             |
| FR-004 | The system shall enforce a minimum password length of 12 characters and a common-password check.                                                                                                  | Must     | Validation tests                                  |
| FR-005 | The system shall impose a 15-minute lockout immediately after the fifth failed login attempt within a rolling 15-minute window. The lockout response shall not reveal whether the account exists. | Must     | Boundary, timing and response-equivalence tests   |
| FR-006 | The system shall return the same externally observable login failure message for an unknown account, an incorrect password and an account currently locked.                                       | Must     | Response-equivalence tests across all three cases |
| FR-007 | The system shall enforce a 30-minute idle session timeout and an 8-hour absolute session lifetime.                                                                                                | Must     | Session expiry tests                              |
| FR-008 | The system shall invalidate the active session on logout and password change.                                                                                                                     | Must     | Session tests                                     |
| FR-009 | The system shall allow an authenticated user to change their own password after verifying the current password.                                                                                   | Must     | Password-change tests                             |
| FR-010 | The system shall allow only Administrators to create user accounts and assign an initial application role at creation.                                                                            | Must     | RBAC and account-creation tests                   |
| FR-011 | The system shall allow only Administrators to edit another user's account details, deactivate another user, unlock an account and initiate an administrative password reset.                      | Must     | Positive and negative RBAC tests                  |
| FR-012 | The system shall audit successful login, failed login, lockout, logout, password change, administrative unlock, administrative password reset and every user-role change.                         | Must     | Event-by-event audit tests                        |

### 4.2 Procuring entities

| ID     | Requirement                                                                                                                                | Priority | Verification                     |
| ------ | ------------------------------------------------------------------------------------------------------------------------------------------ | -------- | -------------------------------- |
| FR-013 | The system shall allow Administrators to create, edit and manage procuring-entity records.                                                 | Must     | CRUD and RBAC tests              |
| FR-014 | The system shall allow Compliance Officers and Auditors to view all procuring entities.                                                    | Must     | RBAC tests                       |
| FR-015 | The system shall allow Procurement Officers to view only their assigned procuring entity.                                                  | Must     | Object-level authorisation tests |
| FR-016 | The system shall associate each Procurement Officer account with an authorised procuring entity.                                           | Must     | Model and validation tests       |
| FR-017 | The system shall prevent a user from accessing records belonging to an unauthorised entity by changing an identifier or request parameter. | Must     | Cross-entity API and UI tests    |

### 4.3 Suppliers

| ID     | Requirement                                                                                                                                                                                                    | Priority | Verification                                   |
| ------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | ---------------------------------------------- |
| FR-018 | The system shall allow a Procurement Officer to create a shared supplier master record and edit only its non-eligibility identity fields.                                                                      | Must     | RBAC and field-level permission tests          |
| FR-019 | The system shall make shared supplier master records visible to Procurement Officers, Compliance Officers and Auditors; supplier records shall not carry a procuring-entity ownership key.                     | Must     | RBAC and schema tests                          |
| FR-020 | The system shall capture supplier identity separately from a dated supplier-eligibility history containing status, valid-from date, valid-to date and recorded source/reference.                               | Must     | Model, history and date-boundary tests         |
| FR-021 | The system shall allow only Compliance Officers to create a new supplier-eligibility period or supersede an existing period; prior periods shall remain historically inspectable and changes shall be audited. | Must     | RBAC, history and audit tests                  |
| FR-022 | The system shall prevent Administrators from viewing supplier, tender, contract and procurement-finding data through application interfaces or API endpoints.                                                  | Must     | Negative authorisation tests across UI and API |

### 4.4 Tenders and lifecycle

| ID     | Requirement                                                                                                                                                                                                                                                                                                                                | Priority | Verification                                |
| ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------- | ------------------------------------------- |
| FR-023 | The system shall allow a Procurement Officer to create and edit Draft tenders belonging to their own entity.                                                                                                                                                                                                                               | Must     | RBAC and CRUD tests                         |
| FR-024 | The system shall capture a tender reference, title, procurement method, estimated value, publication instant, closing instant and lifecycle state.                                                                                                                                                                                         | Must     | Form, model and timezone tests              |
| FR-025 | The system shall reject creation or amendment of a tender whose reference duplicates any existing tender reference in the same procuring entity, including a soft-deleted tender. The database shall enforce the uniqueness invariant.                                                                                                     | Must     | Constraint and duplicate-write tests        |
| FR-026 | The system shall enforce only the valid tender state transitions defined in Section 7.                                                                                                                                                                                                                                                     | Must     | State-transition tests                      |
| FR-027 | The system shall reject invalid state transitions and return a clear validation response.                                                                                                                                                                                                                                                  | Must     | Negative workflow tests                     |
| FR-028 | The system shall validate the provisional method-specific required-field matrix in Section 8.1 before a tender can be Published. The matrix is prototype logic pending OA-004 legal verification and is not a statement of statutory requirements.                                                                                         | Must     | Per-method validation tests                 |
| FR-029 | The system shall require an evaluation outcome record, evaluator identity and evaluation timestamp before a tender can be advanced to Awarded.                                                                                                                                                                                             | Must     | Workflow and audit tests                    |
| FR-030 | The system shall require the selected supplier and award date when a tender is advanced to Awarded. The tender is the source of truth for these award facts; no separate award-value field shall be stored on the tender.                                                                                                                  | Must     | Validation and data-model tests             |
| FR-031 | The system shall require a cancellation reason when a tender is cancelled.                                                                                                                                                                                                                                                                 | Must     | Validation tests                            |
| FR-032 | The system shall prohibit edits to tender identity, method, estimate and publication fields after publication, while allowing only the state-specific evaluation and award fields in Section 8.2 to be completed by the authorised Procurement Officer. Awarded, Contracted and Cancelled tenders shall be locked.                         | Must     | Field-by-state permission tests             |
| FR-033 | The system shall permit soft deletion only for Draft tenders and shall record actor, timestamp and reason; the deleted tender reference shall remain reserved and shall not be reused. Suppliers and procuring entities shall be deactivated, not deleted. Contracts shall not be deleted; Draft contracts may be cancelled with a reason. | Must     | Lifecycle, uniqueness and audit tests       |
| FR-034 | The system shall not hard-delete users, procuring entities, suppliers, tenders, contracts, findings, eligibility history or audit events through application functionality.                                                                                                                                                                | Must     | Negative deletion tests and database review |
| FR-035 | The system should support four-eyes award approval, requiring the awarding user to differ from the user who recorded the evaluation outcome.                                                                                                                                                                                               | Should   | Workflow tests if implemented               |

### 4.5 Contracts

| ID     | Requirement                                                                                                                                                                                                                                                                          | Priority | Verification                                     |
| ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------- | ------------------------------------------------ |
| FR-036 | The system shall allow authorised Procurement Officers to create and edit contracts for their own entity before the record is locked.                                                                                                                                                | Must     | RBAC and CRUD tests                              |
| FR-037 | The system shall associate each contract with a tender and supplier.                                                                                                                                                                                                                 | Must     | Relationship and validation tests                |
| FR-038 | The system shall capture a unique contract reference, contract value, start date and end date. Supplier and award date shall be inherited read-only from the associated tender.                                                                                                      | Must     | Field validation and read-only inheritance tests |
| FR-039 | The system shall enforce at most one active contract per tender. A contract is active while its status is Draft or Executed; Completed and Terminated contracts are inactive. The invariant shall be enforced by a database constraint.                                              | Must     | Status and partial-unique-constraint tests       |
| FR-040 | The system shall permit a Draft contract to be edited while its associated tender is Awarded or Contracted. The contract shall become immutable when its own status changes from Draft to Executed; subsequent status transitions shall not permit changes to its commercial fields. | Must     | Contract lifecycle and field-lock tests          |
| FR-041 | The system shall store monetary values as decimal amounts with an explicit currency; the MVP currency is KES.                                                                                                                                                                        | Must     | Model and API tests                              |

### 4.6 Compliance rules and findings

| ID     | Requirement                                                                                                                                                                                                                                                                                                                                             | Priority | Verification                        |
| ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | ----------------------------------- |
| FR-042 | The system shall evaluate applicable enabled detective rules within the same database transaction as the business write that triggers evaluation; the business write shall roll back if required rule evaluation or finding persistence fails.                                                                                                          | Must     | Atomicity and rollback tests        |
| FR-043 | The system shall provide an on-demand re-evaluation command and shall record its start, completion, scope and result.                                                                                                                                                                                                                                   | Must     | Command and result tests            |
| FR-044 | The system shall create or update one finding per rule and affected record, rather than creating a duplicate finding on every re-run.                                                                                                                                                                                                                   | Must     | Idempotency and uniqueness tests    |
| FR-045 | Each finding shall store the rule ID and version, parameters snapshot, affected record, severity, first-detected timestamp, last-evaluated timestamp, status and timestamps.                                                                                                                                                                            | Must     | Persistence and snapshot tests      |
| FR-046 | The system shall support finding transitions Open → Acknowledged → Resolved, Open → Resolved, and Open/Acknowledged → False Positive. Resolved findings may reopen to Open if the condition recurs; False Positive findings remain suppressed until the underlying record or rule version changes and a Compliance Officer explicitly reopens them. | Must     | Transition and re-open tests        |
| FR-047 | The system shall store finding comments as separate records with author and timestamp, and audit every comment and status transition.                                                                                                                                                                                                                   | Must     | Comment persistence and audit tests |
| FR-048 | The system shall allow Procurement Officers to view and acknowledge findings for their own entity; acknowledgement shall not resolve or suppress the finding.                                                                                                                                                                                           | Must     | RBAC and status tests               |
| FR-049 | The system shall allow Compliance Officers to view findings across entities, add comments, resolve findings, mark findings as False Positive and explicitly reopen a False Positive finding.                                                                                                                                                            | Must     | RBAC and lifecycle tests            |
| FR-050 | The system shall prevent Auditors from creating comments or changing finding status.                                                                                                                                                                                                                                                                    | Must     | Negative RBAC tests                 |
| FR-051 | The system shall expose a restricted API or administrative path through which only Compliance Officers can change rule parameters; every change shall be audited and versioned.                                                                                                                                                                         | Must     | API/RBAC/audit tests                |
| FR-052 | The system shall not generate active findings from disabled or unverified rules.                                                                                                                                                                                                                                                                        | Must     | Rule-engine tests                   |
| FR-053 | RULE-007 shall be treated as a low-severity heuristic and shall not be described as proof of misconduct or a legal breach.                                                                                                                                                                                                                              | Should   | Rule configuration and UI review    |

### 4.7 Dashboard and export

| ID     | Requirement                                                                                                                                                                                                                                          | Priority | Verification                                    |
| ------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | ----------------------------------------------- |
| FR-054 | The system shall show, at minimum, total tenders by lifecycle state, total contracts by status, total contract value (KES), tenders by procuring entity, open findings by severity, findings by rule ID and the five most recently updated findings. | Must     | Metric-by-metric seed-data reconciliation tests |
| FR-055 | Dashboard data shown to Procurement Officers shall be restricted to their own entity.                                                                                                                                                                | Must     | RBAC and query tests                            |
| FR-056 | Dashboard data shown to Compliance Officers and Auditors may cover all entities.                                                                                                                                                                     | Must     | RBAC tests                                      |
| FR-057 | The system should allow authorised users to export permitted procurement and findings data to CSV.                                                                                                                                                   | Should   | Export content and access tests                 |
| FR-058 | Dashboard totals shall be reconcilable to the underlying seed records.                                                                                                                                                                               | Must     | Known-data test                                 |

### 4.8 Audit trail

| ID     | Requirement                                                                                                                                                                                                  | Priority | Verification                                  |
| ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------- | --------------------------------------------- |
| FR-059 | The system shall create an audit event for every successful business write and every authentication/security event listed in FR-012. Business writes and audit inserts shall commit or roll back atomically. | Must     | Endpoint, event-coverage and rollback tests   |
| FR-060 | Audit events shall record a monotonically increasing sequence number, actor (human or system), action, target, canonicalised payload, UTC timestamp, previous hash and current SHA-256 hash.                 | Must     | Schema, canonicalisation and integrity tests  |
| FR-061 | The audit log shall be append-only from the application runtime database role; audit inserts shall lock a singleton chain-head row within a transaction so concurrent workers cannot fork the chain.         | Must     | Database privilege and concurrent-write tests |
| FR-062 | The system shall provide a verification command that recomputes the audit hash chain and reports integrity failures.                                                                                         | Must     | Tamper-detection test                         |
| FR-063 | Only Administrators and Auditors may view the audit log; Auditor access shall be read-only.                                                                                                                  | Must     | RBAC and negative write tests                 |
| FR-064 | No application role shall be able to modify or delete existing audit entries through application functionality.                                                                                              | Must     | Negative tests and database inspection        |

| FR-071 | The system shall allow an Administrator to unlock a locked account and initiate a password reset that forces a password change at the next successful login. | Must | Account administration tests |
| FR-072 | The system shall prevent deactivation of the last active Administrator and prevent an Administrator from deactivating their own account. | Must | Last-admin and self-deactivation tests |
| FR-073 | The system shall store tender evaluation outcome as a controlled enum (Pass, Fail, or Inconclusive) with an optional explanatory note, `evaluated_by` user reference and `evaluated_at` timestamp. Only Pass or Fail may proceed to award; Inconclusive shall not. | Must | Enum, actor and workflow tests |
| FR-074 | The system shall maintain a separate immutable eligibility-period history for each supplier, with non-overlapping validity intervals; corrections shall supersede rather than erase prior entries. | Must | History integrity and overlap tests |
| FR-075 | The system shall create a system-actor audit identity for automated rule evaluation and finding lifecycle changes, distinct from human user accounts. | Must | System actor and audit tests |
| FR-076 | The system shall support contract states Draft, Executed, Completed and Terminated, with only Draft → Executed → Completed or Terminated transitions; Draft → Terminated is permitted with a reason. | Must | Contract state-machine tests |
| FR-077 | The system shall permit Compliance Officers to reopen a False Positive finding explicitly, recording the actor, reason and time. | Must | Reopen and audit tests |
| FR-078 | The system shall allow an authenticated user to update only their own non-privileged profile fields; role, entity assignment and account status shall remain Administrator-controlled. | Must | Field-level profile permission tests |
| FR-079 | The system should provide a rule-parameter management user interface; the Must API/administrative path remains available if this UI is deferred. | Should | UI test if implemented |
| FR-080 | The system shall prevent clients from mass-assigning server-controlled fields, including role, entity ownership, audit metadata, finding state and lifecycle state except through explicit authorised transition actions. | Must | Serializer allow-list and mass-assignment tests |
| FR-081 | The system shall return 404 for a resource identifier that exists but is outside the caller's entity scope, to avoid confirming its existence. | Must | Cross-entity response tests |
| FR-082 | The system shall evaluate supplier eligibility changes against all affected awarded tenders/contracts and update existing findings idempotently. | Must | Supplier-triggered re-evaluation tests |
| FR-083 | The system shall re-evaluate affected findings when a Compliance Officer changes a rule parameter or rule version. | Must | Parameter-triggered re-evaluation tests |
| FR-084 | The system shall preserve a finding's history when its underlying condition is corrected, changing Open/Acknowledged to Resolved by system evaluation with a recorded resolution reason; it shall not silently delete the finding. | Must | Condition-correction lifecycle tests |

### 4.9 REST API

| ID     | Requirement                                                                                                                                            | Priority | Verification                                                                                               |
| ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------ | -------- | ---------------------------------------------------------------------------------------------------------- |
| FR-065 | The system shall expose REST API endpoints for the core resources used by the application.                                                             | Must     | API integration tests                                                                                      |
| FR-066 | The web interface and REST API shall share the same application service layer and business rules.                                                      | Must     | Code review and parity tests                                                                               |
| FR-067 | The API shall enforce authentication, RBAC and procuring-entity scoping equivalent to the web interface.                                               | Must     | API authorisation tests                                                                                    |
| FR-068 | The API shall validate input and return the standard error representation defined in Section 9.2 and Annex D without exposing stack traces or secrets. | Must     | API error-contract tests                                                                                   |
| FR-069 | The API should publish a generated OpenAPI schema or interactive documentation page.                                                                   | Should   | Schema validation                                                                                          |
| FR-070 | A Postman collection and Newman CI execution should be provided if schedule permits.                                                                   | Should   | Newman run evidence; collection uses session login and obtains/submits a CSRF token before unsafe requests |

## 5. Non-functional requirements

| ID      | Requirement                                                                                                                                                                                                                                               | Priority | Acceptance measure / verification                                                                                                                 |
| ------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| NFR-001 | Security: access shall be denied by default and authorised at object level.                                                                                                                                                                               | Must     | Every RBAC allow/deny cell has an automated test; cross-entity access returns 404.                                                                |
| NFR-002 | Security: unsafe browser and API requests shall be protected by CSRF controls.                                                                                                                                                                            | Must     | Automated tests and ZAP baseline.                                                                                                                 |
| NFR-003 | Security: session cookies shall use HttpOnly, Secure and SameSite=Lax.                                                                                                                                                                                    | Must     | Cookie inspection over HTTPS.                                                                                                                     |
| NFR-004 | Security: the same-origin MVP shall not enable cross-origin access by default.                                                                                                                                                                            | Must     | No permissive CORS policy; configuration inspection.                                                                                              |
| NFR-005 | Security: responses shall include CSP, HSTS, X-Content-Type-Options, Referrer-Policy and frame-ancestors none.                                                                                                                                            | Must     | Response-header inspection and ZAP.                                                                                                               |
| NFR-006 | Security: database access shall use separate migration and runtime roles; runtime shall not be a superuser.                                                                                                                                               | Must     | Database privilege inspection.                                                                                                                    |
| NFR-007 | Security: the database service port shall not be published to the host.                                                                                                                                                                                   | Must     | Compose inspection and connectivity test.                                                                                                         |
| NFR-008 | Security: secrets shall not be committed; `.env` shall be ignored and `.env.example` shall contain placeholders only.                                                                                                                                 | Must     | gitleaks and repository review.                                                                                                                   |
| NFR-009 | Security: Nginx and application containers shall run as non-root users; pinned image versions and health checks shall be used.                                                                                                                            | Must     | Container user, image and health-check inspection.                                                                                                |
| NFR-010 | Security: application errors shall not expose stack traces, credentials or secrets.                                                                                                                                                                       | Must     | Error-path tests and review.                                                                                                                      |
| NFR-011 | Security: dependencies and container images shall be scanned using tools selected in DEL-001.                                                                                                                                                             | Must     | CI logs and scan reports.                                                                                                                         |
| NFR-012 | Auditability: audit events shall be tamper-evident through a serialised hash chain.                                                                                                                                                                       | Must     | Deliberate tampering is detected.                                                                                                                 |
| NFR-013 | Performance: dashboard shall load within 2 seconds on the agreed seed dataset in the local demonstration environment.                                                                                                                                     | Must     | Repeatable timing evidence; environment and method recorded.                                                                                      |
| NFR-014 | Performance: paginated list API endpoints shall achieve p95 response time below 500 ms on the agreed seed dataset in the local demonstration environment.                                                                                                 | Must     | Timed API test; environment and method recorded.                                                                                                  |
| NFR-015 | Recovery: backup and restore shall be documented and rehearsed; elapsed time shall be recorded against the 15-minute target.                                                                                                                              | Must     | Restore into a clean database, row-count comparison and elapsed-time record. The target is measured, not an automatic pass/fail gate if exceeded. |
| NFR-016 | Deployability: the Project Owner shall perform a clean-clone deployment using the README, targeting 15 minutes from clone to healthy stack.                                                                                                               | Must     | Timed rehearsal includes image pulls; prerequisite installation is completed before the timed run and recorded separately.                        |
| NFR-017 | Usability: main pages shall be keyboard navigable and provide field-specific validation and actionable error messages.                                                                                                                                    | Must     | Accessibility checklist and functional review.                                                                                                    |
| NFR-018 | Compatibility: the application shall work in current Chrome, Firefox and Edge versions available during testing.                                                                                                                                          | Must     | Browser test record.                                                                                                                              |
| NFR-019 | Maintainability: code shall use consistent formatting, linting and automated tests.                                                                                                                                                                       | Must     | Ruff and pytest CI results.                                                                                                                       |
| NFR-020 | Testability: every Must requirement shall have a Given/When/Then acceptance criterion and at least one linked automated or explicitly justified manual test.                                                                                              | Must     | Requirements coverage CI check and traceability report.                                                                                           |
| NFR-021 | Data integrity: monetary values shall use decimal storage; timestamps shall be UTC and business-date comparisons shall use Africa/Nairobi.                                                                                                                | Must     | Model, timezone and boundary tests.                                                                                                               |
| NFR-022 | Privacy: only fictional user and procurement data shall be used; demo account data shall be minimised and deleted or anonymised at project end.                                                                                                           | Must     | Seed-data review and end-of-project checklist.                                                                                                    |
| NFR-023 | Motion/accessibility: animations should respect reduced-motion preferences and shall not be necessary to understand status or complete a task.                                                                                                            | Should   | Manual accessibility review if included.                                                                                                          |
| NFR-024 | Responsive design: core workflows should remain usable on desktop and tablet-width viewports.                                                                                                                                                             | Should   | Responsive layout review if included.                                                                                                             |
| NFR-025 | Transport security: Nginx shall serve HTTPS; HTTP requests shall redirect to HTTPS. Local demonstration certificates may be self-signed or locally trusted as documented.                                                                                 | Must     | HTTP redirect and HTTPS handshake tests.                                                                                                          |
| NFR-026 | Network protection: Nginx shall apply rate limits to login and API routes, with documented limits and appropriate 429 responses.                                                                                                                          | Must     | Configuration inspection and rate-limit tests.                                                                                                    |
| NFR-027 | Backup protection: database backups shall be stored outside the database container volume and outside Git, with a SHA-256 checksum verified before restore.                                                                                               | Must     | Backup path, checksum and restore evidence.                                                                                                       |
| NFR-028 | Audit concurrency: audit-chain writes shall be serialised across concurrent application workers using a database transaction and locked chain-head row or equivalent database-enforced mechanism.                                                         | Must     | Concurrent-write test confirms a single valid sequence.                                                                                           |
| NFR-029 | Audit canonicalisation: hash input shall use a documented deterministic UTF-8 canonical serialisation, stable field ordering, normalised UTC timestamps, decimal strings and a monotonically increasing sequence number.                                  | Must     | Repeatability and independent recomputation tests.                                                                                                |
| NFR-030 | API pagination: collection endpoints shall use deterministic pagination with default page size 25 and maximum page size 100; stable ordering shall include a unique tie-breaker.                                                                          | Must     | Pagination boundary and performance tests.                                                                                                        |
| NFR-031 | API error contract: errors shall use a consistent `application/problem+json` representation with stable `type`, `title`, `status`, `detail`, `instance` and application `code` fields; field errors may be included in an `errors` array. | Must     | Schema and endpoint error tests.                                                                                                                  |
| NFR-032 | API throttling: rate-limited API requests shall return HTTP 429 and a `Retry-After` header when the retry interval is known.                                                                                                                            | Must     | Throttle tests.                                                                                                                                   |
| NFR-033 | Time handling: stored instants shall be timezone-aware UTC. Publication and closing instants shall be interpreted in Africa/Nairobi for procurement business rules; the application clock shall be injectable in tests.                                   | Must     | Frozen-clock and timezone-boundary tests.                                                                                                         |
| NFR-034 | Audit atomicity: business writes and their required audit events shall commit or roll back together.                                                                                                                                                      | Must     | Forced audit-write failure rolls back the business transaction.                                                                                   |
| NFR-035 | Finding consistency: rule evaluation and finding updates triggered by a business write shall be atomic with that write; deferred work, if used for supplier/rule changes, shall be durable, idempotent and visibly pending.                               | Must     | Transaction rollback and task-retry tests.                                                                                                        |

Performance targets are prototype targets, not production service-level guarantees. Results are valid only when the test environment, dataset size, request method and measurement approach are recorded.

## 6. Confirmed business rules and defaults

The following values are baselined from DEL-001 and must not be silently changed during implementation.

| Setting                           | Confirmed value                                                                                      |
| --------------------------------- | ---------------------------------------------------------------------------------------------------- |
| Failed login attempts             | 5                                                                                                    |
| Lockout window                    | 15 minutes                                                                                           |
| Idle session timeout              | 30 minutes                                                                                           |
| Absolute session lifetime         | 8 hours                                                                                              |
| Password minimum length           | 12 characters                                                                                        |
| RULE-002 estimate tolerance       | 0%                                                                                                   |
| RULE-007 repeat-award threshold   | More than 3 awards (at least 4) by the same entity to the same supplier in a rolling 12-month window |
| Compliance module test coverage   | At least 80%                                                                                         |
| Dashboard response target         | 2 seconds                                                                                            |
| List API p95 target               | Under 500 ms                                                                                         |
| Backup restore target             | 15 minutes                                                                                           |
| Seed procuring entities           | 5                                                                                                    |
| Seed suppliers                    | 60                                                                                                   |
| Seed tenders                      | 250                                                                                                  |
| Seed contracts                    | 150                                                                                                  |
| Procurement Officers per entity   | At least 2, to support the optional four-eyes workflow                                               |
| API default/max page size         | 25 / 100                                                                                             |
| Tender and contract time handling | UTC storage; Nairobi-local business interpretation                                                   |
| Account lockout duration          | 15 minutes from the fifth qualifying failed attempt                                                  |

Money shall be represented using decimal values and explicit currency (KES). All instants are stored as timezone-aware UTC values. Procurement publication and closing instants are interpreted in `Africa/Nairobi` for business rules. The application clock must be injectable so tests can freeze time at boundaries. Seed data shall be fictional and reproducible.

## 7. Tender and contract workflows

### 7.1 Tender state machine

| Current state                         | Permitted next state | Actor and precondition                                                                                                                                      |
| ------------------------------------- | -------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Draft                                 | Published            | Procurement Officer for owning entity; provisional method-specific fields complete                                                                          |
| Published                             | Closed               | Procurement Officer for owning entity; injected current instant is on or after closing instant in Africa/Nairobi                                            |
| Closed                                | Evaluated            | Procurement Officer for owning entity; evaluation outcome, evaluator and timestamp recorded                                                                 |
| Evaluated                             | Awarded              | Procurement Officer for owning entity; outcome is Pass or Fail, supplier and award date recorded; if four-eyes is implemented, actor differs from evaluator |
| Awarded                               | Contracted           | Procurement Officer for owning entity; at least one Draft contract exists and is associated with the tender                                                 |
| Draft, Published, Closed or Evaluated | Cancelled            | Procurement Officer for owning entity; cancellation reason recorded                                                                                         |

All other transitions shall be rejected. Awarded, Contracted and Cancelled tenders are locked against changes to all tender fields, except permitted linked Draft contract work where stated below. Tender identity and published procurement terms are not editable after publication.

### 7.2 Tender field editability by state

| Field group                                 | Draft                            | Published    | Closed                         | Evaluated                    | Awarded   | Contracted                       | Cancelled                      |
| ------------------------------------------- | -------------------------------- | ------------ | ------------------------------ | ---------------------------- | --------- | -------------------------------- | ------------------------------ |
| Tender reference, title, method, estimate   | Edit                             | Read-only    | Read-only                      | Read-only                    | Read-only | Read-only                        | Read-only                      |
| Publication and closing instants            | Edit                             | Read-only    | Read-only                      | Read-only                    | Read-only | Read-only                        | Read-only                      |
| Method-required publication fields          | Edit                             | Read-only    | Read-only                      | Read-only                    | Read-only | Read-only                        | Read-only                      |
| Evaluation outcome, evaluator, evaluated_at | Empty/editable only after Closed | Not editable | Set on transition to Evaluated | Read-only                    | Read-only | Read-only                        | Read-only                      |
| Awarded supplier and award date             | Empty                            | Empty        | Empty                          | Set on transition to Awarded | Read-only | Read-only                        | Read-only                      |
| Cancellation reason                         | Empty                            | Empty        | Empty                          | Empty                        | Empty     | Empty                            | Set on transition to Cancelled |
| Linked Draft contract fields                | N/A                              | N/A          | N/A                            | N/A                          | Editable  | Editable until contract Executed | No new contract                |

No general-purpose update endpoint may bypass this matrix. Explicit transition actions are required.

### 7.3 Evaluation outcome

Evaluation outcome is a controlled enum: `PASS`, `FAIL` or `INCONCLUSIVE`, with an optional explanatory note. A valid evaluation record includes `evaluated_by` and `evaluated_at`. The user performing evaluation must be an authorised Procurement Officer for the owning entity. `INCONCLUSIVE` cannot proceed to Awarded. The optional four-eyes control requires the award actor to differ from `evaluated_by`.

### 7.4 Contract lifecycle and active definition

Contract states are:

- **Draft:** created against an Awarded tender; editable while the tender is Awarded or Contracted.
- **Executed:** contract has been marked executed; commercial fields become immutable.
- **Completed:** contract obligations are recorded as complete; inactive.
- **Terminated:** contract is terminated with a mandatory reason; inactive.

Permitted transitions are Draft → Executed → Completed, Draft → Terminated, and Executed → Terminated. No other transitions are permitted. A contract is **active** when its status is Draft or Executed. The database enforces at most one active contract per tender. A tender may move from Awarded to Contracted when a Draft contract is created; the Draft contract remains editable until it becomes Executed. This resolves the previous contract-locking deadlock.

### 7.5 Award and contract data ownership

The tender is the source of truth for the awarded supplier and award date. The tender does not store a separate award-value field. The contract stores the contract value, reference, start instant and end instant. Supplier and award date are displayed on the contract as read-only values inherited from the linked tender. RULE-001 and RULE-003 use the tender award date; RULE-002 compares contract value to the tender estimate.

## 8. Compliance rules catalogue

Rules are prototype checks, not legal advice, a substitute for procurement review, or determinations of misconduct. Each rule is classified by execution behaviour:

- **Preventive:** blocks an invalid write; does not create a finding for a condition that the application prevents.
- **Detective:** evaluates persisted data and creates or updates findings.
- **Both:** prevents invalid application writes and detects the same condition in deliberately injected or legacy data that bypasses normal validation.

| Rule ID  | Violation condition                                                                                                            | Type       | Severity | Priority                               | Seed/test strategy                                                                                                                                                             |
| -------- | ------------------------------------------------------------------------------------------------------------------------------ | ---------- | -------- | -------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| RULE-001 | Tender award date is before tender closing instant.                                                                            | Detective  | High     | Must                                   | Plant one deliberately invalid tender/award fixture through a test-only fixture path that bypasses workflow validation; expect one finding.                                    |
| RULE-002 | Contract value exceeds tender estimate by more than the configured tolerance; default 0%.                                      | Detective  | Medium   | Must                                   | Plant one contract above estimate; expect one finding.                                                                                                                         |
| RULE-003 | Supplier eligibility is not valid on the tender award date.                                                                    | Detective  | High     | Must                                   | Plant one award whose award date falls outside all valid eligibility periods; expect one finding.                                                                              |
| RULE-004 | Required field(s) for the selected provisional procurement method are absent at publication.                                   | Both       | Medium   | Must                                   | Normal publication is blocked; separately inject one invalid published fixture through test-only bypass; expect one finding.                                                   |
| RULE-005 | Publication-to-closing duration is below the verified minimum for the method.                                                  | Detective  | Medium   | Must, subject to verification fallback | Only seed/test an expected finding if OA-004 verifies and enables the rule; otherwise disabled, unverified and zero findings.                                                  |
| RULE-006 | An award exists without a valid recorded evaluation outcome.                                                                   | Both       | High     | Must                                   | Normal transition is blocked; inject one invalid Awarded fixture through test-only bypass; expect one finding.                                                                 |
| RULE-007 | Same supplier receives more than 3 awards from the same entity within a rolling 12-month period, counted by tender award date. | Detective  | Low      | Should                                 | Plant 4 qualifying contracts for the same supplier/entity, each linked to a tender award date within the rolling 12-month window; expect one finding for the threshold breach. |
| RULE-008 | Contract start instant is before tender award instant, or contract end instant is not after contract start instant.            | Detective  | Medium   | Must                                   | Plant one contract with invalid chronology; expect one finding.                                                                                                                |
| RULE-009 | Tender reference is duplicated within an entity, or more than one active contract exists for a tender.                         | Preventive | Medium   | Must                                   | Database constraints reject duplicate writes; no finding is expected from the normal seed dataset. Constraint-bypass tests verify database integrity separately.               |

### 8.1 Provisional procurement-method catalogue and required fields

The following list is a **prototype method catalogue**, not a legal classification or statement of statutory requirements. Method names and required-field sets must be checked against the Public Procurement and Asset Disposal Act, 2015 and applicable regulations under OA-004 before any legal claim is made. No publication-duration threshold is inferred here.

Common required fields for every method: tender reference, title, procuring entity, method code, procurement description, estimated value in KES, publication instant, closing instant, contact/reference metadata and responsible Procurement Officer.

| Prototype method code | Provisional display name                       | Additional required fields before publication                                                    |
| --------------------- | ---------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| PM-01                 | Open tender                                    | Eligibility/qualification criteria summary; submission instructions; evaluation criteria summary |
| PM-02                 | Restricted tender                              | Restriction/shortlisting rationale; invited supplier list reference; selection criteria summary  |
| PM-03                 | Request for quotations                         | Quotation specification; invited supplier count; quotation submission deadline                   |
| PM-04                 | Request for proposals                          | Terms of reference; technical evaluation criteria; financial evaluation criteria                 |
| PM-05                 | Direct procurement                             | Direct-procurement justification; approval/reference identifier; selected supplier rationale     |
| PM-06                 | Other / special method (prototype placeholder) | Method description; justification; approving reference; manual compliance-review flag            |

The prototype shall not infer that these six entries exhaust or precisely reproduce statutory procurement methods. OA-004 must verify or revise the catalogue. Until then, the interface labels the method catalogue “prototype categories — legal verification pending.” RULE-005 remains disabled unless its legal thresholds are verified.

### 8.2 Finding lifecycle and re-evaluation semantics

A finding is uniquely identified by `(rule_id, affected_record_type, affected_record_id)` for the current rule version. Re-running evaluation updates `last_evaluated_at` and the current condition; it does not create duplicate findings.

Permitted transitions:

| Current state        | Event                             | Next state     | Actor                                                    |
| -------------------- | --------------------------------- | -------------- | -------------------------------------------------------- |
| Open                 | Acknowledge                       | Acknowledged   | Procurement Officer for own entity or Compliance Officer |
| Open                 | Resolve after condition corrected | Resolved       | System evaluation or Compliance Officer                  |
| Acknowledged         | Resolve after condition corrected | Resolved       | System evaluation or Compliance Officer                  |
| Acknowledged         | Explicitly close after review     | Resolved       | Compliance Officer                                       |
| Open or Acknowledged | Mark False Positive with reason   | False Positive | Compliance Officer                                       |
| Resolved             | Condition recurs on re-evaluation | Open           | System                                                   |
| False Positive       | Explicit reopen with reason       | Open           | Compliance Officer                                       |

If an Open or Acknowledged condition is corrected, the finding is retained and moved to Resolved with a system resolution reason. A Resolved finding reopens if the same condition recurs. A False Positive remains suppressed for the same rule version and affected record until a Compliance Officer explicitly reopens it; a new rule version creates a separately traceable evaluation result.

Every finding has a separate comment record (author, body, timestamp) and an immutable status history. Procurement Officers may acknowledge findings for their entity but cannot resolve or mark them False Positive. Auditors are strictly read-only.

### 8.3 Evaluation triggers and atomicity

Evaluation occurs on:

- tender publication, closing, evaluation and award transitions;
- contract creation, status transition and relevant contract-field changes;
- supplier eligibility-period creation or supersession, for every affected award/contract;
- rule enablement, version change or parameter change;
- an authorised full re-evaluation command.

A synchronous evaluation triggered by a business write and its finding changes commit in the same database transaction. If evaluation or required audit persistence fails, the business write rolls back. If a durable background task is used for a fan-out operation, its outbox/task record is committed with the initiating change, is idempotent, exposes a pending state and can be retried without losing the evaluation request.

### 8.4 RULE-005 verification fallback

RULE-005 must not use guessed or unverified legal thresholds. If OA-004 has not verified primary sources and parameters before Slice 4 on Day 5, the rule ships disabled and labelled **Unverified — disabled**. It produces no findings, is excluded from expected seed findings, and may only be enabled after source references and parameters are recorded.

### 8.5 Seed manifest

The authoritative seed manifest shall be committed as `database/seed/seed-manifest.json` and shall include each planted case, rule ID, fixture identifier, injection method, expected outcome, expected finding count, and rationale. Expected counts for the baseline fixture set:

| Rule     | Seed case                                                |         Expected findings | Notes                                                     |
| -------- | -------------------------------------------------------- | ------------------------: | --------------------------------------------------------- |
| RULE-001 | Invalid award date before closing                        |                         1 | Test-only invalid fixture                                 |
| RULE-002 | Contract value above estimate at 0% tolerance            |                         1 | Detective                                                 |
| RULE-003 | Award outside supplier eligibility period                |                         1 | Historical eligibility fixture                            |
| RULE-004 | Published tender missing method-required field           |                         1 | Test-only invalid fixture; ordinary publish is blocked    |
| RULE-005 | Below verified threshold                                 | 0 unless verified/enabled | If disabled, explicitly marked not applicable             |
| RULE-006 | Award without evaluation outcome                         |                         1 | Test-only invalid fixture; ordinary transition is blocked |
| RULE-007 | Four awards to same supplier by same entity in 12 months |   1 if Should implemented | Heuristic only                                            |
| RULE-008 | Invalid contract date chronology                         |                         1 | Detective                                                 |
| RULE-009 | Duplicate tender reference / second active contract      |                         0 | Preventive-only; rejection tests, not findings            |

The seed manifest shall distinguish normal demonstration data from deliberately invalid fixtures. Invalid fixtures must be isolated in a clearly labelled test/demo dataset and must not be silently loaded into the ordinary clean seed dataset if they violate database constraints.

## 9. Role-based access control matrix

Principles:

- Deny by default.
- Enforce object-level checks and entity scoping on every query and API endpoint.
- Administrators manage accounts and entities but cannot read procurement data.
- Administrators cannot change their own role.
- Auditors are strictly read-only for application data and audit records; they may change only their own password and permitted own profile fields.
- Every role change is audited.

| Resource and action                                     | Administrator                                             | Procurement Officer | Compliance Officer | Auditor |
| ------------------------------------------------------- | --------------------------------------------------------- | ------------------- | ------------------ | ------- |
| User accounts: manage                                   | Allow                                                     | Deny                | Deny               | Deny    |
| Own password and profile: change                        | Allow                                                     | Allow               | Allow              | Allow   |
| Procuring entities: manage                              | Allow                                                     | Deny                | Deny               | Deny    |
| Procuring entities: view                                | Allow                                                     | Own                 | All                | All     |
| Supplier master: create and edit identity fields        | Deny                                                      | Allow               | Deny               | Deny    |
| Supplier eligibility history: create/supersede          | Deny                                                      | Deny                | Allow              | Deny    |
| Suppliers: view                                         | Deny                                                      | Allow               | Allow              | Allow   |
| Tenders: create and edit Draft                          | Deny                                                      | Own entity          | Deny               | Deny    |
| Tenders: advance state or cancel                        | Deny                                                      | Own entity          | Deny               | Deny    |
| Tenders: view                                           | Deny                                                      | Own entity          | All                | All     |
| Contracts: create and edit before lock                  | Deny                                                      | Own entity          | Deny               | Deny    |
| Contracts: view                                         | Deny                                                      | Own entity          | All                | All     |
| Soft delete Draft tenders                               | Deny                                                      | Own entity          | Deny               | Deny    |
| Deactivate supplier / procuring entity                  | Allow for entity; supplier eligibility restrictions apply | Deny                | Deny               | Deny    |
| Findings: view                                          | Deny                                                      | Own entity          | All                | All     |
| Findings: acknowledge and comment                       | Deny                                                      | Own entity          | All                | Deny    |
| Findings: resolve or mark False Positive                | Deny                                                      | Deny                | All                | Deny    |
| Rule parameters: edit through restricted API/admin path | Deny                                                      | Deny                | Allow              | Deny    |
| Rule-parameter management UI                            | Deny                                                      | Deny                | Should             | Deny    |
| Dashboard and CSV export                                | Deny                                                      | Own entity          | All                | All     |
| Audit log: view                                         | Allow                                                     | Deny                | Deny               | Allow   |
| Audit log: modify or delete                             | Deny                                                      | Deny                | Deny               | Deny    |

**Interpretation:** “Own” means records associated with the Procurement Officer's assigned procuring entity. “All” means all entities. The shared supplier register is visible to the roles indicated. Procurement Officers may create supplier master records and edit non-eligibility identity fields; only Compliance Officers may create or supersede eligibility periods. Auditors cannot create, edit, delete, acknowledge, comment on, resolve or otherwise change procurement or compliance records.

### 9.1 Rule-parameter access split

Changing parameters through a restricted API or administrative path is Must (FR-051). A dedicated rule-parameter management UI is Should (FR-079). Deferring the UI does not remove the Must authorisation, validation, versioning or audit requirements.

### 9.2 API contract

- Collection endpoints use deterministic pagination: default page size 25, maximum 100; response includes `count`, `next`, `previous` and `results`.
- Ordering is allow-listed and includes a unique tie-breaker to prevent unstable page boundaries.
- Errors use `application/problem+json` as specified by NFR-031.
- Rate-limited requests return 429 and `Retry-After` where known.
- Client payloads cannot set server-controlled fields such as role, entity ownership, audit fields, finding state or lifecycle state except through explicit transition actions.
- A resource outside a caller's entity scope returns 404, not 403.
- Unsafe session-authenticated API requests require CSRF tokens. The Newman collection (Should) must obtain a session cookie and CSRF token before unsafe requests.
- The same-origin API does not enable permissive CORS.

## 10. Data requirements

### 10.1 Core entities

| Entity                      | Minimum information                                                                                                        |
| --------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| User                        | Name, username, fictitious email, password hash, role, active status and entity assignment where applicable                |
| Procuring Entity            | Unique identifier, name, code and active status                                                                            |
| Supplier                    | Global unique identifier, name and registration identity fields; no procuring-entity ownership key                         |
| Supplier Eligibility Period | Supplier, eligibility status, valid-from, valid-to, source/reference, recorded_by and recorded_at                          |
| Tender                      | Entity, reference, title, procurement method, estimate, publication date, closing date, state and evaluation outcome       |
| Contract                    | Tender, supplier, contract reference, value, award date, start date and end date                                           |
| Compliance Rule             | Rule ID, description, enabled status, severity and versioned parameters                                                    |
| Finding                     | Rule ID/version, affected record, parameters used, severity, state, first/last evaluation timestamps                       |
| Finding Comment             | Finding, author, comment body and timestamp                                                                                |
| Finding Status History      | Finding, previous/new state, actor (human or system), reason and timestamp                                                 |
| Tender Evaluation           | Tender, outcome enum, note, evaluated_by and evaluated_at                                                                  |
| Contract                    | Tender, contract reference, value, start/end instants and contract state                                                   |
| Audit Event                 | Sequence number, actor (human or system), action, target, canonical payload, UTC timestamp, previous hash and current hash |

Supplier master records are shared reference data and are not entity-owned. Eligibility periods are historical records controlled by Compliance Officers. Procuring-entity scoping applies to tenders, contracts and entity-specific operational records, not to the shared supplier master.

The detailed relational design, constraints and data-flow diagrams belong in DEL-003.

### 10.2 Data quality and integrity

- Tender reference shall be unique within a procuring entity.
- Each tender shall belong to one procuring entity.
- Each contract shall reference a valid tender; supplier and award date are inherited from the tender and are read-only on the contract.
- Supplier eligibility history shall retain prior periods; validity periods for one supplier shall not overlap.
- Evaluation outcome shall include the actor and timestamp.
- Finding comments and status changes shall be stored as separate history records.
- No more than one active contract shall exist for a tender.
- Monetary values shall use decimal storage and explicit currency.
- Date validation shall enforce the chronology requirements.
- Procurement records shall not be hard-deleted.
- Soft deletion shall be limited to Draft tenders and retain actor, time and reason; the tender reference remains reserved.
- Suppliers and procuring entities are deactivated, not deleted.
- Contracts are never deleted; a Draft contract may be Terminated with a reason.
- Seed records shall be fictional and reproducible.

### 10.3 Seed dataset

The fixed-seed dataset shall target 5 procuring entities, 60 suppliers, 250 tenders and 150 contracts. It shall include at least two Procurement Officer accounts per entity and deliberately planted violations for applicable rules. The dataset shall not contain real personal or procurement data.

## 11. User interface requirements

The MVP shall use Django templates, HTMX, Tailwind CSS and self-hosted Chart.js in a same-origin architecture.

- Navigation and page access shall reflect the authenticated user's role.
- Forms shall display clear field-level validation messages.
- Destructive or state-changing actions shall provide an appropriate confirmation or clear feedback.
- Lists shall provide readable status, key identifiers and relevant actions.
- Dashboard figures shall be traceable to the underlying fictional records.
- The interface shall be responsive for desktop and tablet-width viewports.
- Keyboard focus shall be visible and controls shall be usable without a mouse.
- Animations shall be restrained, shall respect reduced-motion preferences and shall not convey information by motion alone.
- Visual styling, including any glass-like surfaces or transitions, shall not reduce contrast, readability or performance.

Detailed visual tokens and component specifications may be maintained as a UI design standard without changing the approved application architecture.

## 12. Requirements traceability and automated coverage

The detailed requirement-level traceability register is `docs/requirements-traceability.csv`, generated from the requirement catalogue and acceptance criteria in this document. It shall contain one row per requirement ID, its priority, story link, design reference, implementation area, test marker and verification status. Group-only traceability is not sufficient.

### 12.1 Test marker convention

Every automated test that verifies a requirement shall carry a pytest marker, for example:

```python
@pytest.mark.req("FR-005")
def test_lockout_after_fifth_failed_attempt(...):
    ...
```

A test may carry multiple `req` markers if it genuinely verifies each linked requirement. Tests shall not be tagged merely to satisfy coverage; the assertion must exercise the stated acceptance criterion.

### 12.2 CI coverage gate

- The canonical Must-requirement list shall be stored in `docs/requirements-matrix.json`.
- A pytest collection plugin shall collect all `req` markers during `pytest --collect-only`.
- A CI script shall compare collected markers with the canonical Must IDs.
- CI shall fail if any Must FR has no collected test, if a marker refers to an unknown ID, or if a Must requirement has no acceptance criterion.
- Manual-only verification for a Must requirement requires an explicit reason and Project Owner approval; it does not silently satisfy the automated coverage gate.
- The CSV traceability report shall be generated from the canonical requirement matrix and collected test markers, not hand-maintained as an independent source of truth.

The matrix is generated evidence. DEL-002 remains the human-readable requirements source; the machine-readable matrix is its controlled companion.

## 13. Acceptance and verification approach

Requirements will be verified through a combination of:

- Automated unit tests.
- Integration tests across application services and database operations.
- API tests, including negative authorisation and validation cases.
- Browser-based functional tests and manual workflow checks.
- Security checks and scanning defined in DEL-001 and DEL-004.
- Performance measurements using the agreed seed dataset.
- Backup and restore rehearsal.
- Clean-clone deployment rehearsal.

### 13.1 Minimum acceptance expectations

- Every Must FR has a Given/When/Then acceptance criterion in Annex A and a collected pytest `req` marker.
- Every RBAC allow and deny cell is covered by an automated test.
- Cross-entity access attempts are denied.
- Every Must compliance rule has pass, fail and boundary tests; preventive-only rules use rejected-write/constraint tests instead of expecting findings.
- Seeded finding counts match the committed seed manifest.
- Compliance module coverage is at least 80%.
- Disabled or unverified RULE-005 produces no active findings.
- The audit verification command detects deliberate tampering.
- Seeded dashboard counts reconcile to the stored records.
- The backup restore drill records elapsed time against the 15-minute target, with table row counts compared; exceeding the target is reported honestly rather than hidden.
- The Project Owner performs the clean-clone deployment, records elapsed time and documents prerequisites and image-download time separately.
- HTTPS redirect, Nginx rate limits, backup location/checksum and demo-account end-of-project handling are verified.
- All Must requirements are either verified or explicitly reported as unmet.

## 14. Assumptions, dependencies and open verification items

| ID      | Item                           | Required action / dependency                                                                                               | Deadline or owner                          |
| ------- | ------------------------------ | -------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------ |
| DEP-001 | Development environment        | Verify WSL2, Docker and Docker Compose can run the intended stack.                                                         | Before Day 1; Project Owner                |
| DEP-002 | PPRA role alignment            | Verify the actual ICT Officer II job description and update the competency mapping if needed.                              | G1; Project Owner                          |
| DEP-003 | Security verification baseline | Select and record the OWASP ASVS edition in DEL-004.                                                                       | G1; Project Owner                          |
| DEP-004 | RULE-005 legal thresholds      | Verify primary legal sources and procurement-method periods; otherwise disable and label Unverified.                       | Before Slice 4, Day 5; Project Owner       |
| DEP-005 | External review                | Seek one or two genuine reviewers and record comments with source and date.                                                | By Day 8; Project Owner                    |
| DEP-006 | Four-eyes workflow             | If implemented, ensure at least two Procurement Officer accounts per entity in seed data.                                  | Before workflow test; Project Owner        |
| DEP-007 | Procurement method catalogue   | Verify prototype categories and required fields against primary legal sources; do not label them statutory until verified. | OA-004 / before Slice 4; Project Owner     |
| DEP-008 | Parent baseline                | DEL-001 Version 1.0 approved; maintain traceability to that baseline and raise coordinated change requests for any parent amendments. | Completed 24 November 2026; Project Owner |
| DEP-009 | Audit-chain implementation     | Use a locked chain-head row or equivalent serialisation mechanism across Gunicorn workers.                                 | Before audit implementation; Project Owner |

The project is an individual portfolio exercise. Stakeholder roles are simulated. Any external review must be genuine and separately identified.

## 15. Approval

This document is approved as the requirements baseline subordinate to DEL-001 Version 1.0. It has been self-reviewed by the Project Owner and has not been independently reviewed. Annex E records parent-charter alignment items for controlled future change; approval does not silently revise DEL-001. The Project Owner separately approved this document on 24 November 2026.

| Approval role | Name                     | Decision                    | Date           |
| ------------- | ------------------------ | --------------------------- | -------------- |
| Project Owner | Eugen Walubengo Nyongesa | Approved — Version 1.0 established as the requirements baseline | 24 November 2026 |

**Approval record:** The Project Owner explicitly approved this document on 24 November 2026. Version 1.0 is the approved requirements baseline subordinate to DEL-001 Version 1.0. Annex E remains a controlled alignment/change backlog, not an amendment to the parent charter. Approval does not imply independent human review, legal verification of provisional procurement rules, or implementation/testing of specified requirements.

---

**END OF BASE DOCUMENT — PPMCMS-DEL-002 — VERSION 1.0 APPROVED**

# Annex A. Requirement-level Given/When/Then acceptance criteria

This annex is the acceptance-criteria baseline for all Must functional requirements in DEL-002 v1.0. Each row is an independently traceable acceptance criterion. Complex requirements require additional tests for every branch, negative case and boundary; a single happy-path test is not sufficient.

| Requirement | Given                                                                                                                    | When                                                                  | Then                                                                                                                                                                                                                                                                                                                                                                                    | Required test marker           |
| ----------- | ------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------ |
| FR-001      | Given a controlled test fixture is prepared for FR-001, including the role and data conditions stated in the requirement | When the operation or event specified by FR-001 is invoked            | Then the application satisfies this requirement: require an authenticated user to access protected application pages and API resources.                                                                                                                                                                                                                                                 | `@pytest.mark.req("FR-001")` |
| FR-002      | Given a controlled test fixture is prepared for FR-002, including the role and data conditions stated in the requirement | When the operation or event specified by FR-002 is invoked            | Then the application satisfies this requirement: allow a user to log in and log out using Django session authentication.                                                                                                                                                                                                                                                                | `@pytest.mark.req("FR-002")` |
| FR-003      | Given a controlled test fixture is prepared for FR-003, including the role and data conditions stated in the requirement | When the operation or event specified by FR-003 is invoked            | Then the application satisfies this requirement: store passwords using Argon2 password hashing and shall not store plaintext passwords.                                                                                                                                                                                                                                                 | `@pytest.mark.req("FR-003")` |
| FR-004      | Given a controlled test fixture is prepared for FR-004, including the role and data conditions stated in the requirement | When the operation or event specified by FR-004 is invoked            | Then the application satisfies this requirement: enforce a minimum password length of 12 characters and a common-password check.                                                                                                                                                                                                                                                        | `@pytest.mark.req("FR-004")` |
| FR-005      | Given an account has four qualifying failed attempts in the rolling 15-minute window                                     | When a fifth qualifying login attempt fails                           | Then the account is locked for 15 minutes and the response does not disclose account existence.                                                                                                                                                                                                                                                                                         | `@pytest.mark.req("FR-005")` |
| FR-006      | Given the test has an unknown account, a wrong password and a locked account                                             | When a login is attempted for each case                               | Then the externally observable failure message and response shape are equivalent.                                                                                                                                                                                                                                                                                                       | `@pytest.mark.req("FR-006")` |
| FR-007      | Given a controlled test fixture is prepared for FR-007, including the role and data conditions stated in the requirement | When the operation or event specified by FR-007 is invoked            | Then the application satisfies this requirement: enforce a 30-minute idle session timeout and an 8-hour absolute session lifetime.                                                                                                                                                                                                                                                      | `@pytest.mark.req("FR-007")` |
| FR-008      | Given a controlled test fixture is prepared for FR-008, including the role and data conditions stated in the requirement | When the operation or event specified by FR-008 is invoked            | Then the application satisfies this requirement: invalidate the active session on logout and password change.                                                                                                                                                                                                                                                                           | `@pytest.mark.req("FR-008")` |
| FR-009      | Given a controlled test fixture is prepared for FR-009, including the role and data conditions stated in the requirement | When the operation or event specified by FR-009 is invoked            | Then the application satisfies this requirement: allow an authenticated user to change their own password after verifying the current password.                                                                                                                                                                                                                                         | `@pytest.mark.req("FR-009")` |
| FR-010      | Given a controlled test fixture is prepared for FR-010, including the role and data conditions stated in the requirement | When the operation or event specified by FR-010 is invoked            | Then the application satisfies this requirement: allow only Administrators to create user accounts and assign an initial application role at creation.                                                                                                                                                                                                                                  | `@pytest.mark.req("FR-010")` |
| FR-011      | Given a controlled test fixture is prepared for FR-011, including the role and data conditions stated in the requirement | When the operation or event specified by FR-011 is invoked            | Then the application satisfies this requirement: allow only Administrators to edit another user's account details, deactivate another user, unlock an account and initiate an administrative password reset.                                                                                                                                                                            | `@pytest.mark.req("FR-011")` |
| FR-012      | Given a controlled test fixture is prepared for FR-012, including the role and data conditions stated in the requirement | When the operation or event specified by FR-012 is invoked            | Then the application satisfies this requirement: audit successful login, failed login, lockout, logout, password change, administrative unlock, administrative password reset and every user-role change.                                                                                                                                                                               | `@pytest.mark.req("FR-012")` |
| FR-013      | Given a controlled test fixture is prepared for FR-013, including the role and data conditions stated in the requirement | When the operation or event specified by FR-013 is invoked            | Then the application satisfies this requirement: allow Administrators to create, edit and manage procuring-entity records.                                                                                                                                                                                                                                                              | `@pytest.mark.req("FR-013")` |
| FR-014      | Given a controlled test fixture is prepared for FR-014, including the role and data conditions stated in the requirement | When the operation or event specified by FR-014 is invoked            | Then the application satisfies this requirement: allow Compliance Officers and Auditors to view all procuring entities.                                                                                                                                                                                                                                                                 | `@pytest.mark.req("FR-014")` |
| FR-015      | Given a controlled test fixture is prepared for FR-015, including the role and data conditions stated in the requirement | When the operation or event specified by FR-015 is invoked            | Then the application satisfies this requirement: allow Procurement Officers to view only their assigned procuring entity.                                                                                                                                                                                                                                                               | `@pytest.mark.req("FR-015")` |
| FR-016      | Given a controlled test fixture is prepared for FR-016, including the role and data conditions stated in the requirement | When the operation or event specified by FR-016 is invoked            | Then the application satisfies this requirement: associate each Procurement Officer account with an authorised procuring entity.                                                                                                                                                                                                                                                        | `@pytest.mark.req("FR-016")` |
| FR-017      | Given a controlled test fixture is prepared for FR-017, including the role and data conditions stated in the requirement | When the operation or event specified by FR-017 is invoked            | Then the application satisfies this requirement: prevent a user from accessing records belonging to an unauthorised entity by changing an identifier or request parameter.                                                                                                                                                                                                              | `@pytest.mark.req("FR-017")` |
| FR-018      | Given a controlled test fixture is prepared for FR-018, including the role and data conditions stated in the requirement | When the operation or event specified by FR-018 is invoked            | Then the application satisfies this requirement: allow a Procurement Officer to create a shared supplier master record and edit only its non-eligibility identity fields.                                                                                                                                                                                                               | `@pytest.mark.req("FR-018")` |
| FR-019      | Given a controlled test fixture is prepared for FR-019, including the role and data conditions stated in the requirement | When the operation or event specified by FR-019 is invoked            | Then the application satisfies this requirement: make shared supplier master records visible to Procurement Officers, Compliance Officers and Auditors; supplier records shall not carry a procuring-entity ownership key.                                                                                                                                                              | `@pytest.mark.req("FR-019")` |
| FR-020      | Given a controlled test fixture is prepared for FR-020, including the role and data conditions stated in the requirement | When the operation or event specified by FR-020 is invoked            | Then the application satisfies this requirement: capture supplier identity separately from a dated supplier-eligibility history containing status, valid-from date, valid-to date and recorded source/reference.                                                                                                                                                                        | `@pytest.mark.req("FR-020")` |
| FR-021      | Given a controlled test fixture is prepared for FR-021, including the role and data conditions stated in the requirement | When the operation or event specified by FR-021 is invoked            | Then the application satisfies this requirement: allow only Compliance Officers to create a new supplier-eligibility period or supersede an existing period; prior periods shall remain historically inspectable and changes shall be audited.                                                                                                                                          | `@pytest.mark.req("FR-021")` |
| FR-022      | Given a controlled test fixture is prepared for FR-022, including the role and data conditions stated in the requirement | When the operation or event specified by FR-022 is invoked            | Then the application satisfies this requirement: prevent Administrators from viewing supplier, tender, contract and procurement-finding data through application interfaces or API endpoints.                                                                                                                                                                                           | `@pytest.mark.req("FR-022")` |
| FR-023      | Given a controlled test fixture is prepared for FR-023, including the role and data conditions stated in the requirement | When the operation or event specified by FR-023 is invoked            | Then the application satisfies this requirement: allow a Procurement Officer to create and edit Draft tenders belonging to their own entity.                                                                                                                                                                                                                                            | `@pytest.mark.req("FR-023")` |
| FR-024      | Given a controlled test fixture is prepared for FR-024, including the role and data conditions stated in the requirement | When the operation or event specified by FR-024 is invoked            | Then the application satisfies this requirement: capture a tender reference, title, procurement method, estimated value, publication instant, closing instant and lifecycle state.                                                                                                                                                                                                      | `@pytest.mark.req("FR-024")` |
| FR-025      | Given a tender reference is already used by a tender in the same entity, including a soft-deleted tender                 | When a user attempts to create or amend a tender using that reference | Then the write is rejected by the uniqueness invariant.                                                                                                                                                                                                                                                                                                                                 | `@pytest.mark.req("FR-025")` |
| FR-026      | Given a controlled test fixture is prepared for FR-026, including the role and data conditions stated in the requirement | When the operation or event specified by FR-026 is invoked            | Then the application satisfies this requirement: enforce only the valid tender state transitions defined in Section 7.                                                                                                                                                                                                                                                                  | `@pytest.mark.req("FR-026")` |
| FR-027      | Given a controlled test fixture is prepared for FR-027, including the role and data conditions stated in the requirement | When the operation or event specified by FR-027 is invoked            | Then the application satisfies this requirement: reject invalid state transitions and return a clear validation response.                                                                                                                                                                                                                                                               | `@pytest.mark.req("FR-027")` |
| FR-028      | Given a controlled test fixture is prepared for FR-028, including the role and data conditions stated in the requirement | When the operation or event specified by FR-028 is invoked            | Then the application satisfies this requirement: validate the provisional method-specific required-field matrix in Section 8.1 before a tender can be Published. The matrix is prototype logic pending OA-004 legal verification and is not a statement of statutory requirements.                                                                                                      | `@pytest.mark.req("FR-028")` |
| FR-029      | Given a controlled test fixture is prepared for FR-029, including the role and data conditions stated in the requirement | When the operation or event specified by FR-029 is invoked            | Then the application satisfies this requirement: require an evaluation outcome record, evaluator identity and evaluation timestamp before a tender can be advanced to Awarded.                                                                                                                                                                                                          | `@pytest.mark.req("FR-029")` |
| FR-030      | Given a controlled test fixture is prepared for FR-030, including the role and data conditions stated in the requirement | When the operation or event specified by FR-030 is invoked            | Then the application satisfies this requirement: require the selected supplier and award date when a tender is advanced to Awarded. The tender is the source of truth for these award facts; no separate award-value field shall be stored on the tender.                                                                                                                               | `@pytest.mark.req("FR-030")` |
| FR-031      | Given a controlled test fixture is prepared for FR-031, including the role and data conditions stated in the requirement | When the operation or event specified by FR-031 is invoked            | Then the application satisfies this requirement: require a cancellation reason when a tender is cancelled.                                                                                                                                                                                                                                                                              | `@pytest.mark.req("FR-031")` |
| FR-032      | Given a controlled test fixture is prepared for FR-032, including the role and data conditions stated in the requirement | When the operation or event specified by FR-032 is invoked            | Then the application satisfies this requirement: prohibit edits to tender identity, method, estimate and publication fields after publication, while allowing only the state-specific evaluation and award fields in Section 8.2 to be completed by the authorised Procurement Officer. Awarded, Contracted and Cancelled tenders shall be locked.                                      | `@pytest.mark.req("FR-032")` |
| FR-033      | Given a controlled test fixture is prepared for FR-033, including the role and data conditions stated in the requirement | When the operation or event specified by FR-033 is invoked            | Then the application satisfies this requirement: permit soft deletion only for Draft tenders and shall record actor, timestamp and reason; the deleted tender reference shall remain reserved and shall not be reused. Suppliers and procuring entities shall be deactivated, not deleted. Contracts shall not be deleted; Draft contracts may be cancelled with a reason.              | `@pytest.mark.req("FR-033")` |
| FR-034      | Given a controlled test fixture is prepared for FR-034, including the role and data conditions stated in the requirement | When the operation or event specified by FR-034 is invoked            | Then the application satisfies this requirement: not hard-delete users, procuring entities, suppliers, tenders, contracts, findings, eligibility history or audit events through application functionality.                                                                                                                                                                             | `@pytest.mark.req("FR-034")` |
| FR-036      | Given a controlled test fixture is prepared for FR-036, including the role and data conditions stated in the requirement | When the operation or event specified by FR-036 is invoked            | Then the application satisfies this requirement: allow authorised Procurement Officers to create and edit contracts for their own entity before the record is locked.                                                                                                                                                                                                                   | `@pytest.mark.req("FR-036")` |
| FR-037      | Given a controlled test fixture is prepared for FR-037, including the role and data conditions stated in the requirement | When the operation or event specified by FR-037 is invoked            | Then the application satisfies this requirement: associate each contract with a tender and supplier.                                                                                                                                                                                                                                                                                    | `@pytest.mark.req("FR-037")` |
| FR-038      | Given a controlled test fixture is prepared for FR-038, including the role and data conditions stated in the requirement | When the operation or event specified by FR-038 is invoked            | Then the application satisfies this requirement: capture a unique contract reference, contract value, start date and end date. Supplier and award date shall be inherited read-only from the associated tender.                                                                                                                                                                         | `@pytest.mark.req("FR-038")` |
| FR-039      | Given a tender already has a Draft or Executed contract                                                                  | When a second contract is created or activated for the same tender    | Then the database rejects the second active contract.                                                                                                                                                                                                                                                                                                                                   | `@pytest.mark.req("FR-039")` |
| FR-040      | Given a controlled test fixture is prepared for FR-040, including the role and data conditions stated in the requirement | When the operation or event specified by FR-040 is invoked            | Then the application satisfies this requirement: permit a Draft contract to be edited while its associated tender is Awarded or Contracted. The contract shall become immutable when its own status changes from Draft to Executed; subsequent status transitions shall not permit changes to its commercial fields.                                                                    | `@pytest.mark.req("FR-040")` |
| FR-041      | Given a controlled test fixture is prepared for FR-041, including the role and data conditions stated in the requirement | When the operation or event specified by FR-041 is invoked            | Then the application satisfies this requirement: store monetary values as decimal amounts with an explicit currency; the MVP currency is KES.                                                                                                                                                                                                                                           | `@pytest.mark.req("FR-041")` |
| FR-042      | Given a business write changes data relevant to an enabled detective rule                                                | When the service saves the business change and evaluates the rule     | Then the business write and finding changes commit together or all roll back if evaluation/persistence fails.                                                                                                                                                                                                                                                                           | `@pytest.mark.req("FR-042")` |
| FR-043      | Given a controlled test fixture is prepared for FR-043, including the role and data conditions stated in the requirement | When the operation or event specified by FR-043 is invoked            | Then the application satisfies this requirement: provide an on-demand re-evaluation command and shall record its start, completion, scope and result.                                                                                                                                                                                                                                   | `@pytest.mark.req("FR-043")` |
| FR-044      | Given a finding already exists for the same rule version and affected record                                             | When the same condition is evaluated again                            | Then the existing finding is updated and no duplicate finding is created.                                                                                                                                                                                                                                                                                                               | `@pytest.mark.req("FR-044")` |
| FR-045      | Given a controlled test fixture is prepared for FR-045, including the role and data conditions stated in the requirement | When the operation or event specified by FR-045 is invoked            | Then the application satisfies this requirement: Each finding shall store the rule ID and version, parameters snapshot, affected record, severity, first-detected timestamp, last-evaluated timestamp, status and timestamps.                                                                                                                                                           | `@pytest.mark.req("FR-045")` |
| FR-046      | Given a controlled test fixture is prepared for FR-046, including the role and data conditions stated in the requirement | When the operation or event specified by FR-046 is invoked            | Then the application satisfies this requirement: support finding transitions Open → Acknowledged → Resolved, Open → Resolved, and Open/Acknowledged → False Positive. Resolved findings may reopen to Open if the condition recurs; False Positive findings remain suppressed until the underlying record or rule version changes and a Compliance Officer explicitly reopens them. | `@pytest.mark.req("FR-046")` |
| FR-047      | Given a controlled test fixture is prepared for FR-047, including the role and data conditions stated in the requirement | When the operation or event specified by FR-047 is invoked            | Then the application satisfies this requirement: store finding comments as separate records with author and timestamp, and audit every comment and status transition.                                                                                                                                                                                                                   | `@pytest.mark.req("FR-047")` |
| FR-048      | Given a controlled test fixture is prepared for FR-048, including the role and data conditions stated in the requirement | When the operation or event specified by FR-048 is invoked            | Then the application satisfies this requirement: allow Procurement Officers to view and acknowledge findings for their own entity; acknowledgement shall not resolve or suppress the finding.                                                                                                                                                                                           | `@pytest.mark.req("FR-048")` |
| FR-049      | Given a controlled test fixture is prepared for FR-049, including the role and data conditions stated in the requirement | When the operation or event specified by FR-049 is invoked            | Then the application satisfies this requirement: allow Compliance Officers to view findings across entities, add comments, resolve findings, mark findings as False Positive and explicitly reopen a False Positive finding.                                                                                                                                                            | `@pytest.mark.req("FR-049")` |
| FR-050      | Given a controlled test fixture is prepared for FR-050, including the role and data conditions stated in the requirement | When the operation or event specified by FR-050 is invoked            | Then the application satisfies this requirement: prevent Auditors from creating comments or changing finding status.                                                                                                                                                                                                                                                                    | `@pytest.mark.req("FR-050")` |
| FR-051      | Given a controlled test fixture is prepared for FR-051, including the role and data conditions stated in the requirement | When the operation or event specified by FR-051 is invoked            | Then the application satisfies this requirement: expose a restricted API or administrative path through which only Compliance Officers can change rule parameters; every change shall be audited and versioned.                                                                                                                                                                         | `@pytest.mark.req("FR-051")` |
| FR-052      | Given a controlled test fixture is prepared for FR-052, including the role and data conditions stated in the requirement | When the operation or event specified by FR-052 is invoked            | Then the application satisfies this requirement: not generate active findings from disabled or unverified rules.                                                                                                                                                                                                                                                                        | `@pytest.mark.req("FR-052")` |
| FR-054      | Given the database contains the controlled seed dataset                                                                  | When an authorised user opens the dashboard                           | Then all specified metrics are displayed and match independently calculated dataset totals.                                                                                                                                                                                                                                                                                             | `@pytest.mark.req("FR-054")` |
| FR-055      | Given a controlled test fixture is prepared for FR-055, including the role and data conditions stated in the requirement | When the operation or event specified by FR-055 is invoked            | Then the application satisfies this requirement: Dashboard data shown to Procurement Officers shall be restricted to their own entity.                                                                                                                                                                                                                                                  | `@pytest.mark.req("FR-055")` |
| FR-056      | Given a controlled test fixture is prepared for FR-056, including the role and data conditions stated in the requirement | When the operation or event specified by FR-056 is invoked            | Then the application satisfies this requirement: Dashboard data shown to Compliance Officers and Auditors may cover all entities.                                                                                                                                                                                                                                                       | `@pytest.mark.req("FR-056")` |
| FR-058      | Given a controlled test fixture is prepared for FR-058, including the role and data conditions stated in the requirement | When the operation or event specified by FR-058 is invoked            | Then the application satisfies this requirement: Dashboard totals shall be reconcilable to the underlying seed records.                                                                                                                                                                                                                                                                 | `@pytest.mark.req("FR-058")` |
| FR-059      | Given a business write or listed authentication/security event occurs                                                    | When the event is committed                                           | Then the corresponding audit event is committed in the same transaction or the initiating write rolls back.                                                                                                                                                                                                                                                                             | `@pytest.mark.req("FR-059")` |
| FR-060      | Given an audit event is ready to be appended                                                                             | When the audit service commits the event                              | Then sequence, actor, canonical payload, UTC timestamp and valid previous/current hashes are stored.                                                                                                                                                                                                                                                                                    | `@pytest.mark.req("FR-060")` |
| FR-061      | Given two application workers attempt to append audit events concurrently                                                | When both transactions acquire the audit chain lock and commit        | Then the resulting sequence is unique, ordered and verifies as one chain.                                                                                                                                                                                                                                                                                                               | `@pytest.mark.req("FR-061")` |
| FR-062      | Given a controlled test fixture is prepared for FR-062, including the role and data conditions stated in the requirement | When the operation or event specified by FR-062 is invoked            | Then the application satisfies this requirement: provide a verification command that recomputes the audit hash chain and reports integrity failures.                                                                                                                                                                                                                                    | `@pytest.mark.req("FR-062")` |
| FR-063      | Given a controlled test fixture is prepared for FR-063, including the role and data conditions stated in the requirement | When the operation or event specified by FR-063 is invoked            | Then the application satisfies this requirement: Only Administrators and Auditors may view the audit log; Auditor access shall be read-only.                                                                                                                                                                                                                                            | `@pytest.mark.req("FR-063")` |
| FR-064      | Given a controlled test fixture is prepared for FR-064, including the role and data conditions stated in the requirement | When the operation or event specified by FR-064 is invoked            | Then the application satisfies this requirement: No application role shall be able to modify or delete existing audit entries through application functionality.                                                                                                                                                                                                                        | `@pytest.mark.req("FR-064")` |
| FR-065      | Given a controlled test fixture is prepared for FR-065, including the role and data conditions stated in the requirement | When the operation or event specified by FR-065 is invoked            | Then the application satisfies this requirement: expose REST API endpoints for the core resources used by the application.                                                                                                                                                                                                                                                              | `@pytest.mark.req("FR-065")` |
| FR-066      | Given a controlled test fixture is prepared for FR-066, including the role and data conditions stated in the requirement | When the operation or event specified by FR-066 is invoked            | Then the application satisfies this requirement: The web interface and REST API shall share the same application service layer and business rules.                                                                                                                                                                                                                                      | `@pytest.mark.req("FR-066")` |
| FR-067      | Given a controlled test fixture is prepared for FR-067, including the role and data conditions stated in the requirement | When the operation or event specified by FR-067 is invoked            | Then the application satisfies this requirement: enforce authentication, RBAC and procuring-entity scoping equivalent to the web interface.                                                                                                                                                                                                                                             | `@pytest.mark.req("FR-067")` |
| FR-068      | Given a controlled test fixture is prepared for FR-068, including the role and data conditions stated in the requirement | When the operation or event specified by FR-068 is invoked            | Then the application satisfies this requirement: validate input and return the standard error representation defined in Section 9.2 and Annex D without exposing stack traces or secrets.                                                                                                                                                                                               | `@pytest.mark.req("FR-068")` |
| FR-071      | Given a controlled test fixture is prepared for FR-071, including the role and data conditions stated in the requirement | When the operation or event specified by FR-071 is invoked            | Then the application satisfies this requirement: allow an Administrator to unlock a locked account and initiate a password reset that forces a password change at the next successful login.                                                                                                                                                                                            | `@pytest.mark.req("FR-071")` |
| FR-072      | Given a controlled test fixture is prepared for FR-072, including the role and data conditions stated in the requirement | When the operation or event specified by FR-072 is invoked            | Then the application satisfies this requirement: prevent deactivation of the last active Administrator and prevent an Administrator from deactivating their own account.                                                                                                                                                                                                                | `@pytest.mark.req("FR-072")` |
| FR-073      | Given a tender is in Closed state and an authorised Procurement Officer is assigned to its entity                        | When the officer records an evaluation outcome                        | Then only PASS, FAIL or INCONCLUSIVE is accepted and evaluator identity/time are persisted.                                                                                                                                                                                                                                                                                             | `@pytest.mark.req("FR-073")` |
| FR-074      | Given a controlled test fixture is prepared for FR-074, including the role and data conditions stated in the requirement | When the operation or event specified by FR-074 is invoked            | Then the application satisfies this requirement: maintain a separate immutable eligibility-period history for each supplier, with non-overlapping validity intervals; corrections shall supersede rather than erase prior entries.                                                                                                                                                      | `@pytest.mark.req("FR-074")` |
| FR-075      | Given a controlled test fixture is prepared for FR-075, including the role and data conditions stated in the requirement | When the operation or event specified by FR-075 is invoked            | Then the application satisfies this requirement: create a system-actor audit identity for automated rule evaluation and finding lifecycle changes, distinct from human user accounts.                                                                                                                                                                                                   | `@pytest.mark.req("FR-075")` |
| FR-076      | Given a controlled test fixture is prepared for FR-076, including the role and data conditions stated in the requirement | When the operation or event specified by FR-076 is invoked            | Then the application satisfies this requirement: support contract states Draft, Executed, Completed and Terminated, with only Draft → Executed → Completed or Terminated transitions; Draft → Terminated is permitted with a reason.                                                                                                                                                 | `@pytest.mark.req("FR-076")` |
| FR-077      | Given a controlled test fixture is prepared for FR-077, including the role and data conditions stated in the requirement | When the operation or event specified by FR-077 is invoked            | Then the application satisfies this requirement: permit Compliance Officers to reopen a False Positive finding explicitly, recording the actor, reason and time.                                                                                                                                                                                                                        | `@pytest.mark.req("FR-077")` |
| FR-078      | Given a controlled test fixture is prepared for FR-078, including the role and data conditions stated in the requirement | When the operation or event specified by FR-078 is invoked            | Then the application satisfies this requirement: allow an authenticated user to update only their own non-privileged profile fields; role, entity assignment and account status shall remain Administrator-controlled.                                                                                                                                                                  | `@pytest.mark.req("FR-078")` |
| FR-080      | Given a client submits a payload containing a server-controlled field                                                    | When the API validates the request                                    | Then the field is rejected or ignored and cannot alter role, ownership, audit or workflow state.                                                                                                                                                                                                                                                                                        | `@pytest.mark.req("FR-080")` |
| FR-081      | Given a resource exists but belongs to an entity outside the caller's scope                                              | When the caller requests the resource by identifier                   | Then the API returns 404 without disclosing the resource.                                                                                                                                                                                                                                                                                                                               | `@pytest.mark.req("FR-081")` |
| FR-082      | Given a supplier eligibility period is created or superseded                                                             | When the change is committed                                          | Then all affected tenders/contracts are re-evaluated and findings update idempotently.                                                                                                                                                                                                                                                                                                  | `@pytest.mark.req("FR-082")` |
| FR-083      | Given a Compliance Officer changes a rule parameter or version                                                           | When the change is committed                                          | Then affected findings are re-evaluated and the parameter change is audited.                                                                                                                                                                                                                                                                                                            | `@pytest.mark.req("FR-083")` |
| FR-084      | Given an Open or Acknowledged finding's condition is no longer present                                                   | When the rule is re-evaluated                                         | Then the finding remains in history and moves to Resolved with a system reason.                                                                                                                                                                                                                                                                                                         | `@pytest.mark.req("FR-084")` |

## A.1 Test coverage automation

Register the custom marker in `pytest.ini`:

```ini
[pytest]
markers =
    req(requirement_id): requirement ID verified by this test
```

Example:

```python
@pytest.mark.req("FR-005")
def test_account_lockout_after_fifth_failure(...):
    ...
```

The CI coverage plugin shall inspect collected tests and compare markers with `docs/requirements-matrix.json`. CI shall fail if any Must FR has no collected test marker, any marker refers to an unknown ID, or any Must FR has no acceptance criterion. Tests shall not be tagged merely to satisfy coverage; each test must assert the behaviour linked to its marker. Manual-only verification requires an explicit reason and Project Owner approval and does not silently satisfy the automated coverage gate.

## A.2 Requirement coverage definition

A requirement is covered only when it has a criterion, at least one collected test with its exact marker, assertions for the expected result and relevant negative/boundary cases, and a passing CI result. A marker alone is not proof of implementation or success.

# Annex B. Seed manifest and rule injection design

The committed `database/seed/seed-manifest.json` shall identify each fixture by stable ID and include: `fixture_id`, `rule_id`, `record_ids`, `injection_method`, `expected_finding_count`, `expected_preventive_rejection`, `dataset` and `notes`.

Normal valid seed data and intentionally invalid compliance fixtures shall be separate seed profiles. The normal profile must satisfy database constraints. The invalid-fixture profile may bypass application services only in isolated tests or a clearly labelled demonstration fixture loader. It shall never disable or drop database constraints in the ordinary application database.

Expected outcomes are specified in Section 8.5. Preventive-only RULE-009 has zero expected findings; its expected outcome is rejected writes and database constraint tests. RULE-005 has zero expected findings unless OA-004 has been completed and the rule is enabled. RULE-007 is Should and may be omitted if cut under the charter's cut order.

# Annex C. Canonical audit event format

The audit chain shall use a single serialised chain head protected by a database row lock in the same transaction as the audit insert. This mechanism must work across all Gunicorn workers, not merely within one process.

Each event shall include:

- monotonically increasing `sequence`;
- actor identifier, with a reserved system actor for automated actions;
- action and target type/identifier;
- canonical event payload;
- UTC timestamp;
- previous event hash;
- current event hash.

Canonicalisation shall be deterministic: UTF-8 encoding, Unicode NFC normalisation, lexicographically sorted object keys, compact JSON separators, timestamps normalised to UTC ISO-8601, decimal values represented as strings, and null represented consistently. SHA-256 shall be calculated over the canonical event representation including the previous hash and sequence number. The genesis event shall use a documented fixed genesis value. The verification command shall recompute the entire sequence and report the first mismatch.

# Annex D. API and time conventions

- Pagination: default 25, maximum 100, stable ordering and a unique tie-breaker.
- Out-of-scope object lookup: 404.
- Throttling: 429 and `Retry-After` where known.
- Error body: `application/problem+json`.
- Client-writable fields: explicit allow-list; ownership, role, audit metadata and workflow states are server-controlled.
- Newman session flow: obtain session cookie, obtain CSRF token, then send unsafe methods with both.
- Persisted instants: timezone-aware UTC.
- Business comparison timezone: `Africa/Nairobi`.
- Tender publication/closing are instants, not date-only values.
- Tests shall inject/freeze the clock and cover instants immediately before, at and after the closing instant.

# Annex E. Charter alignment and change-control items

The following parent-charter alignment items remain open for controlled change management. They are recorded transparently and do not silently amend the approved DEL-001 Version 1.0 baseline.

| Parent item | Current charter baseline                                                                     | Required action                                                                                                                                                                                              |
| ----------- | -------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| SC-001      | 100% of Must user stories have acceptance criteria and a linked test                         | Proposed wording: “100% of Must functional requirements have Given/When/Then acceptance criteria and at least one linked test; user stories group stakeholder needs.” Retain story-level links in DEL-002. |
| SC-006      | Every Must rule has pass/fail/boundary tests and seeded violations produce expected findings | Clarify that preventive-only rules have rejected-write/constraint tests and no expected finding; detective/both rules have seeded invalid fixtures and manifest counts.                                      |
| SC-009      | HTTPS, non-root containers and 15-minute clean-clone target                                  | NFR-009 keeps non-root as Must for Nginx and application containers. Clean-clone timing records prerequisites and image-download time separately.                                                            |
| SC-010      | Restore target 15 minutes or less                                                            | DEL-002 treats 15 minutes as a measured target; report actual elapsed time and any miss. Parent criterion should explicitly say whether missing the target is a failure or a disclosed deviation.            |
| SC-016      | Browser compatibility, keyboard navigation and API p95                                       | Tablet responsiveness and reduced-motion are Should in DEL-002 unless new success criteria are approved.                                                                                                     |
| §9.2       | Five simulated stakeholder personas                                                          | DEL-002 now uses Procurement Officer, ICT Officer, ICT Security Officer, System Administrator and End User.                                                                                                  |
| Annex A3    | RULE-009 listed with the compliance rules                                                    | Proposed clarification: RULE-009 is preventive-only; no seed finding expected.                                                                                                                               |
| Annex A4    | Every record carries an entity reference                                                     | Proposed clarification: shared Supplier master and global reference data are not entity-owned; entity scoping applies to procurement records and entity-specific operational records.                        |
| Annex A3    | RULE-007 phrased as awards/contracts                                                         | Canonical count is awarded tenders/contracts counted by tender award date, with threshold more than 3 in a rolling 12-month window.                                                                          |
| Annex A2/A4 | Tender lock and data conventions                                                             | Add field-level tender editability, evaluation metadata, contract lifecycle and UTC/Nairobi time semantics from this specification.                                                                          |

DEL-002 Version 1.0 is approved against DEL-001 Version 1.0. The Project Owner may address these items through a coordinated DEL-001 v1.1 and DEL-002 v1.1 change set. Until such revisions are approved, this requirements specification must be interpreted alongside the currently approved parent charter; where a conflict exists, the parent charter controls and the conflict must be logged for resolution. No proposed wording in this annex changes DEL-001.

**END OF DOCUMENT — PPMCMS-DEL-002 — VERSION 1.0 APPROVED**
