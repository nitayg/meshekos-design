# MeshekOS — Technical Design Document

---

## 1. Overview and System Context

### 1.1 Purpose

MeshekOS is an AI-assisted governance operating system for kibbutz committees and governance bodies. The Community Manager is the primary daily user and cross-body operator. Coordinators, approvers, participants, and task assignees use narrower capabilities according to organization authority, governance-body permissions, and task context.

The system supports the complete pilot governance loop: meeting and source intake, asynchronous transcription and extraction, evidence-linked protocol review, formal approval, publication or distribution, durable decisions, human-confirmed tasks, escalation, unresolved-matter return, dashboard oversight, historical import, and permission-safe knowledge retrieval.

MeshekOS does not make governance decisions. AI output remains advisory and distinguishable from human edits and official records. An authorized human action is required before content becomes official or an AI-suggested task becomes active.

### 1.2 Target Users

| Role | Primary Responsibilities |
|---|---|
| Community Manager / Organization Administrator | Primary daily operator; cross-body oversight, permissions, escalations, unresolved-return decisions, and management outcomes |
| Administrative Coordinator | Meeting and document intake, operational coordination, protocol review when permitted, and historical import |
| Chairperson or Designated Approver | Formal whole-protocol approval and other separately granted governance actions for the relevant body |
| Governance-Body Participant | Reads permitted content and performs separately assigned review or other body actions |
| Task Assignee | A person or organizational role responsible for an assigned task; assignment grants only the required task context |

Persona names do not grant authority. Organization administration, body-specific permissions, and task assignments are separate records evaluated from current system state.

### 1.3 Core Design Principles

- **Human authority is mandatory.** AI candidates cannot become official records or active tasks without the required authorized human action.
- **AI assists, does not decide.** Source evidence, uncertainty, missing fields, and the immutable AI original remain visible through review.
- **One owner per domain rule.** Each authoritative record and transition has one module owner; other modules use its application interface or consume its durable events.
- **Provenance is preserved.** Original source, provider result, AI candidate, human edit, and approved official artifact are separate layers.
- **Authorization is current-state and resource-scoped.** A valid session or prior job decision is not continuing permission.
- **Local read access and AI/provider eligibility are independent.** Readable content may still be ineligible for AI use or external transmission.
- **Customer-controlled operation.** The dedicated local deployment starts and preserves governed records without mandatory external providers.
- **Audit is distinct from diagnostics.** Critical governance and security actions do not succeed if their required durable audit event cannot be persisted.

### 1.4 Scope Boundaries

The pilot architecture covers the full value loop defined in the product specification, including governance context, content intake, AI processing, review and approval, tasks and escalation, unresolved matters, notifications, dashboard projections, historical import and lifecycle, keyword/filter search, semantic retrieval, cited Q&A, and pilot measurement.

The architecture intentionally excludes shared-database multi-tenancy, microservices, Kubernetes, a dedicated vector database, WebSockets, arbitrary custom roles, voting or quorum management, automated agenda construction, mobile applications, advanced OCR, additional notification channels, and external business-system integrations. These are deferred scope, not seams that justify speculative pilot infrastructure.

---

## 2. Architecture Style and Module Boundaries

### 2.1 Architectural Decision

MeshekOS is a **modular monolith** with asynchronous worker processes and replaceable provider adapters. The browser API and workers are built from the same versioned application and domain modules. Logical boundaries are enforced inside one deployable product; they are not separate network services.

This is the smallest architecture that supports the full pilot loop while remaining operable in a customer-controlled on-premises environment. Extraction into services is considered only if measured scale, isolation, or deployment requirements justify the added operational cost.

The backend and frontend technology stacks remain explicit implementation gates. Whatever technologies are selected must preserve the ownership, transaction, security, Hebrew/RTL, accessibility, and verification constraints in this design.

### 2.2 Internal Modules

| Module | Sole Ownership |
|---|---|
| **Identity & Session** | Local accounts, credentials, MFA material, recovery codes, password recovery, account state, and server-side sessions |
| **Authorization** | Organization authority, governance-body permission assignments, narrow task-context grants, policy evaluation, and revocation semantics |
| **Governance Context** | Organization root, governance bodies, role holders, meetings, participants, agenda items, and required context linkage |
| **Content & Provenance** | Source intake, storage references, processing metadata, canonical document lifecycle, historical import validation/activation, versions, and amendments |
| **AI Job Orchestration** | Durable jobs and attempts, stage transitions, retry/cancellation/recovery, policy checkpoints, and normalized candidate envelopes |
| **Provider Gateway** | Transcription, extraction, embedding, and Q&A adapters; provider authentication; outbound policy enforcement; optional callback verification |
| **Protocol & Review** | Agenda/topic drafts, review cards, low-confidence resolution, human edits, whole-protocol approval, official artifacts, and publication authorization |
| **Decisions & Tasks** | Decisions, task candidates and confirmation, assignment, deadlines, extensions, escalation, and unresolved-return records |
| **Semantic Search & Q&A** | Keyword/filter retrieval, pgvector-derived representations, permission-safe semantic retrieval, cited Q&A, and index rebuild/lag state |
| **Notifications** | In-app notification records, email rendering and dispatch, retries, delivery state, and permitted channel preferences |
| **Governance Overview** | Read-only cross-domain projections for the Community Manager dashboard and narrower operational views |
| **Audit** | Registered event taxonomy, append-only audit persistence, minimized metadata, and permission-scoped inspection |
| **Operations & Support** | Deployment metadata, readiness, maintenance, backup/restore coordination, support sessions, and diagnostic bundles |
| **Pilot Evidence** | Versioned pilot metrics, baseline provenance, agreed outcome measures, and privacy-minimized reports |

Each module is the only writer of its authoritative records. Cross-module commands call the owning application service. Read models use published query interfaces or owned projections rather than making command decisions from another module's private tables.

Audit and Notifications remain separate: an audit fact is evidence that an action occurred; a notification is a derived delivery record and cannot serve as governance or audit truth.

### 2.3 Background Worker Topology

AI processing, notification delivery, indexing, scheduled task evaluation, and projection updates run outside browser requests. Workers:

- operate from durable PostgreSQL job or event state and complete independently of browser connectivity;
- use least-privilege runtime identities and call the same domain owners as synchronous entry points;
- treat job creation context as a scope reference, not continuing authorization;
- revalidate current authorization, lifecycle, sensitivity, AI eligibility, purpose, and destination at every consequential boundary;
- lease and acknowledge work with idempotent effects so retries cannot duplicate official artifacts, tasks, notifications, or index generations;
- expose pending, running, stalled, failed, and completed state without placing governed content in operational telemetry.

Domain owners atomically persist a state transition and a minimized producer-owned outbox event. Consumers process events at least once and deduplicate by stable event identity. The exact queue, scheduler, relay, leasing, retry, and backoff implementation remains an open technical decision; no additional broker is introduced without demonstrating that the PostgreSQL-backed baseline is inadequate.

---

## 3. Deployment Model and Tenant Isolation

### 3.1 Single-Tenant On-Premises Deployment

Each customer receives a dedicated deployment boundary containing:

- an application/API runtime and one or more workers from the same release;
- a dedicated PostgreSQL database;
- dedicated customer-controlled file/object storage;
- a locally served browser client;
- deployment-specific configuration and secrets;
- optional outbound connections to individually approved AI, transcription, email, and support endpoints.

The pilot uses a simple single-host container bundle. Separate physical servers, Kubernetes, a broker, remote telemetry, or cloud-managed storage are not required. Customer data is never shared with another customer deployment.

### 3.2 Organization/Deployment Root Entity

Every governed record belongs to an explicit **Organization** root even though a pilot deployment serves one organization. Governance bodies, users, meetings, sources, protocols, decisions, tasks, configuration, and audit events preserve that ownership chain.

Installed application release, schema compatibility, maintenance state, deployment identity, and upgrade state belong to a separate deployment-metadata record. They are operational facts, not attributes of the customer's governance organization.

This root provides consistent ownership, export, authorization, and future migration boundaries without implementing shared-database multi-tenancy.

### 3.3 External AI Processing

External provider use is optional and customer-controlled. Provider Gateway is the only owner permitted to send governed content to transcription, extraction, embedding, or Q&A providers. Local/private providers use the same adapter contracts.

Every transmission requires a currently approved provider capability and a current decision for resource authorization, source revision, sensitivity, AI eligibility, permitted purpose, allowed content class, and destination. A prior job decision or local read permission does not authorize external transmission. The payload is minimized for the purpose, and the policy decision and outcome are auditable without copying the content into logs.

Loss of external connectivity degrades only the affected provider-dependent capability. It does not prevent local access to sources, approved records, tasks, keyword/filter search, or operational recovery.

### 3.4 Packaging and Orchestration

The release is packaged as a reproducible single-host container bundle with explicit persistent storage, customer configuration, and secret boundaries. Application and worker containers share the same versioned code and schema contract. The deployment does not include unused placeholder services.

Routine start never changes schema. `/health` reports process liveness, while `/ready` evaluates whether the current application version may receive normal traffic based on database, storage, extension, schema compatibility, capacity, configuration, and maintenance state.

The exact supported host, storage, network, container tooling, and support-connectivity envelope must be fixed in the installation runbook before pilot deployment.

### 3.5 Deployment-Agnostic Codebase

Customer-specific behavior is configuration and governed data, not a code fork. Core modules depend on repository, storage, provider, mail, and support ports rather than customer-specific implementations. This portability permits deployment evolution without weakening the pilot's dedicated isolation model.

---

## 4. Data Model and Storage Strategy

### 4.1 Core Relational Model

**PostgreSQL** is the authoritative store for relational governance data, current authorization, workflow and job state, audit events, notifications, deployment metadata, and keyword/vector indexes.

The principal ownership relationships are:

```
Organization
├── Identity & Authorization: users, sessions, organization authority, body permissions
├── Governance Context: governance bodies, role holders, meetings, participants, agenda items
├── Content & Provenance: sources, versions, processing metadata, lifecycle and amendment links
├── Protocol & Review: drafts, sections, review cards, human edits, official artifacts
├── Decisions & Tasks: decisions, task candidates, assignments, deadlines, return links
├── Derived Records: notifications, overview projections, keyword/vector representations
└── Audit and Deployment Records
```

The tree describes ownership links, not shared write access. Each record has one module owner. Every governance-body resource also retains its body scope, and material records preserve their meeting and source linkage where required.

Official records and fields used for authorization, lifecycle, approval, escalation, filtering, or reporting are normalized relational data. JSONB is limited to versioned candidate envelopes, provider-specific material, and sparse metadata that has no stable cross-record meaning.

Four state concerns remain separate:

| Concern | Owner | Purpose |
|---|---|---|
| Intake and processing state | Content & Provenance / AI Job Orchestration | Upload, transcription, analysis, draft generation, ready, and failure progression |
| Document lifecycle | Content & Provenance | Raw, pending, draft, approved/official, superseded, archived, cancelled/revoked, or partially amended |
| Protocol review and approval | Protocol & Review | Section review progress, whole-protocol readiness, approval, publication authorization, and distribution request |
| Task execution | Decisions & Tasks | Candidate, confirmed, awaiting assignment, active, overdue, blocked, completed, extended, and escalated |

Transitions are explicit, authorized, concurrency-protected operations with actor, prior/new state, reason, context, and audit requirements.

### 4.2 File Storage

Large immutable sources and derived artifacts are stored in customer-controlled local file/object storage. PostgreSQL stores opaque storage keys and metadata rather than exposing raw filesystem paths.

Stored metadata includes integrity checksum, size, detected and declared media type, original filename, uploader, organization/body/meeting provenance, sensitivity, access scope, AI eligibility, lifecycle, source revision, and retention status where applicable.

Storage keys are generated server-side. Writes are streamed safely and verified before metadata commits. A compensating cleanup handles failed unreferenced writes. Original files remain preserved through processing failures, review, lifecycle transitions, and re-indexing.

### 4.3 AI Output Storage

The provenance layers remain independently identifiable:

1. immutable uploaded file or pasted-text snapshot;
2. provider transcript/result, with provider and source timing metadata;
3. immutable AI candidate/original, with model, prompt-policy version, uncertainty, and source references;
4. attributed human edits and review actions;
5. immutable approved official artifact created by the relevant governance owner.

A later layer references but never overwrites an earlier layer. Corrections create a new revision or attributed action.

Raw provider responses may be retained only when permitted by the final retention, sensitivity, and provider policy. They are not retained automatically and are never the governance source of truth. Normalized AI results are durable advisory candidates; only the approved official layer is an official record.

### 4.4 Semantic Search — pgvector

The pilot uses the PostgreSQL **pgvector** extension for semantic retrieval. A dedicated vector database is not part of the pilot architecture.

Every derived chunk references stable organization, body, source, revision, lifecycle, sensitivity, AI-eligibility, and corpus-eligibility metadata. Those fields support candidate filtering and reconciliation but do not grant access; current authoritative policy is rechecked at query time.

Keyword/filter retrieval and semantic/Q&A retrieval are distinct paths owned by the same module:

- keyword/filter search covers all currently authorized records and does not depend on Q&A activation or embedding availability;
- semantic/Q&A indexing accepts official records from their approval or confirmation event, and curated historical or otherwise policy-gated sources only after explicit human activation;
- approved official records do not require a second activation;
- explicit knowledge deactivation removes a source from Knowledge Retrieval without deleting source or audit history;
- superseded and archived sources remain searchable and citable behind active material when corpus eligibility, access, and AI eligibility permit it.

Embedding model, Hebrew chunking, vector index parameters, ranking blend, cache strategy, and lag threshold remain open until benchmarked on representative content.

### 4.5 Schema Migration Process

Database migrations never run during routine application startup. They execute only through a controlled version-upgrade procedure:

1. acquire an exclusive upgrade lock and enter maintenance/non-write mode for all writers;
2. validate source/target compatibility, current schema, required extensions, storage, disk, and configuration;
3. verify a current recoverable pre-upgrade backup;
4. obtain explicit operator approval;
5. run the versioned migration tool selected with the backend stack;
6. start the target release in non-write mode and validate `/health`, `/ready`, schema, storage, and representative workflows;
7. update deployment version metadata only after validation, then enable writes;
8. on failure, keep writes disabled until the release-specific reverse or isolated restore procedure completes.

The pgvector extension is enabled and verified through a versioned migration. Automatic down migrations are not assumed safe, and schema ownership credentials remain separate from application runtime credentials.

---

## 5. AI and Provider Integration Layer

### 5.1 Adapter Interface

Each transcription, extraction, embedding, or Q&A provider is implemented behind a normalized **Provider Gateway** adapter. Vendor SDK types and provider-specific protocols stop at this boundary.

The adapter contract covers:

- capability and provider identity;
- minimized governed input plus source revision and purpose;
- submit, poll, retrieve, cancel where supported, and health/capability description;
- normalized errors and provider-native result/uncertainty metadata;
- outbound authentication and optional callback verification.

AI Job Orchestration owns durable job policy, stage progression, retries, cancellation, and result promotion. Provider Gateway owns protocol execution and egress. Governance modules consume normalized candidates and never call vendor APIs directly.

Outbound submit, status polling, and result retrieval are the default flow. A completion webhook is optional only where the provider and customer network policy require it; it must be narrowly bound to one job and protected against forgery and replay.

### 5.2 Confidence Normalization

Provider-native confidence or uncertainty is preserved with the provider, model/version, source revision, and normalization-policy version. MeshekOS derives user-facing qualitative uncertainty and reasons through a centrally owned, versioned normalization policy.

The reviewer experience follows the product specification: low-confidence material is separated for explicit disposition, missing fields remain visibly incomplete, and numeric scores are not the primary signal. No provider-independent threshold, category list, reason taxonomy, or numeric-detail audience is hard-coded before validation on representative Hebrew material.

Normalization never promotes an item to official status, invents a missing field, or substitutes a confidence threshold for human review.

### 5.3 Data-Transmission Policy Enforcement

Provider policy is evaluated from current state:

1. before dispatch and before any external transmission;
2. before a returned result is stored or promoted into an advisory candidate layer;
3. before a candidate is displayed, enters Q&A context, or is converted into a downstream artifact.

Each check covers organization/body scope, source revision, current authorization, lifecycle, sensitivity, AI eligibility, approved purpose, provider capability, destination, and customer external-processing policy. Stored job context is not authority.

If policy changes while work is in flight, the result is quarantined or discarded according to the approved retention policy and cannot alter a governed record. Transmission decisions and outcomes are audited with identifiers and minimized metadata; prompts, transcripts, source passages, credentials, and provider payloads are excluded from normal logs.

---

## 6. API Design and Communication Patterns

### 6.1 REST API

MeshekOS exposes a **versioned REST API** for browser communication. Exact URI placement, resource naming, error envelope, pagination, optimistic-concurrency representation, and idempotency conventions are selected once as a cross-cutting API decision rather than invented independently by feature teams.

Every command identifies its resource and appropriate scope. Governance-body actions require an explicit body context; organization, identity, deployment, recovery, and support actions use their corresponding scope rather than an inapplicable body. Organization and authority are derived from the authenticated session and current database state, not trusted from client role or organization fields.

The API distinguishes unauthenticated, forbidden, access-revoked, validation, concurrency conflict, transient dependency, and terminal-processing outcomes without confirming inaccessible resource existence. Server-side session cookies and CSRF protection apply to state-changing browser requests.

Transport handlers parse and format requests only. Domain rules remain in the owning application service, and all data selection, sorting, filtering, and pagination occurs in the authorized query path.

### 6.2 Real-Time Status Updates

Two independent communication flows remain distinct:

| Flow | Direction | Purpose |
|---|---|---|
| Application → Provider | Outbound submit/poll/retrieve | Execute provider-dependent jobs |
| Application → Browser | Authenticated SSE / REST polling | Observe durable job and notification state |

SSE is the primary browser update signal and REST polling is the mandatory recovery path. Neither is the source of truth. A browser disconnect does not cancel work, and reconnect reconciles against durable current state.

The underlying resource is authorized on connection, polling, reconnect, and consequential emission. Slow or disconnected clients cannot block worker execution. Processing status presents the seven product states and the configured over-one-hour warning from authoritative server state rather than elapsed-time guesses.

WebSockets are not part of the pilot.

### 6.3 Notifications

As part of the same database transaction that commits a governed transition, its domain owner appends a minimized producer event to the transactional outbox. Notifications consumes only committed outbox events and, after commit, idempotently materializes per-recipient in-app records and per-channel delivery attempts.

The pilot channels are persisted in-app notifications and email. Email rendering contains the minimum permitted information and resolves protected detail only after current authorization. Retryable and terminal delivery failures are distinct and visible. Delivery failure never rolls back protocol approval, publication authorization, task state, or another completed governance action.

Authentication recovery has stricter semantics: the interface must not claim that a reset message was delivered when the configured mail path could not accept it. The selected SMTP/relay topology and outage experience are governed by the corresponding product decision in the specification and deployment configuration.

Governance Overview separately consumes domain events into read-only projections for the five predefined Community Manager dashboard areas. Dashboard commands return to the authoritative owner and recheck permission.

---

## 7. Authentication and Session Management

### 7.1 Locally Managed Accounts

The pilot uses locally managed accounts so authentication works in restricted-network deployments. External SSO is deferred and does not shape the pilot session model.

The first Community Manager is created through a local privileged bootstrap procedure as a pending account with a one-time credential. Password change and any configured mandatory MFA enrollment are required before the associated privilege becomes usable. Accounts are deactivated rather than generically hard-deleted.

### 7.2 Server-Side Sessions

Sessions are stored server-side in PostgreSQL. The browser receives only a high-entropy opaque identifier in a Secure, HttpOnly cookie with an appropriate SameSite policy. The stored identifier is hashed. Login and renewal rotate identifiers.

Logout, password reset, account deactivation, forced revocation, and relevant authority or body-permission removal invalidate affected sessions promptly. A session proves authentication only; authorization is evaluated from current state for every protected operation. Authentication tokens are never stored in browser local or session storage.

State-changing browser requests require CSRF protection. Login, recovery, MFA, and reset flows use centrally configured failed-attempt and rate protections and avoid account enumeration.

### 7.3 Privileged Role Protections

The architecture supports TOTP MFA with encrypted secrets and one-time recovery codes stored only as hashes. A role configured as MFA-required cannot be used until enrollment and confirmation complete. Regeneration or recovery invalidates superseded material and is audited.

Which customer-specific roles require MFA, enrollment timing, and restricted-network recovery governance are onboarding decisions in the product specification. The design does not hard-code Community Manager or support persona names as the policy.

Password reset supports an approved email path and a secure administrator-assisted restricted-network path. Reset tokens are random, hashed, expiring, and single-use. Successful reset changes the credential and revokes active sessions. Administrator assistance issues a one-time credential and never reveals or sets an existing password.

---

## 8. Authorization and Data-Access Enforcement

### 8.1 Permission Enforcement Model

Authorization is centrally owned and enforced wherever governed data is selected or mutated. HTTP middleware may reject obvious failures, but it is never the only control. Repositories and query services accept an evaluated current scope or invoke Authorization directly.

Organization authority, governance-body permissions, narrow task-context grants, lifecycle/access state, and AI/provider eligibility are separate dimensions. Unknown actions and mismatched organization/body/resource context fail closed.

### 8.2 Read Access

Read access is additive across governance bodies. A user may search or view permitted material from several bodies in one result set. Filtering, sorting, and pagination are applied inside the authorized query rather than after broad records are loaded.

The Community Manager has default organization-administrator authority, subject to explicit sensitive-body exceptions and delegated administration configured during onboarding. Cross-body overview does not remove the need for current body-specific permission when reading restricted material.

Task assignment grants only the task and minimum permitted context necessary to do the work; it does not create governance-body membership or protocol-wide access.

### 8.3 Material Actions

Material governance actions require explicit resource and governance-body context: meeting changes, intake classification, protocol review, approval, publication authorization, task confirmation or assignment, lifecycle transition, and permission management.

Organization/deployment actions—such as account administration, backup restore, upgrade, or support authorization—use their organization or deployment scope instead of inventing a governance-body context.

Reviewer and approver permissions are independent. Approval does not imply edit authority, and review does not imply approval. Permission and lifecycle changes are concurrency-protected, atomically audited when critical, and visible to subsequent requests immediately.

### 8.4 Background Job Authorization

Each job records requester, purpose, resource, organization/body scope, source revision, selected capability, and policy version. That record bounds the job but does not authorize future steps.

Current authorization and provider policy are revalidated:

1. before dispatch or external transmission;
2. before storing or promoting a result;
3. before displaying, using as context, or converting the result.

Scheduler, indexing, and notification workers likewise recheck current resource eligibility before consequential effects. If access or policy changes, future use stops even when an external operation cannot be cancelled. The original source remains preserved and the policy lapse is auditable.

---

## 9. Semantic Search and AI Q&A Permission Enforcement

### 9.1 Index-Time Chunk Tagging

Every derived keyword or semantic record carries stable organization, body, source, revision, lifecycle, sensitivity, AI-eligibility, and source-class corpus-eligibility references. Official records carry approval or confirmation as their eligibility basis; historical or otherwise policy-gated sources carry explicit curation activation.

These references support filtering, invalidation, status-aware re-indexing, and reconciliation. They are not permission grants. Access revocation, source change, lifecycle change, or deactivation is handled through versioned events and current-state checks rather than trusting an old chunk.

### 9.2 Dynamic Query-Time Permission Application

Keyword/filter search operates over all currently authorized records independently of Q&A activation or embedding availability. Semantic retrieval and Q&A operate only over the approved or explicitly activated corpus.

Candidate retrieval applies current organization/body scope, lifecycle, sensitivity, AI eligibility, and corpus eligibility inside the query. Results are materialized only after current authoritative source and access state are rechecked. A stale index may reduce recall but must never broaden access.

Active material ranks first. Superseded and archived material remains searchable and citable with clear status when policy permits. Explicit knowledge deactivation removes the source from Knowledge Retrieval but may leave direct permission-scoped source or audit access intact.

### 9.3 Three-Boundary Defense-in-Depth

AI Q&A applies independent checks:

| Boundary | What is checked |
|---|---|
| **Before retrieval** | Current authorization, lifecycle/currentness, sensitivity, AI eligibility, and corpus eligibility constrain candidates |
| **Before context assembly** | Every selected passage, exact provider destination, permitted purpose, and transmission policy are revalidated before model context |
| **Before response** | Source revision, lifecycle, access, citations, and policy are revalidated before release |

Retrieved documents are untrusted data, not instructions. The model cannot override authorization, provider policy, citation requirements, or output handling. When remaining eligible evidence does not support an answer, the product returns the specified insufficient-information outcome.

### 9.4 Permission-Scoped Result Caches

Any result cache includes the effective authorization-policy version, source revisions, lifecycle and eligibility versions, and provider/purpose policy. Revocation or policy change invalidates or bypasses affected entries immediately. Cached prose is never returned solely because a user had access when it was generated.

Final citation and source validation occurs even on a cache hit. Cache strategy and lifetime remain part of the retrieval benchmark decision rather than an arbitrary global constant.

### 9.5 AI Q&A Audit Logging

AI Q&A records requesting actor, effective authorization/body scope, purpose, provider and external-transmission outcome, selected source revision identifiers, citations returned, and outcome. Audit stores identifiers and minimized structured metadata, not prompts, retrieved passages, or generated answer text.

Cross-body queries record the effective scopes actually used; they do not force one active governance-body context.

---

## 10. Protocol Review Interface Architecture

### 10.1 Navigation Model

Protocol review is organized by agenda item or detected topic, with a meeting-level overview and a global queue of low-confidence, unresolved, or incomplete cards. Each section can contain discussion summary, proposed decisions, task candidates, unresolved matters, and review flags.

The desktop experience preserves the product specification's source-and-draft split. Smaller screens use sequential panels. Navigation must support:

- section status and outstanding issue counts;
- global traversal of flagged cards;
- direct jump from an item to its draft location and exact source passage, recording timestamp, page, or document location;
- speaker attribution where supplied;
- enough surrounding context to verify an item without exposing model reasoning;
- stable keyboard focus and Hebrew RTL/BiDi behavior.

Loading strategy is an implementation choice, but it cannot break meeting-wide completeness, global queue ordering, source navigation, or concurrency detection.

### 10.2 Review and Approval States

Section review progress and formal protocol approval are separate state machines. Section state records whether its cards and required fields have been dispositioned; it is not a governance approval.

Whole-protocol readiness requires all sections reviewed, every low-confidence card promoted, classified, dismissed with reason, or otherwise resolved, mandatory fields completed, source integrity verified, and meeting-level completeness confirmed.

Reviewer permission authorizes section work. Approver permission authorizes formal approval of the complete protocol. The permissions are independent even when one person holds both.

Formal approval atomically creates one immutable official artifact and the required durable audit event, and appends minimized producer events to the transactional outbox. Failure or stale concurrency leaves the draft reviewable and creates no partial official state. Only human-confirmed decision and task candidates may be activated. Decisions & Tasks performs activation through its owner interface, either within a coordinated officialization workflow or afterward; unconfirmed candidates remain advisory, and protocol approval alone never activates them.

Publication or distribution is a separate authorized post-approval transition. Protocol & Review records the official revision, approved audience/access policy, and distribution request. Notifications owns actual recipient/channel pending, delivered, and failed state. Delivery failure cannot change the immutable official artifact or falsely report receipt.

### 10.3 Confidence Presentation

Low-confidence or incomplete material appears as structured review cards containing proposed classification, qualitative uncertainty and reason, exact evidence, source location, agenda/topic context, and missing fields.

The immutable AI original remains separate from human edits. A reviewer may promote, classify, edit, link, dismiss with a recorded reason, or request clarification. Dismissed evidence remains traceable rather than being deleted.

Provider-native numeric confidence is optional technical metadata and is not the primary reviewer signal. Exact qualitative categories, reason taxonomy, threshold, and any numeric-detail exposure require representative Hebrew validation and cannot be assumed by an adapter.

---

## 11. Infrastructure and Deployment Operations

### 11.1 Backup Strategy

The pilot uses verified **scheduled whole-deployment backups**, not PostgreSQL WAL-based point-in-time recovery. The recovery-point objective is up to 24 hours; the schedule must be sufficient to meet that bound. Recovery time remains provisional until measured by a full restore test. No interface or runbook may claim PITR.

The recoverable unit includes a consistent PostgreSQL snapshot, referenced file/object artifacts, and sanitized non-secret configuration metadata. Capture pauses writes or uses an equivalently proven consistency boundary. A manifest binds application/schema compatibility, timestamps, file set, and checksums.

Backups use an established authenticated-encryption tool and a customer-controlled off-host destination. A host-local copy alone is not disaster recovery. Success requires integrity verification and visible local/off-host status. Restore occurs into an isolated compatible target and validates database integrity, extension/schema state, file references, authorization, and representative governed workflows before production writes resume.

Exact schedule within the RPO, retention, destination, and key-recovery procedure are installation and policy decisions referenced from the specification and runbook rather than design defaults.

### 11.2 Support Access Model

MeshekOS provides no standing support credentials or permanent tunnel.

An authorized customer administrator initiates a time-limited support session. Diagnostic metadata is the default scope. Content access and change authority are separate explicit grants; one never implies the other. Scope, use, expiry, revocation, and attempts after expiry are evaluated from current state and audited.

The customer can revoke access locally even when external support infrastructure is unavailable. A local CLI provides status and revocation control when the web interface is unavailable. An encrypted diagnostic bundle is generated from an allow-list, excludes governed content and secrets by default, and is delivered only through a customer-controlled procedure.

The exact support connectivity or tunnel mechanism remains an installation/security decision. No mechanism may bypass customer initiation, expiry, revocation, scope separation, or audit.

### 11.3 Observability

Application diagnostics and governance audit are separate systems. Structured local logs use request, job, event, and provider-call correlation identifiers while excluding secrets and governed content.

Local metrics and status cover request failures/latency, job stage and age, retries, worker progress, provider health, notification delivery, index lag, storage capacity, backup outcome, restore-test result, and audit-write failure. Exact lightweight tooling is selected with the backend/deployment stack; remote telemetry, distributed tracing, Prometheus, or Grafana are not pilot requirements.

`/health` is process liveness. `/ready` owns dependency, schema, storage, configuration, capacity, and maintenance readiness. Worker backlog, providers, mail, backups, and indexing appear in an authenticated operator view and may degrade individual capabilities without making the local governance record unavailable.

---

## 12. Security Architecture

### 12.1 Input Validation

Browser input, filenames, uploaded bytes, parsed text, provider responses, AI output, retrieved documents, and support data are untrusted. Transport validation is followed by domain-invariant validation at the owning service.

Uploads validate declared and detected type, configured size and expansion limits, filename/path safety, parser eligibility, and checksum. Storage keys are server-generated. Parsers run with bounded resources and no unnecessary network or secret access. Unreadable scanned PDFs enter the specified manual-handling state rather than an unapproved OCR path.

### 12.2 Injection Prevention

- Database queries are parameterized; dynamic filter/order/full-text/vector expressions use server-defined allow-lists.
- Source and AI content renders as text by default; permitted rich content is sanitized before storage or display.
- User filenames never become storage paths, and provider-returned URLs or commands are never executed automatically.
- Prompt construction separates trusted policy from untrusted evidence. Source text cannot override access, provider policy, citations, or tool capability.
- Final Q&A claims and citations are checked against the selected evidence and current access before display.

### 12.3 Secret Management

Database, provider, SMTP, encryption, session, and support credentials are scoped per deployment and injected through the approved customer-controlled secret mechanism. They are never committed, baked into images, exposed by health/status APIs, copied into diagnostic bundles, or included in sanitized backup metadata.

TOTP secrets are encrypted with key separation. Passwords use the approved password-hashing configuration. Session identifiers, reset tokens, bootstrap credentials, and recovery codes are stored only as hashes.

Database identities are separated by purpose: schema ownership/migration, application runtime, worker where materially different, backup/restore, and read-only audit/support inspection. Runtime credentials cannot change schema or audit history.

### 12.4 Security Headers and Transport

Production browser, provider, mail, support, and backup-transfer traffic uses authenticated encrypted transport. The browser surface applies an approved content security policy, anti-framing, content-type, referrer, and transport-security controls appropriate to the selected frontend and deployment topology.

Inbound exposure is limited to the application surface and an optional provider callback explicitly enabled by policy. A callback verifies provider identity/signature, binds to one job, rejects replay and invalid state, and cannot directly create official content.

### 12.5 Data Sensitivity Handling

Sensitivity, local read access, AI eligibility, external-transmission permission, retention, and lifecycle are distinct policy axes. A readable item is not automatically eligible for AI use or transmission. A body default may be narrowed or otherwise dispositioned per content according to the approved customer policy, but the design does not invent sensitivity levels or permissive defaults.

Every provider use and selected retention path re-evaluates current policy. Historical imports default to restricted and inactive until authorized validation. Official approval does not broaden source permissions.

### 12.6 Log Sanitization

Application logs and audit records exclude passwords, password hashes, session identifiers, credentials, encryption material, raw prompts, transcripts, protocol text, retrieved passages, provider payloads, and file paths. Personal identifiers are minimized to what diagnosis or accountability requires.

Diagnostic bundles use an explicit allow-list and secret/content canaries during verification. Audit references governed content by stable identifiers and structured state evidence rather than reproducing it.

---

## 13. Audit Logging Scope and Design

### 13.1 Auditable Event Categories

Audit is an append-only PostgreSQL store distinct from diagnostics and notifications. Event type is a namespaced text value selected from a centrally registered typed application taxonomy; adding a type does not require a database enum migration.

| Category | Examples |
|---|---|
| **Authentication and credentials** | Login/logout outcomes, MFA enrollment/recovery, reset, bootstrap completion, session revocation |
| **Authority and permissions** | Organization authority, body assignment, sensitive-body exception, account deactivation |
| **Governance actions** | Meeting changes, protocol review/approval/publication authorization, decision creation, task confirmation/assignment/extension/escalation |
| **Content and lifecycle** | Intake, sensitivity/access/AI-eligibility disposition, historical validation, activation, supersession, archive, revocation, amendment |
| **Support access** | Request, approval/denial, granted scope, use, expiry, revocation, diagnostic export |
| **AI and provider activity** | Job lifecycle, policy denial, external transmission, candidate creation, Q&A request |
| **Recovery and deployment** | Backup/restore, restore test, maintenance, upgrade, recovery, configuration change |

Critical permission, approval, lifecycle, task-confirmation, support, restore, and upgrade actions persist their audit event in the same success boundary. If the required audit event cannot be persisted, the action fails or remains in the pre-action state. An external operation that has already completed is not falsely rolled back; its audit failure follows a centrally defined durability class and raises a high-severity diagnostic.

The runtime may insert audit events but cannot update or delete them. Audit review uses a least-privilege, permission-scoped read path. Retention and deletion remain governed by the final legal/customer policy.

### 13.2 AI Activity Log Fields

AI audit records use identifiers and minimized structured metadata.

Required fields include organization and body/resource scope where applicable, actor or worker identity, source and revision identifiers, job/attempt identity, purpose, provider/model and adapter version, policy version and outcome, transmission destination/outcome, timestamps, and final status.

Q&A adds requesting actor, effective authorization scopes, lifecycle and corpus-eligibility basis, selected source revision identifiers, citations released, provider/purpose decision, and insufficient-information or failure outcome. Cross-body queries record their actual effective scopes rather than one artificial active body.

Prompts, retrieved passages, generated answers, transcripts, provider payloads, credentials, and secret material are excluded.

### 13.3 Sensitive Content Referencing

Audit entries reference sensitive material by stable identifier, revision, action, prior/new state, and minimized reason/evidence metadata. A protocol approval records the draft and official artifact identifiers; a lifecycle transition records the source revision and relationship; a review dismissal records the card and source-reference identifiers. None reproduces the governed text.

---

## 14. AI-First Product Design Process Constraint

MeshekOS uses an **AI-first, human-gated delivery process**. Architecture and implementation artifacts must make product, security, provider, and operational policy boundaries explicit so implementers do not have to infer unresolved decisions.

### 14.1 Required Output Standard

Every implementation task identifies:

| Element | Requirement |
|---|---|
| **Ownership and upstream contract** | One authoritative module plus the applicable product and design sections |
| **Trust and authorization boundaries** | Actor, resource scope, current-state checks, external transmission, and prohibited bypasses |
| **Persistence and failure behavior** | Authoritative versus derived state, transactions, idempotency, retry, rollback or containment |
| **User experience** | Required Hebrew/RTL/BiDi, responsive, keyboard, accessibility, loading, empty, error, conflict, and revocation states where visible |
| **Verification** | Automated owner-level tests, cross-boundary failure/security tests, and operational evidence |
| **Dependencies and prohibited scope** | Hard prerequisites separated from parallel work; explicit exclusion of adjacent or deferred capabilities |
| **Human gates** | Technology, provider, security, governance, change review, and release approval where applicable |

Producing code or documentation is not completion evidence. The acceptance and verification conditions must pass, and unresolved implementation choices must remain visible to the responsible reviewers.

### 14.2 Tool-Agnostic Process

The process does not depend on one design, coding, or automation tool. Specifications use portable structured text; provider adapters, build verification, and test suites expose stable contracts. A tool may accelerate execution but cannot become the authority for product behavior, permissions, official records, or release acceptance.

---

## 15. Key Risks and Mitigations

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| **Broad pilot loop creates integration risk** | High | High | One owner per domain, one deployable monolith, explicit dependency order, and full first-slice/E2E verification |
| **On-premises operational variance** | Medium | High | Simple container bundle, readiness, controlled upgrades, whole-deployment recovery, local operator view, and customer-controlled support |
| **Scheduled backup accepts data loss** | Medium | High | Verified consistent off-host backups constrained to RPO up to 24 hours; provisional RTO measured by restore; no PITR claim |
| **Email path is unavailable or misconfigured** | Medium | Medium | Durable in-app records, explicit mail status/retries, secure admin-assisted recovery, and no core-startup dependency |
| **External AI is unavailable or prohibited** | Medium | High | Local core independence, per-purpose policy, preserved sources, replaceable adapters, and actionable delayed/failed jobs |
| **Hebrew provider quality is inadequate** | High | High | Representative Hebrew evaluation, immutable source, evidence-linked review, qualitative uncertainty, and pilot-quality metrics |
| **Provider behavior changes** | Medium | Medium | Provider Gateway isolation, normalized contracts, adapter-version evidence, and independent contract tests |
| **Derived indexes or caches expose revoked content** | Low | High | Current query filtering, three Q&A checks, policy-versioned caches, final citation validation, and immediate invalidation/bypass |
| **Asynchronous work duplicates or loses effects** | Medium | High | Durable jobs/outbox, at-least-once relay, idempotent consumers, version reconciliation, and stalled-delivery observability |
| **Upgrade or migration fails after writes begin** | Low | High | Exclusive lock, stopped writers, compatible backup, non-write validation, and release-specific reverse or restore recovery |
| **Unknown retention obligations conflict with immutability** | Medium | High | Per-aggregate lifecycle, minimized optional artifacts, no blanket deletion rule, and production acceptance gated by final policy |
| **Sensitive-body exceptions conflict with default Community Manager access** | Medium | High | Explicit onboarding configuration, current-state DAL enforcement, and audited grants/revocations |
| **Audit durability reduces command availability** | Low | High | Critical actions fail closed by design; operational audit failures use a distinct diagnostic path |
| **pgvector or ranking is inadequate at measured corpus size** | Low | Medium | Benchmark Hebrew corpus, tune within PostgreSQL first, keep derived data rebuildable, and revisit only with measured evidence |
| **Hebrew/RTL accessibility defects block adoption** | High | Medium | RTL/BiDi foundation, logical layout primitives, keyboard/screen-reader verification, and representative user workflows |

---

## 16. Open Questions and Deferred Decisions

Product, customer-policy, pilot, retention, MFA, email, recovery, provider selection, and support-commitment questions remain in the product specification. Provider-adapter implementation begins only after the applicable provider choices are resolved there. The table contains only unresolved technical choices that block implementation; each requires explicit approval by the responsible technical owner before adoption.

| Item | Status | Notes |
|---|---|---|
| **Backend technology and conventions** | Unresolved | Select language/framework, package management, PostgreSQL access, migration, testing, lint/type checks, bootstrap, and module-boundary enforcement before backend scaffolding |
| **Frontend technology and Hebrew/RTL realization** | Unresolved | Select language/framework, component and test stack, state/data approach, styling, RTL/BiDi, accessibility, and build conventions before frontend scaffolding |
| **Job, scheduler, and event-relay implementation** | Unresolved | Select PostgreSQL queue/table or another justified lightweight mechanism, leasing, wake-up, scheduler, retry/backoff, and operational limits while preserving durable state and idempotent effects |
| **REST API conventions** | Unresolved | Fix version placement, resource naming, pagination, concurrency/idempotency representation, and error envelope once for all modules |
| **Confidence normalization** | Unresolved | Define qualitative categories and reasons, system threshold, and any numeric-detail exposure from representative Hebrew/provider evaluation; per-customer thresholds remain deferred |
| **Provider-adapter integration and technical validation** | Unresolved | After the actual provider choices in Spec OQ-8 and OQ-9 are resolved, define and validate their transcription, extraction, embedding, and Q&A adapters against the Provider Gateway contracts, including provider-format isolation, error normalization, and representative integration tests |
| **Semantic retrieval implementation** | Unresolved | Select embedding model, Hebrew chunking, vector index parameters, ranking blend, cache strategy, and acceptable index lag from representative corpus benchmarks |
| **Deployment and support technical envelope** | Unresolved | Fix supported host/storage/network/container assumptions and the exact customer-initiated support connectivity mechanism before the installation runbook is accepted |
