# MeshekOS — Kibbutz Governance Platform
## Task Breakdown Document

---

## Overview

MeshekOS is a dedicated, on-premises governance operating system for kibbutz communities. This implementation plan covers the complete meeting-to-decision loop: governance context, content intake, advisory AI processing, evidence-linked human review and approval, durable decisions and confirmed tasks, notifications and return loops, historical import, permission-safe search and cited Q&A, and pilot readiness.

The plan contains 14 milestones and exactly 113 implementation tasks. Hebrew and RTL/BiDi behavior, current-state authorization, immutable source evidence, advisory-only AI, durable audit, customer-controlled deployment, and explicit human gates apply throughout. The pilot uses verified scheduled backups with an RPO of no more than 24 hours, a provisional RTO established by restore testing, and no point-in-time-recovery claim.

Tasks are ordered by hard prerequisites and safe parallelism rather than calendar estimates. A task is complete only when its acceptance criteria and verification conditions pass.

---

## Implementation Order

Recommended milestone sequence:

1. **M0 — Foundation & Infrastructure** — No milestone dependency; establishes the shared foundation.
2. **M1A — Authentication & Authorization Backend** — Follows the M0 persistence, organization, and audit foundations.
3. **M-UX — Frontend Foundation** — May proceed alongside M1A after the relevant M0 architecture and readiness foundations.
4. **M1B — Authentication UI** — Consumes the relevant M1A authentication contracts and M-UX components.
5. **M-OPS — Operational Tooling** — Builds on M0 operational foundations and may proceed alongside early feature work.
6. **M2 — Governance Body & Meeting Management** — Uses the M1A identity and authorization foundation; backend work need not wait for all M1B UI work.
7. **M3 — Content Intake & File Management** — Follows governance-body, meeting, provenance, storage, and audit foundations.
8. **M4 — AI Processing Pipeline & Provider Adapters** — Follows intake and authorization; provider evaluation may begin earlier, but a real adapter remains human-gated.
9. **M5 — Protocol Draft Review & Approval** — Consumes authorized, normalized AI results and the review/approval permission boundary.
10. **M6 — Decisions, Tasks, and Minimal Operational View** — Follows formal protocol approval and closes the first demonstrable governance slice.
11. **M7 — Notifications, Dashboard, and Unresolved Matters** — Consumes committed M4–M6 events and projections for notification, dashboard, and unresolved-matter behavior.
12. **M8 — Historical Document Import & Knowledge Base Preparation** — Uses intake and processing foundations for curated historical import and activation.
13. **M9 — Semantic/Keyword Search and AI Q&A** — Provides keyword/filter search over all currently authorized records independently of activation or embeddings; semantic retrieval and cited Q&A consume approved official records and explicitly activated curated sources.
14. **M10 — Hardening, Pilot Acceptance, and Deployment Readiness** — Validates the complete M0–M9 product, recovery, support, security, quality, and pilot evidence.

---

## Task List

---

## M0 — Foundation & Infrastructure

*12 tasks. Hard prerequisites and safe parallel work are stated per task.*

**Critical path:** Follow the hard prerequisites stated per task; work identified as parallelizable may proceed concurrently.

**Purpose:** Establish the smallest production-shaped backend, deployment, persistence, storage, audit, health, recovery, and upgrade foundations on which every later milestone can rely. M0 decides only the backend stack; frontend technology remains owned by M-UX.

**Prerequisites and parallelism:** M0 has no milestone prerequisite. After M0-T01 fixes the backend conventions and M0-T02 establishes the repository/container baseline, database, storage, logging, and recovery-design work may proceed in parallel where their listed dependencies permit. No task may introduce a runtime solely for hypothetical future scale.

**Exit gate:** The chosen backend stack and module conventions have explicit Architecture approval; the local deployment starts without mandatory external services; PostgreSQL and pgvector are migrated through the selected versioned tool; Organization and deployment metadata are separate; health/readiness, durable audit, consistent encrypted scheduled backup/restore, controlled upgrade/recovery, and the operator runbook are verified. The recovery evidence demonstrates an RPO of at most 24 hours, records a provisional measured RTO, and makes no PITR claim.

---

### M0-T01 — Backend Technology Selection and Architecture Decision Record

**Priority:** P0

**Human-review gate:** Architecture and Security

**Description**

Resolve the backend technology decision in Design §16 and record the language/framework, package and dependency management, PostgreSQL access, migration, test, lint/type-check, application bootstrap, and modular-monolith conventions before backend scaffolding begins.

**Upstream references:** Spec §§11, 13; Design §§2, 3, 4, 7, 11, 12, 13, 16.

**Owning module/component:** Architecture across the backend modular monolith; this task owns an ADR, not product code.

**In scope:**

- Compare realistic choices against on-premise simplicity, security, maintainability by the implementation team, PostgreSQL support, and worker reuse. Define one source of truth for verification commands and module dependency enforcement. Obtain explicit technical-lead approval before adoption.

**Out of scope:**

- Frontend technology selection, provider selection, queue technology, SMTP topology, invented timeouts/limits, microservices, Kubernetes, or speculative infrastructure.

**Dependencies:** Hard prerequisites: none. Parallelizable related work: recovery requirements analysis in M0-T08 may proceed without selecting a stack.

**Acceptance Criteria**

- The ADR records the decision, rejected alternatives and trade-offs
- maps every Design §4 logical owner into enforceable conventions
- defines production/test command families
- and leaves queue, SMTP, provider, and frontend choices open.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration | Validate the ADR against a machine-checkable architecture/configuration checklist; after selection, run the documented minimal compile/type, lint, unit-test, and application-start commands in a clean environment. |

---

### M0-T02 — Repository Structure and Local Container Deployment

**Priority:** P0

**Human-review gate:** Architecture and Operations

**Description**

Create the reproducible repository and single-host container baseline for the application/API and worker runtimes, PostgreSQL, customer-controlled file storage binding, and customer-specific configuration boundary.

**Upstream references:** Spec §§1, 11, 13; Design §§2, 3, 11, 12.

**Owning module/component:** Operations & Support plus application bootstrap.

**In scope:**

- The application and worker use the same versioned codebase
- local core startup and governed-data preservation require no external provider. Configuration distinguishes secrets from non-secret settings, uses persistent volumes deliberately, and contains no unused placeholder services.

**Out of scope:**

- Dependency on later `/health` or pgvector acceptance checks, frontend stack selection, Kubernetes, shared multi-tenancy, mandatory broker/object-store/telemetry services, or dummy containers reserved for future use.

**Dependencies:** Hard prerequisite: M0-T01. Parallelizable related work: M0-T03, T05, and T06 begin after the repository contract is stable.

**Acceptance Criteria**

- A clean checkout can be configured and started using documented local commands
- process restart preserves database/files
- required services expose only intended boundaries
- missing optional providers degrade only their capabilities
- the configuration template contains no credentials.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration | Exercise clean build/start/stop/restart and configuration-failure paths; inspect the effective container configuration for accidental public exposure, ephemeral governed storage, or undeclared services. |

---

### M0-T03 — PostgreSQL Schema and Versioned Migration Tooling

**Priority:** P0

**Human-review gate:** Architecture, Security, and Operations

**Description**

Establish PostgreSQL ownership, migration, schema-version, and least-privilege conventions required by all authoritative relational data.

**Upstream references:** Spec §§8, 11; Design §§4, 11, 12.

**Owning module/component:** Shared persistence foundation under the owning domain modules; Operations & Support owns migration execution.

**In scope:**

- Use the migration tool approved by M0-T01. Enable and verify pgvector through a versioned migration, not only first-run database initialization. Separate migration/ownership credentials from runtime credentials
- make forward schema state explicit and auditable.

**Out of scope:**

- Application-start auto-migration, destructive down-migration assumptions, production seed data, organization records, or storing installed application version on the Organization aggregate.

**Dependencies:** Hard prerequisites: M0-T01 and M0-T02. Parallelizable related work: M0-T05 and T06.

**Acceptance Criteria**

- An empty database reaches the expected schema deterministically
- repeat execution is safe
- an incompatible or partial schema is detected
- pgvector availability is migration-controlled
- runtime identities cannot alter schema
- migration status is inspectable without write credentials.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration | Run migrations from empty and previous supported states, test interruption/failure behavior and least-privilege denial, and verify extension/schema versions with the selected tooling's documented commands. |

---

### M0-T04 — Organization Root and Deployment Metadata

**Priority:** P0

**Human-review gate:** Architecture and Security

**Description**

Create the single-organization ownership root and a distinct deployment/system metadata record without implementing shared-database multi-tenancy.

**Upstream references:** Spec §§8, 9, 13; Design §§3, 4, 8.

**Owning module/component:** Governance Context owns Organization; Operations & Support owns deployment metadata.

**In scope:**

- Every governed record can be rooted in Organization. Installed application/schema compatibility, maintenance state, deployment identity, and upgrade metadata remain outside the Organization aggregate. Initial production organization creation is reserved for the provisioning procedure documented by M0-T12.

**Out of scope:**

- Shared-tenant row isolation, billing/subscription implementation, organization settings UI, production organization creation through a development seed script, or customer-specific code forks.

**Dependencies:** Hard prerequisite: M0-T03. Parallelizable related work: M0-T05–T08 after the ownership boundary is agreed.

**Acceptance Criteria**

- Database constraints prevent orphaned organization-scoped records
- exactly one active Organization is supported per pilot deployment without hard-coding a universal identifier
- deployment version changes do not mutate customer governance facts
- development seed and production provisioning are distinct paths.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Integration / operational | Test organization/deployment constraint boundaries, transaction rollback, and denial of cross-root references; verify migration and provisioning fixtures separately. |

---

### M0-T05 — Customer-Controlled Storage Abstraction

**Priority:** P1

**Human-review gate:** Architecture and Security

**Description**

Define and implement the binary/object storage port used for immutable sources, large derived artifacts, backups, and later historical imports without coupling domain code to filesystem paths or a vendor.

**Upstream references:** Spec §§7, 8, 13; Design §§3, 4, 11, 12.

**Owning module/component:** Content & Provenance storage port; Operations & Support supplies the deployment adapter.

**In scope:**

- Generate opaque keys server-side
- retain original filename as metadata only
- persist checksum, size and media type
- stream safely
- prevent traversal/overwrite
- verify durable write before metadata commit
- and support cleanup of unreferenced failed writes.

**Out of scope:**

- User upload workflow, document parsing, lifecycle deletion policy, cloud-vendor selection, or automatic deletion of governed/retained content.

**Dependencies:** Hard prerequisites: M0-T01 and M0-T02. Parallelizable related work: M0-T03, T04 and T06.

**Acceptance Criteria**

- Store/read/delete-of-unreferenced-artifact operations behave consistently through the port
- governed artifacts cannot be overwritten by name collision
- callers never receive a raw storage path
- restart preserves stored data
- failure before metadata commit leaves no referenced corrupt object.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration | Contract-test the selected local adapter with duplicate names, large streamed input, checksum mismatch, traversal strings, interrupted write, unavailable storage, and compensating cleanup. |

---

### M0-T06 — Structured Logging and Correlation

**Priority:** P1

**Human-review gate:** Security and Operations

**Description**

Establish local structured diagnostic logging and correlation across requests, jobs, events, provider calls, and operational procedures while keeping audit separate.

**Upstream references:** Spec §11; Design §§11, 12, 13.

**Owning module/component:** Operations & Support diagnostic observability; consuming modules attach correlation context.

**In scope:**

- Define stable severity, event, correlation, and error fields
- redact secrets and governed content
- carry correlation across API/worker boundaries
- make local rotation/retention configurable through the deployment's operational configuration.

**Out of scope:**

- Remote telemetry requirement, distributed tracing platform, audit-event persistence, raw prompts/transcripts/passages, or fixed retention values not approved through operations/legal policy.

**Dependencies:** Hard prerequisites: M0-T01 and M0-T02. Parallelizable related work: M0-T03–T05 and T08.

**Acceptance Criteria**

- Request-to-worker failures can be traced without source content
- password/session/provider/SMTP/encryption material is absent
- audit events are not derived after the fact from logs
- logging failure does not silently alter domain behavior.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration | Capture representative success/failure logs, assert correlation propagation and redaction, fuzz sensitive field names/values, and verify log output remains parseable during exception paths. |

---

### M0-T07 — Process Health and Dependency Readiness

**Priority:** P1

**Human-review gate:** Operations and Security

**Description**

Provide distinct liveness and readiness signals for safe local operation and later controlled upgrades.

**Upstream references:** Spec §11; Design §§3, 11.

**Owning module/component:** Operations & Support with application bootstrap adapters.

**In scope:**

- `/health` returns success whenever the application process can respond and contains no invented “unrecoverable state.” `/ready` evaluates database connectivity, required extension, application/schema compatibility, storage access, capacity/configuration prerequisites, and maintenance/write-enable state. Anonymous details are non-sensitive.

**Out of scope:**

- Queue/provider/mail status in public liveness, user-facing health navigation, schema migration, or making optional external services a local-core readiness dependency.

**Dependencies:** Hard prerequisites: M0-T02, M0-T03 and M0-T05. Parallelizable related work: M0-T06, T08 and T10.

**Acceptance Criteria**

- Liveness stays successful during a dependency outage while readiness fails with a stable non-secret category
- maintenance or incompatible schema prevents normal write readiness
- restored dependencies recover readiness without process replacement where safe.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Integration / operational | Probe healthy, database-down, storage-down, missing-extension, schema-mismatch, maintenance, low-capacity/configuration-failure, and recovery cases; assert no credentials or topology details leak. |

---

### M0-T08 — Scheduled Backup and Recovery Strategy ADR

**Priority:** P1

**Human-review gate:** Architecture, Operations, and Security

**Description**

Record the selected pilot recovery strategy: verified scheduled whole-deployment backups, RPO no greater than 24 hours, provisional RTO measured by restore, and no WAL/PITR claim.

**Upstream references:** Spec §§11, 14; Design §§11, 15, 16.

**Owning module/component:** Operations & Support; this task owns the recovery ADR and acceptance boundary.

**In scope:**

- Define a consistent recoverable unit of PostgreSQL, referenced files, and sanitized non-secret configuration metadata
- require authenticated encryption, a customer-controlled off-host destination, integrity verification, visible status, and restore testing. Leave exact retention, destination, and key-recovery choices to the deployment and support technical-envelope decision in Design §16 while constraining the schedule to the RPO.

**Out of scope:**

- WAL archiving/PITR claims, arbitrary schedule/retention values, custom cryptography, copying `.env`, or treating an on-host copy as disaster recovery.

**Dependencies:** Hard prerequisites: none. Parallelizable related work: M0-T03–T07; implementation waits for their relevant contracts.

**Acceptance Criteria**

- The approved ADR explicitly rejects unimplemented PITR, identifies accepted up-to-24-hour data loss, defines how RTO is measured, distinguishes backup from disaster recovery, and names every unresolved operational/legal input without choosing it silently.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Integration / operational | Lint runbooks/configuration for forbidden PITR claims and missing recovery-unit components; trace each ADR requirement to M0-T09 and M10 recovery acceptance. |

---

### M0-T09 — Verified Backup and Restore CLI

**Priority:** P1

**Human-review gate:** Operations and Security

**Description**

Implement operator-controlled creation, verification, status inspection, and isolated restoration of the recovery unit defined by M0-T08.

**Upstream references:** Spec §11; Design §§4, 11, 12, 13.

**Owning module/component:** Operations & Support, using PostgreSQL and the M0-T05 storage port.

**In scope:**

- Temporarily prevent writes or use an equivalently proven consistent snapshot boundary
- bind database, file set, compatible application/schema version, checksums, and sanitized metadata in a manifest
- encrypt with an established authenticated-encryption tool
- verify local and off-host writes
- restore only through compatible-version validation into an isolated target. Backup/restore lifecycle events use M0-T10 durable audit.

**Out of scope:**

- PITR/WAL recovery, custom encryption, secret-bearing environment snapshots, destructive in-place restore without validated isolation, or success when only the application-host copy exists.

**Dependencies:** Hard prerequisites: M0-T03, M0-T04, M0-T05, M0-T08 and M0-T10. Parallelizable related work: M0-T11 design may proceed, but upgrade execution depends on this verified backup path.

**Acceptance Criteria**

- A scheduled run can meet the RPO
- incomplete/off-host-missing results are not marked successful
- last-success/failure is queryable
- a full restore reproduces database/file consistency
- mismatch has no generic force path
- measured RTO is recorded as provisional evidence.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Integration / operational | Test consistent capture under attempted writes, tampered manifest/artifact, wrong key, missing file, incompatible version, interrupted transfer, off-host failure, duplicate invocation, isolated restore, and audit-write failure. |

---

### M0-T10 — Audit Event Schema and Durable Audit Writer

**Priority:** P0

**Human-review gate:** Security and Architecture

**Description**

Establish the append-only audit store, centrally registered event taxonomy, least-privilege access, and shared durable `recordAuditEvent()` boundary used by later sensitive operations.

**Upstream references:** Spec §§9, 11; Design §§4, 13.

**Owning module/component:** Audit.

**In scope:**

- Store namespaced event types in TEXT with safe syntax/length only
- typed application constants are the authoritative registry. Require Organization, actor/type, governed context, resource, action/outcome, time, minimized metadata, and prior/new state references where applicable. Runtime may insert but cannot update/delete. Critical operations fail or remain pre-action if audit persistence fails.

**Out of scope:**

- PostgreSQL ENUM or fixed known-values CHECK list, fire-and-forget critical audit, update/delete APIs, after-the-fact derivation from logs, or a separate late “audit everything” task in M1A.

**Dependencies:** Hard prerequisites: M0-T03 and M0-T04. Parallelizable related work: M0-T06–T08.

**Acceptance Criteria**

- Registered events persist atomically with a caller transaction
- unregistered or malformed types fail
- critical durability cannot be downgraded by a caller
- append-only and read-only roles are enforced
- secret/session/source content is rejected or excluded.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration | Test registry validation, mandatory organization, transaction rollback on audit failure, append-only permissions, concurrency, least-privilege reads, and metadata redaction/size boundaries. |

---

### M0-T11 — Controlled Upgrade and Recovery Procedure

**Priority:** P1

**Human-review gate:** Architecture, Operations, and Security

**Description**

Implement and document an explicit, locked, auditable upgrade path that changes schema only during approved version upgrades and has a release-appropriate recovery path.

**Upstream references:** Spec §11; Design §§3, 4, 11, 12, 13.

**Owning module/component:** Operations & Support, invoking the migration and readiness owners.

**In scope:**

- Acquire an exclusive upgrade lock
- preflight source/target compatibility, schema, disk, storage, and configuration
- verify a pre-upgrade backup
- stop every write-capable process
- run the M0-T01-selected migration tool
- start the target in maintenance/non-write mode
- pass `/health` and `/ready`
- update deployment version metadata only after validation
- then enable writes. On failure keep writes disabled until explicit reverse/restore recovery completes and audit state is safe.

**Out of scope:**

- Auto-migration on startup, generic force/continue path, optimistic automatic down migrations, changing version metadata before readiness, or releasing the lock before failure/audit state is recorded.

**Dependencies:** Hard prerequisites: M0-T03, M0-T04, M0-T07, M0-T09 and M0-T10. Parallelizable related work: M0-T12 documentation can begin after the procedure contract stabilizes.

**Acceptance Criteria**

- Concurrent upgrades are rejected
- routine restart never migrates
- failure before readiness preserves previous version metadata
- non-transactional migrations require an approved restore-based plan
- write traffic cannot resume early
- success/failure is durably audited.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Integration / operational | Simulate lock contention, failed preflight, migration interruption, target readiness failure, metadata failure, audit failure, and restore recovery; verify all writers remain stopped and version metadata is truthful. |

---

### M0-T12 — Operator Runbook and Production Provisioning

**Priority:** P2

**Human-review gate:** Operations, Security, and Release

**Description**

Produce the living operator procedure for provisioning, maintenance, safe start/stop, health diagnosis, backup/restore, upgrade/recovery, and least-privilege audit review.

**Upstream references:** Spec §§1, 11, 13; Design §§3, 11, 12, 13, 16.

**Owning module/component:** Operations & Support.

**In scope:**

- Production Organization creation uses the deployment provisioning procedure, never a development seed script. Define maintenance entry/exit, safe shutdown/startup, post-failure recovery and the exact condition for re-enabling writes. Audit inspection uses a read-only least-privilege identity rather than application write credentials. Mark unresolved retention, host, backup, support, and RTO choices as gates.

**Out of scope:**

- Customer credentials in documentation, manual steps that bypass product-owned verification, application write credentials for audit review, invented support tunnel/retention choices, or declaring RTO/PITR capabilities not demonstrated.

**Dependencies:** Hard prerequisites: M0-T07, M0-T09, M0-T10 and M0-T11. Parallelizable related work: none within M0 after final integration begins.

**Acceptance Criteria**

- A qualified operator unfamiliar with the implementation can provision and recover a clean deployment from the runbook
- every command identifies required role and expected observable result
- secrets are never printed/copied
- unsafe or failed states direct the operator to a controlled recovery path.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Integration / operational | Execute a runbook smoke rehearsal in a disposable environment; lint referenced commands/configuration; verify read-only audit credentials cannot mutate data and production provisioning cannot invoke development seeds. |

---

## M1A — Authentication & Authorization Backend

*9 tasks. Hard prerequisites and safe parallel work are stated per task.*

**Critical path:** Follow the hard prerequisites stated per task; work identified as parallelizable may proceed concurrently.

**Purpose:** Deliver the server-side identity, session, MFA, recovery, account-management, bootstrap, and deny-by-default authorization foundations needed before governed data is exposed or mutated.

**Prerequisites and parallelism:** Milestone prerequisites are M0-T03, M0-T04, and M0-T10. After M1A-T01 establishes the minimal body-aware schema, credential and session work proceeds before the dependent endpoint paths. M-UX may begin independently once its M0 prerequisites are met; M2 backend work may begin after M1A-T01 and T04 stabilize without waiting for M1B.

**Exit gate:** Local users can authenticate through PostgreSQL-backed server-side sessions; current database permissions are enforced at the data-access layer; privilege revocation takes effect promptly; privileged roles are unusable until required MFA enrollment; password reset sends through an approved authentication mail path or uses the secure restricted-network administration path, prevents enumeration, and revokes sessions; accounts deactivate without hard deletion; the first Community Manager is bootstrapped as a pending account; every sensitive operation writes its required audit event inline.

---

### M1A-T01 — User, Role, Session, and Permission Schema

**Priority:** P0

**Human-review gate:** Architecture and Security

**Description**

Define the minimal Identity & Session and Authorization persistence model, separating organization-level Community Manager authority from governance-body permission assignments.

**Upstream references:** Spec §§5, 9; Design §§4, 7, 8.

**Owning module/component:** Identity & Session owns accounts/sessions; Authorization owns organization authority and per-body assignments.

**In scope:**

- Model account activation, credentials, sessions, MFA/recovery material, organization authority, predefined per-body permissions, and policy/revocation versioning without collapsing them into persona labels. The governance-body reference is a minimal authorization boundary only
- full body/membership management remains M2.

**Out of scope:**

- Full M2 governance-body CRUD, arbitrary end-user-defined roles, JWT authorization snapshots, fixed sensitive-body exceptions, or a five-persona capability matrix treated as the data model.

**Dependencies:** Hard prerequisites: M0-T03 and M0-T04; M0-T10 is the milestone audit prerequisite. Parallelizable related work: M-UX-T00 and T01.

**Acceptance Criteria**

- Constraints prevent cross-organization assignments, orphaned sessions/roles, and invalid active privilege states
- reviewer and approver are independent grants
- task ownership is not a broad body role
- deactivation and revocation have explicit state rather than generic deletion.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration | Migration and model tests cover valid/invalid organization/body scope, multiple body roles, additive read assignments, independent review/approve grants, duplicate assignment, deactivation, and policy-version changes. |

---

### M1A-T02 — Password Hashing and Credential Storage

**Priority:** P0

**Human-review gate:** Security

**Description**

Implement secure credential primitives and storage boundaries for passwords, session identifiers, TOTP secrets, recovery codes, reset tokens, and bootstrap credentials.

**Upstream references:** Spec §§9, 11; Design §§7, 12.

**Owning module/component:** Identity & Session.

**In scope:**

- Use the security-reviewed password hashing configuration selected under the backend ADR
- store session identifiers, recovery codes, reset/bootstrap tokens only as hashes
- encrypt TOTP secrets at rest with key separation
- compare secrets safely
- never expose existing passwords or recovery material after issuance.

**Out of scope:**

- Inventing password/MFA timeouts or strength thresholds outside their approved source, reversible password storage, plaintext session/recovery codes, custom cryptography, or browser storage decisions.

**Dependencies:** Hard prerequisite: M1A-T01. Parallelizable related work: M-UX foundation and M1A-T04 policy-contract design.

**Acceptance Criteria**

- Plaintext secrets are absent from persistence/logs/audit
- verification succeeds only for valid material
- one-time materials cannot be reused
- key/configuration absence fails safely
- credential replacement invalidates the superseded verifier.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration | Unit-test hash/verify, rotation, one-time consumption, malformed inputs, timing-safe comparisons, encryption/decryption and wrong-key failure; scan fixtures/logs for plaintext secret leakage. |

---

### M1A-T03 — PostgreSQL-Backed Server-Side Session Management

**Priority:** P0

**Human-review gate:** Security and Architecture

**Description**

Implement server-held session lifecycle, renewal/identifier rotation, expiry, revocation, and cookie/CSRF contracts without JWT refresh semantics.

**Upstream references:** Spec §9; Design §§7, 8, 12.

**Owning module/component:** Identity & Session.

**In scope:**

- The browser holds only a Secure, HttpOnly, appropriately SameSite-protected opaque identifier
- PostgreSQL stores its hash and session state. Login/renewal rotates identifiers. Logout, password reset, account deactivation, forced revocation, and relevant permission removal invalidate affected sessions promptly. State-changing requests require CSRF protection.

**Out of scope:**

- JWT access/refresh tokens, authentication tokens in browser storage, permission claims cached as authority in the session, or values for idle/absolute lifetimes chosen outside centralized security configuration.

**Dependencies:** Hard prerequisites: M1A-T01 and M1A-T02. Parallelizable related work: M1A-T04 authorization evaluation.

**Acceptance Criteria**

- A valid cookie authenticates only an active account/session
- old identifiers fail after rotation
- every listed invalidation path is immediate at the next consequential request
- concurrent renewal cannot leave multiple unintended valid identifiers
- sessions survive process restart.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration | Integration-test cookie flags, login/renewal races, expiry, logout, reset/deactivation/role-removal revocation, forged/missing CSRF tokens, database outage, and replay of rotated identifiers. |

---

### M1A-T04 — Deny-by-Default RBAC and Data-Access Authorization

**Priority:** P0

**Human-review gate:** Security and Product/Governance

**Description**

Implement current-state policy evaluation and require it at the data-selection/mutation boundary, with middleware as an optional early rejection rather than the sole control.

**Upstream references:** Spec §§5, 9, 10; Design §8.

**Owning module/component:** Authorization; every governed repository/query consumes its evaluated scope.

**In scope:**

- Derive organization/actor from the session
- evaluate current organization authority, per-body assignment, action, resource, sensitivity and explicit body context
- combine authorized read scopes additively
- fail closed on mismatch or unknown action
- use stable non-enumerating denial reasons. Background/resource-scoped authorization is extended later but cannot bypass this owner.

**Out of scope:**

- UI-only or middleware-only authorization, session/JWT-captured permission authority, permissive default, inferred permissions from persona names, or M2's full membership-management surface.

**Dependencies:** Hard prerequisites: M1A-T01 and M1A-T03. Parallelizable related work: M1A-T02 and M-UX.

**Acceptance Criteria**

- Repository calls cannot return/mutate governed rows without an authorized scope
- Community Manager defaults and explicit exceptions are representable
- reviewer/approver actions are independent
- assignment grants only minimum task context
- revoked permissions are ineffective immediately despite a valid session.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration | Matrix-test cross-organization IDs, multi-body additive reads, explicit-context commands, sensitive exceptions, stale client claims, reviewer-versus-approver separation, task-grant limits, revocation, and denial non-enumeration at middleware and DAL seams. |

---

### M1A-T05 — Login and Logout Endpoints

**Priority:** P1

**Human-review gate:** Security

**Description**

Expose local-account login/logout through the versioned authentication boundary with failed-attempt protection, session issuance/revocation, and inline durable audit.

**Upstream references:** Spec §9; Design §§6, 7, 8, 13.

**Owning module/component:** Identity & Session transport adapter calling the session and Audit owners.

**In scope:**

- Validate credentials without revealing account existence
- apply centrally configured rate limiting and progressive delay/lockout behavior
- establish/rotate a server session only after required authentication steps
- logout revokes the current session. Audit success, failure, throttling and logout with minimized metadata.

**Out of scope:**

- Frontend pages, SSO, JWTs, hard-coded rate/lockout constants, revealing account existence, or deferring login audit to another task.

**Dependencies:** Hard prerequisites: M1A-T02, M1A-T03 and M1A-T04; milestone prerequisite M0-T10 supplies audit. Parallelizable related work: M1A-T06–T08 after shared contracts settle.

**Acceptance Criteria**

- Invalid/deactivated accounts receive indistinguishable public failures
- throttling cannot be bypassed by trivial identifier variations
- successful login has correct cookie/CSRF behavior
- logout is idempotent
- required audit failure prevents a falsely successful security action.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration | API tests cover valid/invalid credentials, deactivated and MFA-gated accounts, rate/backoff boundaries, concurrent logins, cookie attributes, logout/replay, database/audit failure, and response/timing anti-enumeration checks. |

---

### M1A-T06 — Privileged-Role MFA and Recovery Codes

**Priority:** P1

**Human-review gate:** Security and Product/Governance

**Description**

Implement TOTP enrollment/verification and one-time recovery-code lifecycle, enforcing that an MFA-required privilege cannot be used before enrollment.

**Upstream references:** Spec §§9, 14; Design §§7, 8, 12, 13.

**Owning module/component:** Identity & Session enforces authentication strength; Authorization blocks privileged actions/roles until the requirement is met.

**In scope:**

- Support secure enrollment, confirmation before activation, encrypted TOTP secret, hashed single-use recovery codes, regeneration/revocation, failed-attempt protection, and inline audit. The approved privileged-role policy is centrally configured
- exact role mapping and recovery governance remains the privileged MFA and recovery decision in Spec §14, not an implementation choice.

**Out of scope:**

- Silently selecting which roles require MFA, SMS/email OTP, plaintext TOTP/recovery data, frontend flows, or treating “MFA supported” as sufficient when a configured privileged role requires it.

**Dependencies:** Hard prerequisites: M1A-T01, T02, T03 and T04. Parallelizable related work: M1A-T07 and T08.

**Acceptance Criteria**

- A role marked MFA-required is unusable until enrollment succeeds
- a partial/abandoned enrollment grants nothing
- recovery codes work once and invalidate atomically
- reset/regeneration cannot leave old codes valid
- authentication strength is retained server-side and rechecked for privileged operations.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Integration / operational | Test enrollment confirmation, invalid/replayed TOTP, recovery-code race/reuse, role granted before enrollment, policy change while session active, secret-key failure, rate limiting, and durable audit failure. |

---

### M1A-T07 — Password Reset and Restricted-Network Recovery

**Priority:** P1

**Human-review gate:** Security and Operations

**Description**

Deliver the complete backend password-reset lifecycle, including authentication-only email delivery through the approved mail path and a secure administrator-assisted fallback for restricted networks.

**Upstream references:** Spec §§9, 14; Design §§6, 7, 12, 13.

**Owning module/component:** Identity & Session; Notification/mail adapter is used only through its approved authentication contract.

**In scope:**

- Reset requests are anti-enumerating
- tokens are random, hashed, single-use and expiring under centralized configuration
- successful reset replaces the credential and revokes every active session atomically. Administrator-assisted reset requires current authorization, issues a one-time credential, forces password change, exposes no existing password, and audits the action.

**Out of scope:**

- UI pages, general product notifications, insecure manual password setting, returning reset tokens through the public API, choosing SMTP topology, or leaving old sessions active.

**Dependencies:** Hard prerequisites: M1A-T01, T02 and T03; milestone prerequisite M0-T10 supplies audit. Parallelizable related work: M1A-T06 and T08. Production email enablement remains gated by the email-delivery decision in Spec §14.

**Acceptance Criteria**

- Request behavior does not reveal account existence
- approved email dispatch contains no sensitive governance content
- redemption cannot be replayed or race-used
- all prior sessions fail after success
- unavailable email offers only the explicitly authorized local recovery path rather than insecure bypass.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration | Test unknown/deactivated account requests, delivery failure, expired/invalid/reused token, concurrent redemption, password policy failure, session revocation, admin authorization, one-time credential, enumeration resistance, and audit failure. |

---

### M1A-T08 — User Management Endpoints

**Priority:** P1

**Human-review gate:** Security and Product/Governance

**Description**

Provide authorized account creation, activation/deactivation, organization-authority and per-body permission assignment endpoints with current-state enforcement and inline audit.

**Upstream references:** Spec §9; Design §§6, 7, 8, 13.

**Owning module/component:** Identity & Session for accounts; Authorization for grants/revocations.

**In scope:**

- Separate account management from governance-body membership administration while supporting the minimal assignment boundary from T01. Deactivation replaces hard deletion and revokes sessions. Role changes validate explicit Organization/body/action context, enforce independent reviewer/approver grants, update revocation policy state, and persist critical audit in the same success boundary.

**Out of scope:**

- Generic user deletion, full M2 body configuration, arbitrary custom-role builder, invitation UX, automatic CM access exceptions, or a separate deferred auth-audit task.

**Dependencies:** Hard prerequisites: M1A-T01, T03 and T04; milestone prerequisite M0-T10 supplies audit. Parallelizable related work: M1A-T06, T07 and T09.

**Acceptance Criteria**

- Only currently authorized administrators can manage accounts/assignments
- self-escalation and cross-organization changes fail
- deactivation is idempotent and immediately blocks sessions
- revocation is visible to DAL checks
- audit failure rolls back the change.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration | API/security tests cover create, duplicate identity, deactivate/reactivate policy, attempted hard delete, self/peer escalation, sensitive-body exception, per-body role changes, concurrent edits, session invalidation, and audit rollback. |

---

### M1A-T09 — CLI Bootstrap for the First Community Manager

**Priority:** P1

**Human-review gate:** Security and Operations

**Description**

Provide the production provisioning command that creates the first Community Manager as a pending account with a one-time credential and no full privileged access until password change and required MFA enrollment.

**Upstream references:** Spec §9; Design §§3, 7, 8, 11, 13.

**Owning module/component:** Identity & Session and Authorization through an Operations & Support CLI adapter.

**In scope:**

- Require local privileged operator access and an already provisioned Organization
- create at most the intended first administrative authority
- hash the one-time credential
- avoid outputting reusable secrets to logs/audit
- force credential change and the MFA required by the privileged-access decision in Spec §14 before full access
- record bootstrap and completion events inline.

**Out of scope:**

- Development seed scripts in production, immediate fully privileged account, plaintext credential persistence/logging, remote unauthenticated bootstrap, invitation UI, or combining this task back into M1A-T08.

**Dependencies:** Hard prerequisites: M1A-T01, T02 and T04; milestone prerequisite M0-T10 supplies audit. Parallelizable related work: M1A-T06–T08 after shared contracts stabilize.

**Acceptance Criteria**

- Re-running cannot create duplicate first administrators or silently replace credentials
- the pending account cannot perform privileged actions
- one-time credential use transitions safely to required enrollment
- failure at any step leaves no partially privileged account
- bootstrap is distinct from development seeding.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Integration / operational | CLI/integration tests cover missing/multiple Organization, duplicate invocation, interrupted transaction, invalid operator context, one-time credential replay, pre-enrollment privilege denial, successful password/MFA completion, and audit failure. |

---

## M-UX — Frontend Foundation

*9 tasks. Hard prerequisites and safe parallel work are stated per task.*

**Critical path:** Follow the hard prerequisites stated per task; work identified as parallelizable may proceed concurrently.

**Purpose:** Establish the approved browser architecture and reusable Hebrew-first, RTL/BiDi-correct, responsive, accessible interaction foundations that all subsequent UI milestones consume. M-UX supplies shells and primitives; it does not implement the real authentication flows owned by M1B.

**Prerequisites and parallelism:** M-UX-T00 has no hard prerequisite and may run immediately in parallel with backend work. After its decision, the visual/component path (T01–T07) and API/data path (T08) proceed in parallel, consuming the relevant M0 contracts as stated per task. M1A and M2 backend work remain independent of this frontend decision; no subsequent product UI should bypass these foundations.

**Exit gate:** The frontend technology and Hebrew/RTL realization decision in Design §16 has explicit Architecture and UX approval; the component foundation renders and operates in Hebrew RTL, preserves mixed-direction content, adapts to supported responsive sizes, supports keyboard-only operation with visible predictable focus, exposes perceivable labels and states, and remains understandable and operable with assistive technologies; common form/error/empty/toast behaviors are consistent; the application shell and AuthLayout are reusable; the API client uses session cookies and CSRF, never browser-stored auth tokens, and distinguishes 401 from 403/access revocation.

---

### M-UX-T00 — Frontend Architecture ADR

**Priority:** P0

**Human-review gate:** Architecture and UX/Accessibility

**Description**

Resolve the frontend technology and Hebrew/RTL realization decision in Design §16 and record the browser language/framework, build, component/testing, styling, state/data-fetching, localization, RTL/BiDi, and accessibility approach before frontend scaffolding.

**Upstream references:** Spec §§11, 13; Design §§2, 6, 10, 16.

**Owning module/component:** Frontend architecture across the browser client; this task owns an ADR, not feature screens.

**In scope:**

- Evaluate choices against Hebrew-first rendering, accessible component behavior, responsive review workflows, server-session/CSRF compatibility, maintainability by the implementation team, and local deployment. Define frontend ownership boundaries and the standard verification commands.

**Out of scope:**

- Backend stack changes, actual login/MFA/reset screens, product-feature routes, provider/SMTP choices, design-provider lock-in, or adopting a library without a Hebrew/RTL/accessibility validation path.

**Dependencies:** Hard prerequisites: none. Parallelizable related work: M0-T01 and M0-T07 contract work, M1A backend implementation, and M2 backend implementation.

**Acceptance Criteria**

- The approved ADR records alternatives/trade-offs and fixes conventions for routing, component composition, state/data ownership, styling/tokens, testing, i18n/RTL and accessibility
- it preserves versioned REST and leaves product screens to their milestones.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration / E2E | Validate the ADR against a required-decision checklist; after selection, run the documented clean install/build/type/lint/unit/browser-smoke commands in the selected toolchain. |

---

### M-UX-T01 — Design Tokens and Global Styles

**Priority:** P0

**Human-review gate:** UX/Accessibility

**Description**

Establish semantic design tokens and global document behavior for Hebrew RTL, mixed-direction data, responsive layout, focus, motion, contrast, typography, spacing, and status communication.

**Upstream references:** Spec §11; Design §§10, 12, 16.

**Owning module/component:** Frontend foundation.

**In scope:**

- Use logical direction-aware properties
- isolate LTR identifiers, timestamps, email, numbers and source snippets safely
- keep focus visibly discernible
- support reduced motion
- define breakpoints and density semantically rather than per-screen magic values.

**Out of scope:**

- Product-specific page styling, arbitrary per-component breakpoints, fixed left/right assumptions, provider-specific branding, or silently choosing content truncation that changes a later product experience.

**Dependencies:** Hard prerequisite: M-UX-T00. Parallelizable related work: M-UX-T08.

**Acceptance Criteria**

- Representative Hebrew and mixed-direction content renders without reversed identifiers or broken reading order
- color/status meaning is not color-only
- focus and contrast meet the approved accessibility bar
- responsive tokens support desktop split and sequential small-screen experiences.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration / E2E | Run style/token linting, contrast checks, RTL/LTR visual fixtures, long-text/zoom/reduced-motion snapshots, and browser checks at supported viewport boundaries. |

---

### M-UX-T02 — Primitive Component Library

**Priority:** P0

**Human-review gate:** UX/Accessibility

**Description**

Provide the small reusable set of interactive primitives required by forms, navigation, review cards, dialogs, status indicators, tables/lists, and later workflows.

**Upstream references:** Spec §11; Design §§10, 12.

**Owning module/component:** Frontend foundation.

**In scope:**

- Primitives expose semantic states, labels, focus behavior, keyboard interaction, disabled/busy semantics, and direction-safe layout without embedding domain rules. Prefer native semantics and simple composition over a parallel design framework.

**Out of scope:**

- Domain-specific auth or governance logic, a full screen library, inaccessible custom replacements for native controls, arbitrary token duplication, or components that call APIs directly.

**Dependencies:** Hard prerequisite: M-UX-T01. Parallelizable related work: M-UX-T08.

**Acceptance Criteria**

- Each primitive has an accessible name/role/state, keyboard behavior, focus restoration where applicable, Hebrew and mixed-direction examples, and documented composition rules
- disabled and loading states do not cause duplicate actions.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration / E2E | Component tests cover keyboard/pointer interaction, focus trapping/restoration, ARIA semantics, RTL, zoom, reduced motion, long labels, loading/disabled transitions, and automated accessibility scanning. |

---

### M-UX-T03 — Application Shell and Routing Structure

**Priority:** P1

**Human-review gate:** UX/Accessibility and Architecture

**Description**

Build the authenticated application shell and routing/error-boundary structure that preserves navigation context and can later display explicit governance-body/action context.

**Upstream references:** Spec §§5, 9, 11; Design §§6, 8, 10.

**Owning module/component:** Frontend application shell.

**In scope:**

- Define public/authenticated route boundaries, skip/navigation landmarks, responsive shell regions, loading transitions, route-level recovery, and a stable place for active-body/role/action visibility. Treat client route guards as UX only
- server authorization remains authoritative.

**Out of scope:**

- Product dashboard/navigation inventory, actual login flow, authorization enforced only in client routing, user-facing `/health` link, or a mandatory governance-body selector for read-only views.

**Dependencies:** Hard prerequisites: M-UX-T01 and M-UX-T02. Parallelizable related work: M-UX-T05–T08.

**Acceptance Criteria**

- Direct navigation, refresh and back/forward restore an allowed route coherently
- unknown/error routes recover accessibly
- keyboard focus moves predictably after navigation
- Hebrew RTL order remains correct
- forbidden/revoked results cannot be mistaken for unauthenticated state.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Integration / operational | Browser tests cover direct/deep links, refresh, history navigation, responsive shell, keyboard landmarks/focus, route error recovery, and placeholder handling of normalized 401/403/revoked outcomes. |

---

### M-UX-T04 — AuthLayout Component

**Priority:** P1

**Human-review gate:** UX/Accessibility

**Description**

Provide the shared accessible layout wrapper for login, MFA, recovery-code, enrollment, password-reset, and restricted-access authentication screens implemented in M1B.

**Upstream references:** Spec §§9, 11; Design §§7, 10.

**Owning module/component:** Frontend foundation.

**In scope:**

- Supply consistent heading/help/error/status regions, responsive Hebrew RTL layout, safe mixed-direction credential display, focus placement, and space for recovery/support guidance. It contains no authentication state machine or endpoint calls.

**Out of scope:**

- Real login/MFA/enrollment/reset behavior, password policy decisions, authentication API calls, user-facing health navigation, or product dashboard shell.

**Dependencies:** Hard prerequisites: M-UX-T01 and M-UX-T02. Parallelizable related work: M-UX-T03 and T05–T08.

**Acceptance Criteria**

- All intended auth content variants fit without clipping or reordering
- errors are announced and do not move focus unpredictably
- sensitive values are not exposed through decorative/help regions
- consumers can compose flows without overriding foundation accessibility.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration / E2E | Component/visual tests cover Hebrew, long validation copy, mixed-direction usernames/codes, small screens, zoom, keyboard focus, screen-reader landmarks, and empty/loading/error slots. |

---

### M-UX-T05 — Composed Form Components

**Priority:** P1

**Human-review gate:** UX/Accessibility

**Description**

Compose reusable field, validation-summary, action, secret/code-entry, confirmation, and async-submission patterns for M1B and later governed forms.

**Upstream references:** Spec §§9, 11; Design §§7, 10, 12.

**Owning module/component:** Frontend foundation.

**In scope:**

- Associate labels/help/errors correctly
- preserve user input on recoverable failure
- prevent accidental duplicate submission
- distinguish client hints from server authority
- support RTL labels with LTR identifiers/codes
- restore focus to the actionable error or confirmation.

**Out of scope:**

- Product-specific validation rules, API endpoints, authentication state machines, hidden security decisions, or components that silently discard server errors/user input.

**Dependencies:** Hard prerequisites: M-UX-T01 and M-UX-T02. Parallelizable related work: M-UX-T03, T04, T06–T08.

**Acceptance Criteria**

- Components expose idle, validating, submitting, success, field-error, form-error and disabled states
- errors are programmatically associated
- paste/autofill/password-manager use is not blocked without reason
- server validation maps predictably without leaking restricted facts.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration / E2E | Test keyboard submission, duplicate clicks, async race/out-of-order responses, server/client errors, focus/error announcement, RTL/LTR entry, paste/autofill, and unmount/cancellation behavior. |

---

### M-UX-T06 — Toast Notification System

**Priority:** P2

**Human-review gate:** UX/Accessibility

**Description**

Provide an accessible transient-feedback mechanism for non-critical confirmations and recoverable errors without replacing durable notification or form/status UI.

**Upstream references:** Spec §§7, 11; Design §§6, 10.

**Owning module/component:** Frontend foundation; Notification Delivery remains the later domain owner for in-app/email records.

**In scope:**

- Support semantic severity, announcement, dismissal, deduplication, reduced motion, and RTL placement. Critical authentication, access-revocation, destructive confirmation, processing state, and durable inbox events must remain in persistent UI.

**Out of scope:**

- Durable notification inbox, email delivery, domain-event policy, authentication failure as toast-only UI, or invented timeout values outside the centralized token/config owner.

**Dependencies:** Hard prerequisite: M-UX-T02. Parallelizable related work: M-UX-T03–T05, T07 and T08.

**Acceptance Criteria**

- Toasts are keyboard/screen-reader accessible, do not steal focus, do not obscure primary actions at supported sizes, and avoid repeated announcements for the same event
- transient disappearance never removes the only copy of required guidance.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration / E2E | Test severity announcements, queue/deduplication, timeout pause/dismiss, reduced motion, RTL/small-screen placement, burst events, unmount/navigation, and persistent-error fallback. |

---

### M-UX-T07 — Empty and Error State Components

**Priority:** P2

**Human-review gate:** UX/Accessibility

**Description**

Provide reusable, actionable empty, forbidden, access-revoked, unavailable, not-found, recoverable-error, and terminal-error presentations.

**Upstream references:** Spec §§7, 11; Design §§10, 12.

**Owning module/component:** Frontend foundation.

**In scope:**

- Keep 401, 403, revoked, validation, conflict, transient dependency, and terminal processing outcomes distinct
- avoid confirming inaccessible resource existence
- include retry/support/navigation actions only when valid
- retain stable focus and context.

**Out of scope:**

- Backend error semantics, invented support contact/topology, toast-only critical errors, user-facing raw stack/provider errors, or product-specific empty-state actions.

**Dependencies:** Hard prerequisite: M-UX-T02. Parallelizable related work: M-UX-T03–T06 and T08.

**Acceptance Criteria**

- Each normalized outcome maps to one coherent persistent state
- no state relies on color/icon alone
- retry is absent for permanent denial and safe/idempotent where offered
- Hebrew copy slots handle long text and mixed-direction references.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration / E2E | Component tests cover each state, accessibility semantics, retry success/failure, repeated errors, focus behavior, responsive/RTL layouts, and resource-enumeration-safe copy. |

---

### M-UX-T08 — API Client and Data-Fetching Foundation

**Priority:** P1

**Human-review gate:** Architecture and Security

**Description**

Implement the shared browser data boundary for versioned REST, server-session cookies, CSRF, normalized errors, cancellation, and later authenticated SSE/REST status consumption.

**Upstream references:** Spec §9; Design §§6, 7, 8, 12.

**Owning module/component:** Frontend data-access foundation.

**In scope:**

- Send same-origin credentials through the selected cookie contract
- acquire/refresh CSRF material through the approved server flow
- never persist authentication tokens in localStorage/sessionStorage
- normalize 401 separately from 403 and access revocation
- support abort/stale-response handling and correlation identifiers without logging sensitive bodies.

**Out of scope:**

- Endpoint-specific product features, GraphQL/WebSockets, JWT refresh logic, auth tokens in browser storage, permission decisions from cached UI data, or choosing the REST API conventions in Design §16 without approval.

**Dependencies:** Hard prerequisite: M-UX-T00. Parallelizable related work: M-UX-T01–T07; endpoint-specific clients wait for the relevant backend contracts.

**Acceptance Criteria**

- Requests use the approved REST/error conventions
- one shared policy handles authentication expiry/revocation without redirect loops
- mutations cannot proceed without valid CSRF
- canceled or out-of-order responses do not overwrite newer UI state
- no auth token appears in browser storage.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration / E2E | Contract-test success/validation/401/403/revoked/conflict/transient outcomes, CSRF failure/renewal, cookie behavior, cancellation and race ordering, network loss/retry eligibility, correlation propagation, and browser-storage inspection. |

---

## M1B — Authentication UI

*6 tasks. Hard prerequisites and safe parallel work are stated per task.*

**Critical path:** Follow the hard prerequisites stated per task; work identified as parallelizable may proceed concurrently.

**Purpose:** Deliver the user-facing local authentication, MFA/recovery, password-reset, account/role administration, and cross-flow failure experiences on top of M1A and the approved frontend foundation.

**Prerequisites and parallelism:** Milestone prerequisites are M-UX-T04, M-UX-T05, M-UX-T08, M1A-T05, M1A-T06, M1A-T07, and M1A-T08. M1B-T02 and T03 proceed in parallel after T01. T04 can proceed directly from the shared layout/form/API contracts. T05 integrates all auth paths; T06 follows the shared shell and backend management contracts without blocking M2 backend work.

**Exit gate:** Real local users can log in, complete required MFA or use a one-time recovery code, enroll MFA before privileged-role use, request/redeem a password reset without account enumeration, recover coherently from deactivation/session expiry/revocation/lockout, and—when authorized—create/deactivate users and manage per-body assignments. Hebrew RTL, responsive, keyboard, and accessibility verification passes; browser storage contains no authentication token; every security decision remains server-enforced.

---

### M1B-T01 — Login Page and Session Flow

**Priority:** P0

**Human-review gate:** Security and UX/Accessibility

**Description**

Implement the Hebrew-first login experience and post-authentication routing against M1A's server-side session endpoints.

**Upstream references:** Spec §§9, 11; Design §§6, 7, 10, 12.

**Owning module/component:** Authentication UI using AuthLayout, composed forms, and the API client.

**In scope:**

- Accept email/username and password
- preserve anti-enumerating server responses
- show submitting and configured throttling/lockout states
- follow server-directed MFA/enrollment/password-change requirements
- rotate into the authenticated session without exposing an auth token
- redirect only to an authorized safe destination.

**Out of scope:**

- Client-side authorization, JWT/localStorage/sessionStorage tokens, SSO, exposing raw lockout/account facts beyond the approved response, or implementing MFA/reset logic inside the login component.

**Dependencies:** Hard prerequisites: M-UX-T04, M-UX-T05 and M-UX-T08; M1A-T05 is the backend milestone prerequisite. Parallelizable related work: M1B-T04 can proceed independently; T02/T03 follow this shared session flow.

**Acceptance Criteria**

- Successful login reaches an authorized route with a cookie-backed session
- invalid/deactivated/unknown accounts do not reveal existence
- repeated submission cannot create races
- unsafe return URLs are rejected
- keyboard/screen-reader/RTL/responsive behavior is coherent.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration / E2E | Browser/API contract tests cover success, invalid credentials, throttling/lockout, deactivation, MFA/enrollment/password-change branch, network/audit failure, double submit, safe redirect, focus/error announcement, and absence of auth tokens in storage/URL/logs. |

---

### M1B-T02 — MFA Verification and Recovery-Code Flow

**Priority:** P1

**Human-review gate:** Security and UX/Accessibility

**Description**

Implement the second-factor challenge experience, including one-time recovery-code use, for sessions that M1A marks as requiring verification.

**Upstream references:** Spec §§9, 11; Design §§7, 10, 12.

**Owning module/component:** Authentication UI; M1A-T06 remains authoritative for verification, rate limits, code use, and session strength.

**In scope:**

- Present TOTP and recovery-code modes without revealing stored material
- support safe code paste/mixed direction
- handle expired/replayed/failed challenges and rate limiting
- continue only on the server's strengthened session
- provide approved recovery guidance without inventing policy.

**Out of scope:**

- Client-side TOTP validation, displaying stored secrets/codes, SMS/email OTP, choosing the privileged MFA and recovery policy in Spec §14, or treating the UI challenge as the privilege enforcement point.

**Dependencies:** Hard prerequisite: M1B-T01; backend prerequisite M1A-T06. Parallelizable related work: M1B-T03.

**Acceptance Criteria**

- Valid TOTP or unused recovery code completes authentication
- reuse and out-of-order responses fail safely
- failure does not create an authenticated route
- recovery-code consumption is clearly confirmed without displaying remaining codes
- accessibility and RTL checks pass.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Integration / operational | Browser/contract tests cover valid/invalid/expired/replayed codes, recovery-code race, throttling, navigation/back/refresh, session expiry mid-challenge, network recovery, paste/autofill, focus announcements, and server-enforced privilege denial. |

---

### M1B-T03 — MFA Enrollment Flow

**Priority:** P1

**Human-review gate:** Security and UX/Accessibility

**Description**

Implement enrollment, confirmation, recovery-code presentation/acknowledgement, and blocked-privilege guidance for users whose role requires MFA.

**Upstream references:** Spec §§9, 11, 14; Design §§7, 10, 12.

**Owning module/component:** Authentication UI; M1A-T06 owns the enrollment state and privilege gate.

**In scope:**

- Display the server-provided enrollment material only in the intended authenticated flow
- require a successful TOTP confirmation before activation
- present newly issued recovery codes once with safe copy/download guidance and explicit acknowledgement
- recover from abandoned/expired setup without granting privilege.

**Out of scope:**

- Backend enforcement in the UI, storing TOTP/recovery material, silently marking enrollment complete, choosing which roles require MFA, or placing this real flow in M-UX.

**Dependencies:** Hard prerequisite: M1B-T01; backend prerequisite M1A-T06. Parallelizable related work: M1B-T02.

**Acceptance Criteria**

- A user with required-but-unenrolled MFA is routed to enrollment and cannot use the privileged role
- completed enrollment enables the server-authorized path
- refresh/back/duplicate confirmation cannot reactivate old material
- recovery codes are not retained in client storage after the flow.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Integration / operational | Browser tests cover initial/expired enrollment, invalid/valid confirmation, abandon/resume, refresh/back, recovery-code acknowledgement, screen capture-safe rendering considerations, small-screen/RTL/keyboard accessibility, and pre/post-enrollment server authorization. |

---

### M1B-T04 — Password Reset Flow

**Priority:** P1

**Human-review gate:** Security and UX/Accessibility

**Description**

Implement password-reset request and token-redemption screens plus clear restricted-network/admin-assisted recovery guidance.

**Upstream references:** Spec §§9, 11, 14; Design §§6, 7, 10, 12.

**Owning module/component:** Authentication UI using M1A-T07.

**In scope:**

- The request screen always gives an anti-enumerating outcome
- redemption handles invalid/expired/used tokens without exposing account state
- successful reset communicates that sessions were revoked and returns the user to authentication. Admin-assisted recovery is described only through the approved local path and never asks an administrator to discover/set the existing password.

**Out of scope:**

- General notification preferences, choosing SMTP provider/topology, returning reset tokens in API responses, insecure admin password assignment, or client-only password/session invalidation.

**Dependencies:** Hard prerequisites: M-UX-T04, M-UX-T05 and M-UX-T08; backend prerequisite M1A-T07. Parallelizable related work: M1B-T01–T03.

**Acceptance Criteria**

- Known and unknown account requests are publicly indistinguishable
- reset links/tokens do not leak into logs or browser storage
- password update follows server validation
- success invalidates old sessions and token replay
- mail unavailability yields approved actionable guidance rather than false success claims about delivery.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration / E2E | Browser/contract tests cover known/unknown/deactivated requests, delivery unavailable, malformed/expired/reused token, validation errors, double submit/race, post-reset old-session denial, navigation history, RTL/keyboard/accessibility, and URL/log/storage leakage. |

---

### M1B-T05 — Authentication Error and Cross-Flow Edge States

**Priority:** P2

**Human-review gate:** Security and UX/Accessibility

**Description**

Integrate persistent user experiences for deactivated accounts, session expiry, forced revocation, permission loss, lockout/backoff, CSRF failure, and interrupted auth flows across T01–T04.

**Upstream references:** Spec §§9, 11; Design §§6, 7, 10, 12.

**Owning module/component:** Authentication UI integration using M-UX error states and API normalization.

**In scope:**

- Keep unauthenticated, forbidden, revoked, throttled, validation, transient dependency, and terminal outcomes distinct
- clear sensitive transient UI state
- preserve safe return context when appropriate
- avoid redirect loops and resource/account enumeration
- never attempt to override server decisions.

**Out of scope:**

- New backend status codes, client-side security workarounds, toast-only critical states, leaking resource/account existence, or changing approved server policy to simplify UI.

**Dependencies:** Hard prerequisites: M1B-T01, T02, T03 and T04. Parallelizable related work: M1B-T06 can start after its own prerequisites but exit waits for shared integration.

**Acceptance Criteria**

- Every cross-flow event reaches one deterministic accessible state
- revocation during an active flow stops further protected actions
- CSRF/session recovery does not repeat unsafe mutations
- lockout messaging follows server policy
- recovery/back navigation is coherent on desktop and small screens.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration / E2E | End-to-end tests inject expiry/revocation/deactivation/permission removal at each flow step, plus CSRF failure, network interruption, duplicate tabs, stale responses, lockout, browser back/refresh, keyboard/focus, and RTL/accessibility checks. |

---

### M1B-T06 — User Management and Per-Body Role Assignment UI

**Priority:** P1

**Human-review gate:** Security, Product/Governance, and UX/Accessibility

**Description**

Provide the authorized administration UI for account creation/deactivation, organization authority, per-governance-body assignments, independent reviewer/approver permissions, and privileged-role MFA status.

**Upstream references:** Spec §§5, 9, 11; Design §§6, 8, 10, 12.

**Owning module/component:** Administration UI calling M1A-T08; Identity & Session and Authorization remain authoritative.

**In scope:**

- Clearly separate organization-level Community Manager/delegated authority from per-body grants
- require explicit body context for body-permission changes
- display current MFA readiness before an MFA-required privilege can be used
- deactivate rather than delete
- show concurrency/conflict/revocation results without applying optimistic unauthorized state.

**Out of scope:**

- Hard user deletion, arbitrary role designer, full M2 governance-body management, invitations as a substitute for account creation, client-authoritative permissions, or making Community Manager exceptions implicitly.

**Dependencies:** Hard prerequisites: M1B-T01, M-UX-T05 and M-UX-T08; backend prerequisite M1A-T08. M1B-T02–T05 are parallelizable related work, with full milestone integration required at exit.

**Acceptance Criteria**

- Authorized admins can create/deactivate users and manage supported assignments
- sensitive-body exceptions and insufficient authority fail closed
- reviewer and approver remain independent
- a newly required privileged role shows blocked-until-enrolled state
- audit-backed failures leave the displayed state unchanged.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration / E2E | Browser/security tests cover organization versus body authority, multi-body assignments, reviewer/approver separation, self-escalation, cross-organization IDs, sensitive exceptions, concurrent edits, deactivation/session revocation, MFA-required status, 401/403/revoked outcomes, and Hebrew/RTL/accessibility. |

---

## M-OPS — Operational Tooling

*7 tasks. Hard prerequisites and safe parallel work are stated per task.*

**Critical path:** Follow the hard prerequisites stated per task; work identified as parallelizable may proceed concurrently.

**Purpose:** Give the customer administrator a safe local view of deployment condition and a customer-controlled support path. This milestone consumes M0 logging, health/readiness, backup, and audit foundations; it does not rebuild them.

**Prerequisites and parallelism:** M0-T06, M0-T07, M0-T09, and M0-T10 provide the operational sources. M-UX-T03 provides the UI shell. Backend status aggregation and support-access work may proceed in parallel after their direct prerequisites. Metrics may proceed independently of the support flow.

**Exit gate:** A customer administrator can inspect deployment and backup status, create and revoke a time-limited diagnostic support session, produce an encrypted sanitized bundle, and perform the same essential recovery actions from a local CLI. All support lifecycle actions are durably audited, no standing support access exists, and the milestone has not introduced a second health, backup, or log system.

---

### M-OPS-T01 — Operator Status Dashboard (Backend)

**Priority:** P0

**Human-review gate:** Required — operations and security review of every exposed field and authorization rule.

**Description**

Provide one read-only operator query that aggregates the existing M0 health/readiness and backup state without duplicating their implementation.

**Upstream references:** Spec §§9, 11; Design §§8, 11, 13.

**Owning module/component:** Operations & Support, consuming M0 operational interfaces and Audit read contracts.

**In scope:**

- Return current process/readiness condition, installed and schema compatibility state, storage/disk condition, last completed backup and off-host outcome, restore-test status, and active support-session summary where available.
- Read authoritative M0 status records and checks; do not reimplement probes, backup commands, log ingestion, or audit persistence.
- Restrict the query to an authenticated customer operator/administrator. Suppress secrets, paths, connection strings, raw logs, customer content, and unnecessary identifiers.
- Distinguish unavailable, stale, failed, and never-run status so absence is not reported as success.

**Out of scope:**

- New health probes, backup execution, log aggregation, remote monitoring platforms, governance dashboards, or write operations.

**Dependencies:** Hard prerequisites: M0-T07, M0-T09, M0-T10. Parallelizable related work: M-OPS-T03 and M-OPS-T07.

**Acceptance Criteria**

- Authorized operators receive a timestamped aggregation whose values agree with the underlying M0 sources
- unauthorized users are denied at the data-access boundary
- stale or missing evidence is explicit
- no sensitive value is returned.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Integration / operational | Unit-test aggregation and status precedence; integration-test each unavailable dependency and authorization denial; run secret/path fixtures through response inspection; compare representative responses with the underlying health, backup, and audit records. |

---

### M-OPS-T02 — Operator Status Dashboard (UI)

**Priority:** P1

**Human-review gate:** Required — customer-operator UX, Hebrew/RTL, accessibility, and sensitive-detail review.

**Description**

Present M-OPS-T01 as an actionable local operator view that clearly separates healthy, degraded, failed, stale, and unknown conditions.

**Upstream references:** Spec §11; Design §§10, 11.

**Owning module/component:** Operations & Support UI using the shared application shell.

**In scope:**

- Show readiness, backup/off-host status, restore-test evidence, storage capacity, version compatibility, and active support sessions with last-updated times and safe next-action guidance.
- Preserve distinct loading, partial-data, authorization-revoked, and backend-unavailable states; never turn an unknown status into a green state.
- Use Hebrew-first RTL/BiDi and keyboard-accessible status semantics. Do not expose the public liveness endpoint as a user support action.
- Link only to operator actions that the current user is authorized to perform.

**Out of scope:**

- Recreating health logic in the browser, editing operational configuration, a user-facing `/health` link, or the Community Manager governance dashboard.

**Dependencies:** Hard prerequisites: M-OPS-T01, M-UX-T03. Parallelizable related work: M-OPS-T04 and M-OPS-T05 after their backends exist.

**Acceptance Criteria**

- Every backend status has a perceivable, accessible UI representation
- stale/unknown evidence is visible
- forbidden data is never rendered
- refresh preserves context and handles partial failure.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration / E2E | Component-test every status and empty/error state; integration-test 401, 403/access-revoked, partial aggregation, and refresh; verify RTL, focus order, keyboard navigation, and accessible labels at supported responsive sizes. |

---

### M-OPS-T03 — Support Access — Request & Approval Flow

**Priority:** P1

**Human-review gate:** Required — security/authorization approval and customer validation of scope wording.

**Description**

Let an authorized customer administrator initiate a narrowly scoped, time-limited support session with diagnostic-only access by default.

**Upstream references:** Spec §§9, 11; Design §§8, 11, 13.

**Owning module/component:** Operations & Support; Authorization evaluates the customer approver and requested support scope.

**In scope:**

- Create a pending request that identifies the customer initiator, requested diagnostic purpose, requested duration from approved configuration, and support recipient identity.
- Default to diagnostic metadata only. Content access and change authority are separate explicit scopes; granting either requires a distinct informed approval and does not imply the other.
- Issue no standing customer credential, permanent tunnel, or general application account. A request that cannot be bound to a current customer approval fails closed.
- Persist request, approval/denial, scope, start/expiry, and actor through guaranteed audit before access becomes usable.

**Out of scope:**

- Standing credentials, permanent VPN/tunnel access, implicit content access, provider selection, or a generic remote administration account.

**Dependencies:** Hard prerequisites: M1A-T04, M0-T10. Parallelizable related work: M-OPS-T01 and M-OPS-T05.

**Acceptance Criteria**

- Only an authorized customer administrator can approve
- the default grant is diagnostic-only
- content/change scopes remain separately visible and approved
- no access exists before approval and durable audit
- repeated or stale approvals cannot create duplicate sessions.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration | Test unauthorized initiation/approval, expired approval, duplicate submission, cross-organization identifiers, audit failure, and each scope combination; verify the support identity receives only the granted capabilities. |

---

### M-OPS-T04 — Support Access — Expiry, Revocation & Audit

**Priority:** P1

**Human-review gate:** Required — adversarial security review of expiry, revocation, and active-session enforcement.

**Description**

Enforce support-session lifetime and immediate customer revocation at every support operation.

**Upstream references:** Spec §§9, 11; Design §§8, 11, 13.

**Owning module/component:** Operations & Support with Authorization and Audit.

**In scope:**

- Evaluate session active state, expiry, revocation, support identity, and granted scope from current database state for every operation; a cached grant is not authority.
- Allow an authorized customer administrator to revoke before expiry and ensure revocation takes effect for subsequent requests and active channels.
- Record requested, approved, started, scope-changed, revoked, expired, denied, and attempted-after-revocation/expiry events with minimized metadata.
- Guarantee audit persistence for approval, scope escalation, use of content/change scope, and revocation; the protected operation fails if its required audit event cannot persist.

**Out of scope:**

- Renewal without fresh customer approval, silent scope escalation, indefinite sessions, or relying only on UI hiding.

**Dependencies:** Hard prerequisites: M-OPS-T03, M-OPS-T01. Parallelizable related work: M-OPS-T05.

**Acceptance Criteria**

- Expired or revoked sessions cannot perform another operation
- diagnostic access cannot mutate or read content
- revocation is available locally when external support infrastructure is unavailable
- every lifecycle transition is queryable in Audit.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration / E2E | Exercise exact expiry boundaries with a controllable clock, concurrent revoke/use races, stale browser/channel state, audit-store failure, and attempts outside scope; verify no forbidden content appears in denial responses. |

---

### M-OPS-T05 — Encrypted Diagnostic Bundle

**Priority:** P1

**Human-review gate:** Required — security review of the allow-list, redaction evidence, and selected authenticated-encryption mechanism.

**Description**

Produce a customer-controlled diagnostic artifact that is useful for support without copying secrets or governed content.

**Upstream references:** Spec §§9, 11; Design §§11, 12, 13.

**Owning module/component:** Operations & Support, consuming structured logs, status, deployment metadata, and permission-scoped audit summaries.

**In scope:**

- Build from an explicit allow-list: sanitized diagnostic logs, health/readiness summary, installed/schema version, resource status, backup status, and minimized support/audit summary.
- Exclude credentials, session/cookie values, encryption material, environment files, source documents, transcripts, prompts, AI results, and raw governance records. Mask personal identifiers not required for diagnosis.
- Encrypt the finished artifact with an established authenticated-encryption tool approved for the deployment; do not implement cryptography. Delete intermediate plaintext through the selected platform's safe temporary-file procedure.
- Require a current authorized customer operation and record creation/download outcome without logging the bundle key or contents.

**Out of scope:**

- Database dumps, source-file export, `.env` capture, custom encryption, automatic remote upload, or rebuilding M0 logs/backups.

**Dependencies:** Hard prerequisites: M-OPS-T03, M0-T06. Parallelizable related work: M-OPS-T04. This task blocks M10-T08.

**Acceptance Criteria**

- Known secret/content fixtures do not appear in plaintext or decrypted output outside the explicit allow-list
- tampering is detected
- failed generation leaves no usable partial artifact
- bundle metadata identifies version and collection time.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration | Run secret/PII canary fixtures, malformed-log inputs, encryption failure, disk-full, cancellation, and interrupted generation; decrypt with the intended recipient procedure and inspect the manifest; verify temporary artifacts and logs contain no sensitive values. |

---

### M-OPS-T06 — Local CLI Fallback

**Priority:** P1

**Human-review gate:** Required — operator runbook and least-privilege security review.

**Description**

Preserve essential status and support-session control when the web operator UI is unavailable.

**Upstream references:** Spec §§9, 11; Design §§8, 11, 12, 13.

**Owning module/component:** Operations & Support CLI using the same application services as the web flow.

**In scope:**

- Query the M-OPS-T01 status contract and list, revoke, or inspect support sessions through M-OPS-T04; do not duplicate business rules in shell scripts.
- Authenticate the local operator through the deployment-approved OS/administrative boundary and use least-privilege database/application credentials.
- Produce machine-readable and human-readable output without secrets, raw content, or connection details. Destructive or privileged actions require explicit target and confirmation appropriate to the selected CLI conventions.
- Audit support lifecycle actions identically to the web path.

**Out of scope:**

- Direct table edits, bypass credentials, a second support-session implementation, backup/restore commands already owned by M0, or interactive exposure of secrets.

**Dependencies:** Hard prerequisites: M-OPS-T01, M-OPS-T04. Parallelizable related work: none after these contracts stabilize.

**Acceptance Criteria**

- Status and revocation work while the browser UI is unavailable
- CLI and web enforce identical authorization, scope, expiry, and audit behavior
- invalid or ambiguous targets fail without changing state.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Integration / operational | Test command parsing, unavailable API/database paths, unauthorized OS identity, ambiguous identifiers, expired/revoked sessions, and audit failure; compare CLI results with service-level queries and the operator runbook. |

---

### M-OPS-T07 — Lightweight Metrics & Alerting

**Priority:** P2

**Human-review gate:** Required — operations review of signal usefulness, threshold ownership, and data minimization.

**Description**

Surface pilot-critical operational failures with local, lightweight metrics and alerts built from existing structured signals.

**Upstream references:** Spec §11; Design §§11, 12.

**Owning module/component:** Operations & Support observability.

**In scope:**

- Cover readiness failures, storage/disk pressure, backup age/failure, audit-write failure, and support-session anomalies. Define extension points for later worker/provider/mail/index signals without implementing later milestones.
- Keep thresholds and notification destinations in one documented configuration source. An unset or stale threshold is visible; no arbitrary production constant is embedded in code.
- Emit no governance content, prompts, credentials, or high-cardinality personal identifiers.
- Reuse M0 structured logging and health state. Keep tooling deployable locally and optional for core data preservation.

**Out of scope:**

- Mandatory Prometheus/Grafana, distributed tracing, remote telemetry, governance analytics, or duplicate audit review.

**Dependencies:** Hard prerequisites: M0-T06, M0-T07. Parallelizable related work: all other M-OPS tasks.

**Acceptance Criteria**

- Each covered failure produces a detectable local signal and recovery clears it
- alert repetition is bounded
- observability failure cannot block core governed reads/writes
- sensitive canaries never appear.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Integration / operational | Inject each failure/recovery, stale signal, and alert-delivery failure; verify metric labels remain bounded and sanitized; exercise deployment with the optional alert consumer absent. |

---

## M2 — Governance Body & Meeting Management

*8 tasks. Hard prerequisites and safe parallel work are stated per task.*

**Critical path:** Follow the hard prerequisites stated per task; work identified as parallelizable may proceed concurrently.

**Purpose:** Establish governance-body ownership and the advance/retrospective meeting context that every later content and official action relies on. This milestone is meeting management, not organization-settings expansion.

**Prerequisites and parallelism:** M1A-T01 and M1A-T04 provide identity and current-state authorization; M0-T10 provides durable audit. After the body schema, body API and meeting backend may proceed in parallel. Agenda, provenance, and participants build on the meeting contract; UI follows stable APIs.

**Exit gate:** An authorized user can create and manage a governance body, membership/permissions, meetings, agenda items, and participants. Every material action carries explicit body context and data-layer authorization. The system can validate the required organization/body/meeting/date/uploader provenance contract for M3 without adding invitations, broad organization settings, or unrelated audit-view features.

---

### M2-T01 — Governance Body Schema & DAL

**Priority:** P0

**Human-review gate:** Required — architecture, authorization, and migration review.

**Description**

Define the authoritative Governance Context persistence boundary for governance bodies under the existing Organization root.

**Upstream references:** Spec §§5, 7, 8, 9; Design §§4, 8, 13.

**Owning module/component:** Governance Context; Authorization remains sole owner of authority evaluation.

**In scope:**

- Persist organization ownership, stable body identity, body metadata needed by meeting/context workflows, and an explicit active/inactive condition without implementing a configurable workflow engine.
- Expose owner-scoped data-access operations that always require Organization scope and never allow cross-organization reads or writes.
- Reserve membership and permission mutation for M2-T03; do not collapse organization administrator authority into body roles.
- Use versioned migrations and concurrency protection according to the backend technology decision in Design §16; preserve existing organization and audit boundaries.

**Out of scope:**

- Organization settings, invitations, subscription data, arbitrary custom roles, meetings, or a new tenant model.

**Dependencies:** Hard prerequisites: M1A-T01, M0-T10. Parallelizable related work: none before the schema; T02 and T04 may begin after it.

**Acceptance Criteria**

- Every body belongs to exactly one Organization
- owner-scoped queries cannot cross that boundary
- unsupported destructive deletion is absent
- migrations apply and validate through the approved tooling.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration | Migration round-trip/compatibility test, organization-scope query tests, duplicate/invalid identity tests, concurrent update test, and least-privilege database-role verification. |

---

### M2-T02 — Governance Body CRUD — API

**Priority:** P1

**Human-review gate:** Required — product and authorization review of commands and errors.

**Description**

Expose authorized body creation, retrieval, update, and supported status transitions through the versioned application API.

**Upstream references:** Spec §§5, 7, 9; Design §§6, 8, 13.

**Owning module/component:** Governance Context API using Authorization and Audit.

**In scope:**

- Require a current Organization administrator or explicitly delegated authority for material body changes; reads use current additive scope.
- Validate the Organization from the session/context rather than trusting a client organization identifier. Return conflict behavior for stale versions and deny without confirming inaccessible resource existence.
- Persist critical permission/governance audit context for create, update, and status changes according to the central event registry.
- Keep request/response conventions aligned with the REST API conventions decision in Design §16 rather than inventing route or envelope standards in this task.

**Out of scope:**

- Membership/permission mutation, user invitations, organization-wide settings, meeting behavior, or hard deletion of referenced bodies.

**Dependencies:** Hard prerequisite: M2-T01. Parallelizable related work: M2-T04.

**Acceptance Criteria**

- Authorized operations produce the expected body state and audit
- unauthorized, cross-organization, stale, and invalid transitions fail without mutation
- deactivated bodies cannot silently accept new material actions.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration | API/data-layer tests for each authority path, sensitive-body exception, cross-organization identifier, concurrency conflict, audit failure, and inactive state; verify no role or invitation side effects. |

---

### M2-T03 — Governance Body Membership & Permissions — API

**Priority:** P1

**Human-review gate:** Required — security/authorization and governance-policy review.

**Description**

Manage predefined governance-body memberships and action permissions while preserving separate organization-level Community Manager authority.

**Upstream references:** Spec §§5, 9; Design §§6, 8, 13.

**Owning module/component:** Authorization for assignments/policy; Governance Context supplies body and role-holder identities.

**In scope:**

- Create, change, and revoke predefined body assignments for read, review/edit, approve, import/classify, and other settled actions without allowing arbitrary role construction.
- Preserve additive reads across bodies and explicit single-body context for material actions. Approver and reviewer are independent permissions; task assignment is not body membership.
- Enforce current privileged-role MFA eligibility before a privileged assignment becomes usable, without choosing unresolved customer role mappings.
- Guarantee audit of actor, affected user, body/scope, previous/new state, timestamp, and delegation context; permission change fails if audit persistence fails.

**Out of scope:**

- Custom role editor, organization account lifecycle, task assignment, standing support access, or permission checks only in middleware/UI.

**Dependencies:** Hard prerequisites: M2-T01, M1A-T04, M1A-T06. Parallelizable related work: M2-T04 after T01.

**Acceptance Criteria**

- Revocation takes effect on the next protected operation and invalidates affected authorization state
- reviewer/approver combinations work independently
- cross-body and cross-organization escalation is impossible
- every change is durably audited.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration | Matrix-test predefined roles/actions, additive reads, explicit context mismatch, MFA-incomplete privilege, sensitive-body exception, revocation during an active session, audit failure, and concurrent assignment changes. |

---

### M2-T04 — Meeting Lifecycle — Backend

**Priority:** P1

**Human-review gate:** Required — domain review of state transitions and retrospective-flow behavior.

**Description**

Own meeting records for both advance planning and retrospective content assignment.

**Upstream references:** Spec §§6, 7, 8; Design §§4, 6, 8, 13.

**Owning module/component:** Governance Context.

**In scope:**

- Create and update meetings with Organization, governance body, meeting date, and lifecycle state; support planned/scheduled, open, and closed transitions needed by the pilot without introducing a generic workflow engine.
- Allow records to remain open for attachments across sessions and support retrospective creation before processing begins.
- Require current body-scoped create/edit permission and explicit body context. Preserve change history and audit material transitions.
- Define stable query/application contracts for agenda, participant, provenance, dashboard, and later return-loop owners.

**Out of scope:**

- Automated agenda generation, calendar integration, protocol review, content binaries, voting/quorum, or organization settings.

**Dependencies:** Hard prerequisite: M2-T01. Parallelizable related work: M2-T02 and M2-T03.

**Acceptance Criteria**

- Both advance and retrospective meeting creation produce equivalent valid context
- invalid body/date/state transitions fail
- closed/inactive state behavior is explicit
- concurrent edits do not overwrite each other.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration | State-transition table tests, authorization/context mismatch, retrospective path, cross-organization body, concurrency conflict, audit failure, and attachment-across-sessions contract test. |

---

### M2-T05 — Agenda Item Management — Backend

**Priority:** P1

**Human-review gate:** Required — product review of ordering and meeting-context semantics.

**Description**

Manage ordered agenda items within a meeting and expose stable references for protocol sections and optional content linkage.

**Upstream references:** Spec §§6, 7; Design §§4, 6, 8, 13.

**Owning module/component:** Governance Context.

**In scope:**

- Create, update, order, and remove only unreferenced agenda items under an authorized meeting; preserve stable identity once referenced.
- Support multiple items and an explicit unassigned/detected-topic path downstream, because intake linkage may be unavailable.
- Apply body-context authorization at the data layer and reject items whose meeting/body ownership does not match.
- Expose a future-return association contract without implementing M7 return-loop confirmation or automatic agenda construction.

**Out of scope:**

- Collaborative real-time editing, automated agenda construction, return-loop decisions, protocol section review, or voting.

**Dependencies:** Hard prerequisite: M2-T04. Parallelizable related work: M2-T07.

**Acceptance Criteria**

- Ordering is deterministic and conflict-safe
- referenced item identity survives title/order changes
- unauthorized/cross-meeting mutations fail
- omission of an agenda link does not invalidate an otherwise valid intake.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration | Order/reorder and concurrent edit tests, referenced-item deletion behavior, authorization/body mismatch, empty agenda, and optional-link contract verification. |

---

### M2-T06 — Content Item Provenance & Linkage — Backend

**Priority:** P1

**Human-review gate:** Required — data-integrity and authorization review of the processing gate.

**Description**

Provide the canonical context-validation contract that M3 uses to bind every intake to its governed origin.

**Upstream references:** Spec §§7, 8, 9; Design §§4, 8, 13.

**Owning module/component:** Governance Context for linkage validity; Content & Provenance owns intake records beginning in M3.

**In scope:**

- Validate required Organization, governance body, meeting, meeting date, and authorized uploader relationships from authoritative records. Agenda-item linkage is optional but, when supplied, must belong to the meeting.
- Return a stable governed-context reference and current access scope suitable for M3 persistence; do not copy stale role claims into content authority.
- Prevent processing eligibility when required context is incomplete, inconsistent, inactive, or inaccessible.
- Preserve body/date/uploader history even if later display metadata changes.

**Out of scope:**

- File storage, text parsing, AI eligibility policy, processing jobs, historical bulk import, or treating agenda linkage as mandatory.

**Dependencies:** Hard prerequisites: M2-T04, M2-T05. Parallelizable related work: M2-T07 and the M3 schema design after the contract stabilizes.

**Acceptance Criteria**

- Valid advance and retrospective contexts resolve identically
- each required missing/mismatched field is rejected
- an omitted agenda item succeeds
- revocation before intake/processing is respected.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration | Contract tests for every required field, wrong-body agenda, changed meeting date, inactive body, revoked uploader, cross-organization identifiers, and optional agenda linkage. |

---

### M2-T07 — Participant Tracking — Backend

**Priority:** P1

**Human-review gate:** Required — privacy and governance-context review.

**Description**

Record meeting participation without turning participation into authorization or task responsibility.

**Upstream references:** Spec §§6, 7, 8; Design §§4, 6, 8.

**Owning module/component:** Governance Context.

**In scope:**

- Add, update, and remove meeting participant records under current body permission, supporting known users and the minimal external/display identity needed by the pilot.
- Keep participant, speaker attribution, presenter, reviewer, approver, and task owner as distinct relationships.
- Do not grant body access merely because a person appears in the participant list.
- Apply retention/data-minimization policy and audit material changes without storing unnecessary personal data.

**Out of scope:**

- Invitations, attendance/quorum rules, voting eligibility, speaker-identification provider logic, HR/directory integration, or task assignment.

**Dependencies:** Hard prerequisites: M2-T04, M2-T03. Parallelizable related work: M2-T05.

**Acceptance Criteria**

- Participant changes remain meeting/body scoped
- participant addition grants no permission
- duplicate handling is deterministic
- inaccessible identities do not leak through API errors.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration | Test known/external participant paths, duplicates, cross-meeting/body operations, no permission side effect, revoked editor, concurrent updates, and minimized audit output. |

---

### M2-T08 — Governance Body & Meeting UI

**Priority:** P2

**Human-review gate:** Required — Community Manager/coordinator workflow, governance authorization, Hebrew/RTL, responsive, and accessibility review.

**Description**

Deliver the primary UI for body context, meetings, agenda items, participants, and customer-specific membership/permission assignment.

**Upstream references:** Spec §§5, 6, 7, 9, 11; Design §§6, 8, 10.

**Owning module/component:** Governance Context UI; permission controls call Authorization APIs.

**In scope:**

- Center the Community Manager's cross-body workflow while supporting authorized coordinator operations. Display active body, current role/permissions, and allowed actions during material work.
- Support advance meeting creation and the context portion of retrospective creation, agenda ordering, participant management, and predefined body assignments.
- Preserve entered state and explain validation, conflict, 401, 403/revocation, and partial backend failure. UI action visibility is guidance, never the enforcement boundary.
- Meet shared Hebrew RTL/BiDi, keyboard, focus, responsive, and accessible error conventions.

**Out of scope:**

- Broad organization settings, invitations, audit-log viewer, protocol review, content upload, automatic agenda generation, voting, or custom roles.

**Dependencies:** Hard prerequisites: M2-T02, M2-T03, M2-T04, M2-T05, M2-T07, M-UX-T03. Parallelizable related work: M2-T06 is consumed by M3 rather than blocking all meeting UI.

**Acceptance Criteria**

- Authorized personas complete body/meeting/agenda/participant flows
- explicit body context remains visible
- forbidden actions fail safely if invoked
- responsive RTL behavior is coherent and no unrelated settings replace meeting management.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration / E2E | Component/integration journeys for CM, coordinator, read-only, reviewer/approver separation, sensitive-body exception, revocation mid-form, concurrency conflict, keyboard/focus, RTL/BiDi strings, and small screens. |

---

## M3 — Content Intake & File Management

*5 tasks. Hard prerequisites and safe parallel work are stated per task.*

**Critical path:** Follow the hard prerequisites stated per task; work identified as parallelizable may proceed concurrently.

**Purpose:** Accept supported text and file sources, preserve the original, and bind every intake to valid governance provenance, sensitivity, access, and AI-eligibility state. This milestone stops at intake and initial processing readiness.

**Prerequisites and parallelism:** M2-T01, M2-T04, and M2-T06 provide governance context; M0-T05 provides the storage port; M0-T10 provides audit. Schema precedes storage/API. Scanned detection may proceed with API work after storage. UI follows both behavior paths.

**Exit gate:** Authorized users can paste text or upload each supported pilot format, with immutable source preservation and required organization/body/meeting/date/uploader linkage. Unsafe or unsupported input fails safely, unreadable scanned PDFs receive the specified manual-handling notice, and nothing in M3 publishes, versions, archives, searches, views, or AI-processes content.

---

### M3-T01 — Content & Attachment Schema

**Priority:** P0

**Human-review gate:** Required — data/provenance, authorization, and migration review.

**Description**

Define Content & Provenance records for original sources, attachments, governance linkage, sensitivity, access scope, AI eligibility, and initial processing state.

**Upstream references:** Spec §§7, 8, 9; Design §§4, 8, 13.

**Owning module/component:** Content & Provenance, consuming the M2 governed-context contract.

**In scope:**

- Require Organization, governance body, meeting, meeting date, and uploader on every intake; allow agenda-item linkage to be absent.
- Represent pasted text and stored binary attachments through one provenance model while keeping immutable original content distinguishable from later provider, AI, human, and official layers.
- Record detected/declared type, integrity reference, access scope, sensitivity, AI eligibility, retention status, and initial processing/status history as queryable fields where they affect policy.
- Keep processing, document lifecycle, protocol review, and official approval states separate; do not introduce later milestone entities.

**Out of scope:**

- Versioning/publishing/archive workflows, search indexes, document viewer/diff, protocol drafts, historical bulk import, or AI job execution.

**Dependencies:** Hard prerequisite: M2 provenance foundations, specifically M2-T06. This task blocks M3-T02 through M3-T04.

**Acceptance Criteria**

- The database rejects missing required provenance and mismatched ownership
- agenda linkage is nullable but validated when present
- original source references are immutable through the owner API
- AI eligibility is independent of read access.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration | Migration and constraint tests, pasted/binary variants, each missing provenance field, wrong-body agenda, revoked uploader at creation, source immutability, and separation of the four state concerns. |

---

### M3-T02 — File Storage & Validation

**Priority:** P0

**Human-review gate:** Required — security review of upload, parser, storage, and failure cleanup boundaries.

**Description**

Store supported originals through the M0 storage abstraction and validate hostile input before parsing or downstream use.

**Upstream references:** Spec §§7, 8, 11; Design §§4, 11, 12.

**Owning module/component:** Content & Provenance using the M0 storage port.

**In scope:**

- Accept direct text and the pilot formats defined in Spec §7.2; verify detected type rather than trusting extension or browser MIME. Video is accepted only through a later approved provider capability, not by building a local video pipeline here.
- Generate opaque storage keys, retain original filename only as metadata, compute/verify integrity data, and commit database metadata only after durable storage succeeds.
- Enforce centrally configured size, expansion, parser resource, and type limits. Run document parsing without unnecessary network or secret access.
- Clean up unreferenced partial artifacts after failure while never deleting a previously committed original.

**Out of scope:**

- OCR, provider calls, content publication, storage-vendor selection beyond M0, antivirus-product selection without a gate, full document viewer, or derived search text indexing.

**Dependencies:** Hard prerequisites: M3-T01, M0-T05. Parallelizable related work: M3-T04 after the storage contract exists.

**Acceptance Criteria**

- Supported valid sources are retrievable byte-for-byte through authorized content access
- spoofed/oversized/malformed input is rejected
- traversal is impossible
- storage/database failure cannot produce a referenced partial or silently lose an original.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration | File-signature mismatch, traversal names, empty/large/expanding/malformed files, parser timeout, storage interruption, checksum mismatch, duplicate retry, pasted text, and least-privilege storage access; inspect orphan cleanup and logs for sensitive data. |

---

### M3-T03 — Content Intake API

**Priority:** P1

**Human-review gate:** Required — product, data-layer authorization, and security review.

**Description**

Expose authorized text/file intake and context assignment while enforcing the no-processing-without-governance-context rule.

**Upstream references:** Spec §§6, 7, 9; Design §§6, 8, 13.

**Owning module/component:** Content & Provenance API with Governance Context and Authorization.

**In scope:**

- Support advance attachment to an existing meeting and retrospective intake/context assignment. Persist only after M2 validates Organization, body, meeting, date, uploader, and optional agenda relationship.
- Require current body-scoped upload/classify permission at the data-access boundary; do not restrict the flow to a invented CM-only rule when an authorized coordinator is configured.
- Capture access scope, sensitivity, and AI-eligibility separately. An uploaded item may remain preserved but non-processable until policy is confirmed.
- Return stable content identity, original-source metadata, and initial state; do not start M4 work implicitly unless the later explicit job operation is invoked.

**Out of scope:**

- Automatic AI dispatch, versioning, publication, archive/lifecycle transitions, keyword/full-text search, viewer/diff, historical batch import, or approval.

**Dependencies:** Hard prerequisites: M3-T01, M3-T02. Parallelizable related work: M3-T04. This task enables M4 intake submission.

**Acceptance Criteria**

- Every successful record has all required provenance
- invalid/mismatched/revoked context fails without a committed item
- retries are idempotent
- read access does not automatically imply AI eligibility or external transmission.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration | Advance/retrospective paths, direct text/all file categories, each missing context field, optional agenda, cross-body/organization IDs, revoked uploader, inactive meeting/body, duplicate retry, storage failure, and audit sanitization. |

---

### M3-T04 — Scanned Document Detection & OCR Notice

**Priority:** P1

**Human-review gate:** Required — product UX and document-security review of classification and recovery guidance.

**Description**

Detect PDFs without usable text and route them to the pilot's explicit manual handling path.

**Upstream references:** Spec §7; Design §§4, 6, 12.

**Owning module/component:** Content & Provenance.

**In scope:**

- Inspect a safely parsed PDF for usable text and distinguish text-readable, unreadable/scanned, malformed, and indeterminate outcomes.
- Preserve the original in every accepted outcome. Mark unreadable/scanned material as requiring manual handling and provide structured reason/guidance for the UI.
- Offer re-upload of a text-readable version or the configured accompanied manual path. Do not send the file to OCR or an external provider in this milestone.
- Do not classify a transient parser failure as definitely scanned; expose a retry/supportable failure state.

**Out of scope:**

- Automated OCR, OCR-provider selection, image transcription, lifecycle activation, content viewer, or historical archive cleanup.

**Dependencies:** Hard prerequisites: M3-T01, M3-T02. Parallelizable related work: M3-T03.

**Acceptance Criteria**

- Representative text and image-only PDFs are distinguished
- malformed/indeterminate files do not receive misleading OCR claims
- originals remain accessible to authorized users
- scanned items are not AI-processable through the text path.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration / provider validation | Fixtures for Hebrew text PDFs, image-only PDFs, mixed pages, encrypted/malformed files, parser timeout, and retry; verify notice reason, source preservation, and no network/provider call. |

---

### M3-T05 — Content Intake UI

**Priority:** P1

**Human-review gate:** Required — Community Manager/coordinator workflow, Hebrew/RTL, accessibility, responsive, and upload-security review.

**Description**

Provide coherent text/file intake for advance and retrospective meeting flows with visible governance and policy context.

**Upstream references:** Spec §§6, 7, 11; Design §§6, 8, 10, 12.

**Owning module/component:** Content & Provenance UI.

**In scope:**

- Support text paste and supported file selection, existing-meeting attachment, and retrospective meeting/context assignment. Clearly display body, meeting, date, uploader, access scope, sensitivity, AI eligibility, and optional agenda choice before submission.
- Preserve user input across validation or recoverable upload errors without retaining secrets or stale authorization assumptions.
- Render progress for the intake transfer only; later AI-processing stages belong to M4. Show the exact scanned/manual notice and re-upload path from M3-T04.
- Handle 401, 403/revocation, context conflict, unsupported/malformed input, interrupted upload, and storage failure accessibly in Hebrew RTL.

**Out of scope:**

- AI status/SSE, protocol review, publication, archive/version/search/viewer/diff, OCR execution, or bulk historical import.

**Dependencies:** Hard prerequisites: M3-T03, M3-T04. Parallelizable related work: none after behavior contracts stabilize.

**Acceptance Criteria**

- Authorized users complete every supported path
- required context is visible and confirmed
- agenda remains optional
- failures do not create duplicate records or imply processing/official status
- responsive and keyboard flows remain usable.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration / E2E | End-to-end text and file journeys, advance/retrospective context, scanned notice, upload interruption/retry, revocation mid-submit, validation focus, RTL/BiDi filenames/text, keyboard and responsive checks. |

---

## M4 — AI Processing Pipeline & Provider Adapters

*7 tasks. Hard prerequisites and safe parallel work are stated per task.*

**Critical path:** Follow the hard prerequisites stated per task; work identified as parallelizable may proceed concurrently.

**Purpose:** Turn eligible governed sources into traceable advisory candidates through durable asynchronous jobs and replaceable provider adapters. Provider choice remains human-gated, and no M4 output is official.

**Prerequisites and parallelism:** M3-T03 supplies eligible content; M1A-T04 supplies current-state authorization; M0-T10 supplies audit. The job schema precedes adapters. Stub/contract work may proceed while real provider evaluation runs in parallel. Execution precedes policy/result work; status delivery can proceed once lifecycle and stored-result contracts stabilize.

**Exit gate:** A permitted source can run through a stub and at least one explicitly approved, realistically Hebrew-validated provider path; jobs survive browser disconnect/restart, expose SSE plus polling status, retry/recover safely, and preserve originals. Current authorization, sensitivity, AI eligibility, purpose, and provider policy are revalidated at every consequential boundary. Failed/cancelled jobs create no partial protocol or official content, and processing events are available to M5/M7.

---

### M4-T01 — AI Job Schema & Processing Status Model

**Priority:** P0

**Human-review gate:** Required — architecture, data-provenance, and security review.

**Description**

Persist durable resource-scoped jobs, attempts, stage history, and candidate-result linkage independently of transport or queue technology.

**Upstream references:** Spec §§7, 8, 10; Design §§2, 4, 5, 8, 13.

**Owning module/component:** AI Job Orchestration.

**In scope:**

- Record source/content revision, Organization/body/meeting scope, requester, purpose, capability, selected approved provider reference, policy version, current stage, attempts, cancellation/failure reason, timestamps, and result reference.
- Support the user-visible processing stages while retaining finer internal attempt/stage evidence without conflating content lifecycle or protocol state.
- Make authoritative job state durable and concurrency-safe; possession of a job ID grants no access.
- Define idempotency identities for submission and attempt/result promotion without selecting the queue, scheduler, or framework.

**Out of scope:**

- Queue/vendor selection, provider SDK types in domain records, voting/quorum, protocol drafts, official records, or search indexing.

**Dependencies:** Hard prerequisite: M3-T01. Milestone prerequisites M3-T03, M1A-T04, and M0-T10 must exist before executable jobs. This task blocks M4-T02 through M4-T07.

**Acceptance Criteria**

- Jobs cannot exist without valid governed source scope
- duplicate submission is deterministic
- legal state transitions are enforced
- attempt history survives restarts
- content/protocol/official state is not mutated by schema creation.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration / E2E | Migration/constraint tests, transition table, duplicate/concurrent submission, inaccessible job lookup, source revision mismatch, attempt history, terminal-state immutability, and least-privilege worker persistence. |

---

### M4-T02 — Provider Adapter Contract, Stub & Real Integrations

**Priority:** P0

**Human-review gate:** Required — architecture and security review for the contract; separate provider-selection, DPA/data-policy, and realistic Hebrew validation before each real adapter is enabled.

**Description**

Isolate transcription and extraction provider protocols behind a normalized outbound adapter boundary.

**Upstream references:** Spec §§10, 12; Design §§3, 5, 12, 16.

**Owning module/component:** Provider Gateway; AI Job Orchestration owns job policy.

**In scope:**

- Define capability discovery and submit/poll/retrieve/cancel-when-supported behavior, normalized errors, provider correlation, and a candidate envelope that preserves provider-native evidence.
- Supply a deterministic local stub for development/failure testing without making external services mandatory for deployment startup or governed-data preservation.
- Add real transcription/extraction adapters only after the named provider is explicitly approved. Test with representative Hebrew, multi-speaker governance material and approved retention/training/residency settings.
- Keep vendor SDK requests/responses inside the adapter. Optional webhooks expose only a separately approved, signed, replay-protected completion path.

**Out of scope:**

- Choosing a provider, general inbound provider access, mandatory webhook, provider-specific domain schema, building transcription models, voting/quorum, or queue selection.

**Dependencies:** Hard prerequisite: M4-T01. Real integration additionally depends on provider human approval; no provider name is selected by this task. This task blocks M4-T03.

**Acceptance Criteria**

- Orchestration can substitute stub and approved adapters without domain changes
- local startup succeeds with real providers disabled
- errors are normalized without losing diagnostic correlation
- unapproved adapters cannot transmit.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration / provider validation | Contract suite against stub/real adapters, submit-poll-retrieve, unsupported cancel, timeouts/malformed results, webhook signature/replay when enabled, credential/log sanitization, and human-reviewed Hebrew benchmark evidence. |

---

### M4-T03 — Async Job Execution — Dispatch & Lifecycle

**Priority:** P1

**Human-review gate:** Required — architecture/operations review of durability, retry, cancellation, and recovery behavior.

**Description**

Execute jobs outside the request/browser lifecycle with durable dispatch, polling, retry, timeout, cancellation, and restart recovery.

**Upstream references:** Spec §§7, 11; Design §§2, 5, 6, 11, 12.

**Owning module/component:** AI Job Orchestration workers using Provider Gateway.

**In scope:**

- Persist producer state before dispatch; lease and acknowledge work through the job, scheduler, and event-relay mechanism selected under Design §16 without choosing its implementation here. Consumers are idempotent under at-least-once delivery.
- Poll outbound provider jobs, classify transient/permanent/policy/cancel errors, and apply centrally configured provider-aware timeouts, retry limits, and backoff.
- Continue without a browser and recover expired work after process restart without duplicating results. Cancellation is best effort externally but authoritative locally before downstream promotion.
- Preserve the source and record the failed stage; no failed/exhausted/cancelled attempt creates a partial draft or official content.

**Out of scope:**

- Exact queue/broker selection, hard-coded retry constants, browser-owned execution, officialization, voting/quorum, or automatic provider failover without approval.

**Dependencies:** Hard prerequisite: M4-T02. Parallelizable related work: none until adapter contract stabilizes. This task enables M4-T04 and M4-T05.

**Acceptance Criteria**

- Jobs complete/retry/fail/cancel deterministically across disconnect and worker restart
- duplicate delivery has one business effect
- exhausted failures are actionable
- configured limits have one source and can be observed.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration / provider validation | Worker restart mid-stage, duplicate lease/delivery, provider transient/permanent failure, timeout, cancellation race, result-after-cancel, source revision change, and unavailable external network; verify original preservation and no draft creation. |

---

### M4-T04 — Authorization Revalidation & External-Transmission Policy

**Priority:** P1

**Human-review gate:** Required — security/authorization and customer external-provider policy approval.

**Description**

Make every provider job and result obey current access, sensitivity, AI eligibility, purpose, and provider-destination policy throughout its lifetime.

**Upstream references:** Spec §§9, 10, 11; Design §§5, 8, 12, 13.

**Owning module/component:** AI Job Orchestration for checkpoints; Provider Gateway for the final network egress gate; Authorization/Audit own policy evaluation/evidence.

**In scope:**

- Revalidate before dispatch, immediately before any external transmission, before accepting/storing a returned candidate, and before exposing/promoting the candidate downstream.
- Evaluate current Organization/body access, source revision, sensitivity, AI eligibility, approved purpose, provider capability/destination, and customer external-processing policy; stored job context is not authority.
- Block, quarantine, or discard returned data according to settled retention policy when permission/policy changed, without mutating governed content.
- Durably audit permit/block and transmission decisions with minimized metadata. If required security audit cannot persist, external transmission or protected promotion fails closed.

**Out of scope:**

- UI-only checks, policy captured only at job creation, implicit consent, standing provider access, provider selection, or logging prompts/source passages.

**Dependencies:** Hard prerequisites: M4-T01, M4-T02, M4-T03. Parallelizable related work: provider-policy onboarding can proceed while execution is built.

**Acceptance Criteria**

- Revocation or eligibility/policy change at every checkpoint prevents the next consequential action
- locally readable but externally ineligible content is not transmitted
- stale jobs/caches cannot broaden access
- audit proves the decision without copying content.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration / provider validation | Revoke between creation/dispatch, dispatch/result, and result/display; change sensitivity/provider policy/purpose/source revision; cross-body job, audit failure, disabled provider, and callback after revocation; inspect network mock for zero blocked payloads. |

---

### M4-T05 — Confidence Normalization & Result Storage

**Priority:** P1

**Human-review gate:** Required — AI/product review of normalization semantics and data/provenance review.

**Description**

Store authorized provider results as traceable advisory candidates and normalize uncertainty without inventing a universal provider-independent score.

**Upstream references:** Spec §§8, 10, 12; Design §§4, 5, 13, 16.

**Owning module/component:** AI Job Orchestration for candidate envelope; Content & Provenance supplies source references.

**In scope:**

- Preserve provider-native uncertainty/confidence and result provenance, then record the versioned normalization policy that derives qualitative review signals.
- Validate structured output and exact source references before storage. Invalid, unsupported, or out-of-contract output fails the job or enters an explicit non-promotable diagnostic state.
- Store only after M4-T04 current-state revalidation. Candidate output is clearly AI-generated and cannot overwrite source/provider layers or become official.
- Leave the exact system threshold, qualitative mapping, and optional numeric display to the confidence-normalization decision in Design §16 and representative Hebrew validation; do not embed an example threshold as fact.

**Out of scope:**

- Choosing thresholds, configurable customer thresholds, protocol section review, official tasks/decisions, destructive source overwrite, or retaining raw provider payloads automatically.

**Dependencies:** Hard prerequisites: M4-T01, M4-T02, M4-T03, M4-T04. This task enables M5-T01.

**Acceptance Criteria**

- Every stored candidate references a source revision, job/attempt, provider/model where available, normalization policy, and uncertainty evidence
- blocked/invalid results are not promoted
- retry cannot duplicate candidates.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration / provider validation | Provider formats with/without numeric confidence, malformed/out-of-range native values, missing source reference, duplicate result, policy revocation, raw-response retention disabled/enabled, and source-layer immutability. |

---

### M4-T06 — SSE / Polling Status Updates

**Priority:** P1

**Human-review gate:** Required — API, authorization, reconnect, and frontend integration review.

**Description**

Expose and render authoritative job progress through authenticated SSE with a REST polling fallback.

**Upstream references:** Spec §§7, 11; Design §§6, 8, 11.

**Owning module/component:** AI Job Orchestration owns status truth; its transport adapter and the frontend processing-status surface consume that one state model.

**In scope:**

- Stream stage changes from durable job state and allow polling of the same state. Disconnect/reconnect does not control job execution or imply cancellation.
- Authorize the underlying source/job resource on connection, polling, reconnect, and each consequential emission; job ID possession is insufficient.
- Support resume/reconciliation from authoritative current state so missed events do not leave the UI permanently stale.
- Close or clearly mark terminal outcomes and distinguish failed/requires-attention, cancelled, access-revoked, and unavailable transport.
- Render the seven user-visible stages exactly as Uploaded, Awaiting transcription, Transcribing, Analysing content, Generating structured draft, Ready for review, and Failed / requires attention. Show the failure stage and an authorized retry path where retry is safe; do not infer progress from elapsed time.
- Before the user leaves the upload flow, show a clear warning when the authoritative estimate says audio/video processing is expected to exceed one hour. The estimate source and boundary remain centrally configured; the client must not invent a timeout or SLA.

**Out of scope:**

- WebSockets, browser-owned jobs, unauthenticated status, notification-center behavior, provider callback implementation, or a new status source.

**Dependencies:** Hard prerequisites: M4-T03, M4-T05. Parallelizable related work: M4-T07 event contracts.

**Acceptance Criteria**

- SSE, polling, and the frontend agree on all seven visible states
- browser disconnect leaves work running
- reconnect catches up
- revocation stops further detail
- another body's job cannot be observed
- terminal state is stable
- the over-one-hour warning appears only from the authoritative estimate and processing failures name their stage without publishing partial official content.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration / E2E | SSE disconnect/reconnect, missed events, polling fallback, concurrent viewers, every visible state and transition, failure-stage and retry presentation, authoritative estimates just below/at/above the one-hour warning boundary, absent estimate, worker restart, unauthorized/cross-body ID, revocation mid-stream, slow-client/backpressure behavior, and Hebrew/RTL accessible status announcements. |

---

### M4-T07 — In-App Processing Events

**Priority:** P1

**Human-review gate:** Required — event ownership, payload-minimization, and notification semantics review.

**Description**

Publish the durable, idempotent processing-failed producer event and define the handoff boundary from durable AI results to the Protocol Review owner that creates reviewable drafts.

**Upstream references:** Spec §7; Design §§2, 5, 6, 13.

**Owning module/component:** AI Job Orchestration owns processing-failed production and the durable-result handoff; Protocol Review & Approval owns the later draft-ready transition; Notification Delivery consumes committed producer events.

**In scope:**

- Append processing-failed only when an actionable terminal failure is durably committed. A successful durable candidate result uses the M5 handoff contract but is not itself the user-facing draft-ready notification event.
- Include stable event/job/content/body/meeting identifiers, recipient-resolution context, stage/outcome, and occurred time without source text, prompts, or provider payloads.
- Persist each producer-owned outbox event atomically with its triggering state and make downstream consumption idempotent. Notification Delivery materializes its separate recipient/channel intents only after commit; backfill/retry cannot create duplicate recipient effects for the same source event.
- Define the versioned handoff that lets M5 create a reviewable draft and append draft-ready in its own transaction. Provide a temporary in-app integration seam until M7 owns full dispatch/email; do not choose SMTP or queue technology.

**Out of scope:**

- SMTP/email delivery, notification preferences/center, emitting draft-ready before the M5 transition, queue selection, or publishing raw AI output.

**Dependencies:** Hard prerequisites: M4-T05, M4-T06. This task supports M5-T06.

**Acceptance Criteria**

- Each actionable terminal failure creates exactly one durable producer outbox event
- transient retries do not
- a successful result handoff cannot emit draft-ready
- payloads are sufficient for later authorized recipient resolution and contain no governed content
- notification persistence/delivery failure cannot erase or roll back the committed domain event.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration | Success handoff versus actionable failed/exhausted/cancelled transitions, duplicate consumer/delivery, transaction rollback, draft-creation handoff without premature draft-ready emission, recipient revoked before consumption, and payload secret/content canaries. |

---

## M5 — Protocol Draft Review & Approval

*8 tasks. Hard prerequisites and safe parallel work are stated per task.*

**Critical path:** Follow the hard prerequisites stated per task; work identified as parallelizable may proceed concurrently.

**Purpose:** Turn authorized M4 candidates into evidence-linked, agenda/topic-structured protocol drafts that humans can review and formally approve. AI originals and source provenance remain immutable; section review and whole-protocol approval are separate; approval creates a distinct official artifact.

**Prerequisites and parallelism:** M4-T05 supplies normalized candidates, M4-T07 supplies processing event contracts, M1A-T04 supplies authorization, and M0-T10 supplies guaranteed audit. Schema precedes permission/API work. Low-confidence backend follows the section API and can proceed in parallel with formal approval integration once its blocking contract is stable. UI follows the relevant APIs.

**Exit gate:** An authorized reviewer can work through agenda/topic sections and the global evidence-linked review queue, preserve immutable AI originals, resolve or dismiss every flagged item with reason, and confirm meeting-level completeness. A separately authorized approver can create exactly one immutable official protocol only when every readiness condition passes. Failure, stale edits, missing fields, unresolved low confidence, or audit failure creates no official artifact. Approval-stage in-app intents and accessible Hebrew/RTL desktop/mobile flows are verified.

---

### M5-T01 — Protocol Draft Schema & Status Model

**Priority:** P0

**Human-review gate:** Required — architecture, provenance, and governance-record integrity review.

**Description**

Persist structured protocol drafts, agenda/topic sections, immutable AI originals, exact source references, human edits, review-card state, and whole-protocol workflow state without conflating any of them with an official artifact.

**Upstream references:** Spec §§7, 8, 10; Design §§4, 10, 13.

**Owning module/component:** Protocol Review & Approval.

**In scope:**

- Link each draft to Organization, governance body, meeting, source/content revision, and originating AI job/result. Group sections by agenda item where known or stable detected topic where not.
- Preserve immutable AI original plus exact source passage/segment, speaker when available, timestamp/page/location, and stable full-context pointer. Store human edits and attribution separately.
- Model per-section/card review state and separate whole-protocol readiness/approval/publication state. A reviewed section is not a formally approved protocol.
- Keep low-confidence cards outside the main draft until a reviewer explicitly promotes/classifies them; retain dismissed evidence and its reason rather than deleting it.

**Out of scope:**

- Official protocol creation, task activation, publishing/distribution, configurable customer confidence thresholds, voting/quorum, or overwriting M3/M4 records.

**Dependencies:** Hard prerequisites: M3-T01, M4-T01. Milestone prerequisites M4-T05, M1A-T04, and M0-T10 must exist before draft creation. This task blocks M5-T02 through M5-T08.

**Acceptance Criteria**

- AI originals and source-reference fields cannot be changed through owner operations
- edits are attributable
- agenda link may be absent without losing topic/source trace
- protocol and section states remain independently queryable
- no schema operation creates official content.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration | Migration/constraint tests, source/AI immutability, edit attribution, agenda/detected-topic variants, low-confidence separation, status-transition table, duplicate candidate ingestion, source revision mismatch, and cross-body ownership. |

---

### M5-T02 — Reviewer & Approver Permission Model

**Priority:** P0

**Human-review gate:** Required — security/authorization and customer governance-policy review.

**Description**

Define and enforce separate per-body permissions for draft review/edit and formal whole-protocol approval.

**Upstream references:** Spec §§5, 7, 9; Design §§8, 10.

**Owning module/component:** Authorization owns permission evaluation; Protocol Review & Approval consumes it for each operation.

**In scope:**

- Evaluate review/edit and formal approval as independent current-state permissions. An approver does not implicitly receive review/edit authority, and a reviewer cannot approve unless separately assigned.
- Require explicit governance-body context and verify that it matches draft, meeting, and source. Organization administrator authority follows configured CM defaults/exceptions rather than bypassing resource context.
- Revalidate on every data-access operation and at formal approval; session or UI state is not authority. Revocation stops subsequent edits/approval immediately.
- Reuse predefined body assignments from M2; do not create custom roles or a workflow engine.

**Out of scope:**

- Persona-name inference, approval inherited from review or vice versa, UI-only authorization, custom roles, task confirmation, or publication permission.

**Dependencies:** Hard prerequisites: M1A-T04, M5-T01. Parallelizable related work: M5-T03 contract design after permission actions stabilize.

**Acceptance Criteria**

- Reviewer-only, approver-only, both, neither, CM default, delegated admin, and sensitive-body exception cases behave distinctly
- cross-body IDs fail closed
- revocation during a session takes effect without waiting for login expiry.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration / E2E | Full permission matrix at data layer/API, explicit-context mismatch, additive read versus material action, MFA-incomplete privileged assignment, revoke during edit/approval, inaccessible-resource error behavior, and stale client permission list. |

---

### M5-T03 — Section Review & Edit API

**Priority:** P1

**Human-review gate:** Required — product, provenance, concurrency, and authorization review.

**Description**

Let an authorized reviewer inspect, edit, classify, and resolve protocol sections/cards while preserving AI and source evidence.

**Upstream references:** Spec §§7, 10; Design §§6, 8, 10, 13.

**Owning module/component:** Protocol Review & Approval API.

**In scope:**

- Return agenda/topic sections, AI original, current human edit, qualitative uncertainty, missing fields, review state, and immutable exact source references with full-context target.
- Accept human text edits and section-level review actions without accepting changes to AI original or source-reference fields. Attribute every action to current reviewer/body context and expected draft revision.
- Support promote/classify/link/edit/reject/dismiss-with-reason/clarification actions for review cards through owner operations; keep unresolved/missing fields explicit.
- Own the transition that makes a stored draft reviewable and append draft-ready atomically with that transition. Own return-for-revision as a separate committed transition and append returned-for-revision atomically; neither event is inferred from provider completion or delivery state.
- Detect stale concurrent edits and return a conflict that preserves both versions for user resolution. Audit review actions with minimized content references.

**Out of scope:**

- Formal approval, official artifact creation, task activation, publication, notification recipient/channel mapping, AI reprocessing, destructive evidence deletion, or numeric confidence as the primary user signal.

**Dependencies:** Hard prerequisites: M5-T01, M5-T02. This task enables M5-T04, M5-T05, and M5-T07.

**Acceptance Criteria**

- Editing changes only the human layer
- source jumps remain stable
- all actions are attributed and each reviewable/returned transition produces exactly one producer-owned event in the same transaction
- missing fields and unresolved cards cannot be hidden by a generic “reviewed” flag
- unauthorized/stale operations do not mutate state.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration | Edit/accept/classify/promote/link/dismiss/clarify paths; first reviewable transition; returned-for-revision; duplicate/replayed transitions; outbox/audit rollback; attempted source/AI mutation; missing fields; stale revisions; revocation mid-review; cross-body section ID; and immutable provenance retrieval. |

---

### M5-T04 — Formal Protocol Approval API

**Priority:** P1

**Human-review gate:** Required — governance, security/authorization, transaction-integrity, and audit review.

**Description**

Enforce whole-protocol readiness and create a separate immutable official protocol artifact through an atomic, audited approval operation.

**Upstream references:** Spec §§7, 8, 9, 10; Design §§6, 8, 10, 13.

**Owning module/component:** Protocol Review & Approval; an application use-case coordinator owns the atomic transaction, not another domain's tables.

**In scope:**

- Revalidate approver permission, MFA eligibility where required, explicit body context, draft/source integrity, and expected revision at command time.
- Require every section reviewed, every low-confidence/unresolved card resolved or explicitly dismissed with reason, every mandatory field complete, and an explicit reviewer confirmation of meeting-level completeness.
- In one transaction, create exactly one immutable official protocol linked to source/draft, mark the draft approved, persist guaranteed approval audit, and append one canonical protocol-approved producer event plus durable downstream handoff records. Roll back all effects if any required write fails.
- Keep approval distinct from publication/access authorization, distribution request, and M6 decision/task creation. After approval, expose a separate idempotent publish/distribute command owned by Protocol Review & Approval. It revalidates current authorization, body configuration, official revision, audience/destination policy, and guaranteed audit before changing awaiting-publication to published/access-authorized and, where configured, recording distribution-requested. Configured automatic publication invokes the same command and rules.
- The approval use-case exposes a transaction boundary through which the Decisions, Tasks & Return Loop owner may activate already human-confirmed candidates in the same coordinated officialization transaction; approval never writes M6 tables and unconfirmed candidates remain advisory. The same M6 owner also supports later confirmation/activation after officialization.
- Append producer-owned protocol-publication-awaiting, protocol-publication-authorized, and protocol-distribution-requested events atomically with their respective committed transitions. If an accepted publication attempt reaches a durable failed state, append protocol-publication-failed atomically with that state. Notification Delivery exclusively owns per-recipient/channel pending, delivered, and failed truth. A delivery outage must not falsify publication authorization, claim distribution success, or roll back approval.

**Out of scope:**

- Per-section formal approval, automatic approval, direct M6 table writes, automatic activation of unconfirmed candidates, inventing publication audiences/channels outside governance-body configuration, M6 task/decision schema, voting/quorum, or overwriting source/draft/official layers.

**Dependencies:** Hard prerequisites: M5-T02, M5-T03. M5-T05 supplies a readiness blocker consumed before completion. This task enables M5-T06 and M5-T08.

**Acceptance Criteria**

- Only a fully ready current draft can create one official artifact
- repeated approval returns the existing outcome or a deterministic conflict
- audit failure, stale revision, revocation, or any readiness blocker leaves no partial official state. Approval leaves a truthful awaiting-publication state until the separate authorized/configured command succeeds
- publication retry is idempotent, is impossible before approval, and records its audience/destination policy and actor/system basis. Distribution-requested never equals delivered
- only Notification Delivery may report recipient/channel success
- protocol-approved is the sole approval event identity; no duplicate officialized/approved event pair is produced.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration | One test per readiness blocker, reviewer-confirmation absence, reviewer/approver permission combinations, revocation/MFA state, stale/concurrent/double approval, canonical-event uniqueness, coordinated activation success/rollback and unconfirmed-candidate exclusion, audit/outbox/artifact write failures, transaction rollback, and official/source immutability; plus manual and configured-automatic publication, awaiting/authorized/distribution-requested/failed transitions, not-approved denial, unauthorized destination/audience, stale official revision, duplicate/concurrent publish, publication-audit/outbox failure, notification materialization/delivery outage, partial recipient failure, no false distributed-success state, and immutable published content. |

---

### M5-T05 — Low-Confidence Queue Backend

**Priority:** P1

**Human-review gate:** Required — product/AI review of uncertainty semantics and governance review of resolution actions.

**Description**

Provide the global, evidence-linked queue for low-confidence and unresolved candidates and feed its resolution state into whole-protocol readiness.

**Upstream references:** Spec §§7, 10; Design §§5, 10.

**Owning module/component:** Protocol Review & Approval.

**In scope:**

- List low-confidence cards separately from the main draft, grouped/filterable by agenda/topic while retaining a whole-meeting queue.
- Return proposed classification, qualitative confidence, available flag reason, missing/unclear fields, and exact source passage/speaker/time-or-page/full-context pointer. Numeric provider scores are optional technical detail only.
- Support the settled resolution actions through M5-T03 and preserve reviewer, timestamp, reason, and resulting draft linkage. Dismissed cards remain traceable.
- Calculate readiness from explicit card states. Do not choose the system threshold or normalization mapping in this task; consume the versioned confidence-normalization policy selected under Design §16.

**Out of scope:**

- Hard-coded example threshold, customer-configurable thresholds, deleting dismissed evidence, automatic promotion, AI reasoning explanations, or formal approval.

**Dependencies:** Hard prerequisite: M5-T03. This task contributes a hard readiness condition to M5-T04 and enables M5-T07.

**Acceptance Criteria**

- Every flagged candidate appears exactly once
- unresolved cards block readiness
- resolved/dismissed cards remain auditable and source-linked
- no low-confidence item is presented or promoted as a confirmed decision/task without reviewer action.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration | Below/at/above policy boundary using an injected policy version, missing confidence, each resolution action, dismissal reason, duplicate candidate, agenda-less topic, concurrent resolution, revocation, and readiness recalculation. |

---

### M5-T06 — Approval-Stage In-App Notifications

**Priority:** P1

**Human-review gate:** Required — product event semantics, authorization, and payload-minimization review.

**Description**

Integrate and verify the Protocol Review producer-event contracts used by downstream notification handling without duplicating their source transitions.

**Upstream references:** Spec §7; Design §§6, 10, 13.

**Owning module/component:** Protocol Review & Approval transition tasks own event production; this task owns event-contract registration and integration verification. Notification Delivery owns later recipient intents, transport, and delivery state.

**In scope:**

- Register the event schemas emitted by M5-T03 and M5-T04: draft-ready, returned-for-revision, canonical protocol-approved, protocol-publication-awaiting, protocol-publication-authorized, protocol-distribution-requested, and protocol-publication-failed. Relay occurs only after commit; this task must not append a second event for any source transition.
- Carry only recipient-resolution context; Notification Delivery resolves recipients from current per-body reviewer/approver access when it materializes its own notification intents rather than copying stale membership into the producer event.
- Include stable draft/protocol/body/meeting identifiers, event identity, state, and occurred time; exclude protocol text, source passages, prompts, and provider payloads.
- Verify producer outbox persistence is atomic/idempotent with each triggering state and that event identity is canonical across replay. Integrate with M4-T07 conventions without implementing M7 recipient/channel mappings, email, preferences, or notification center.

**Out of scope:**

- Emitting transition events, low-confidence notifications absent a product recipient/channel contract, SMTP/email topology, notification preferences, delivery retries/cleanup owned by M7, publication distribution, or embedding full protocol/source content.

**Dependencies:** Hard prerequisites: M5-T04, M4-T07. Parallelizable related work: M5-T07 UI can consume local intent behavior once stable.

**Acceptance Criteria**

- Each valid upstream transition is observed as exactly one producer outbox event and this integration creates none
- rolled-back/failed approval produces none
- a committed event survives Notification Delivery outage
- revoked recipients cannot retrieve protected details after materialization
- payload contains no governed content
- protocol-approved is canonical and no officialized/approved duplicate can pass contract validation.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Integration / operational | Reviewable and returned-for-revision transitions; successful/failed/double approval; awaiting/authorized/distribution-requested/publication-failed transitions; canonical-event uniqueness; transaction/outbox rollback; post-commit relay; duplicate relay/materialization; proof that M5-T06 emits no duplicate; recipient permission change; cross-body recipient; Notification Delivery outage/recovery; and content/secret canary inspection. |

---

### M5-T07 — Draft Review UI — Section-Level Review Cards

**Priority:** P1

**Human-review gate:** Required — reviewer workflow, source-trace integrity, Hebrew/RTL, responsive, accessibility, and permission review.

**Description**

Deliver the evidence-first protocol review experience across agenda/topic sections and the global low-confidence queue.

**Upstream references:** Spec §§7, 10, 11; Design §§10, 12.

**Owning module/component:** Protocol Review & Approval UI.

**In scope:**

- On desktop, keep source/transcript and structured draft visible together; organize by agenda/topic with whole-meeting overview and a global flagged queue. On smaller screens, use sequential panels without losing review position/context.
- Show immutable AI original, separate human edit, qualitative uncertainty/reason, missing fields, exact source passage/speaker/timestamp-or-page, and direct jump to full source for every section/card.
- Support M5-T03 review actions and M5-T05 queue resolution with reasoned dismissal. Clearly distinguish saved, unsaved, stale-conflict, access-revoked, and failed states.
- Preserve keyboard/focus behavior across source jumps, filters, edits, and panel changes; do not let optional numeric confidence dominate or imply certainty.

**Out of scope:**

- Formal approval authority, automatic classification/promotion, AI reasoning display, editable source/AI originals, publication, voting/quorum, or task management.

**Dependencies:** Hard prerequisites: M5-T03, M5-T05, M-UX-T05. Parallelizable related work: M5-T06 event integration.

**Acceptance Criteria**

- A reviewer can verify and resolve every item without losing provenance
- immutable fields cannot be edited
- global and section views agree
- stale/revoked states protect work and data
- desktop and small-screen Hebrew RTL flows are coherent and accessible.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration / E2E | Component/end-to-end journeys for agenda and detected topic, flagged/missing fields, all resolution actions, source jump, recording timestamp/page references, edit conflict, revocation mid-review, keyboard/focus, screen reader labels, RTL/BiDi, and responsive sequencing. |

---

### M5-T08 — Formal Approval UI

**Priority:** P1

**Human-review gate:** Required — approver governance workflow, security, Hebrew/RTL, accessibility, and destructive-action confirmation review.

**Description**

Present whole-protocol readiness, a deliberate formal approval action, and separate post-approval publication authorization, distribution request, and delivery state backed by M5-T04 and Notification Delivery.

**Upstream references:** Spec §§7, 9, 10, 11; Design §§8, 10, 12.

**Owning module/component:** Protocol Review & Approval UI.

**In scope:**

- Show meeting/body identity, reviewer completeness confirmation, section totals, unresolved/low-confidence and missing-field blockers, source/provenance status, and current officialization readiness.
- Enable approval only for a currently authorized approver and still submit explicit body context and expected draft revision; the backend remains authoritative if state changes.
- Require a deliberate confirmation that clearly states approval applies to the complete protocol and creates an official record. On success, navigate to or identify the immutable official artifact.
- Distinguish 401, 403/revocation/MFA, stale draft, readiness failure, audit/transaction failure, duplicate approval, and transport uncertainty without falsely reporting success.
- After approval, show awaiting publication, publication authorized/published, and distribution requested as Protocol Review & Approval truth. Separately show Notification Delivery's per-recipient/channel pending, delivered, partially failed, and failed outcomes; never label a request as successfully distributed. Offer the publish/distribute action only when the current body configuration requires an authorized user; show configured automatic behavior without simulating client-side publication or delivery.
- Before a manual publish/distribute command, show the configured audience/destination and require deliberate confirmation. Notification materialization/channel failure is shown separately from approval and publication authorization.

**Out of scope:**

- Section-by-section formal approval, automatic approval, client-invented publication configuration, task/decision activation UI, voting/quorum, or modifying the official artifact.

**Dependencies:** Hard prerequisites: M5-T04, M5-T06, M5-T07. Parallelizable related work: none after readiness and event behavior stabilize.

**Acceptance Criteria**

- Approval cannot be completed or represented as successful while any blocker exists
- reviewer-only users cannot approve
- approver-only users can inspect readiness without receiving review-edit authority
- retry after uncertain transport does not create a second artifact. Approved-but-unpublished is visible, unauthorized users cannot publish, and manual or configured-automatic publication converges on the same authoritative state. Distribution-requested, recipient pending/delivered/failed, publication failure, and approval failure remain distinct.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Integration / operational | Ready and every blocked state, reviewer/approver/both permission matrix, revocation/MFA change between render and submit, stale revision, audit/transaction failure, duplicate/ambiguous response recovery, approved-awaiting-publication, manual/configured-automatic publication, unauthorized or stale publish, destination confirmation, distribution requested with pending/partial/success/failure delivery, no false distributed-success label, publication failure and notification-delivery failure, keyboard/focus, screen reader, RTL/BiDi, and responsive confirmation. |

---

## M6 — Decisions, Tasks, and Minimal Operational View

*7 tasks. Hard prerequisites and safe parallel work are stated per task.*

**Critical path:** Follow the hard prerequisites stated per task; work identified as parallelizable may proceed concurrently.

**Purpose:** Close the first demonstrable governance slice by turning protocol evidence into durable decisions and human-confirmed tasks, then exposing the minimum operational view needed to act on them. A human-confirmed candidate may be activated through the coordinated officialization transaction or afterward; an unconfirmed AI candidate always remains advisory. This milestone contains no voting, quorum, ballot, or tally behavior.

**Prerequisites and parallelism:** M6 begins after the M5 approval transaction contract, M2 governance context, M1A authorization boundary, and M4 normalized-result contract are available. Schema and API work proceeds in dependency order. The confirmation owner integrates with M5's officialization coordinator for same-transaction activation and also exposes the later post-officialization path; UI work may begin once the minimal-view contract stabilizes, while overdue-event work can proceed beside the view. M7 consumes M6 events and projections but does not own task lifecycle rules.

**Exit gate:** From one protocol, an authorized user can create a source-linked durable decision and explicitly confirm an immutable AI task suggestion into an active or awaiting-assignment task either atomically with officialization or afterward. Both timings converge on the same authoritative task and producer events; rollback leaves neither a partial officialization nor a partial activation, and an unconfirmed candidate never becomes authoritative. The user can then manage the governed lifecycle and inspect the result in a minimal Hebrew-capable operational view. The first demo ends only after these paths, authorization checks, audit guarantees, and failure tests pass.

---

### M6-T01 — Decision & Task Schema

**Priority:** P0

**Human-review gate:** Architecture, Security, and Product/Governance

**Description**

Establish authoritative decision, AI task-candidate, confirmed task, assignment, deadline/review-date, collaborator, lifecycle, extension, escalation, and source-link records.

**Upstream references:** Spec §§4, 7, 8, 9; Design §§4, 8, 10, 13.

**Owning module/component:** Decisions, Tasks & Return Loop owns all authoritative writes; Content & Provenance supplies immutable source references and Audit supplies the durable audit boundary.

**In scope:**

- Keep the immutable AI suggestion separate from human-confirmed wording and fields. Support assignment to a person, a governance role, or an explicit awaiting-assignment state
- preserve original and current deadline/review values, provenance, version/concurrency state, and distinct decision/task lifecycles. Database constraints must keep every record rooted in organization, governance body, approved protocol, and source evidence where applicable.

**Out of scope:**

- Voting, quorum, ballots, tallies, automatic agenda construction, notification transport, custom workflow engines, or invented deadline/grace defaults.

**Dependencies:** Hard prerequisites: M5-T04 and M2-T01. Parallelizable related work: M6-T02 API-contract design may proceed after the ownership and state model stabilizes.

**Acceptance Criteria**

- The schema represents every required confirmation field without making an AI value authoritative
- invalid lifecycle combinations and orphaned/cross-body links are rejected
- role assignments retain the role identity independently of its current holder
- immutable source and AI layers cannot be overwritten by later human edits.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration | Migration and repository tests cover empty/partial fields, person/role/awaiting-assignment variants, cross-organization/body references, invalid transitions, concurrent versions, immutable-column attempts, rollback, and least-privilege writes. |

---

### M6-T02 — Decision Creation API

**Priority:** P0

**Human-review gate:** Security and Product/Governance

**Description**

Provide the authorized command surface that creates and maintains official decisions from approved protocol evidence without bypassing the Decisions owner.

**Upstream references:** Spec §§4, 6, 7, 8, 9; Design §§6, 8, 10, 13.

**Owning module/component:** Decisions, Tasks & Return Loop business service and repository; the API layer handles transport only.

**In scope:**

- Require a currently authorized actor, approved protocol, governance-body context, decision wording/status, and exact source reference. Creation, correction/supersession, and any official-state transition must be explicit, concurrency-safe, provenance-preserving, and atomically audited
- historical wording remains inspectable rather than overwritten.

**Out of scope:**

- AI-generated official decisions without review, voting/quorum semantics, task activation, hard deletion, notification delivery, or a second writer to decision tables.

**Dependencies:** Hard prerequisites: M6-T01 and M1A-T04. Parallelizable related work: M6-T03 can define candidate-confirmation payloads after the decision contract is stable.

**Acceptance Criteria**

- Authorized creation returns the durable decision and source link
- unapproved protocols, inaccessible bodies, stale versions, invalid sources, and missing audit persistence produce stable failure responses with no partial official record
- correction preserves prior official facts and actor/time provenance.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration | Always-on API/service tests cover create, duplicate/idempotent retry, authorization denial, exists-but-wrong-body source, unapproved protocol, stale update, audit failure, transaction rollback, and immutable history. Integration tests verify DAL enforcement cannot be bypassed through alternate entry points. |

---

### M6-T03 — AI Task Suggestion Intake & Human Confirmation API

**Priority:** P1

**Human-review gate:** Product/Governance, Security, and Provider where real model output is used

**Description**

Accept source-linked AI task suggestions and require explicit human confirmation of all authoritative task fields before activation, whether confirmation is coordinated with protocol officialization or occurs afterward.

**Upstream references:** Spec §§7, 8, 9, 10; Design §§5, 6, 8, 10, 13.

**Owning module/component:** Decisions, Tasks & Return Loop owns confirmation; AI Job Orchestration supplies normalized advisory candidate envelopes without writing official tasks, and Provider Gateway remains the provider-adapter owner.

**In scope:**

- Preserve the original suggestion, provider/model provenance, confidence, and exact protocol passage. A human confirms or edits wording, person-or-role owner or explicit awaiting assignment, deadline, reporting/review date, and collaborators
- omission of an owner must never silently become an active unassigned task. Revalidation is required immediately before confirmation.
- Support two owner-controlled activation timings without changing the confirmation rules: participate through the owner interface in M5's coordinated officialization transaction for candidates already human-confirmed before commit, or activate after an existing official protocol through the same idempotent command. The coordinator calls this owner; M5 never writes task tables.
- When confirmation creates an active task with a resolved person/role owner, append the canonical task-assigned producer event atomically with activation. Awaiting-assignment creates no task-assigned event. Preserve a single idempotency identity so retry or movement between the two timings cannot create a second task or event.

**Out of scope:**

- Automatic activation of unconfirmed candidates, inferred owner/deadline values, replacing human judgment with confidence thresholds, voting, notification delivery, or model-provider selection.

**Dependencies:** Hard prerequisites: M6-T01, M6-T02, M4-T05, and the M5-T04 approval transaction contract. Parallelizable related work: M6-T04 transition design and M6-T06 UI interaction design may proceed against the stabilized contract.

**Acceptance Criteria**

- A candidate cannot become confirmed without a positive human action and every required authoritative disposition
- edits do not mutate the AI layer
- inaccessible/revoked source context, stale candidate versions, or invalid decision links fail without activation
- the audit record distinguishes suggestion, reviewer edits, and confirmation
- coordinated and later activation produce the same authoritative result; transaction failure produces neither partial officialization nor partial activation
- initial active assignment creates exactly one task-assigned event, while awaiting-assignment and unconfirmed candidates create none.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration / provider validation | Deterministic tests cover accept-as-suggested; edited confirmation; person/role/awaiting-assignment; coordinated officialization success and rollback; later activation; same idempotency identity across both timings; canonical initial task-assigned event/no event for awaiting assignment; missing required dispositions; stale/replayed request; permission revocation; source mismatch; low confidence; audit failure; and concurrent reviewers. Provider-dependent quality checks run only in the approved model-evaluation gate. |

---

### M6-T04 — Task Lifecycle & Assignment API

**Priority:** P1

**Human-review gate:** Product/Governance and Security

**Description**

Implement the authoritative task lifecycle, assignment, reassignment, reporting, completion, extension, and escalation commands for confirmed tasks.

**Upstream references:** Spec §§7, 9; Design §§6, 8, 13.

**Owning module/component:** Decisions, Tasks & Return Loop exclusively owns task state and deadline rules; notification and dashboard modules consume its durable outcomes.

**In scope:**

- Enforce valid transitions and optimistic concurrency
- resolve person/role assignments through current body membership without erasing the assigned role. A role-holder change preserves responsibility and triggers review. Only the Community Manager can approve an extension
- retain original deadline and reason. A later assignment of an awaiting-assignment task, or a reassignment after confirmation, appends one canonical task-assigned producer event atomically with that assignment transition; the initial confirmation event remains owned by M6-T03. Critical overdue handling has no grace delay and emits simultaneous assignee/Community Manager intent.

**Out of scope:**

- Notification-channel implementation, hard-coded grace/deadline values, custom roles, automated extension approval, voting, or dashboard-owned state mutation.

**Dependencies:** Hard prerequisite: M6-T03. Parallelizable related work: M6-T05, T06, and T07 after command/event contracts stabilize.

**Acceptance Criteria**

- Every state change is authorized, durably audited, idempotent where retried, and reflected once in the task history
- later assignment/reassignment produces exactly one task-assigned event without duplicating the initial confirmation event
- awaiting-assignment tasks cannot masquerade as assigned
- unauthorized extension/reassignment or invalid transition changes nothing
- role-holder changes are visible for review rather than silently transferring accountability.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration | State-machine tests cover every allowed and forbidden edge; person/role/awaiting variants; initial-versus-later task-assigned ownership; reassignment idempotency; role-holder change; stale version; duplicate request; concurrent completion/extension; unauthorized Community Manager action; critical flag; audit failure; and rollback. |

---

### M6-T05 — Minimal Operational View Backend

**Priority:** P1

**Human-review gate:** Product/Governance and Security

**Description**

Expose the smallest permission-safe read projection needed to inspect durable decisions and active tasks at the first-demo boundary.

**Upstream references:** Spec §§7, 9; Design §§6, 8.

**Owning module/component:** Decisions, Tasks & Return Loop owns the projection contract; its repository performs query-side filtering/sorting and current-state authorization.

**In scope:**

- Return source-linked decisions and tasks with current status, owner/person-or-role or awaiting assignment, deadline/review date, critical/overdue state, and permitted actions. Body access is enforced in the data path
- projection lag or stale caches cannot grant access. This is a focused M6 view, not the cross-domain M7 dashboard.

**Out of scope:**

- Five-area operational dashboard, unresolved-matter tracker, search/semantic retrieval, cross-body aggregate leakage, workflow mutations, or invented SLA metrics.

**Dependencies:** Hard prerequisite: M6-T04. Parallelizable related work: M6-T07 and M7 dashboard contract design.

**Acceptance Criteria**

- Authorized users see only records from accessible bodies with truthful lifecycle/assignment data and navigable source provenance
- revoked users immediately lose results
- filters and pagination operate in the query
- empty and partial states are explicit.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration | API/repository tests cover mixed-body data, revoked access, role change, awaiting assignment, overdue/critical states, empty results, sorting/filter/pagination boundaries, stale projection/cache, and injection attempts. |

---

### M6-T06 — Decisions & Tasks UI

**Priority:** P1

**Human-review gate:** UX/Accessibility, Product/Governance, and Security

**Description**

Provide Hebrew/RTL-capable interfaces for reviewing decisions, confirming AI task candidates, operating task lifecycle commands, and using the minimal operational view.

**Upstream references:** Spec §§7, 9, 10, 11; Design §§8, 10, 12.

**Owning module/component:** Frontend feature surfaces consume M6 APIs through the M-UX foundation; domain decisions remain in Decisions, Tasks & Return Loop.

**In scope:**

- Visually separate immutable AI suggestion/source evidence from editable human-confirmed fields. Make person, role, and awaiting-assignment states unambiguous
- expose lifecycle actions only when permitted but still rely on server authorization
- handle loading, empty, stale, conflict, revoked-access, and failure states without losing edits or implying success.

**Out of scope:**

- Voting UI, custom dashboard builder, notification center, hidden client-side authorization, browser-stored auth tokens, or automatic confirmation.

**Dependencies:** Hard prerequisites: M6-T05 and M-UX-T05. Parallelizable related work: M6-T07 backend event work.

**Acceptance Criteria**

- A keyboard-only Hebrew user can complete the first-demo flow, inspect exact provenance, confirm/edit all task fields, understand critical/overdue and extension states, and recover from validation/concurrency errors. UI and API representations agree after refresh and role/permission change.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration / E2E | Component and browser tests cover RTL/BiDi text, keyboard/focus, labels/errors, suggestion-versus-confirmed distinction, all assignment variants, optimistic conflict, forbidden action, revoked session, network retry, responsive layout, and accessible status announcements. |

---

### M6-T07 — Overdue & Escalation Notification Events

**Priority:** P2

**Human-review gate:** Product/Governance, Architecture, and Operations

**Description**

Detect governed deadline and extension boundaries and emit durable, idempotent task-domain events named task-approaching-deadline, task-overdue, task-grace-escalated, task-critical-overdue, and extension-request-received. Extension outcome events may remain domain events but are not notification-mapped without a product recipient/channel contract.

**Upstream references:** Spec §§7, 9; Design §§2, 6, 8, 11, 13.

**Owning module/component:** Decisions, Tasks & Return Loop owns deadline evaluation and producer outbox events; Notification Delivery later materializes recipient intents and owns channels/retries.

**In scope:**

- Use the task's authoritative deadline/critical/extension state and a single approved source for the onboarding-configured reminder and grace policies. Atomically append an approaching-deadline outbox event at the configured boundary
- standard overdue first targets the owner, then escalates to the Community Manager after the configured grace period without response
- critical overdue targets owner and Community Manager simultaneously with no grace. Append extension-request-received when the request is durably accepted. Extension outcome state remains truthful in the task domain, but Notification Delivery must not infer outcome recipients or channels absent a Spec contract. Extension invalidates obsolete scheduled work without erasing original history.

**Out of scope:**

- Hard-coded grace periods, SMTP/SSE implementation, notification preferences, business rules in the delivery service, voting, or treating an outbox event or notification intent as delivered notification.

**Dependencies:** Hard prerequisites: M6-T04 and M4-T07. Parallelizable related work: M7-T01/T02 may define the generic delivery model against this event contract.

**Acceptance Criteria**

- Each applicable reminder, deadline, escalation, and request-receipt boundary produces one durable, correlated producer outbox event under retry/concurrency
- early, completed, extended, or already-acknowledged tasks do not produce obsolete reminders/escalation
- scheduling failure is visible and recoverable
- notification materialization/transport failure changes neither task state nor source-event truth.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Integration / operational | Use a controllable clock to test just-before/at/after reminder, deadline, and grace boundaries; approaching-deadline owner mapping; critical simultaneous recipients; extension-request receipt; unmapped extension outcomes; completion/extension races; role-owner resolution; missing assignee; duplicate worker delivery; restart recovery; event/audit persistence failure; and configured-policy changes without embedding a numeric value. |

---

## M7 — Notifications, Dashboard, and Unresolved Matters

*11 tasks. Hard prerequisites and safe parallel work are stated per task.*

**Critical path:** Follow the hard prerequisites stated per task; work identified as parallelizable may proceed concurrently.

**Purpose:** Add durable in-app/email notification delivery, a read-only operational dashboard, and the explicit unresolved-matter return loop. M7 does not own search, semantic retrieval, historical import, or any upstream governance state transition.

**Prerequisites and parallelism:** Notification foundations depend on M0 audit/persistence and can integrate M4–M6 event producers once their contracts exist. Dashboard and unresolved-tracker backends may proceed in parallel with delivery work after M6/M5/M2 projections stabilize; their UI follows the M-UX shell and notification UI. SMTP topology and outage UX remain human-gated open decisions.

**Exit gate:** Durable notification intents reach authorized recipients through an in-app inbox and the approved email path with retry/failure visibility; the five required operational areas are permission-safe; and a Community Manager can explicitly confirm an unresolved matter's future return and track its linked resolution. Delivery failures never roll back completed domain actions, and notification cleanup cannot destroy audit or governance truth.

---

### M7-T01 — Notification Schema & Delivery Model

**Priority:** P0

**Human-review gate:** Architecture, Security, and Product/Governance

**Description**

Define the durable notification, recipient visibility, delivery-attempt, channel, and correlation model plus the namespaced event-type registry used by all producers.

**Upstream references:** Spec §§7, 8, 9; Design §§2, 4, 6, 8, 13.

**Owning module/component:** Notification Delivery owns notification/delivery records; each domain owner still owns and emits its business outcome.

**In scope:**

- Treat the committed producer outbox event as immutable external input, not a Notification-owned row. Materialize one Notification-owned recipient intent independently from per-channel attempts and per-recipient visibility. Preserve source event identity for idempotency, organization/body/resource context, minimized render data, delivery/read state, failure classification, and correlation without copying governed source content. Use the M0 typed event registry and append-only audit boundary.

**Out of scope:**

- Domain escalation rules, SMTP/SSE transport, search, historical import, fixed retention periods, or copying complete protocol/document/task content into notification rows.

**Dependencies:** Hard prerequisites: M0-T03 and M0-T10. Parallelizable related work: producer event mapping for M7-T03 and SMTP decision analysis for M7-T11.

**Acceptance Criteria**

- Duplicate source events cannot create duplicate recipient intents
- cross-body recipients are structurally invalid
- channel attempts do not mutate domain state
- delivery/read/failure states are distinguishable and traceable
- protected content is referenced and resolved only after authorization.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration | Migration/repository tests cover duplicate idempotency keys, multiple recipients/channels, invalid state transitions, cross-organization/body data, least-privilege writes, missing source, retry history, redaction, and transaction rollback. |

---

### M7-T02 — Notification Service & Dispatch API

**Priority:** P0

**Human-review gate:** Architecture, Security, and Product/Governance

**Description**

Implement the single service that consumes committed producer outbox events, resolves currently permitted recipients/channels, materializes notification intents, exposes the inbox API, and dispatches delivery work.

**Upstream references:** Spec §§7, 9; Design §§2, 6, 8, 11, 13.

**Owning module/component:** Notification Delivery business service/repository; API and worker adapters call this owner rather than writing notification storage directly.

**In scope:**

- Consume only committed producer outbox events and materialize Notification-owned recipient intents idempotently after commit. Recheck current authorization when materializing and reading protected notification detail
- deny by default when event type/channel policy is absent. Enforce user preferences without suppressing required domain/audit events, queue retryable attempts durably, make list/read operations permission-safe, and keep recipient materialization/channel delivery asynchronous from the completed domain transaction.

**Out of scope:**

- Reimplementing task/protocol/import business rules, provider-specific mail topology, notification-producing database triggers, search, or treating successful enqueue as successful delivery.

**Dependencies:** Hard prerequisite: M7-T01. Parallelizable related work: M7-T03, T04, T09, and T10 after the service contract stabilizes.

**Acceptance Criteria**

- Valid intents create the intended recipient records once
- unauthorized/revoked recipients cannot retrieve protected detail
- unknown types/channels fail closed
- a dispatch outage leaves recoverable pending work and does not reverse the source action
- APIs distinguish authentication, authorization, validation, and unavailable-delivery failures.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration | Service/API tests cover duplicates, mixed-body recipients, revoked access, unknown event type, preference combinations, retryable/non-retryable failure, worker restart, concurrent dispatch, stale cached membership, and persistence/audit failure. |

---

### M7-T03 — Notification Backfill — Prior Milestone Events

**Priority:** P1

**Human-review gate:** Architecture and Product/Governance

**Description**

Map only the producer events that have an approved recipient/channel contract in Spec §7.12 through the M7 dispatch owner, without changing upstream domain semantics.

**Upstream references:** Spec §7; Design §§2, 6, 13.

**Owning module/component:** Notification Delivery owns adapters/mappings; M4, M5, M6, and the M7 unresolved-matter owner retain authoritative event production.

**In scope:**

- Map draft-ready to the currently authorized reviewer through in-app and email, and processing-failed to the uploading user through in-app and email.
- Map task-assigned, task-approaching-deadline, and task-overdue to the current task owner through in-app and email; task-grace-escalated to the Community Manager through both channels; task-critical-overdue to the responsible person and Community Manager simultaneously through both channels; and extension-request-received to the Community Manager through both channels.
- Map unresolved-matter-flagged to the Community Manager through in-app only. Reuse stable source-event identities so replay/backfill is safe.
- Explicitly leave protocol-approved, returned-for-revision, protocol-publication-awaiting, protocol-publication-authorized, protocol-publication-failed, protocol-distribution-requested, processing-success, low-confidence, revision, and extension-outcome events unmapped unless the product contract later defines their recipient and channel behavior.
- materialize Notification-owned intents only after commit, resolve content at read time, and do not create an alternate event bus or rewrite historical domain records.

**Out of scope:**

- Retrofitting M8 notification infrastructure, changing M4–M6 lifecycle rules, inventing recipients/channels for unmapped events, fabricating missing historical events, direct SMTP calls from producers, or search.

**Dependencies:** Hard prerequisites: M7-T02, M4-T07, M5-T06, and M6-T07. Parallelizable related work: M7-T04 delivery and T05 UI.

**Acceptance Criteria**

- Every mapped event has an explicit producer-owned outbox event, the exact Notification-owned recipient/channel policy from Spec §7.12, and an idempotency key
- replay produces no duplicate notification intents
- unsupported/ambiguous events are reported rather than guessed
- existing M4–M7 workflows remain correct while materialization/delivery is unavailable
- non-contracted producer events create no notification intents, and approval, publication authorization, distribution request, and per-recipient/channel delivery outcomes remain distinct.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Integration / operational | Contract tests replay representative and duplicate mapped events; cover draft-ready/reviewer/both channels, processing-failed/uploading-user/both channels, task-assigned/deadline/overdue owner mappings, grace/CM mapping, critical simultaneous responsible-person-and-CM mapping, extension-request/CM mapping, unresolved-flag/CM/in-app-only mapping, missing/revoked recipients, unknown versions/types, delivery outage/recovery, and prove no upstream domain state changes. Negative cases prove protocol approval/publication/return/revision, processing success, low-confidence, and extension-outcome events materialize no intents. |

---

### M7-T04 — Real-Time Notification Delivery (SSE) & SMTP Email

**Priority:** P1

**Human-review gate:** Security, Operations, and UX/Accessibility

**Description**

Deliver durable notifications through SSE with REST polling recovery and through the approved SMTP email adapter with retry and explicit failure status.

**Upstream references:** Spec §§7, 9, 11, 14; Design §§6, 7, 8, 11, 12.

**Owning module/component:** Notification Delivery adapters/workers; authentication/session ownership remains in M1A and SMTP configuration detail is completed by M7-T11.

**In scope:**

- SSE is an update signal, not the source of truth
- reconnecting clients recover from the durable inbox by polling. Email rendering contains the minimum approved information, respects current authorization/policy, records attempts and terminal/retryable failures, and never rolls back a completed governance action. Exact SMTP topology, retry values, and outage UX require their approved configuration source.

**Out of scope:**

- Treating SSE as durable storage, hard-coded retry/timing values, browser push/SMS/WhatsApp, domain rollback on email failure, or selecting SMTP topology without Operations/Security approval.

**Dependencies:** Hard prerequisites: M7-T02 and M4-T06. Parallelizable related work: M7-T05 and T11 after adapter contracts stabilize.

**Acceptance Criteria**

- Connected clients receive new-item signals and recover every missed item after disconnect/restart
- accepted SMTP attempts and failures are recorded distinctly
- transient outages leave retryable work
- permanent failure is actionable
- no event is exposed across users/bodies.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Integration / operational | Integration tests cover SSE reconnect/last-seen recovery, polling fallback, duplicate/out-of-order signals, session revocation, mixed users, SMTP accept/reject/timeout, retry/restart, malformed address, template escaping, sensitive-content minimization, and unavailable mail service. |

---

### M7-T05 — Notification UI — Bell & Notification Center

**Priority:** P1

**Human-review gate:** UX/Accessibility and Security

**Description**

Provide an accessible Hebrew/RTL bell and notification center backed by the durable inbox and resilient to SSE disconnection.

**Upstream references:** Spec §§7, 9, 11; Design §§6, 8, 10, 12.

**Owning module/component:** Frontend notification feature through the M-UX app shell; Notification Delivery remains the state owner.

**In scope:**

- Display unread/read/dismissible state supported by the service, event type, safe summary, time, delivery/failure state where relevant, and authorized navigation to the source. Announce new items accessibly without disruptive focus changes
- poll after disconnect
- handle loading, empty, error, stale, and revoked-access states.

**Out of scope:**

- Client-only notification truth, browser-stored authorization, custom inbox workflows, search, exposing raw governed content, or assuming SSE is always available.

**Dependencies:** Hard prerequisites: M7-T04 and M-UX-T03. Parallelizable related work: M7-T06/T07 backends.

**Acceptance Criteria**

- Keyboard and assistive-technology users can discover, inspect, navigate, and update notification state in Hebrew/RTL
- reconnect retrieves missed items once
- inaccessible source detail is withheld and clearly explained
- unread counts converge with the server after multi-tab/retry behavior.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration / E2E | Component/browser tests cover RTL/BiDi, focus/order/labels, unread/read/dismiss interactions, duplicate SSE, reconnect/poll, multi-tab refresh, revoked access, broken source, HTML escaping, network failure, responsive view, and accessible live-region behavior. |

---

### M7-T06 — Operational Dashboard Backend

**Priority:** P1

**Human-review gate:** Product/Governance, Security, and Architecture

**Description**

Build permission-safe read projections for the five predefined operational areas: tasks/execution, protocol status, upcoming meetings, unresolved/return items, and recent decisions.

**Upstream references:** Spec §§7, 9; Design §§4, 6, 8.

**Owning module/component:** Governance Overview owns read-only projections; source-domain owners remain the only writers and must handle any command reached from the dashboard.

**In scope:**

- Aggregate only currently authorized organization/body data and expose freshness/projection status. Tasks include assignment/status/overdue views
- protocol truth distinguishes in review, ready for approval, approved awaiting publication, and publication authorized/published. Distribution-requested and Notification-owned recipient/channel pending, delivered, partially failed, and failed states are displayed separately
- meetings show upcoming governed records
- unresolved data comes from its owner
- decisions retain source/status. Query-side filtering/pagination is required
- stale cache cannot grant access.

**Out of scope:**

- Custom dashboards/analytics, search, direct cross-domain writes, duplicated lifecycle calculations, data warehouse, or invented operational thresholds.

**Dependencies:** Hard prerequisites: M6-T05, M5-T04, and M2-T04. Parallelizable related work: M7-T07 and notification-delivery work.

**Acceptance Criteria**

- All five areas return truthful, source-linked, body-scoped data
- partial/empty areas are explicit
- permission revocation removes data immediately
- projection delay is visible and cannot authorize
- no dashboard request mutates a source aggregate.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration | Repository/API tests cover mixed bodies, revoked roles, each empty/partial area, overdue/awaiting assignment, every protocol approval/publication/delivery status including approved-awaiting-publication, upcoming ordering, unresolved linkage, stale projection/cache, pagination/filtering, and source-owner failure. |

---

### M7-T07 — Unresolved Matters Tracker Backend

**Priority:** P1

**Human-review gate:** Product/Governance and Security

**Description**

Implement the explicit unresolved-matter return loop from governed agenda/decision/task context to a Community Manager-confirmed future agenda target and linked resolution record.

**Upstream references:** Spec §§7, 8, 9; Design §§4, 6, 8, 13.

**Owning module/component:** Decisions, Tasks & Return Loop owns unresolved/return state; Governance Context owns meetings/agenda and is invoked for any agenda command.

**In scope:**

- Create a source-linked unresolved record only from authorized governed context and atomically append one producer-owned unresolved-matter-flagged outbox event carrying Community Manager recipient-resolution context. The Community Manager explicitly confirms or rejects a proposed return and selects a future meeting/agenda target or scheduled return date
- the system tracks pending, confirmed, returned, and resolved linkage without constructing an agenda automatically or duplicating meeting state. Notification materialization/delivery failure never changes the unresolved record or silently drops the durable source event.

**Out of scope:**

- Automatic agenda construction, AI-decided return scheduling, voting, generic case management, dashboard-owned writes, or invented return deadlines.

**Dependencies:** Hard prerequisites: M2-T05 and M6-T04. Parallelizable related work: M7-T06 dashboard projection and T08 UI after contracts stabilize.

**Acceptance Criteria**

- An unresolved matter preserves its origin and lifecycle history
- creation produces exactly one correlated producer outbox event for later Community Manager recipient resolution
- only an authorized Community Manager can confirm/reject a return
- invalid/inaccessible/past or otherwise ineligible target context is rejected according to Governance Context
- resolution links back to both origin and return occurrence
- audit/outbox persistence failure prevents creation while notification materialization/transport failure does not roll it back.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Integration / operational | State/API tests cover create plus unresolved-matter-flagged intent, Community Manager recipient mapping, duplicate/retry without duplicate intent, notification outage/recovery, confirm, reject, reschedule, return, resolve, concurrent confirmation, non-Community-Manager denial, revoked access, wrong-body meeting, missing target, source deletion/lifecycle change, and audit/intent rollback. |

---

### M7-T08 — Operational Dashboard & Unresolved Matters UI

**Priority:** P1

**Human-review gate:** UX/Accessibility, Product/Governance, and Security

**Description**

Render the five operational areas and the explicit unresolved-return workflow in an accessible Hebrew/RTL interface.

**Upstream references:** Spec §§7, 9, 11; Design §§6, 8, 10, 12.

**Owning module/component:** Frontend Governance Overview feature; source-domain APIs execute commands and Notification Delivery supplies notification state.

**In scope:**

- Use predefined sections with clear empty/error/freshness states and authorized links to sources. Unresolved actions show origin, proposed return, target meeting/date, confirmation consequence, and current status
- UI visibility is not authorization. Maintain keyboard/focus behavior, RTL/BiDi correctness, responsive hierarchy, and actionable feedback.

**Out of scope:**

- Custom dashboard designer, analytics warehouse, search, client-side business rules, automatic agenda mutation, or hiding unresolved records because notification delivery failed.

**Dependencies:** Hard prerequisites: M7-T06, M7-T07, and M7-T05. Parallelizable related work: M7-T09–T11 operational refinements.

**Acceptance Criteria**

- Authorized personas can understand current work across all five areas and complete a Community Manager return confirmation without losing context
- unauthorized actions fail safely and refresh state
- partial backend failure does not misrepresent other areas as empty/successful.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration / E2E | Component/browser tests cover each area, empty/loading/partial/error/freshness states, Hebrew/RTL/BiDi, keyboard/focus, responsive layout, return confirm/reject/conflict, permission revocation, broken source link, stale data, and accessible status/error announcements. |

---

### M7-T09 — Notification Preferences

**Priority:** P2

**Human-review gate:** Product/Governance, Security, and UX/Accessibility

**Description**

Let users configure approved per-event/channel notification preferences while preserving mandatory domain, audit, and governance behavior.

**Upstream references:** Spec §§7, 9, 11; Design §§4, 6, 8, 13.

**Owning module/component:** Notification Delivery owns preference records and enforcement; event-domain owners do not read preferences to decide whether an event exists.

**In scope:**

- Preferences are user-scoped, validated against the registered event/channel policy, concurrency-safe, and applied at dispatch. Missing or newly introduced policy fails closed to the approved default
- preferences cannot disable audit, alter task/protocol state, broaden access, or silently define whether critical events are optional—such product policy must be explicitly approved.

**Out of scope:**

- Invented quiet hours/digest schedules, organization-wide policy UI, muting domain events, custom channels, or hard-coding critical opt-out behavior without approval.

**Dependencies:** Hard prerequisite: M7-T02. Parallelizable related work: M7-T04/T05 and T10/T11.

**Acceptance Criteria**

- Users can retrieve/update only their preferences
- changes affect subsequent dispatch consistently
- unsupported event/channel combinations are rejected
- revoked access still suppresses protected detail regardless of preference
- no event or audit fact is lost when a channel is disabled.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration | API/service/UI tests cover defaults, supported combinations, partial updates, stale version, duplicate retry, another-user access, new/unknown event type, concurrent dispatch/update, revoked body access, and prove preferences cannot suppress domain/audit persistence. |

---

### M7-T10 — Notification Observability & Cleanup

**Priority:** P2

**Human-review gate:** Operations, Security, and Product/Governance

**Description**

Make notification backlog, attempts, failures, duplicates, and cleanup outcomes diagnosable while retaining only what approved policy permits.

**Upstream references:** Spec §§7, 8, 11, 14; Design §§6, 11, 12, 13.

**Owning module/component:** Notification Delivery operational instrumentation and cleanup; Audit and source-domain owners retain their own immutable records.

**In scope:**

- Emit local structured metrics/logs for pending age, attempt outcome, failure class, duplicate suppression, and cleanup without recording governed content or secrets. Cleanup is explicit, idempotent, scoped, auditable, and disabled until retention/deletion policy is approved
- it may remove eligible delivery artifacts but never audit facts or authoritative governance records.

**Out of scope:**

- Invented retention periods, remote observability requirement, raw notification/governance content in logs, silent deletion, domain-state cleanup, or redefining read/dismiss UX as this task's identity.

**Dependencies:** Hard prerequisites: M7-T02 and M0-T06. Parallelizable related work: M7-T09 and T11.

**Acceptance Criteria**

- Operators can distinguish no-intent, pending, delivered, retrying, permanent-failure, and cleanup-failure states by correlation
- sensitive content is absent
- cleanup dry-run/execute behavior reports exact eligible scope and respects legal hold/retention decisions once approved.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration | Instrumentation tests simulate backlog, duplicate, retry, permanent failure, malformed record, cleanup interruption/retry, audit failure, and redaction. Policy tests prove cleanup is a no-op while the retention and deletion decision in Spec §14 is unresolved and cannot delete domain/audit rows. |

---

### M7-T11 — SMTP Configuration & Delivery Observability

**Priority:** P2

**Human-review gate:** Operations and Security

**Description**

Complete the secure, operator-verifiable configuration and diagnostics for the approved SMTP delivery adapter.

**Upstream references:** Spec §§7, 9, 11, 14; Design §§6, 11, 12, 16.

**Owning module/component:** Notification Delivery SMTP adapter with Operations & Support configuration ownership.

**In scope:**

- Validate non-secret settings separately from credentials, keep secrets out of files/logs/status responses, provide readiness/diagnostic state without making SMTP a core local-readiness dependency, and correlate submission/acceptance/rejection/retry/failure. The approved deployment decides SMTP topology, sender identity, retry values, and outage UX.

**Out of scope:**

- Choosing a provider/topology without approval, embedding credentials, hard-coded retry/timeout values, making email mandatory for core startup, sending test governance content externally, or replacing M1A authentication-email ownership.

**Dependencies:** Hard prerequisite: M7-T04. Parallelizable related work: M7-T09 and T10.

**Acceptance Criteria**

- Missing/invalid configuration disables or fails the email capability explicitly without corrupting in-app delivery
- operators can verify connectivity and diagnose a failed message without viewing protected content
- configuration reload/restart behavior is documented and truthful
- credentials never appear in UI, logs, bundles, or errors.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration | Adapter/configuration tests cover missing and malformed settings, secret redaction, authentication/TLS failure, sender rejection, recipient rejection, timeout, retry/restart, local in-app continuity, diagnostic-bundle sanitation, and optional-service readiness behavior. |

---

## M8 — Historical Document Import & Knowledge Base Preparation

*9 tasks. Hard prerequisites and safe parallel work are stated per task.*

**Critical path:** Follow the hard prerequisites stated per task; work identified as parallelizable may proceed concurrently.

**Purpose:** Provide an authorized, provenance-preserving historical-import workflow in which AI suggests metadata, a human validates authoritative metadata and policy fields, and explicit activation prepares documents for later knowledge indexing. M8 also completes the canonical Content & Provenance lifecycle model and command surface used by every governed document, not only imported history. M8 does not build notification infrastructure, search, embeddings, or Q&A.

**Prerequisites and parallelism:** M8 reuses M0 storage/audit, M3 content safety, and M4 processing. Schema and authorization work can proceed together after ownership is agreed; historical ingestion and processing then feed classification, validation, curation activation, and the M9 handoff in order. M7 delivery may consume M8 events as a parallel integration but is neither import infrastructure nor a hard prerequisite. M9 infrastructure may be prepared in parallel; it consumes M8 activation for historical/otherwise policy-gated sources and authoritative approval/confirmation handoffs for official records.

**Exit gate:** An authorized coordinator can import a historical batch, observe per-document progress, preserve immutable source/provenance, review AI-suggested metadata, explicitly validate access/lifecycle/sensitivity/AI eligibility, and activate or deactivate curated historical/policy-gated content without deletion. Authorized users can apply the evidence-bearing lifecycle transitions to current uploads, official protocols, historical imports, and other governed documents through one owner. Validated and explicitly activated curated sources produce idempotent index-ready handoffs; approved official records remain eligible from their authoritative approval/confirmation event when access, sensitivity, and AI-eligibility policy permit. Unknown values and unresolved policy never become invented metadata or permissive access.

---

### M8-T01 — Import Schema & Provenance Model

**Priority:** P0

**Human-review gate:** Architecture, Security, and Product/Governance

**Description**

Establish import-batch and historical-classification state plus the canonical immutable source/provenance, access, complete document-lifecycle, activation, and handoff model shared by every governed document.

**Upstream references:** Spec §§7, 8, 9; Design §§4, 8, 13.

**Owning module/component:** Content & Provenance exclusively owns source/version and lifecycle state for all governed documents; AI Job Orchestration owns normalized advisory candidate envelopes and Provider Gateway owns provider-specific execution/egress.

**In scope:**

- For historical imports, record title, document type, date/unknown, governance body/committee, meeting reference, participants, effective period, related agenda topics, decisions and review dates, source, batch, importer, format, checksum, processing status, sensitivity, access scope, duplicate/relation assessment, and AI-eligibility disposition. For every governed document—including current uploads, approved official protocols, and imported history—model the five lifecycle states—uploaded/raw, pending processing, draft, approved/official, terminal—and the four explicit terminal statuses—superseded, archived, cancelled/revoked, partially amended—with version, replacement/amendment, effective-date, affected-scope, reason, and supporting-decision relationships. Other domains create or reference document layers only through this owner
- they do not duplicate lifecycle truth. Separate immutable source/provider/AI layers from human-validated authoritative metadata
- default unvalidated imports to restricted/inactive and preserve unknown rather than guessing. Activation/search eligibility is related to but not synonymous with lifecycle status.

**Out of scope:**

- Automatic public/searchable activation, search/vector schema, notification infrastructure, forced metadata inference, document deletion policy, or advanced OCR.

**Dependencies:** Hard prerequisites: none within M8. Parallelizable related work: M8-T04 authorization/audit design and T02 ingestion contract after schema ownership stabilizes.

**Acceptance Criteria**

- Every imported artifact has organization/batch/importer/checksum/provenance and the complete historical metadata shape
- every governed document has exactly one canonical lifecycle state/status plus immutable source/version relationships and explicit activation state where applicable. Unknown historical title/date/body/meeting/participants/effective-period/topics/decision/review-date/status values can be represented without fabrication
- AI suggestions cannot overwrite human metadata
- invalid lifecycle transitions, cross-body/access/replacement/amendment combinations, self/cyclic replacement, and duplicate handoff identities are constrained across document origins.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration | Migration/repository tests cover current upload, official protocol, and historical-import origins; every lifecycle and terminal status; required transition evidence; full and partial amendment relationships across eligible origins; missing/unknown historical metadata; duplicate checksums with distinct provenance; restricted default; immutable source/AI fields; invalid/cyclic transitions; cross-organization/body links; lifecycle-versus-activation independence; concurrent validation; rollback; and least-privilege writes. |

---

### M8-T02 — Bulk Import Ingestion API & CLI

**Priority:** P1

**Human-review gate:** Security, Operations, and Product/Governance

**Description**

Provide reusable service-backed API and operator CLI entry points for authorized coordinators to create and submit historical-import batches.

**Upstream references:** Spec §§7, 8, 9, 11; Design §§4, 6, 8, 11, 12, 13.

**Owning module/component:** Content & Provenance ingestion service; API and CLI are adapters using the same authorization, validation, storage, and audit logic.

**In scope:**

- Require current coordinator authorization for the target body, validate/stream files through M3/M0 storage safety, accept explicit known/unknown batch metadata, generate stable idempotency/checksum evidence, and create restricted inactive records before async processing. Partial batch failure is per-document and recoverable
- neither entry point bypasses audit or policy checks.

**Out of scope:**

- Community-Manager-only ingestion, direct filesystem paths, automatic processing success, notification infrastructure, search/index writes, invented batch/file limits, or separate CLI business logic.

**Dependencies:** Hard prerequisite: M8-T01. Parallelizable related work: M8-T04 and T05 contract/UI work.

**Acceptance Criteria**

- Authorized API and CLI submissions produce the same durable batch/document states
- duplicates and retries are reported rather than silently copied
- invalid, malicious, oversized-by-approved-policy, unsupported, or interrupted items do not corrupt successful items
- unauthorized Community Manager/coordinator assumptions are rejected—the permission is explicit, not role-name hard-coded.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration | Contract/integration tests cover API/CLI parity, mixed-validity batch, duplicate/replay, wrong-body coordinator, revoked access, traversal/polyglot/media mismatch, interrupted stream/storage outage, audit failure, worker enqueue failure, and restart-safe resumption. |

---

### M8-T03 — Historical Document Processing Pipeline Integration

**Priority:** P1

**Human-review gate:** Architecture, Security, and Provider where real processing is enabled

**Description**

Reuse the M4 asynchronous processing lifecycle for imported historical documents while preserving import-specific provenance and policy gates.

**Upstream references:** Spec §§7, 8, 9, 10; Design §§2, 4, 5, 8, 13.

**Owning module/component:** Content & Provenance requests processing; AI Job Orchestration owns job dispatch, checkpoints, status, and normalized candidate envelopes; Provider Gateway owns provider execution, protocols, and egress adapters.

**In scope:**

- Create idempotent jobs linked to import document/version and immutable source. Revalidate authorization, lifecycle, sensitivity, AI eligibility, and external-transmission policy before dispatch, before transmission, and before persistence. Reuse scanned-document detection and report OCR-required rather than claiming advanced OCR
- failure leaves the source available for correction/retry.

**Out of scope:**

- New parallel AI pipeline, advanced OCR, automatic activation/classification acceptance, M9 embedding/search, M7 notification infrastructure, or provider selection.

**Dependencies:** Hard prerequisites: M8-T01 and M4-T01. Parallelizable related work: M8-T04 and T05; T06 begins once normalized results are available.

**Acceptance Criteria**

- Eligible imported documents follow the M4 status model with import correlation
- restricted/ineligible/revoked documents do not reach a provider
- duplicate delivery does not duplicate results
- provider/policy failure is visible per document and does not activate content.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration / provider validation | Integration tests cover eligible text, scanned detection, unsupported input, duplicate job, restart/retry, provider timeout/failure, revoked access at each checkpoint, policy change mid-job, source revision mismatch, result-persistence/audit failure, and no-optional-provider degradation. |

---

### M8-T04 — Import Authorization & Audit Events

**Priority:** P1

**Human-review gate:** Security and Product/Governance

**Description**

Define and enforce authorized-coordinator permissions and complete per-batch/per-document audit coverage for every sensitive import action.

**Upstream references:** Spec §§5, 7, 9; Design §§8, 13.

**Owning module/component:** Content & Provenance enforces import commands through M1A DAL authorization; Audit owns durable event storage.

**In scope:**

- Permission is explicit per organization/body and is not inferred solely from Community Manager identity. Audit creation/submission, source attachment, processing request/retry, AI result, metadata validation, access/sensitivity/eligibility change, activation/deactivation, duplicate disposition, and failure with minimized metadata. Critical policy/state changes fail if audit cannot persist.

**Out of scope:**

- CM-only shortcut, application-log audit replacement, permissive batch-wide authorization after per-document scope changes, search authorization, or copying document content into audit.

**Dependencies:** Hard prerequisite: M8-T01. Parallelizable related work: M8-T02 and T03.

**Acceptance Criteria**

- Every entry point produces the same authorization result and required audit facts
- cross-body, revoked, expired-session, and support-access paths fail closed
- batch summaries remain traceable to per-document outcomes
- sensitive content, provider payloads, and secrets are absent from audit metadata.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration | Permission-matrix/API/CLI tests cover coordinator/non-coordinator, correct/wrong body, revocation, support scope, concurrent policy change, each audited action, audit failure rollback, failure-event behavior, metadata redaction, and read-only audit access. |

---

### M8-T05 — Import Status & Progress UI

**Priority:** P2

**Human-review gate:** UX/Accessibility, Security, and Product/Governance

**Description**

Give authorized coordinators a Hebrew/RTL batch and per-document view of ingestion, processing, validation, activation, retry, and failure status.

**Upstream references:** Spec §§7, 9, 11; Design §§6, 8, 10, 12.

**Owning module/component:** Frontend historical-import feature; Content & Provenance owns import/lifecycle truth and AI Job Orchestration owns job/status truth.

**In scope:**

- Clearly separate upload accepted, processing pending/running/failed, classification ready, human validation required, active/inactive, and index-handoff states. Show per-item actionable errors and safe retry where authorized
- progress is based on durable server state and polling/SSE signals, not client inference. Never imply that processing completion equals validation or activation.

**Out of scope:**

- Metadata approval by progress alone, search UI, notification infrastructure, client-only batch state, automatic activation, or exposing file paths/provider payloads.

**Dependencies:** Hard prerequisites: M8-T02 and M8-T04. Parallelizable related work: M8-T03/T06 backend processing.

**Acceptance Criteria**

- A coordinator can identify every failed/pending/needs-review item in a mixed batch and resume it without reimporting successful items
- revoked users lose detail immediately
- unknown metadata and restricted/inactive defaults are visible
- partial API failure is not rendered as completed.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration / E2E | Component/browser tests cover mixed batch states, reconnect/poll, duplicate submission, retry, revoked permission, wrong body, unknown metadata, scanned/OCR notice, provider outage, stale progress, RTL/BiDi, keyboard/focus, responsive and accessible status announcements. |

---

### M8-T06 — AI-Assisted Metadata Classification

**Priority:** P1

**Human-review gate:** Product/Governance, Security, and Provider

**Description**

Produce immutable, source-linked advisory metadata suggestions for processed historical documents without changing authoritative metadata or access.

**Upstream references:** Spec §§7, 8, 10; Design §§4, 5, 8, 13.

**Owning module/component:** AI Job Orchestration owns normalized candidate envelopes; Provider Gateway owns provider execution/egress adapters; Content & Provenance owns subsequent human validation.

**In scope:**

- Suggest document title, document type, date, governance body/committee, meeting reference, participants, draft/approved status, effective period, related agenda topics, decisions and review dates, lifecycle/replacement/amendment cues, sensitivity/access warnings, duplicate cues, and AI-eligibility recommendation only where the approved model contract supports them. Store model/version/prompt-policy/source revision, confidence/evidence, and unknown/insufficient outcomes
- do not infer permission grants, lifecycle transitions, or erase existing human values.

**Out of scope:**

- Automatic authoritative metadata, access broadening, lifecycle activation, invented confidence thresholds, provider selection, M9 embeddings, or notification infrastructure.

**Dependencies:** Hard prerequisites: M8-T03 and M4-T05. Parallelizable related work: M8-T07 review-contract/UI design.

**Acceptance Criteria**

- Each suggestion is traceable to an exact source/version and clearly advisory
- unsupported/low-confidence fields remain unknown or flagged for review
- reprocessing creates a new candidate version without overwriting prior AI/human layers
- revoked or ineligible content produces no new provider transmission.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration / provider validation | Deterministic adapter/storage tests cover every historical metadata candidate, lifecycle/replacement/amendment cue, per-field unknown/low confidence, malformed/model-failure output, duplicate/retry, source-version mismatch, policy revocation, prompt injection in source text, immutable history, and redaction. Representative Hebrew model-quality evaluation runs only at the human-gated provider suite. |

---

### M8-T07 — Human Validation of AI Classifications

**Priority:** P1

**Human-review gate:** Product/Governance and Security

**Description**

Require an authorized human to validate historical metadata/policy dispositions and provide the canonical evidence-bearing lifecycle command surface for every governed document.

**Upstream references:** Spec §§5, 7, 8, 9, 10; Design §§4, 8, 10, 13.

**Owning module/component:** Content & Provenance owns historical validation state and the only lifecycle command service for all governed documents; AI suggestions are read-only inputs.

**In scope:**

- For historical imports, present source evidence beside AI suggestions and existing authoritative metadata. The reviewer explicitly disposes title, type, date, body/committee, meeting reference, participants, effective period, related agenda topics, decisions/review dates, duplicate/version relationship, lifecycle state/status, sensitivity, access scope, and AI eligibility
- unknown is allowed where factually unknown. For every governed document origin, every lifecycle transition—including superseded, archived, cancelled/revoked, and partially amended—is a separate human-confirmed command through the same Content & Provenance service, recording actor/time, prior/new status, effective date, reason, supporting decision/approval, replacement where applicable, and whole-document or provision-level affected scope. M3 ingestion and M5 approval/publication call or emit into this owner rather than implementing a second lifecycle state machine. Default access remains restricted, every broadened access/sensitivity/eligibility decision is authorized and audited, and stale source/candidate versions require review again.

**Out of scope:**

- Bulk accept without explicit field dispositions, AI-decided permissions, automatic activation, forced invented dates/bodies, hard deletion, search, or lifecycle policy invention.

**Dependencies:** Hard prerequisite: M8-T06. Parallelizable related work: M8-T08 activation-contract design.

**Acceptance Criteria**

- Historical validation cannot complete with an undisposed required policy field
- accepted/edited/rejected/unknown outcomes are distinguishable and attributed
- missing factual metadata may remain explicitly unknown
- the AI record remains unchanged
- no AI suggestion silently changes lifecycle. Current uploads, official protocols, and historical documents use the same authorized transition rules and evidence
- incomplete transition evidence, unsupported origin-specific bypass, unauthorized broadening, stale review, concurrent conflict, or failed audit changes no authoritative state.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration | State/API/UI tests cover every historical field disposition and unknown; current-upload, official-protocol, and historical-import lifecycle commands; all five lifecycle states and four terminal transitions; full/partial amendment scope; replacement required/not-applicable; missing effective date/reason/supporting decision; AI-suggested accept/reject; restricted default; duplicate conflict; cross-origin relationship denial; access broadening denial; stale source/candidate; concurrent reviewers; revoked permission; prompt-injected suggestion text escaping; audit failure; and revalidation after change. |

---

### M8-T08 — Knowledge Base Activation Gating

**Priority:** P1

**Human-review gate:** Product/Governance and Security

**Description**

Make Q&A-corpus eligibility an explicit, authorized activation/deactivation decision for historical or otherwise policy-gated sources after completed validation, without adding a second approval gate to official records.

**Upstream references:** Spec §§7, 8, 9; Design §§4, 8, 9, 13.

**Owning module/component:** Content & Provenance exclusively owns activation state and emits durable lifecycle events; retrieval/indexing only consumes that state.

**In scope:**

- Explicit activation applies to imported historical or other policy-gated sources and requires current validation of source, authoritative metadata, access, lifecycle, sensitivity, duplicate disposition where applicable, and AI eligibility. An official protocol/decision/task becomes Q&A-corpus eligible from its authoritative approval/confirmation event when the same current policies permit
- it must not require a second M8 activation. Keyword/filter search eligibility is evaluated independently across all currently authorized records. Explicit deactivation withdraws the source from Knowledge Retrieval and emits an idempotent invalidation without deleting source/history
- direct authorized audit/source access may remain outside Knowledge Retrieval. Superseded/archived content that has not been explicitly deactivated remains historically retrievable/citable with terminal labels and lower rank when access and AI eligibility permit
- cancelled/revoked follows its explicit policy. Partial amendment preserves affected scope. Recheck current authorization and fail the critical state change if audit persistence fails.

**Out of scope:**

- Automatic activation after processing/validation, physical deletion, indexing/search implementation, cache-based authorization, notification infrastructure, or making active synonymous with public.

**Dependencies:** Hard prerequisite: M8-T07. Parallelizable related work: M8-T09 handoff mapping and M9-T01 infrastructure planning.

**Acceptance Criteria**

- Historical/policy-gated sources require explicit validated activation
- approved official records require no second activation
- keyword/filter search does not depend on Q&A activation. Lifecycle and activation remain independently truthful. Explicitly deactivated sources are absent from Knowledge Retrieval but may remain directly accessible for authorized audit
- superseded/archived sources that remain corpus-eligible are retrievable/citable behind active material
- cancelled/revoked behavior follows explicit policy
- partial amendments retain provision-level context. Reactivation creates a new event, and duplicate/retry/concurrent commands settle to one truthful state with complete history.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration / E2E | State/service tests cover historical/policy-gated activate/deactivate/reactivate; approved official protocol/decision/task eligibility without second activation; keyword/filter independence; missing/stale validation; policy change; inaccessible body; every terminal status; explicit deactivation absent from Knowledge Retrieval but retained for authorized direct audit access; superseded/archived retrieval and active-first ranking; cancelled/revoked policy; partial-amendment scope; duplicate/concurrent lifecycle and activation commands; downstream outage; audit failure; and preserved source/history. |

---

### M8-T09 — Knowledge Base Indexing Handoff

**Priority:** P2

**Human-review gate:** Architecture, Security, and Product/Governance

**Description**

Produce durable, idempotent index-ready, status-change, and invalidation payloads that let M9 derive indexes from currently eligible governed content of every origin.

**Upstream references:** Spec §§7, 8, 9; Design §§4, 8, 9.

**Owning module/component:** Content & Provenance owns handoff publication; Knowledge Retrieval owns all derived index writes and consumption.

**In scope:**

- Identify organization/body/document/source revision, lifecycle version, source-class corpus-eligibility basis/version (authoritative approval/confirmation or explicit curation activation), permitted access-scope reference, sensitivity/AI-eligibility state, provenance, checksum, and event identity without embedding a permission snapshot that can later grant access. Publish approval/confirmation, curation activation, revision, lifecycle-status change, deactivation, and policy-change handoffs durably and replayably. Access revocation, explicit knowledge deactivation, or AI-ineligibility requests invalidation
- supersession/archival requests status-aware reindexing so corpus-eligible, authorized historical evidence remains retrievable but ranked behind active material. Failure leaves an observable pending handoff.

**Out of scope:**

- pgvector/index writes, chunking/embedding, search/Q&A, durable permission grants in payloads, notification infrastructure, or invented index-lag thresholds.

**Dependencies:** Hard prerequisite: M8-T08. Parallelizable related work: M9-T01 schema/infrastructure can be prepared against the versioned contract.

**Acceptance Criteria**

- Each eligible revision yields one replay-safe index-ready event carrying its source-class eligibility basis
- approved official records do not require a curation-activation version. Access revocation/deactivation/ineligibility yields invalidation
- supersession/archival yields status-aware reindex without silent removal when corpus eligibility, access, and AI eligibility still permit historical retrieval
- out-of-order/duplicate events reconcile by source/lifecycle/eligibility version
- missing/ambiguous access or eligibility metadata fails closed
- no index record is written by M8.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Integration / operational | Contract tests cover activate/deactivate/reactivate, access revocation, superseded/archived status-aware reindex, cancelled/revoked and partial-amendment policy outcomes, revision replacement, duplicate/out-of-order delivery, worker restart, missing policy metadata, source checksum mismatch, cross-body event, publish failure/recovery, and current-state reconciliation. |

---

## M9 — Semantic/Keyword Search and AI Q&A

*6 tasks. Hard prerequisites and safe parallel work are stated per task.*

**Critical path:** Follow the hard prerequisites stated per task; work identified as parallelizable may proceed concurrently.

**Purpose:** Deliver permission-safe keyword/filter search across all currently authorized records plus semantic retrieval and Hebrew cited Q&A over the product-defined corpus of approved official records and explicitly activated curated sources. M9 owns retrieval and derived indexes; it has no M4 processing-quality dependency and no M7 search dependency.

**Prerequisites and parallelism:** M9 consumes authoritative approval/confirmation handoffs from official source domains plus M8's versioned curation/lifecycle handoffs and reuses M1A current-state authorization. pgvector infrastructure precedes embedding and semantic search; keyword/filter search over authoritative records does not wait for Q&A activation or embeddings. Index monitoring follows the embedding pipeline. M9-T04 was deliberately removed before approval, so the numbering gap is preserved.

**Exit gate:** Authorized users can search by keyword, semantic meaning, and required filters and can ask Hebrew questions that return only directly cited, currently accessible evidence with status/conflict/insufficient-information handling. Authorization is rechecked before retrieval, before context assembly, and before response; revocation is immediate, and stale/missing indexes can reduce results but can never broaden access.

---

### M9-T01 — pgvector Index Schema & Infrastructure

**Priority:** P0

**Human-review gate:** Architecture, Security, and Operations

**Description**

Establish the PostgreSQL/pgvector-derived index schema, migration, ownership, source-version linkage, and rebuild-safe infrastructure for M9 retrieval.

**Upstream references:** Spec §§7, 8, 9; Design §§4, 8, 9, 11.

**Owning module/component:** Knowledge Retrieval exclusively owns chunk/vector/keyword-derived records; Content & Provenance remains authoritative for source/access/lifecycle.

**In scope:**

- Link every derived row to organization/body/source/revision and index-generation state
- support invalidation/rebuild without mutating source. The migration uses M0 tooling and existing pgvector extension. Authorization is never delegated to vector metadata or cached scope. Exact chunk, embedding dimension/index type, ranking, and lag choices remain semantic-retrieval and provider decisions under Design §16.

**Out of scope:**

- External vector database, provider/model selection, hard-coded semantic-retrieval values, source-of-truth permissions in vectors, M4 result-quality coupling, or search UI.

**Dependencies:** Hard prerequisite: M8-T09. Parallelizable related work: M9-T02/T03 contract design after schema boundaries stabilize.

**Acceptance Criteria**

- Derived records cannot be orphaned or cross-linked
- active revision replacement/invalidation is representable
- schema can coexist with keyword/filter indexes
- runtime writers have least privilege
- empty/rebuild states are explicit and do not make source unavailable.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Integration / operational | Migration/repository tests cover empty/upgrade/repeat migration, extension absence, revision/invalidation, cross-body links, duplicate generation, least privilege, concurrent rebuild writes, cleanup of obsolete derived rows, and rollback. |

---

### M9-T02 — Document Embedding & Indexing Pipeline

**Priority:** P1

**Human-review gate:** Architecture, Security, Provider, and Operations

**Description**

Consume authoritative approved/confirmed source-domain handoffs and M8 curation/lifecycle handoffs to build/rebuild keyword and semantic derived representations asynchronously and remove/relabel/invalidate them when the relevant eligibility changes.

**Upstream references:** Spec §§7, 8, 9, 10, 12; Design §§4, 5, 8, 9, 11, 13.

**Owning module/component:** Knowledge Retrieval owns the indexing pipeline and derived rows; Provider Gateway supplies approved embedding adapters/egress while Content & Provenance supplies current source policy.

**In scope:**

- For semantic/Q&A indexing, require either authoritative approval/confirmation or explicit curation activation according to source class, then revalidate lifecycle, sensitivity, AI eligibility, and external-transmission policy before dispatch/transmission/persistence. For keyword/filter search, derive only what is needed from all currently authorized records without using Q&A activation as an inclusion gate. Chunk/index exact source revisions with provenance, source-class/eligibility basis, idempotent jobs, generation markers, retry/restart, invalidation, and an operator reindex interface. Index output is derived and replaceable
- current text remains available for authorized keyword/filter behavior during rebuild where supported.

**Out of scope:**

- M4 processing-quality feedback loop, automatic source activation, external vector database, invented chunk/rank/lag values, authorization from cached vectors, or Q&A generation.

**Dependencies:** Hard prerequisite: M9-T01. Parallelizable related work: M9-T03 and T06; T07 follows usable embedding/search contracts.

**Acceptance Criteria**

- Eligible revisions converge to one current derived generation
- approved official records need no second activation
- historical/policy-gated sources do. Access-revoked, explicitly deactivated, or AI-ineligible content is removed from semantic/Q&A retrieval as policy requires, while explicit deactivation does not destroy direct source/audit access. Superseded/archived content receives a status-aware generation and remains retrievable/citable behind active material when corpus eligibility, access, and AI eligibility still permit it
- keyword/filter coverage remains independent of Q&A activation. Duplicate/out-of-order handoffs and worker restarts do not mix generations
- provider failure is visible/retryable and cannot activate or alter source
- operator reindex is scoped and auditable.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration / provider validation | Pipeline tests cover activation/deactivation races, access revocation, superseded/archived status-aware reindex and lower-ranking metadata, cancelled/revoked and partial-amendment outcomes, authorized historical retrieval, duplicate/out-of-order events, provider timeout/malformed result, revocation at all checkpoints, source revision change, restart/retry, partial generation, concurrent reindex, prompt-like document content, and no-provider local degradation. Provider quality runs in the approved external suite. |

---

### M9-T03 — Knowledge Base Search API (Semantic + Keyword + Filter)

**Priority:** P1

**Human-review gate:** Architecture, Security, and Product/Governance

**Description**

Provide one permission-safe search contract combining semantic and keyword retrieval with filters over all authorized knowledge content.

**Upstream references:** Spec §§7, 9, 10; Design §§4, 6, 8, 9.

**Owning module/component:** Knowledge Retrieval search service/repositories; source-domain owners supply authoritative status and provenance.

**In scope:**

- Search authorized protocols, decisions, tasks, documents, and agenda material with keyword text and filters for body, date/time, type, status/lifecycle, and other approved metadata
- combine available semantic ranking without making it mandatory for keyword/filter results. Enforce current DAL authorization before candidate retrieval and again before result materialization
- resolve citations/status from authoritative source revisions.

**Out of scope:**

- Search in M7, client-side permission filtering, vector-only access control, hidden fallback that changes semantics, M4 quality coupling, Q&A answer generation, or invented ranking thresholds.

**Dependencies:** Hard prerequisite: M9-T01. Parallelizable related work: M9-T02 improves semantic results; M9-T05 UI follows the stable response contract.

**Acceptance Criteria**

- Keyword, semantic, filter-only, and combined queries return only currently authorized records with exact source links and truthful lifecycle status
- active material ranks first while authorized superseded/archived material remains searchable and citable
- revoked access removes results immediately
- stale/missing vector generation lowers recall or marks semantic unavailable but cannot expose or corrupt keyword results
- invalid filters fail clearly.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration / provider validation | API/repository tests cover each retrieval mode, active-first ranking, superseded/archived search and citation, terminal-status labels, Hebrew forms, empty/partial index, mixed body/permission data, revocation, stale cache/index, source revision/status change, conflicting filters, pagination/ranking determinism under fixed inputs, injection, malformed query, and semantic-provider outage. |

---

### M9-T05 — Knowledge Base Search UI

**Priority:** P1

**Human-review gate:** UX/Accessibility, Security, and Product/Governance

**Description**

Provide an accessible Hebrew/RTL interface for keyword, semantic, filter-only, and combined knowledge search with direct source navigation.

**Upstream references:** Spec §§7, 9, 11; Design §§6, 8, 9, 10, 12.

**Owning module/component:** Frontend Knowledge Retrieval feature through the M-UX foundation; the server owns authorization, ranking, filtering, and source truth.

**In scope:**

- Make search mode/filters, active constraints, status/lifecycle, body/date/type, semantic-unavailable state, result provenance, and exact source link understandable. Preserve Hebrew/RTL/BiDi correctness, keyboard/focus, responsive behavior, query state, empty/insufficient/error states, and safe navigation after permission changes.

**Out of scope:**

- Client-side ranking/authorization, custom analytics, M7 dashboard search, Q&A answers, cached protected snippets after revocation, or invented relevance thresholds.

**Dependencies:** Hard prerequisite: M9-T03. Parallelizable related work: M9-T02/T06 pipeline health and M9-T07 Q&A.

**Acceptance Criteria**

- Users can independently run keyword and filtered searches and use semantic results when available
- filters are removable/visible
- inaccessible or revoked results disappear and protected detail is never rendered from stale client state
- partial semantic failure is distinguished from no matching content.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration / E2E | Component/browser tests cover all modes/filter combinations, Hebrew/RTL/BiDi, keyboard/focus, empty/partial/error/semantic-unavailable, stale request cancellation, pagination, direct citation navigation, permission revocation, malicious snippets/HTML escaping, responsive layout, and accessible result/status announcements. |

---

### M9-T06 — Index Health & Lag Monitoring

**Priority:** P2

**Human-review gate:** Operations, Security, and Architecture

**Description**

Expose sanitized, actionable evidence about indexing backlog, source/index revision divergence, invalidation, failures, and rebuild progress.

**Upstream references:** Spec §§7, 11; Design §§9, 11, 12.

**Owning module/component:** Knowledge Retrieval operational instrumentation, surfaced through Operations & Support conventions.

**In scope:**

- Correlate activated source generations with indexed generations and report pending/running/failed/stale/invalidating/rebuilding states without content, prompts, embeddings, or access details. Exact acceptable lag/alert thresholds remain an approved operational decision
- health never overrides current authorization and optional semantic failure does not make core local startup unready.

**Out of scope:**

- Invented lag SLA/alerts, source content/embeddings in telemetry, remote telemetry dependency, automatic source activation, authorization from health state, or Q&A quality metrics.

**Dependencies:** Hard prerequisite: M9-T02. Parallelizable related work: M9-T03/T05 and T07.

**Acceptance Criteria**

- Operators can identify affected source generation and retry/rebuild path
- stale and failed differ from legitimately inactive
- duplicate suppression and invalidation backlog are visible
- metrics/logs remain sanitized and accurate after restart
- no unapproved fixed threshold is embedded.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration / provider validation | Simulate backlog, stuck/failed/retried job, duplicate event, stale revision, deactivation/invalidation, rebuild, restart, provider outage, metric/log failure, and redaction. Verify optional-semantic outage behavior and configuration-driven threshold handling once the semantic-retrieval decision in Design §16 is approved. |

---

### M9-T07 — Hebrew AI Q&A with Source Citations

**Priority:** P1

**Human-review gate:** Product/Governance, Security, Provider, and UX/Accessibility

**Description**

Answer Hebrew questions from authorized knowledge evidence with direct citations, status/conflict context, and explicit insufficient-information behavior.

**Upstream references:** Spec §§7, 9, 10, 11; Design §§5, 6, 8, 9, 13.

**Owning module/component:** Knowledge Retrieval owns retrieval/context/citation validation and Q&A orchestration; Provider Gateway executes the approved model adapter/egress call; source-domain owners remain authoritative.

**In scope:**

- At each of three checkpoints—(1) before retrieval, (2) before context assembly/provider transmission, and (3) before response/citation release—re-evaluate current DAL authorization, document lifecycle/currentness, sensitivity, AI eligibility, approved/curated-content scope, and governing purpose/provider-transmission policy. At checkpoint 2, additionally validate the exact provider destination and purpose immediately before any external transmission
- at checkpoint 3, fail closed if any policy changed while generation ran. Treat retrieved content as untrusted, separate instructions from evidence, minimize context, require citations to exact accessible source passages, identify body/date/type/status, disclose conflicts/superseded facts, and return insufficient information when the remaining eligible evidence cannot support a reliable answer rather than fabricate. Revalidate every citation against the current source revision before release.

**Out of scope:**

- Uncited free-form advice, answering from model memory as governance fact, broadening access through embeddings/cache, transmitting unauthorized passages, inventing a confidence threshold, M4 quality feedback coupling, or treating Q&A as an official decision.

**Dependencies:** Hard prerequisites: M9-T02 and M9-T03. Parallelizable related work: M9-T05 UI can extend the stable search/source components; M9-T06 monitors index availability.

**Acceptance Criteria**

- Every substantive answer claim is grounded in a released direct citation from currently authorized, lifecycle-eligible, sensitivity-allowed, AI-eligible, approved/curated evidence
- no remaining eligible evidence yields an explicit insufficient-information answer
- conflicting/current-versus-superseded sources are labeled
- permission, lifecycle, sensitivity, eligibility, curation, provider, or purpose-policy change at any checkpoint prevents the affected transmission or release
- provider failure leaves no partial authoritative answer.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration / E2E | Deterministic tests independently revoke/change authorization, lifecycle, sensitivity, AI eligibility, approved/curated status, external-provider permission, and purpose policy at each of the three checkpoints; also cover mixed eligible/ineligible evidence, stale cache/index, citation-source revision change, provider-destination mismatch, missing/unsupported citation, conflicting sources, insufficient remaining evidence, prompt injection/data-exfiltration text, provider timeout/malformed answer, Hebrew/RTL rendering, and audit/log redaction. Representative Hebrew groundedness/citation quality runs at the human-gated provider evaluation. |

---

## M10 — Hardening, Pilot Acceptance, and Deployment Readiness

*9 tasks. Hard prerequisites and safe parallel work are stated per task.*

**Critical path:** Follow the hard prerequisites stated per task; work identified as parallelizable may proceed concurrently.

**Purpose:** Validate the completed M0–M9 product as a recoverable, supportable, secure, measurable pilot and obtain explicit stakeholder acceptance. M10 closes evidence and unresolved-policy gates; it does not hide missing product capability behind a runbook, test waiver, or known-issue entry.

**Prerequisites and parallelism:** M10 follows the implemented prior milestones, although security review, measurement design, migration fixtures, runbook rehearsal, and known-issue capture may begin earlier against stable contracts. Performance, end-to-end, migration, deployment, recovery/support, and measurement evidence can be gathered in parallel after T01 establishes the hardening baseline. Formal sign-off waits for every listed dependency and every release-blocking open decision.

**Exit gate:** The exact release candidate passes security, representative performance, full end-to-end, migration, deployment, backup/restore, upgrade/recovery, support-access, Hebrew/RTL/accessibility, and pilot-measurement validation. Stakeholders explicitly sign off the already-set scheduled-backup RPO of at most 24 hours, the measured provisional RTO, retention/deletion, external-provider, sensitive-body, SMTP/outage, support/availability, baseline/threshold, and unresolved-matter policies; there is no PITR claim or unresolved must-fix defect.

---

### M10-T01 — Security Hardening & Dependency Audit

**Priority:** P0

**Human-review gate:** Security, Architecture, and Release

**Description**

Perform threat-driven hardening of the exact release candidate and its dependencies across every exposed, privileged, provider, support, and governed-data path.

**Upstream references:** Spec §§9, 10, 11, 12; Design §§3, 5, 7, 8, 9, 11, 12, 13, 15.

**Owning module/component:** Security review across all module owners; each owner fixes findings in its authoritative boundary rather than through a parallel guard.

**In scope:**

- Review authentication/session/MFA/CSRF/rate limiting, DAL authorization, uploads/import, AI/provider transmission, prompt injection/Q&A, SMTP, secrets, audit, diagnostics/support, backup/restore, dependencies/images, headers/TLS/deployment exposure, and log/data minimization. Classify findings with reproducible evidence, trace guarantees through API/CLI/worker/cache/export paths, and fail release for unresolved must-fix risk.

**Out of scope:**

- Blanket scanner suppression, disabling security checks, test-only guards, unreviewed dependency upgrades, invented risk acceptance, pentest-complete claims without evidence, or shipping a workaround instead of the owning-boundary fix.

**Dependencies:** Hard prerequisites: implemented M0–M9 release candidate. Parallelizable related work: M10-T02–T05, T07–T09 may begin after the baseline/finding format is agreed.

**Acceptance Criteria**

- Every in-scope attack surface and dependency artifact has recorded review evidence
- exploitable authorization/data-exposure/secret/audit failures are corrected at the owning boundary and regression-tested
- accepted residual risks have named human approval
- scans and manifests are reproducible for the exact build.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Integration / operational | Run the approved dependency/container/static/dynamic checks plus adversarial tests for broken access control, session/CSRF, injection, malicious uploads/imports, prompt injection, revocation races, secrets/logs, support scope, backup artifacts, and audit-persistence failure. Pin exact contract outcomes rather than tool exit alone. |

---

### M10-T02 — Performance Baseline & Load Testing

**Priority:** P1

**Human-review gate:** Architecture, Operations, Product/Governance, and Release

**Description**

Establish a reproducible performance/capacity baseline for representative pilot workflows and datasets without inventing acceptance thresholds.

**Upstream references:** Spec §§3, 7, 11, 14; Design §§3, 5, 6, 9, 11, 15.

**Owning module/component:** Operations & Support coordinates; each logical owner measures and corrects its own hot path.

**In scope:**

- Measure authenticated intake/upload, AI job scheduling/status, protocol review/approval, task/dashboard, notification backlog, import, keyword/semantic search, cited Q&A, and concurrent governed reads/writes on representative Hebrew content and deployment resources. Record workload/data/build/configuration, latency/throughput/error/resource behavior, bottlenecks, and degradation
- thresholds and observation window must come from approved pilot policy.

**Out of scope:**

- invented load profile, arbitrary SLA/thresholds, production-content copy, benchmark-only code paths, remote scale infrastructure not in pilot architecture, or treating average latency as sufficient evidence.

**Dependencies:** Hard prerequisite: M10-T01. Parallelizable related work: M10-T03–T05 and T08–T09.

**Acceptance Criteria**

- Results are rerunnable and attributable to an exact release/configuration/dataset
- critical paths and saturation behavior are known
- optional-provider failure/backpressure does not corrupt unrelated workflows
- no pass/fail claim uses an unapproved numeric target
- regressions have an owned resolution or explicit release risk decision.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Integration / operational | Run scripted baseline/load/soak families at the approved run point, including concurrency, backlog/recovery, large representative import/search corpus, provider latency/failure, index rebuild, and database/storage pressure. Compare identical workload versions and validate result completeness/error accounting. |

---

### M10-T03 — End-to-End Acceptance Test Suite

**Priority:** P1

**Human-review gate:** Release, Product/Governance, Security, UX/Accessibility, Provider, and Operations

**Description**

Build the release-gating suite that proves the complete M0–M9 governed workflow and its critical failure/revocation boundaries.

**Upstream references:** Spec §§3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14; Design §§1, 3, 5, 7, 8, 9, 10, 11, 12, 13, 15, 16.

**Owning module/component:** Cross-functional acceptance harness; assertions remain against public contracts and authoritative state, not private implementation calls.

**In scope:**

- Cover provisioning/auth/MFA/body permissions, meeting/intake, AI processing, source-linked protocol review/approval, decision/task confirmation, deadline/escalation, notification/dashboard/unresolved return, historical import/validation/activation, search, and cited Q&A. Include human gates, Hebrew/RTL/BiDi/accessibility, provider/queue/storage/mail failures, duplicate/retry/concurrency, permission revocation, audit failure, and recovery-safe behavior.

**Out of scope:**

- Mock-only proof of provider/browser/restore properties, assertions that merely check field presence, skipped release-critical tests without approved risk acceptance, production data dependence, or using E2E tests to compensate for missing owner-level tests.

**Dependencies:** Hard prerequisite: M10-T01; cross-milestone prerequisites are the implemented M0–M9 candidate. Parallelizable related work: M10-T02, T04, T05, T08, and T09 evidence collection.

**Acceptance Criteria**

- The suite maps every critical spec/design guarantee to a named falsifiable scenario
- deterministic tests gate every release, provider/browser/restore suites have explicit pre-production run points, and failures identify the violated contract. No expected result is weakened to match observed behavior without resolving the source-of-truth conflict.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration / E2E | Exercise happy, error, boundary, state-dependent, exists-but-wrong-scope, and missing/partial-data cases, including the three Q&A auth checkpoints, critical/no-grace escalation, CM-only extension, unresolved return confirmation, audit atomicity, provider low-confidence/failure, stale index/cache, RTL/a11y, and email fallback. |

---

### M10-T04 — Data Migration & Seed Validation

**Priority:** P1

**Human-review gate:** Architecture, Operations, Security, and Release

**Description**

Prove clean provisioning, supported-version migration, reference/test fixtures, and historical-data paths produce a truthful release schema without conflating development seed with production data.

**Upstream references:** Spec §§7, 8, 11; Design §§3, 4, 11, 12, 13.

**Owning module/component:** Operations & Support executes versioned migrations; each domain owner validates its authoritative invariants.

**In scope:**

- Validate empty install and every supported source version through M0 migration/upgrade tooling
- verify extension, constraints, ownership, source/provider/AI/human/official layer preservation, import activation, derived-index rebuild, and deployment metadata. Production provisioning creates required roots through the runbook, never development seed
- derived data is rebuildable and not the migration oracle.

**Out of scope:**

- Auto-migration on normal startup, arbitrary unsupported-version conversion, production demo seed, destructive data rewrite without provenance, invented retention deletion, or declaring success from schema version alone.

**Dependencies:** Hard prerequisites: M0-T03 and M0-T11. Parallelizable related work: M10-T03, T05, and T08.

**Acceptance Criteria**

- Supported upgrades preserve row/file provenance and official history, reject incompatible/partial states, and leave readiness/version metadata truthful
- clean production provisioning contains no demo credentials/content
- fixture/reference data is deterministic and isolated
- failed migration follows the approved recovery path.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Integration / operational | In disposable environments, test empty and each supported upgrade, repeat/interrupt/fail migration, incompatible schema/extension, missing file, orphan/cross-body constraint, development-seed exclusion, activation/index rebuild, pre/post counts/checksums/invariants, and restore recovery. |

---

### M10-T05 — Deployment Runbook Validation

**Priority:** P1

**Human-review gate:** Operations, Security, and Release

**Description**

Rehearse and verify the complete production provisioning, configuration, startup, maintenance, backup, upgrade, recovery, diagnosis, and shutdown runbook for the release candidate.

**Upstream references:** Spec §§11, 13, 14; Design §§3, 11, 12, 13, 16.

**Owning module/component:** Operations & Support; module owners provide observable verification points rather than undocumented manual repair steps.

**In scope:**

- A qualified operator follows role-scoped commands from clean host through secrets/non-secrets configuration, Organization/bootstrap, health/readiness, optional provider/mail degradation, scheduled backup/off-host verification, upgrade lock, restore/recovery, support access, diagnostic bundle, and safe stop/start. Resolve deployment-specific open choices through approved configuration sources
- redact all output/evidence.

**Out of scope:**

- Customer secrets in documentation/evidence, unverified manual database edits, hidden developer-only recovery, invented host/SMTP/support choices, mandatory optional providers, PITR claims, or treating a checklist read-through as rehearsal.

**Dependencies:** Hard prerequisites: M0-T12 and M-OPS-T06. Parallelizable related work: M10-T03, T04, and T08.

**Acceptance Criteria**

- An operator not relying on developer knowledge completes the rehearsal and observes the documented outcomes
- every unsafe/failed state has a product-owned detection/recovery path
- no secret or governed content enters the runbook evidence
- optional external service loss does not destroy local core capability
- discrepancies are corrected before sign-off.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Integration / operational | Lint/check referenced commands/configuration and execute a disposable runbook smoke covering clean start, invalid/missing config, maintenance/readiness, restart persistence, backup failure, upgrade failure, restore, support expiry/revocation, diagnostic sanitation, and safe shutdown. |

---

### M10-T06 — Formal Acceptance Checklist & Stakeholder Sign-off

**Priority:** P1

**Human-review gate:** Release plus Product/Governance, Security, Architecture, Operations, Provider, and UX/Accessibility as applicable

**Description**

Consolidate objective release evidence and obtain explicit authorized acceptance of product behavior, operational capability, provider policy, pilot measures, and residual risk.

**Upstream references:** Spec §§3, 11, 14; Design §§5, 11, 13, 15, 16.

**Owning module/component:** Release governance; evidence is supplied by the responsible domain/technical owners and sign-off cannot substitute for missing verification.

**In scope:**

- Trace every spec/design acceptance area and open decision to evidence, decision owner, outcome, and residual risk. Explicitly validate—not reopen—the scheduled-backup pilot RPO of at most 24 hours
- record measured provisional RTO and no-PITR boundary. Require sign-off on retention/deletion/legal hold, external providers/data transmission, sensitive-body restrictions, confidence policy, MFA posture, SMTP/outage UX, availability/support, measurement baseline/threshold/window/duration, and unresolved-matter governance policy.

**Out of scope:**

- Implicit approval, auto-signing, invented policy values, reopening the settled RPO, converting provisional RTO to commitment without restore evidence, PITR claim, waiving must-fix defects, or treating generated prose as acceptance evidence.

**Dependencies:** Hard prerequisites: M10-T01, T02, T03, T04, T05, T07, T08, and T09. Parallelizable related work: none after formal acceptance begins.

**Acceptance Criteria**

- Every required gate has named authorized approval or a release-blocking unresolved status
- evidence references the exact release/build/configuration and is internally consistent
- no must-fix known issue/open decision remains
- RPO/RTO/PITR statements match demonstrated recovery
- accepted residual risks and pilot success criteria are explicit.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Integration / operational | Validate checklist completeness and referential integrity against spec/design open-decision registers, 113-task coverage, test/security/performance/recovery/measurement evidence, exact build identifiers, and required signatory roles; fail generation on missing, contradictory, stale, or unattached evidence. |

---

### M10-T07 — Known-Issue Register

**Priority:** P2

**Human-review gate:** Release, Security, Product/Governance, and Operations according to issue impact

**Description**

Maintain a truthful, reviewable register of discovered release-candidate defects, limitations, residual risks, affected scope, evidence, ownership, and acceptance disposition.

**Upstream references:** Spec §§11, 13, 14; Design §§11, 13, 15, 16.

**Owning module/component:** Release governance curates; each finding remains owned by the responsible product/technical component.

**In scope:**

- Record reproducible condition, observable impact, security/governance/data/recovery implications, workaround only if explicitly accepted, owning decision-maker, and resolution or risk-acceptance evidence. Link duplicates and preserve history
- distinguish defect, documented limitation, open policy decision, and future enhancement without assigning invented due dates.

**Out of scope:**

- Hiding defects as enhancements, default risk acceptance, invented resolution dates, workaround shipment without explicit approval, copying sensitive reproduction data, or using the register to defer missing in-scope capability.

**Dependencies:** Hard prerequisites: M10-T01, T02, and T03. Parallelizable related work: M10-T04, T05, T08, and T09 continuously contribute findings.

**Acceptance Criteria**

- Every failed/waived acceptance item appears once with complete traceability
- must-fix items block T06
- accepted residual risks name the authorized accepter and exact release scope
- closed items link corrective verification
- register content contains no secrets/governed source data.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration | Schema/lint checks reject missing severity/impact/owner/evidence/disposition, invalid build/task/test links, contradictory closed/open state, duplicate identity, secret patterns, and a T06-ready state while must-fix items remain. |

---

### M10-T08 — Operational Recovery & Support Validation

**Priority:** P1

**Human-review gate:** Operations, Security, and Release

**Description**

Demonstrate whole-deployment backup/restore, upgrade recovery, and time-limited support/diagnostic behavior on the exact release candidate.

**Upstream references:** Spec §§9, 11, 14; Design §§11, 12, 13, 15, 16.

**Owning module/component:** Operations & Support, invoking authoritative backup/upgrade/audit/support owners; M-OPS diagnostic bundle is reused rather than reimplemented.

**In scope:**

- Run a consistent scheduled-backup capture of PostgreSQL, referenced files, and sanitized configuration metadata
- verify encrypted off-host artifact
- restore into an isolated compatible deployment
- validate schema/files/readiness/governance invariants
- measure provisional RTO and demonstrate the at-most-24-hour RPO schedule. Exercise upgrade failure/recovery, support request/approval/scope/expiry/revocation, read-only audit review, and encrypted sanitized diagnostic export. Make no PITR claim.

**Out of scope:**

- WAL/PITR claims, unmeasured committed RTO, on-host-only disaster-recovery success, custom cryptography, standing support access, unapproved manual repair, secret-bearing `.env` copy, or destructive production rehearsal.

**Dependencies:** Hard prerequisites: M10-T03 and M-OPS-T05. Parallelizable related work: M10-T04 and T05 rehearsals and T07 issue capture.

**Acceptance Criteria**

- Restored data/files are consistent and usable, measured RTO evidence is recorded as provisional, RPO configuration/evidence is within 24 hours, off-host/encryption/integrity checks pass, unsafe version restore is rejected, support access cannot exceed approved scope/time and is fully audited, and diagnostic material exposes no secrets/governed content.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Integration / operational | Rehearse tampered/wrong-key/missing-file/incompatible-version/off-host failure, interrupted backup/restore, attempted writes during capture, upgrade readiness failure, support expiry/revocation/concurrency, audit failure, diagnostic decryption/access and redaction, plus post-restore E2E smoke and consistency checks. |

---

### M10-T09 — Pilot Measurement Instrumentation

**Priority:** P1

**Human-review gate:** Product/Governance, Architecture, Security, and Release

**Description**

Produce reproducible, privacy-minimized pilot evidence for adoption, AI quality/trust, workflow behavior, operational value, and commercial validation.

**Upstream references:** Spec §§3, 8, 11, 14; Design §§4, 6, 13, 15.

**Owning module/component:** Pilot Evidence owns measurement definitions/observations; source-domain owners emit minimized events without surrendering workflow truth.

**In scope:**

- Define versioned metric semantics, eligible populations, observation windows, provenance, deduplication, missing-data treatment, and access controls for the agreed dimensions. Derive counts/timings/review outcomes from authoritative events without copying governance content, prompts, passages, or personal data beyond approved necessity. Baseline, thresholds, pilot duration, qualitative method, and commercial criteria remain explicit human decisions.

**Out of scope:**

- Copying protocol/document/Q&A content into analytics, hard-coded baseline/threshold/window/duration, surveillance analytics, external telemetry dependency, metrics that become workflow truth, or declaring pilot success before T06 approval.

**Dependencies:** Hard prerequisite: M10-T01. Parallelizable related work: M10-T02/T03 validation and T07 issue capture.

**Acceptance Criteria**

- Each reported measure is reproducible from versioned definitions and traceable minimized observations
- retries/duplicates and missing telemetry do not inflate results
- security/privacy review approves the data set and retention/access plan
- dashboard/report distinguishes baseline, observation, and unapproved target
- no invented threshold is used.

**Test Plan**

| Type | What it asserts |
|------|----------------|
| Unit / integration | Metric-contract tests use independently computed fixtures for duplicate/retry, late/out-of-order event, missing field, version change, permission/role change, partial outage, and exact numerator/denominator/window boundaries. Redaction/access/retention-gate tests prove governed content and unauthorized personal data are absent. |
