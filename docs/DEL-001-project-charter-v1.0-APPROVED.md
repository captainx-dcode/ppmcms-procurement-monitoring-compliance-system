# PPMCMS Project Charter

## Public Procurement Monitoring and Compliance Management System

**Document ID:** PPMCMS-DEL-001
**Version:** 1.0 — Approved baseline
**Status:** Approved — Project Owner approval recorded 24 November 2026
**Project Owner and Approver:** Eugen Walubengo Nyongesa
**Date:** 24 November 2026
**Supersedes:** Version 0.4 and earlier drafts
**Repository:** `ppmcms-procurement-monitoring-compliance-system`
**Review basis:** Self-review by the Project Owner, informed by an AI-assisted critique of earlier drafts and the Project Owner's subsequent decisions. This is not an independent human review.
**Version record:** Git history is authoritative. This consolidated version incorporates the accepted decisions from Version 0.4 into the complete Version 0.3 charter.

---

## 1. Purpose and background

PPMCMS is an independent software engineering project demonstrating how a procurement monitoring application, its supporting infrastructure and its security controls work together. It is inspired by public-sector procurement oversight and by the functional competencies relevant to the PPRA ICT Officer II role.

It is not an official PPRA application. It does not integrate with, replace or represent any PPRA or government system. It uses fictional procurement data and does not claim PPRA endorsement.

### The problem

An oversight body monitors procurement activities across multiple procuring entities. Tender, supplier and contract records may be fragmented or inconsistent, making it difficult to identify incomplete records, possible rule breaches and suspicious patterns, produce useful reports and establish who changed a record.

PPMCMS demonstrates a controlled prototype that:

- Holds procurement records for several procuring entities with controlled access.
- Applies explicit, testable compliance rules and presents findings for review.
- Maintains a tamper-evident audit trail.
- Runs on reproducible, hardened infrastructure that can be backed up and restored.

## 2. Aim and objectives

**Aim:** Deliver a working, securely configured and reproducibly deployable prototype within a 10-day MVP schedule, supported by honest evidence of what has been built, tested and demonstrated.

**Objectives**

1. Model procurement using procuring entities, suppliers, tenders with a defined lifecycle, and contracts.
2. Implement the compliance rules in Annex A and present violations as reviewable findings.
3. Enforce authentication, four-role access control and procuring-entity scoping.
4. Record a tamper-evident audit trail of write operations and authentication events.
5. Run the system using Docker Compose behind Nginx, with TLS, separated networks, non-root containers and a database not exposed to the host.
6. Demonstrate backup and recovery through a documented restore drill.
7. Automate quality and security checks in CI from Day 1.
8. Develop an early threat model using a data-flow diagram and STRIDE, keep it current, and assess the application against OWASP ASVS Level 1 and the OWASP API Security Top 10.
9. Maintain accurate documentation that distinguishes planned, implemented, tested and demonstrated capabilities.

## 3. Scope

Application development, infrastructure and security receive balanced attention from Day 1. Infrastructure and security are implemented and tested rather than left solely as documentation tasks.

### 3.1 MVP scope and priorities (MoSCoW)

| Priority                   | Included functionality                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| -------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Must**             | Authentication; four-role RBAC with procuring-entity scoping; append-only, hash-chained audit log; procuring entities, suppliers, tenders with lifecycle, and contracts; soft deletion only for permitted draft records; awarded records locked; eight Must compliance rules (RULE-001 to RULE-006, RULE-008 and RULE-009); findings; basic dashboard; REST API for core resources sharing the application service layer with the UI; reproducible fictional seed data; Docker Compose stack with Nginx, Django/Gunicorn and PostgreSQL; TLS and separated networks; backup script and tested restore drill; CI with security scanning; DFD and STRIDE threat model; ASVS Level 1 checklist; clean-clone README instructions; demonstration video. |
| **Should**           | CSV export; four-eyes approval on award; RULE-007 repeat-award heuristic; UI for editing rule parameters; generated OpenAPI page; Newman in CI; encrypted backups.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| **Could**            | Additional dashboard charts; search and filter improvements.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| **Won't in the MVP** | PDF reports; monitoring and alerting stack; high availability or failover; file uploads; MFA or SSO; multi-VM network lab; React SPA front end; email notifications; external-system integrations.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |

The four-eyes approval control remains a Should requirement. If implemented, the user advancing a tender to Awarded must be different from the user who recorded the evaluation outcome. The seed dataset must provide at least two Procurement Officer accounts for each procuring entity so this workflow can be exercised. If schedule constraints prevent implementation, it may be deferred without compromising the Must baseline.

### 3.2 Out of scope

The project excludes:

- Production deployment for PPRA or any institution.
- Integration with official systems, including PPIP and e-GP.
- Confidential government, supplier or citizen data.
- Real procurement transactions.
- Claims of PPRA endorsement, ISO certification or formal regulatory compliance.
- Production-scale or high-availability guarantees.

**File uploads are explicitly out of scope.** Tender documents are represented as metadata only, such as title and reference. The system does not accept or store files.

If file uploads are considered after the MVP, they require allow-listed file types, size limits, storage outside the web root, malware scanning, random file names and authorised downloads.

### 3.3 Infrastructure scope and honesty boundary

Infrastructure capabilities must be labelled accurately.

| Status                          | Items                                                                                                                                                                                                                                                                                                                                                                                                                              |
| ------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Implemented in MVP**    | Docker Compose stack with Nginx, Django/Gunicorn and PostgreSQL; separate frontend and backend networks, with the backend network internal; database port not published to the host; TLS at Nginx; non-root containers; pinned image versions; health checks; secrets supplied through the environment or Docker secrets and never committed; Nginx rate limits on login and API; `pg_dump` backup script with a tested restore. |
| **Simulated**             | Network segmentation through Docker networks, not a physical or virtual-machine firewall boundary; TLS using a self-signed or mkcert certificate, not a trusted certificate authority; single-host deployment.                                                                                                                                                                                                                     |
| **Design-only, post-MVP** | High availability, monitoring and alerting, multi-VM topology and host firewall rule set.                                                                                                                                                                                                                                                                                                                                          |

Design-only capabilities must never be presented as deployed or implemented.

### 3.4 Security scope and personal data

The security baseline in Annex B covers:

- Authentication, password policy and lockout.
- Rate limiting and session security.
- CSRF, CORS and security headers.
- Input validation and output handling.
- Object-level authorisation and entity scoping.
- Audit-log integrity.
- Database least privilege.
- Secrets management.
- Dependency and container-image security.
- Backup protection and recovery.

Although procurement records are fictional, user accounts still involve personal data. The project will follow a minimisation approach informed by the Kenya Data Protection Act, 2019:

- Use only minimum account details: name, username, role and fictitious email address.
- Hash passwords.
- Restrict access by role.
- Do not use real individuals' personal data.
- Delete or anonymise demonstration accounts at project end.

This describes intended project practice. It is not a claim of legal compliance.

## 4. Approach and technology

### 4.1 Confirmed technology architecture

The Project Owner has confirmed the following architecture for the MVP.

| Component               | Selected technology                                    |
| ----------------------- | ------------------------------------------------------ |
| Backend                 | Python and Django                                      |
| API                     | Django REST Framework                                  |
| User interface          | Django templates with HTMX                             |
| Styling                 | Tailwind CSS using the build-time CLI                  |
| Charts                  | Self-hosted Chart.js                                   |
| Authentication          | Django session authentication                          |
| Session protection      | HttpOnly, Secure and SameSite cookies; CSRF protection |
| Database                | PostgreSQL                                             |
| Web tier                | Nginx reverse proxy                                    |
| Application server      | Gunicorn                                               |
| Containers              | Docker Compose                                         |
| Testing                 | pytest, pytest-django, Postman and Newman              |
| CI/CD                   | GitHub Actions                                         |
| Development environment | Windows, WSL2 Ubuntu, VS Code and Docker               |

The API will expose core resources while sharing the same application service layer as the user interface.

The MVP will use a same-origin architecture and session-based authentication. Browser-stored authentication tokens and a separate React SPA are not part of the MVP.

**Architecture rationale**

A separate React SPA and API would introduce additional CORS, CSRF and token-handling decisions, browser token-storage risks and another build pipeline. For a solo 10-day project, Django templates with HTMX and same-origin session cookies provide a simpler architecture with fewer security and implementation burdens.

React may be reconsidered after the MVP. Any future change must record decisions about token or session handling, storage, refresh and revocation.

ADR-001 will record the selected architecture, alternatives considered and the reasons for the decision.

### 4.2 Delivery method: vertical slices

Development will be iterative. Each slice delivers API, UI, automated tests and audit events together.

| Slice | Deliverables                                               |
| ----- | ---------------------------------------------------------- |
| 1     | Authentication, RBAC and audit log                         |
| 2     | Procuring entities and tenders with lifecycle              |
| 3     | Suppliers and contracts                                    |
| 4     | Compliance rules engine and findings, developed test-first |
| 5     | Dashboard and CSV export if time allows                    |

**Definition of Done for every slice**

- Reviewed against a self-review checklist.
- API and user interface working.
- Automated tests passing, including authorisation tests.
- Required audit events emitted.
- Threat model and ASVS checklist updated where affected.
- Documentation reflects the actual implementation.
- CI pipeline green.

### 4.3 Project gates

| Gate                           | Timing       | Required evidence                                                                                                                                      |
| ------------------------------ | ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **G1 — Foundation**     | End of Day 1 | Repository, Compose stack running, CI green, ADR-001, DFD and STRIDE draft, Annex A baselined, job-description mapping checked, ASVS edition selected. |
| **G2 — Feature freeze** | End of Day 6 | Must-priority features implemented. After this gate, work is limited to fixes, hardening and essential completion.                                     |
| **G3 — Release**        | Day 10       | Success-criteria evidence, demonstration, README, evaluation and release tag v0.1.0.                                                                   |

### 4.4 CI/CD and security automation

GitHub Actions will run on every push and pull request. Evidence should be generated by the pipeline rather than collected manually wherever practical.

| Tool                          | Purpose                                     | Addresses      |
| ----------------------------- | ------------------------------------------- | -------------- |
| Ruff and pytest with coverage | Linting and unit, integration and API tests | SC-006, SC-013 |
| Bandit                        | Python static security analysis             | RSK-005        |
| pip-audit                     | Known-vulnerable Python dependencies        | RSK-007        |
| gitleaks                      | Secret detection in pre-commit and CI       | RSK-008        |
| Trivy                         | Container image and filesystem scanning     | RSK-007        |
| Newman (Should)               | Executes the Postman collection             | SC-003         |
| OWASP ZAP baseline            | Dynamic scan of the running Compose stack   | SC-012         |
| Dependabot                    | Dependency update pull requests             | RSK-007        |

npm audit is only relevant if Node tooling is introduced.

## 5. Deliverables

One identifier scheme, DEL, covers all project deliverables, including formal documents.

| ID      | Deliverable                          | Contents                                                                                                                                |
| ------- | ------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------- |
| DEL-001 | Project Charter                      | Project purpose, objectives, scope, approach, governance and baselines                                                                  |
| DEL-002 | Requirements Specification           | User stories, acceptance criteria, simulated personas, NFRs, compliance rules, RBAC matrix, tender state machine and traceability table |
| DEL-003 | Architecture and Data Design         | Architecture, DFD with trust boundaries, ERD and data classification                                                                    |
| DEL-004 | Threat Model and Security Plan       | STRIDE table, ASVS Level 1 checklist and security control baseline                                                                      |
| DEL-005 | Infrastructure and Deployment Guide  | Compose stack, TLS, network separation, hardening, backup and restore drill, implemented/simulated/design-only distinctions             |
| DEL-006 | Test Plan and Results                | Test strategy, results, coverage and scan reports                                                                                       |
| DEL-007 | Working Application                  | Application source code                                                                                                                 |
| DEL-008 | API Specification                    | Generated OpenAPI schema and Postman collection                                                                                         |
| DEL-009 | Database                             | Database schema through migrations and seed script                                                                                      |
| DEL-010 | CI/CD Pipeline and Security Evidence | Workflows and scan outputs                                                                                                              |
| DEL-011 | User Guide                           | Role-based usage guide                                                                                                                  |
| DEL-012 | Demonstration                        | 3–5 minute video and screenshots                                                                                                       |
| DEL-013 | Final Evaluation Report              | Results against success criteria, limitations and post-MVP backlog                                                                      |
| DEL-014 | GitHub Repository                    | README, LICENSE, SECURITY.md, CHANGELOG, ADRs, issues, project board and release tag v0.1.0                                             |

The Technical Manual is incorporated into the README, DEL-003 and DEL-005. It is not a separate MVP deliverable.

Stakeholder analysis is included in DEL-002. The requirements traceability matrix is a table within DEL-002.

## 6. Success criteria

Each criterion has a pass threshold and named evidence.

| ID     | Criterion                 | Pass threshold                                                                                                                                                                                      | Evidence                |
| ------ | ------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------- |
| SC-001 | Requirements traceability | 100% of Must user stories have acceptance criteria and at least one linked test                                                                                                                     | DEL-002                 |
| SC-002 | Authentication            | Login, logout and session expiry work; account locks after 5 failed attempts in 15 minutes; automated tests demonstrate behaviour                                                                   | DEL-006                 |
| SC-003 | RBAC                      | Every allow and deny cell in the RBAC matrix has an automated test; no unauthorised action succeeds; cross-entity access is denied                                                                  | DEL-006                 |
| SC-004 | Tender lifecycle          | All valid transitions pass; invalid transitions are rejected; locked records cannot be edited or deleted                                                                                            | DEL-006                 |
| SC-005 | Suppliers and contracts   | Records follow the RBAC matrix; invalid and duplicate records are rejected through validation and database constraints                                                                              | DEL-006                 |
| SC-006 | Compliance engine         | Every Must rule has pass, fail and boundary tests; compliance module coverage is at least 80%; seeded violations produce expected findings                                                          | Coverage report         |
| SC-007 | Audit trail               | 100% of write endpoints and authentication events create audit entries; runtime database role cannot update or delete audit rows; hash-chain verification detects deliberate tampering              | Tests and demonstration |
| SC-008 | Dashboard and export      | Figures match known seed-data counts; dashboard loads within 2 seconds on seed data; CSV content is verified if delivered                                                                           | DEL-006                 |
| SC-009 | Reproducible deployment   | From a clean clone, README brings the stack to healthy within 15 minutes; DB port is unreachable from host; HTTPS is served; containers run as non-root                                             | DEL-005                 |
| SC-010 | Backup and restore        | Restore into a clean database; row counts match for every table; elapsed time recorded; target is 15 minutes or less                                                                                | Restore drill report    |
| SC-011 | Threat model              | DFD and STRIDE table complete; every High threat has a mitigation with a test or recorded risk acceptance; updated after Slice 5                                                                    | DEL-004                 |
| SC-012 | Security testing          | No open Critical or High findings from pip-audit, Bandit, fixable Trivy findings and ZAP baseline; zero gitleaks findings; every ASVS Level 1 checklist item marked Pass, Fail or N/A with a reason | CI and reports          |
| SC-013 | Continuous integration    | Pipeline runs on every push and pull request; main is green at release                                                                                                                              | GitHub Actions history  |
| SC-014 | Defects                   | No open Critical defects at release; other defects recorded with status                                                                                                                             | GitHub Issues           |
| SC-015 | Honest documentation      | Every capability claim labelled planned, implemented, tested or demonstrated; a new user can run and demonstrate the system using the README                                                        | Review                  |
| SC-016 | Non-functional baseline   | Works in current Chrome, Firefox and Edge; main pages are keyboard-navigable; list endpoints under 500 ms at the 95th percentile on seed data                                                       | DEL-006                 |
| SC-017 | Demonstration             | 3–5 minute video covers login, RBAC, tender lifecycle, compliance finding, audit tamper check and backup restore                                                                                   | DEL-012                 |

## 7. Timeline

The 10-day schedule is an MVP target, not a guarantee that every feature will be delivered. Quality, security and accurate documentation take precedence over unsupported completion claims.

| Timing                 | Focus                                                                                                                                                                                                     |
| ---------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Before Day 1** | Verify WSL2, Docker and Docker Compose installation and confirm that the development environment can run the intended stack. Resolve environment blockers before G1.                                      |
| **Day 1**        | Repository; Compose skeleton (Nginx, Django, PostgreSQL); CI pipeline; DFD and STRIDE draft; Annex A baseline; ADR-001; job-description mapping; ASVS edition selection; charter approval baseline — G1. |
| **Day 2**        | Slice 1: authentication, RBAC and audit log.                                                                                                                                                              |
| **Day 3**        | Slice 2: procuring entities and tenders.                                                                                                                                                                  |
| **Day 4**        | Slice 3: suppliers and contracts.                                                                                                                                                                         |
| **Day 5**        | Slice 4: compliance rules engine, developed test-first. Verify RULE-005 legal thresholds before implementation.                                                                                           |
| **Day 6**        | Slice 5: dashboard and CSV export — G2 feature freeze.                                                                                                                                                   |
| **Day 7**        | Nginx and TLS hardening, network separation, rate limiting and security headers.                                                                                                                          |
| **Day 8**        | ZAP baseline, ASVS Level 1 checklist, defect fixing and deadline for seeking genuine external feedback.                                                                                                   |
| **Day 9**        | Backup and restore drill, threat model update and deployment documentation.                                                                                                                               |
| **Day 10**       | Demonstration video, README, tag v0.1.0, evaluation and post-MVP backlog — G3.                                                                                                                           |

**Known constraint:** The plan has no slack day, and Day 1 is the heaviest. A slice that is not completed on its planned day is reduced to its smallest working version rather than automatically carried forward.

**Cut order if behind** (first to cut, then subsequent items): CSV export; four-eyes award approval; OpenAPI page and Newman; rule-parameter UI; RULE-007; encrypted backups; extra dashboard charts.

**Never cut:** RBAC tests, audit log, backup drill, CI security scanning and honest documentation.

Post-MVP iterations may address functional and usability gaps, broader infrastructure (multi-VM and host firewall), monitoring and alerting, MFA, PDF reports, expanded security testing and documentation updates.

## 8. Risk management

Risks are scored Low, Medium or High for likelihood (L) and impact (I). The Project Owner owns every risk. Risks are tracked as GitHub Issues labelled `risk`; this table is the baseline.

| ID      | Risk                                                          | L      | I      | Trigger                                              | Mitigation                                                                                                                                              |
| ------- | ------------------------------------------------------------- | ------ | ------ | ---------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| RSK-001 | Skill gap: Django REST, HTMX, Docker and Nginx learning curve | High   | Medium | Slice 1 not completed by end of Day 2                | Use framework defaults; timebox blockers to 60 minutes, then simplify; apply cut order                                                                  |
| RSK-002 | Environment failure involving WSL2, Docker or laptop          | Medium | High   | Stack will not start or hardware fails               | Validate environment before Day 1; push at least daily; maintain reproducible Compose setup and documented virtual-environment fallback for development |
| RSK-003 | Demonstration failure                                         | Medium | Medium | Clean-clone run fails on Day 9                       | Maintain seed script; rehearse clean-clone run; record demonstration video                                                                              |
| RSK-004 | Single-person dependency                                      | Low    | High   | Any working day is lost                              | Early feature freeze at G2; daily push; documentation matches actual implementation                                                                     |
| RSK-005 | Application vulnerabilities                                   | Medium | High   | Bandit, ZAP or ASVS finding                          | Threat model from Day 1; security checks in CI; authorisation tests; ASVS Level 1                                                                       |
| RSK-006 | Compliance rules misstate the law                             | Medium | Medium | A rule is cited to a statutory provision             | Label rules as prototype logic; verify primary text before citing; configurable parameters; no legal-compliance claims                                  |
| RSK-007 | Vulnerable or outdated dependencies and images                | Medium | Medium | pip-audit or Trivy finding                           | Pinned versions; Dependabot; fail CI on Critical/High findings                                                                                          |
| RSK-008 | Credential or secret exposure                                 | Low    | High   | gitleaks alert                                       | Environment variables; `.env.example` only; gitleaks pre-commit and CI; rotate any exposed secret                                                     |
| RSK-009 | Scope creep                                                   | High   | Medium | New feature requested after G2                       | Change-request issues; MoSCoW freeze at G2                                                                                                              |
| RSK-010 | Insufficient testing because features land late               | Medium | High   | Slice finishes without tests                         | Definition of Done requires tests; CI gates merges                                                                                                      |
| RSK-011 | Infrastructure claims exceed implementation                   | Medium | High   | Documentation describes unbuilt features             | Implemented/simulated/design-only table; evidence for every capability claim                                                                            |
| RSK-012 | Little genuine external feedback                              | High   | Low    | No reviewer by Day 8                                 | Use clearly labelled personas for acceptance criteria; seek one or two real reviewers by Day 8; record source and date                                  |
| RSK-013 | Audit log rewritten by privileged database user               | Medium | Medium | Database-role review identifies excessive privileges | Hash chain; INSERT/SELECT-only runtime role; document limitations; external anchoring of head hash post-MVP                                             |
| RSK-014 | Unrealistic seed data makes dashboard and rules meaningless   | Medium | Medium | Dashboard looks empty or uniform                     | Fixed-seed Faker script with deliberately planted violations for every applicable compliance rule                                                       |

## 9. Governance, review and records

### 9.1 Ownership

The Project Owner manages scope, schedule, technical decisions, documentation and quality.

### 9.2 Review integrity

Document statuses are:

- Draft
- Self-reviewed
- Externally reviewed
- Approved
- Superseded

A document is not Approved until approval is explicitly recorded.

Simulated stakeholders — Procurement Officer, ICT Officer, ICT Security Officer, System Administrator and End User — are personas used to generate acceptance criteria in DEL-002. Their input must never be presented as feedback from real people.

One or two genuine reviewers, such as a peer, mentor or community reviewer, will be sought. Each genuine external comment must be recorded with its source and date.

The Project Owner's self-review and AI-assisted critique are not independent human reviews.

### 9.3 Project records

Records are maintained in GitHub rather than separate Markdown registers.

- Defects, change requests, risks, feedback and lessons learned are GitHub Issues with labels `defect`, `change-request`, `risk`, `feedback` and `lesson`.
- Technical decisions are Architecture Decision Records in `docs/adr/`.
- Each change request states the proposed change, reason and impact on scope, schedule and security.
- Progress-meeting notes and separate code-review logs are not required. Pull-request checklists and Git history provide the project record.

### 9.4 Identification conventions

| Prefix       | Meaning                                       | Recorded in           |
| ------------ | --------------------------------------------- | --------------------- |
| DEL          | Deliverable, including documents and software | Section 5             |
| FR, NFR      | Functional and non-functional requirement     | DEL-002               |
| SC           | Success criterion                             | Section 6             |
| RULE         | Compliance rule                               | Annex A3 and DEL-002  |
| THR          | Threat                                        | DEL-004               |
| TC           | Test case                                     | Test code and DEL-006 |
| RSK, DEF, CR | Risk, defect and change request               | GitHub Issues         |
| ADR          | Architecture decision                         | `docs/adr/`         |

Every requirement must be traceable to its design, implementation component and test through the traceability table in DEL-002.

### 9.5 Git practice

- Git is used for source code and project documentation.
- Development follows trunk-based development with short-lived feature branches.
- Every change reaches `main` through a pull request linked to an issue.
- CI must pass before merging.
- Pull requests use a self-review checklist.
- Commit messages should be meaningful and follow a consistent convention.
- Release tags identify completed project versions.
- `SECURITY.md` and `CHANGELOG.md` are maintained.
- Simulated reviews must be distinguished from independent reviews.
- GitHub activity must accurately reflect work actually performed.

### 9.6 Document control

Each formal document carries a short header containing its ID, title, version, status, owner and date.

Git history is the authoritative version record. Separate per-document revision tables are not maintained.

Versioning convention:

- 0.x — draft.
- 1.0 — first approved baseline.
- 1.x — minor amendments.
- 2.0 — significant changes to scope, objectives or governance.

## 10. Standards and legal references

The project uses a focused set of standards and references. Inclusion does not establish certification, formal compliance or legal approval.

| Reference                                                               | Project use                                                                                                      |
| ----------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| OWASP ASVS Level 1, with edition selected and recorded in DEL-004 by G1 | Pass/Fail/N/A verification checklist                                                                             |
| OWASP API Security Top 10                                               | API test cases and threat-model input                                                                            |
| OWASP Top 10, current edition selected for the review                   | Web vulnerability review during Day 8                                                                            |
| STRIDE                                                                  | Threat-modelling method                                                                                          |
| Public Procurement and Asset Disposal Act, 2015                         | Context for procurement compliance rules; provisions must be verified against primary legal text before citation |
| Kenya Data Protection Act, 2019                                         | Reference for handling user-account personal data                                                                |

Background references only, with no formal mapping or compliance claim:

- NIST Cybersecurity Framework 2.0.
- ISO/IEC 27001.
- ISO/IEC 25010.
- ISO/IEC/IEEE 29148, used as guidance for writing singular, testable requirements.

## 11. Assumptions, constraints and test data

### 11.1 Assumptions

- This is an independent portfolio project.
- Procurement data is fictional.
- Windows, WSL2 Ubuntu, VS Code and Docker are intended development tools.
- GitHub is the central project repository.
- Simulated stakeholder personas are used where genuine stakeholders are unavailable.

### 11.2 Constraints

- Limited MVP time.
- One-person delivery.
- No production infrastructure.
- No access to confidential procurement information.
- No official integration or endorsement.
- Development environment availability must be verified before Day 1.

### 11.3 Confirmed test data

A Python seed script using Faker and a fixed random seed will generate approximately:

| Data category      | Target volume |
| ------------------ | ------------: |
| Procuring entities |             5 |
| Suppliers          |            60 |
| Tenders            |           250 |
| Contracts          |           150 |

The dataset must include deliberately planted violations for every applicable compliance rule so the dashboard and findings are meaningful.

For the four-eyes award workflow, the seed dataset must also include at least two Procurement Officer accounts for each procuring entity. This ensures that the workflow can distinguish the user recording an evaluation from the user approving the award.

The data must remain fictional and reproducible. Any later material reduction or change to these volumes must be recorded through the change-control process.

## 12. PPRA job-description mapping

The project is intended to demonstrate competencies relevant to the ICT Officer II role. The mapping below is a working project alignment, not a verified quotation or exhaustive interpretation of the vacancy.

The actual PPRA ICT Officer II job-description document must be checked by G1. Any missing competencies must be added to the mapping.

| Competency area                                          | Project artefacts                                              |
| -------------------------------------------------------- | -------------------------------------------------------------- |
| Business application development and database management | DEL-007, DEL-009                                               |
| Requirements analysis and system design                  | DEL-002, DEL-003                                               |
| ICT infrastructure planning and configuration            | DEL-005, Compose stack                                         |
| Network architecture and service availability            | DEL-005: network separation, health checks, backup and restore |
| Application and API security                             | DEL-004, DEL-008, SC-012                                       |
| Cybersecurity risk assessment and threat modelling       | DEL-004 and `risk` issues                                    |
| Software testing, debugging and quality assurance        | DEL-006, DEL-010                                               |
| Technical documentation and change management            | DEL-001 to DEL-014, `change-request` issues and ADRs         |
| Version control and structured development               | DEL-014, DEL-010                                               |
| Problem-solving and evaluation of feedback               | DEL-013 and `feedback` issues                                |

If the verified job description places greater emphasis on infrastructure, networking or user support, the Project Owner should adjust effort toward Days 7–9 and update the traceability mapping.

## 13. Outstanding verification actions and deadlines

The following tasks are mandatory project controls. Their completion must be recorded in the relevant project artefacts.

| ID     | Action                                                                                                            | Deadline               | Required evidence                                                  |
| ------ | ----------------------------------------------------------------------------------------------------------------- | ---------------------- | ------------------------------------------------------------------ |
| OA-001 | Verify the PPRA ICT Officer II job description and reconcile the competency mapping                               | Before or at G1, Day 1 | Updated Section 12 and competency-to-artefact mapping              |
| OA-002 | Confirm the OWASP ASVS edition to be used                                                                         | Before or at G1, Day 1 | Edition recorded in DEL-004                                        |
| OA-003 | Verify WSL2, Docker and Docker Compose and confirm the stack can start                                            | Before Day 1           | Successful environment check; resolve blockers before G1           |
| OA-004 | Verify the legal basis and procurement-method publication-to-closing periods for RULE-005 against primary sources | Before Slice 4, Day 5  | Source references and verified rule parameters recorded in DEL-002 |
| OA-005 | Seek one or two genuine external reviewers and record any feedback                                                | By Day 8               | Source, date and summary recorded through GitHub feedback issues   |
| OA-006 | Record ADR-001 and confirm the repository reflects the selected architecture                                      | By G1, Day 1           | ADR-001 in `docs/adr/`                                           |

### RULE-005 verification fallback

RULE-005 must not use invented or unverified publication-to-closing thresholds.

If the required periods cannot be verified against primary legal sources before Slice 4 on Day 5:

1. RULE-005 must ship disabled in the MVP.
2. Its status must be explicitly labelled **Unverified — disabled**.
3. No guessed numerical threshold may be configured or applied.
4. The limitation must be documented in DEL-002 and DEL-013.
5. The rule may be enabled in a later iteration only after its source and parameters have been verified and recorded.

This fallback does not authorise the project to claim that the rule is implemented or operational while disabled.

## 14. Completion and acceptance

The MVP is ready for evaluation at G3 when:

- Implemented functionality is demonstrated.
- Limitations are documented.
- Architecture and database design are documented.
- Security controls and known risks are recorded.
- Functional and security tests have run and their results are recorded.
- Unresolved Critical defects are disclosed, with SC-014 requiring none to remain open at release.
- Deployment instructions work from a clean clone.
- The repository contains the documentation and evidence.
- The final evaluation reports completed work, unmet criteria and recommended improvements.

The project must not be described as production-ready merely because the MVP has been demonstrated.

## 15. Approval

This consolidated charter has been prepared following the Project Owner's acceptance of the seven design decisions and subsequent review corrections.

The document has been self-reviewed. The review was informed by AI-assisted critique, but no independent human review has taken place.

| Approval role | Name                     | Decision                                            | Date           |
| ------------- | ------------------------ | --------------------------------------------------- | -------------- |
| Project Owner | Eugen Walubengo Nyongesa | Approved — Version 1.0 established as the project baseline | 24 November 2026 |

**Approval record:** The Project Owner explicitly approved this document on 24 November 2026. Version 1.0 is the approved project baseline. This approval records the project baseline; it does not imply independent human review, external stakeholder endorsement, legal verification of outstanding actions, or implementation of planned capabilities.

---

# Annex A. Domain baseline

These definitions form the initial domain baseline for the Requirements Specification, DEL-002.

The compliance rules are prototype logic, not legal advice. Any legal basis must be verified against the primary text of the Public Procurement and Asset Disposal Act, 2015 and applicable regulations before being cited.

## A1. Roles and RBAC matrix

Principles:

- Deny by default.
- Apply object-level checks and procuring-entity scoping on every query.
- Administrators cannot read procurement data and cannot change their own role.
- Every role change is audited.
- Auditors have read-only access as defined in the matrix.

| Resource and action                                 | Administrator | Procurement Officer | Compliance Officer | Auditor |
| --------------------------------------------------- | ------------- | ------------------- | ------------------ | ------- |
| User accounts: manage                               | Allow         | Deny                | Deny               | Deny    |
| Own password and profile: change                    | Allow         | Allow               | Allow              | Allow   |
| Procuring entities: manage                          | Allow         | Deny                | Deny               | Deny    |
| Procuring entities: view                            | Allow         | Own                 | All                | All     |
| Suppliers (shared across entities): create and edit | Deny          | Allow               | Deny               | Deny    |
| Suppliers: view                                     | Deny          | Allow               | Allow              | Allow   |
| Tenders: create and edit (Draft)                    | Deny          | Own entity          | Deny               | Deny    |
| Tenders: advance state or cancel                    | Deny          | Own entity          | Deny               | Deny    |
| Tenders: view                                       | Deny          | Own entity          | All                | All     |
| Contracts: create and edit (before lock)            | Deny          | Own entity          | Deny               | Deny    |
| Contracts: view                                     | Deny          | Own entity          | All                | All     |
| Soft delete (Draft records only)                    | Deny          | Own entity          | Deny               | Deny    |
| Findings: view                                      | Deny          | Own entity          | All                | All     |
| Findings: acknowledge and comment                   | Deny          | Own entity          | All                | Deny    |
| Findings: resolve or mark false positive            | Deny          | Deny                | All                | Deny    |
| Rule parameters: edit                               | Deny          | Deny                | Allow              | Deny    |
| Dashboard and CSV export                            | Deny          | Own entity          | All                | All     |
| Audit log: view                                     | Allow         | Deny                | Deny               | Allow   |
| Audit log: modify or delete                         | Deny          | Deny                | Deny               | Deny    |

## A2. Tender state machine

| From                                  | To         | Who and condition                                                                                         |
| ------------------------------------- | ---------- | --------------------------------------------------------------------------------------------------------- |
| Draft                                 | Published  | Procurement Officer; fields required for the procurement method are complete                              |
| Published                             | Closed     | Procurement Officer; on or after the closing date                                                         |
| Closed                                | Evaluated  | Procurement Officer; evaluation outcome recorded                                                          |
| Evaluated                             | Awarded    | Procurement Officer; supplier, award date and value recorded; Should: a different user from the evaluator |
| Awarded                               | Contracted | Procurement Officer; contract record created                                                              |
| Draft, Published, Closed or Evaluated | Cancelled  | Procurement Officer; cancellation reason mandatory                                                        |

Every other transition is invalid and rejected.

Awarded, Contracted and Cancelled records are locked: no edit and no delete. Corrections by amendment record are post-MVP.

The compliance engine also checks data the state machine would normally prevent, such as seed data, future imports and direct database changes. This provides defence in depth.

## A3. Compliance rules catalogue

| ID       | Rule                                                                                                                                                                         | Type         | Severity | Priority                               |
| -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | -------- | -------------------------------------- |
| RULE-001 | Contract award date is on or after the tender closing date                                                                                                                   | Chronology   | High     | Must                                   |
| RULE-002 | Contract value does not exceed the tender estimate by more than the configured tolerance; confirmed default is 0%                                                            | Value        | Medium   | Must                                   |
| RULE-003 | Awarded supplier's registration or eligibility was valid on the award date                                                                                                   | Eligibility  | High     | Must                                   |
| RULE-004 | Every field required for the tender's procurement method is present before publication; field sets are defined in DEL-002                                                    | Completeness | Medium   | Must                                   |
| RULE-005 | Time from publication to closing meets the verified minimum for the procurement method; if verification is incomplete by Day 5, the rule is disabled and labelled Unverified | Process      | Medium   | Must, subject to verification fallback |
| RULE-006 | An award exists only after an evaluation outcome is recorded                                                                                                                 | Process      | High     | Must                                   |
| RULE-007 | One supplier is awarded more than 3 contracts by one entity in a rolling 12-month window; configurable; a red flag, not a breach                                             | Pattern      | Low      | Should                                 |
| RULE-008 | Contract start date is on or after the award date, and end date is after start date                                                                                          | Chronology   | Medium   | Must                                   |
| RULE-009 | Tender reference is unique per procuring entity; at most one active contract per tender                                                                                      | Integrity    | Medium   | Must                                   |

For RULE-007, the accepted default is **more than 3 awards in 12 months**, meaning the threshold is exceeded when the count reaches at least 4. The heuristic is not evidence of misconduct or a legal breach.

Findings follow this lifecycle:

- Open
- Acknowledged
- Resolved or False positive

Each finding stores the rule ID, the rule parameters in force when evaluated, the relevant record, severity and timestamps. Every finding change is audited.

Rules run when a record is saved and through an on-demand full re-evaluation command. Disabled or unverified rules must not produce active compliance findings.

## A4. Data conventions

- Procurement records are never hard-deleted. Soft deletion records actor, time and reason.
- Every record carries a procuring-entity reference used for access scoping.
- Money is stored as a decimal with explicit currency (KES), never as floating point.
- Times are stored in UTC.
- The audit log is append-only. Each entry stores actor, action, target, timestamp and a hash chained to the previous entry.
- A verification command recomputes the hash chain.
- Seed data is fictional and reproducible.

---

# Annex B. Security control baseline

| Area                    | MVP control                                                                                                                                | Verified by              |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------ |
| Authentication          | Django session authentication; Argon2 password hashing; minimum 12-character password; common-password check; generic login error messages | Tests and ASVS checklist |
| Brute force             | Lockout after 5 failures in 15 minutes; Nginx rate limits on login and API                                                                 | Automated test and ZAP   |
| Sessions                | HttpOnly, Secure, SameSite=Lax cookies; 30-minute idle and 8-hour absolute timeout; invalidated on logout and password change              | Tests                    |
| CSRF and CORS           | CSRF on every unsafe method, including API; no CORS because the application is same-origin                                                 | Tests and ZAP            |
| Security headers        | CSP, HSTS, X-Content-Type-Options, Referrer-Policy and frame-ancestors none; HTMX and Chart.js self-hosted                                 | ZAP baseline             |
| Input and output        | Server-side validation; ORM only; no raw SQL without review; template auto-escaping                                                        | Bandit and tests         |
| Authorisation           | Deny by default; entity scoping in every query; RBAC matrix tested                                                                         | SC-003                   |
| Audit integrity         | Append-only, hash-chained; runtime database role has INSERT and SELECT only on audit table; verification command                           | SC-007                   |
| Database                | Separate migration and runtime roles; no application superuser; port not published to host                                                 | SC-009                   |
| Secrets                 | Environment or Docker secrets; `.env` ignored by Git; `.env.example` committed; gitleaks pre-commit and CI                             | gitleaks                 |
| Dependencies and images | Pinned versions; pip-audit; Dependabot; Trivy; minimal base images                                                                         | CI                       |
| Errors and logging      | DEBUG off outside development; no stack traces shown to users; no passwords or secrets in logs                                             | Review and tests         |
| Backups                 | Stored outside database volume and outside Git; checksum recorded; restore drill                                                           | SC-010                   |

### Data classification

- **Public:** README, documentation and source code.
- **Internal:** Fictional procurement records.
- **Confidential:** Credentials, secrets, session data, user-account data, audit log and backups.

### Trust boundaries

- **TB1:** Browser to Nginx — untrusted network.
- **TB2:** Nginx to Django — frontend network.
- **TB3:** Django to PostgreSQL — internal backend network.
- **TB4:** Developer machine, GitHub and CI to repository — supply chain.

Detailed trust-boundary diagrams belong in DEL-003 and DEL-004.

---

# Annex C. Repository layout

One home per artefact. Test code lives in `tests/`; generated reports, screenshots and drill records live in `evidence/`; security documents live in `docs/`.

```text
ppmcms-procurement-monitoring-compliance-system/
  README.md
  LICENSE
  SECURITY.md
  CHANGELOG.md
  .gitignore
  .env.example
  docker-compose.yml

  .github/
    workflows/
    ISSUE_TEMPLATE/
    pull_request_template.md
    dependabot.yml

  docs/
    DEL-001-project-charter.md
    DEL-002-requirements-specification.md
    DEL-003-architecture-and-data-design.md
    DEL-004-threat-model-and-security-plan.md
    DEL-005-infrastructure-and-deployment-guide.md
    DEL-006-test-plan-and-results.md
    DEL-011-user-guide.md
    DEL-013-final-evaluation-report.md
    adr/
    diagrams/

  backend/
    Django project, apps, templates and static files

  database/
    seed script and migration notes

  infrastructure/
    nginx/
    docker/
    backup-recovery/

  tests/
    unit/
    integration/
    api/
    security/

  evidence/
    screenshots/
    reports/
    demo/
```

Document identifiers and traceability references must remain consistent when files are moved or renamed.

---

**END OF DOCUMENT — PPMCMS-DEL-001 — VERSION 1.0 APPROVED**
