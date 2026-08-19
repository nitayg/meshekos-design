# MeshekOS Pilot — Product Specification

**Version:** 1.0 — Pilot Release
**Status:** Draft for Review
**Date:** 2026-08-08

---

## Table of Contents

1. [Overview](#1-overview)
2. [Problem Statement](#2-problem-statement)
3. [Goals and Success Criteria](#3-goals-and-success-criteria)
4. [Scope and Boundaries](#4-scope-and-boundaries)
5. [Users and Roles](#5-users-and-roles)
6. [Primary User Flow](#6-primary-user-flow)
7. [Functional Requirements](#7-functional-requirements)
8. [Data Requirements](#8-data-requirements)
9. [Permissions and Access Control](#9-permissions-and-access-control)
10. [AI Behavior and Trust Model](#10-ai-behavior-and-trust-model)
11. [Non-Functional Requirements](#11-non-functional-requirements)
12. [Integrations and Provider Dependencies](#12-integrations-and-provider-dependencies)
13. [Constraints and Assumptions](#13-constraints-and-assumptions)
14. [Open Questions](#14-open-questions)

---

## 1. Overview

MeshekOS is a governance operating system for kibbutz communities. It transforms the way governance decisions are made, recorded, tracked, and acted upon — across the many committees, elected bodies, and working groups that make up a kibbutz's governance structure.

The product covers the complete governance loop: from meeting content intake and AI-assisted protocol drafting, through human review and formal approval, to active task tracking, escalation, and institutional knowledge retrieval. At every stage, AI output is advisory and every official action requires human confirmation.

The pilot is a single-organization, accompanied deployment targeting one kibbutz. It is designed to test the core product assumptions — most critically, whether real Hebrew-language governance meetings can be reliably transformed into structured protocols, decisions, and tasks, and whether governance participants will trust and adopt this output. The pilot is a structured learning vehicle, not a feature-complete release.

**Commercial and deployment model:** Recurring self-hosted software subscription with bundled onboarding, implementation, and maintenance support. The pilot is a dedicated single-tenant deployment in infrastructure controlled by the customer, not a shared multi-tenant SaaS service. The economic buyer is the Community Manager or organizational CEO. The product is not self-service for the pilot.

---

## 2. Problem Statement

Kibbutz governance operates across numerous committees, elected bodies, and working groups. Each body generates a continuous flow of decisions, protocols, and assigned tasks. These are currently managed through fragmented, manual, and inconsistent processes.

The consequences are significant:

- **Decisions get lost.** Meeting outcomes exist in personal notes, email threads, and scattered files rather than in a shared, searchable system.
- **Tasks go untracked.** Assigned actions lack confirmed owners, deadlines, and follow-up mechanisms. Items disappear without resolution.
- **Protocols are delayed or never formally approved.** Drafting is manual and time-consuming; review cycles are informal; approval is not always confirmed or recorded.
- **Institutional knowledge is fragile.** Knowledge lives with individuals, not the organization. When Community Managers or committee chairs change roles, continuity breaks. Previous decisions are recalled from memory rather than retrieved from authoritative records.
- **Governance continuity depends on heroic effort.** The current system places unreasonable demands on individual memory, diligence, and informal coordination to compensate for the absence of a structured governance operating system.

The core product hypothesis is that this problem can be solved by combining AI-assisted processing with mandatory human confirmation — producing a system that is both efficient enough to adopt and trustworthy enough to rely on for official governance outcomes.

---

## 3. Goals and Success Criteria

The pilot must produce evidence across five dimensions. A single impressive AI demonstration is not sufficient for success.

### 3.1 Adoption and Continued Use

The pilot should demonstrate that MeshekOS becomes part of the organization's real governance workflow, not merely a demonstration environment.

Signals of success:
- Real meetings are created or processed through MeshekOS
- Protocols move through review and approval in the system
- Decisions and tasks are created and tracked
- The Community Manager regularly uses the governance dashboard
- Users return to the system without prompting from the MeshekOS implementation team
- The organization continues using the system throughout the pilot period

**Governing question:** Does the organization rely on MeshekOS for ongoing governance work?

### 3.2 AI Quality and User Trust

AI outputs must be sufficiently accurate and useful that human review is faster and less effort than completing the work manually.

Signals of success:
- Hebrew transcription quality is acceptable for real in-person and online meetings
- Decisions, tasks, owners, deadlines, and review dates are identified accurately
- Low-confidence items are surfaced appropriately and separated from main draft content
- Reviewers can verify outputs quickly using exact source evidence and direct navigation
- The AI does not invent unsupported information
- Users report that correcting AI output requires less effort than creating the same output from scratch
- Reviewers are willing to approve AI-assisted protocols after verification

**Governing question:** Does the AI reduce work while preserving trust and human control?

### 3.3 Workflow and Behaviour Change

The pilot should demonstrate that MeshekOS changes how governance work is managed, not only how documents are produced.

Signals of success:
- Decisions are consistently converted into structured tasks with confirmed owners and deadlines, or an explicit awaiting-assignment state
- Task owners update progress through the system
- Overdue matters become visible and are actively managed
- Unresolved matters return to future agendas
- Governance-body chairs and coordinators use the system within their assigned permissions
- The organization relies less on personal memory, email follow-up, and scattered files

**Governing question:** Does MeshekOS create a more continuous and accountable governance process?

### 3.4 Operational Value

The pilot should measure whether MeshekOS improves practical management outcomes.

Signals of success:
- Reduced time to prepare and approve protocols
- Reduced time to locate previous decisions
- Increased proportion of decisions with an owner, deadline, and review date
- Fewer tasks that disappear without follow-up
- Improved visibility into overdue and blocked matters
- Faster preparation for upcoming meetings
- Successful retrieval of historical context from the curated knowledge base

**Baseline requirement:** Baseline measurements must be collected before or at the start of the pilot so that improvement can be evaluated meaningfully.

### 3.5 Commercial Validation

Signals of success:
- The pilot organization wants to continue using MeshekOS after the pilot period
- The economic buyer considers the product valuable enough to pay for
- The organization is willing to expand usage to additional governance bodies
- Onboarding and support effort is reasonable relative to expected subscription and implementation revenue
- The organization is willing to act as a reference customer or provide a case study, subject to confidentiality

**Governing question:** Would the organization choose to continue and pay for MeshekOS after experiencing the product?

### 3.6 Pre-Launch Threshold Agreement

Exact numerical success thresholds must be agreed before the pilot begins. Thresholds should be calibrated to:
- The number of governance bodies included in the pilot
- The number and frequency of meetings
- The number of participating users
- The duration of the pilot
- The organization's current baseline process

Success requires evidence across all five dimensions above.

---

## 4. Scope and Boundaries

### 4.1 Governing Principle

**Full value loop with limited implementation depth.** The pilot includes all ten core capabilities required to demonstrate the complete MeshekOS governance loop. Some capabilities are implemented at full depth; others are simplified or supported through accompanied implementation rather than full product automation.

### 4.2 In Scope for the Pilot

**Full depth:**
- Governance context management (governance body, meeting, date, participants, agenda items)
- Content intake for all supported formats
- AI-assisted structured extraction with confidence handling
- Human review, editing, and formal approval workflow
- Task tracking with owner or awaiting-assignment state, deadline, overdue detection, and escalation
- Unresolved matter flagging and return loop (with Community Manager confirmation)
- Basic keyword and filter search
- Basic permissions model with predefined permission sets
- Governance dashboard (predefined sections)
- AI-assisted Q&A over approved content and a curated historical dataset
- Historical document import with AI-assisted classification and mandatory human validation
- Document lifecycle management with five states and four terminal statuses
- In-app and email notifications

**Simplified (pilot-level implementation):**
- AI protocol drafting with three content entry paths
- Basic deadline reminders
- Agenda management without automation (manual Community Manager confirmation for return items)

**Accompanied implementation (supported through onboarding, not full product automation):**
- Complex scanned document handling
- Sensitive committee access configuration
- Governance permissions and role-holder setup

### 4.3 Explicitly Out of Scope — Deferred Capabilities

- Structured data imports (CSV, Excel, system exports)
- Custom role creation and configurable permission tiers
- Configurable AI confidence thresholds per customer
- Advanced OCR for large historical archives
- AI reasoning explanations and cross-item reasoning chains in source traces
- SMS, mobile push notifications, WhatsApp, and other notification channels
- Video processing as a strategic product capability (supported only if the transcription provider handles it without MeshekOS building a separate pipeline)
- Full historical data migration and archival cleanup
- External business-system integrations such as calendar, ERP, HR, and document-management systems
- Custom report builders, complex charts, predictive governance scoring, benchmarking across kibbutzim, and configurable dashboard layouts
- Automated agenda construction
- Advanced legal interpretation and automatic conflict resolution
- Collaborative real-time agenda editing
- Voting, quorum management, vote sessions, and tally calculation

---

## 5. Users and Roles

### 5.1 Role Overview

MeshekOS distinguishes product personas, organization-level authority, governance-body permissions, and task assignments. A persona describes how a person uses the product; it does not automatically grant every capability associated with that persona name.

| Persona or assignment | Description |
|---|---|
| Community Manager | Primary daily user and default Organization Administrator. Accountable for governance continuity, cross-body oversight, permissions, escalations, and management outcomes. |
| Administrative Coordinator | Secondary operational user. Manages meeting and document intake, reviews AI-generated material when authorized, and supports historical import. |
| Chairperson or designated approver | Performs formal approval actions only for governance bodies where approval permission is assigned. |
| Governance-body participant | Reads permitted content and may perform review or other actions according to assigned permissions. |
| Task assignee | A person or organizational role assigned responsibility for a task. This is an assignment relationship, not necessarily an account-wide or governance-body role. |

### 5.2 Role Capabilities Summary

Organization-level administrative authority is separate from governance-body permissions. Within each governance body, permissions are assigned for reading, reviewing/editing, approving, importing/classifying, publishing, task confirmation, and other material actions. A person may hold different permissions in different governance bodies.

Reviewer and approver authority are separate gates even when the same person holds both permissions. Approval permission does not automatically grant review/edit authority. Task assignment does not itself grant broad governance-body access; the assignee receives only the access required for the assigned work and its permitted context.

The Community Manager is the default Organization Administrator, with cross-body oversight and permission-management authority except where a sensitive-body restriction or delegated administration arrangement is explicitly configured.

### 5.3 Custom Role Configuration

End-user creation of arbitrary custom roles or configurable permission tiers is out of scope for the pilot. Predefined permission sets are configured during accompanied onboarding. The assignment of people to organization-level authority and governance-body actions is customer-specific and must be confirmed rather than inferred from persona names.

---

## 6. Primary User Flow

The complete end-to-end governance flow proceeds through ten steps.

**Step 1 — Meeting record creation**
The Community Manager or authorized coordinator creates a meeting record specifying the governance body, date, participants, and agenda items. Alternatively, content is uploaded first and governance context is assigned before processing begins.

**Step 2 — Content attachment**
Content is attached to the meeting record. Supported paths: upload a meeting recording, upload or paste an existing transcript, upload an existing draft or approved protocol. Multiple intake paths are supported at different stages of content maturity.

**Step 3 — AI processing**
The system transcribes audio/video content through a configured provider and passes the result to the AI extraction pipeline. The AI generates a structured draft protocol containing proposed decisions, tasks, owners, deadlines, review dates, and unresolved matters, with uncertainty information for each extracted item.

**Step 4 — Structured draft review**
The authorized reviewer reviews the AI-generated structured draft by agenda item or topic. On desktop, the review experience presents source material and the draft together; smaller screens use sequential panels. Each review item carries an exact evidence reference and direct navigation to the relevant transcript, recording timestamp, page, or source location.

**Step 5 — Protocol approval**
The authorized approver for the governance body formally approves the complete reviewed protocol. Reviewer and approver permissions are checked independently. Nothing becomes an official record before this step.

**Step 6 — Task confirmation**
AI-suggested tasks are reviewed before activation. The authorized reviewer confirms task validity, wording, owner as a named person or organizational role — or an explicit awaiting-assignment state — deadline, reporting date, and collaborators where applicable. No AI task suggestion becomes active without human confirmation, and a task derived from a protocol cannot become active merely because AI detected it.

**Step 7 — Active task tracking**
Confirmed tasks enter the tracking system, visible to the responsible person, the Community Manager, and governance-body chairs according to current permissions.

**Step 8 — Overdue detection and escalation**
When a deadline passes, the responsible person is notified first. After a configured grace period without response, the Community Manager receives an escalation with full task context. For explicitly critical tasks, both parties are notified simultaneously without a grace period.

**Step 9 — Unresolved matter return loop**
Unresolved matters flagged during AI extraction are surfaced to the Community Manager after protocol approval. The Community Manager confirms whether each should be added to a future agenda. Automatic agenda construction is deferred.

**Step 10 — Knowledge base access**
Approved protocols, decisions, tasks, and historical documents are searchable within authorized scope via keyword/filter search and AI-assisted Q&A, with source citations and permission enforcement. Keyword/filter search does not depend on Q&A activation. Approved official records need no second activation; curated historical or otherwise policy-gated sources require explicit validation and activation for Q&A.

---

## 7. Functional Requirements

### 7.1 Meeting and Governance Context Management

- The system shall allow creation of a meeting record specifying governance body, date, participants, and agenda items before content is attached.
- The system shall support retrospective meeting creation, where content is uploaded first and governance context is assigned before AI processing begins.
- **Hard rule:** No content shall be processed without a confirmed link to Organization, governance body, meeting record, meeting date, and authorized user context.
- The system shall support multiple agenda items per meeting record.
- The system shall allow a meeting record to remain open for content attachment across multiple sessions.

### 7.2 Content Intake

The following input formats are supported at pilot launch:

| Format | Handling |
|---|---|
| Direct text paste | Accepted natively; no file upload required |
| `.docx` | Parsed and ingested as a text-based document |
| Text-based `.pdf` | Parsed and ingested |
| `.txt` | Accepted natively |
| `.md` | Accepted natively |
| `.mp3`, `.m4a`, `.wav` | Passed to a configured transcription provider |
| Video, if provider-supported | Accepted if the transcription provider processes it without a separate MeshekOS pipeline; audio upload is recommended where video adds unnecessary complexity |

- Scanned or image-based PDFs that cannot be read automatically shall surface a clear notice explaining that the file cannot be processed automatically. The user shall be offered the option to upload a text-readable version or use a manual handling path. Advanced OCR for scanned documents is not included in the pilot.
- Structured data imports are deferred.
- The original uploaded file is preserved as the source document at all stages.

### 7.3 AI Processing Pipeline

#### Processing Status Model

The system shall expose the following processing states to the user:

1. Uploaded
2. Awaiting transcription
3. Transcribing
4. Analysing content
5. Generating structured draft
6. Ready for review
7. Failed / requires attention

- Processing is asynchronous. Users may leave the page after upload; the system notifies them through in-app notification and email when the structured draft is ready.
- For audio/video content, the system shall warn the user if processing is expected to exceed one hour.

#### Processing Targets (Pilot — Not Contractual SLAs)

| Input type | Target processing time |
|---|---|
| Text-based input | Within a few minutes |
| Audio/video — typical | 10–30 minutes |
| Audio/video — complex | Under 1 hour |

#### Failure Handling

- On processing failure, the system shall inform the user of the stage at which failure occurred.
- Retry shall be available where appropriate.
- The original source document shall always be preserved regardless of processing outcome.
- No partial or unofficial content shall be created or published as a result of a processing failure.
- A support path shall be available during the pilot for unresolvable processing failures.

### 7.4 Structured Draft Review

#### Review Interface

- Desktop: split-view layout with the source document or transcript on one side and the AI-generated structured draft on the other.
- The draft shall be segmented by agenda item or detected topic, with a full-meeting overview and a global queue for unresolved or low-confidence items.
- A reviewer shall be able to navigate from a flagged or extracted item to its section in the structured draft and to the exact supporting transcript, recording timestamp, page, or source location.
- Smaller screens: sequential panel layout that preserves source-to-draft navigation and review progress.

#### Extracted Item Structure

Each extracted item in the main structured draft shall include:

- Proposed classification (decision, task, unresolved matter, etc.)
- Exact source evidence reference: passage or segment, source location, and direct navigation target
- Speaker attribution, where available from the transcription provider
- Timestamp or document reference
- Confidence or uncertainty indication
- Enough source context for the reviewer to verify the item without relying on AI reasoning
- A clear label indicating that the item is AI-generated and subject to human approval

#### Low-Confidence Item Handling

- Items where the AI is uncertain of classification shall be excluded from the main structured draft.
- Low-confidence items shall be listed separately in an **"Items Requiring Human Review"** section.
- Each low-confidence item shall be presented as a structured review card with its proposed classification, qualitative confidence, available flag reason, exact source evidence, agenda/topic context, and missing or unclear required fields. A raw numeric score may be shown as optional technical detail but is not the primary user-facing signal.
- Low-confidence items shall not be presented as confirmed decisions or tasks.
- The reviewer shall be able to promote an item into the official draft, classify it, edit its wording, link it to an agenda item, reject or dismiss it, or flag it for clarification from meeting participants.
- A dismissal shall record the reason, acting reviewer, and timestamp; dismissed evidence remains traceable and is not silently deleted.

#### Missing Required Fields

- If the AI identifies a likely decision or task but cannot reliably determine a required field, the item may appear in the structured draft with the missing field clearly marked as incomplete.
- A missing required field must be completed by a human reviewer before the item can be included in an approved protocol.
- The system shall never invent or assume a value for a missing required field.

#### Confidence Threshold

- The confidence threshold for separating main-draft items from items requiring review is system-defined for the pilot.
- Configurable thresholds per customer are deferred to a later phase.

### 7.5 Protocol Approval Workflow

The approval workflow follows a four-stage model:

| Stage | Actor | Description |
|---|---|---|
| 1. AI generates draft | System | Structured draft produced from the AI extraction pipeline |
| 2. Review and edit | Authorized reviewer | Reviewer edits content, resolves low-confidence items, and completes missing fields |
| 3. Approve | Authorized approver | Formal whole-protocol approval; content becomes an official record |
| 4. Distribute / publish | System or authorized user | The approved protocol is published or distribution is requested according to governance-body configuration |

- Each governance body has a predefined configuration specifying which permissions are required for each stage action.
- A protocol may become ready for formal approval only after every section is reviewed, every low-confidence item is resolved or explicitly dismissed, every mandatory field is complete, and the reviewer confirms meeting-level completeness.
- Section review state is workflow progress, not formal governance approval. Formal approval applies to the complete protocol.
- Reviewer and approver permissions are independent. Holding approval permission does not automatically grant review/edit permission.
- Nothing becomes an official record without approval by the authorized approver for the relevant governance body.
- Approval is distinct from publication/access authorization and any distribution request. A distribution request must not be represented as successful recipient or channel delivery.
- The audit trail shall record actor identity, action performed, timestamp, and governance-body context for each stage transition.
- Self-configurable workflow roles are deferred; predefined permission sets are assigned during onboarding.

### 7.6 Task Management

#### AI Task Suggestions

- The AI shall suggest a responsible person or role for each identified task only where the source content provides clear evidence.
- Where the source content does not support a reliable suggestion, the owner field shall remain blank.
- All AI task suggestions are advisory. No task is activated without human confirmation.

#### Task Confirmation Flow

Before a task is activated, an authorized reviewer shall confirm:
- Task validity
- Task wording
- Owner — named person or organizational role — or an explicit **awaiting assignment** state
- Deadline
- Reporting date
- Collaborators, where applicable

An awaiting-assignment task is a deliberate, audited human choice. The system must never convert a missing AI owner into an implicit assignment or silently omit the task. Tasks originating in a protocol may activate only as part of or after the protocol's formal approval, never merely because AI detected them.

#### Audit Trail for Tasks

The system shall record:
- Original AI suggestion
- Reviewer identity
- Changes made during confirmation
- Final assignment or awaiting-assignment disposition
- Activation timestamp

#### Overdue Detection and Escalation

- On the deadline date, a task shall be automatically marked overdue and become visible in the Community Manager's dashboard.
- The responsible person shall receive an active notification.
- After a configured grace period with no response from the responsible person, the Community Manager shall receive an escalation notification with full task context.
- **Critical task handling:** Tasks designated as legally required, regulatory, contractual, or explicitly critical shall trigger simultaneous notification to both the responsible person and the Community Manager, without a grace period.
- **Deadline extensions:** Extensions require explicit Community Manager confirmation and are never automatic. The original deadline and extension history remain traceable.
- Grace period duration is configured at onboarding for the pilot.

#### Task Assignment

Tasks may be assigned to a named person or to an organizational role whose current holder is maintained during onboarding and operations. A role-holder change preserves the responsibility history and triggers review of active assignments rather than silently rewriting past assignments.

### 7.7 Unresolved Matters and Return Loop

- The AI shall flag items identified as unresolved matters during extraction and display them as a distinct category in the structured draft.
- After protocol approval, flagged unresolved matters shall generate an alert to the Community Manager.
- The Community Manager shall confirm or reject each proposed future return and select a future meeting, agenda target, or intended return date.
- The system shall track pending, confirmed, returned, and resolved linkage to both the originating matter and the selected future context.
- Automatic agenda construction is deferred. The Community Manager manually confirms agenda inclusion.

### 7.8 Historical Document Import

#### Import Process

- Historical documents are imported by authorized administrative coordinators through a self-service import workflow within the MeshekOS product interface.
- The MeshekOS implementation team accompanies the import process during the pilot but does not operate it on the customer's behalf.

#### AI-Assisted Classification

After upload, the AI shall attempt to extract or suggest:
- Document title
- Document type
- Date
- Governance body or committee
- Meeting reference
- Participants
- Draft or approved status
- Effective period
- Related agenda topics
- Decisions and review dates
- Sensitivity level
- Proposed access permissions
- Possible duplicates or related versions
- Possible lifecycle, replacement, or amendment relationships

All AI classification suggestions are advisory and must not automatically become authoritative metadata, access permissions, lifecycle status, or knowledge-base eligibility.

#### Mandatory Human Validation

An authorized coordinator shall review all AI suggestions before a historical or otherwise policy-gated document becomes active in the Q&A corpus. The coordinator shall be able to:
- Confirm or correct document type
- Assign governance body
- Confirm date or record it as unknown
- Confirm lifecycle status
- Set sensitivity level and access permissions
- Identify duplicates and version relationships
- Confirm or deny AI/Q&A eligibility
- Mark unclear information for later validation

The system shall support unknown values where classification cannot be confirmed. It shall never invent missing metadata.

#### Permission-Safe Defaults

- No imported historical document shall become searchable or available for AI processing until it has a confirmed access scope.
- For documents with uncertain permissions, the default shall be restricted and inactive until an authorized person confirms the appropriate access and eligibility.
- Local read permission and eligibility for external transmission or AI-assisted use are separate decisions.

#### Pilot Dataset Scope

The pilot historical dataset is limited to a curated subset, starting with:
- Approved protocols from selected governance bodies
- Current bylaws and regulations
- Key procedures
- Decisions relating to active matters
- Documents relevant to predefined pilot use cases

Full historical data migration and archival cleanup are deferred.

### 7.9 Document Lifecycle Management

The lifecycle model applies consistently to current uploads, approved official records, historical imports, and other governed documents. Lifecycle status and Q&A-corpus activation are separate: approved official records do not require a second M8-style activation, while curated historical or otherwise policy-gated sources require explicit activation. Keyword/filter search remains available across all currently authorized records independently of Q&A activation.

#### Five Lifecycle States

| State | Description |
|---|---|
| 1. Uploaded / raw | File or text received; governance context not yet confirmed |
| 2. Pending processing | Governance context confirmed; awaiting transcription or AI extraction |
| 3. Draft | AI-generated output under human review and editing |
| 4. Approved / official | Signed off by an authorized approver; eligible for the approved-content knowledge corpus subject to current access, sensitivity, and AI policy |
| 5. Terminal status | Superseded, archived, cancelled/revoked, or partially amended — see below |

#### Four Terminal Statuses

| Status | Meaning |
|---|---|
| Superseded | Replaced by a newer or more authoritative document |
| Archived | Retained for historical or administrative purposes; no longer in the active working set |
| Cancelled / revoked | Formally withdrawn without necessarily being replaced |
| Partially amended | Specific sections changed; remainder remains effective |

#### Automated Status Detection

- When a new document is approved, the system shall attempt to detect likely replacement relationships using indicators including document type and governance body, title similarity, version number, effective date, and explicit replacement wording.
- Where a likely relationship is detected, the system shall flag it, display the related document, explain why the relationship was suggested, and ask an authorized user to confirm or reject it.
- **Hard rule:** The system shall never silently change an official document's status based on AI inference alone. All status transitions require human confirmation.

#### Manual Status Changes

Authorized users may mark a document as superseded, archived, cancelled/revoked, or partially amended. Manual status changes require:
- Replacement document, where applicable
- Effective date
- Reason
- Scope — the entire document or specific provisions
- Supporting decision or approval reference

#### Accessibility of Terminal-Status Documents

- Superseded and archived documents shall remain accessible to authorized users for historical context and audit purposes unless they are separately deactivated from the Q&A corpus under an applicable policy.
- Terminal-status documents shall be clearly labelled with their status.
- Active versions shall appear first in search results.
- AI-assisted answers shall prioritize active documents but may cite corpus-eligible historical documents to show how a rule or decision developed.
- Explicit deactivation removes a source from Knowledge Retrieval without deleting its source or audit history; direct authorized source or audit access may remain.
- The system shall warn users when a document's current status is uncertain.

#### Status Change Audit Trail

All document status changes shall record:
- Who confirmed the change
- Timestamp
- Whether AI-suggested or manually initiated
- Previous status
- Replacement or related document, where applicable
- Effective date
- Reason
- Affected scope for partial amendments

### 7.10 Search and Knowledge Base

#### Keyword and Filter Search

- Keyword search shall be available across all currently authorized content types: protocols, decisions, tasks, documents, and agenda items.
- Keyword/filter inclusion is independent of Q&A-corpus activation or embedding availability.
- Filter dimensions include governance body, document type, date range, approval status, lifecycle status, responsible person, task status, and access scope.
- Search results shall link directly to the authoritative source document or record without AI interpretation in the path.

#### AI-Assisted Q&A

- Users shall be able to submit natural-language questions and receive synthesized answers over approved official records and explicitly activated curated sources.
- Each response shall include:
  - Direct citations to the underlying documents, decisions, or protocol passages that support the answer
  - Governance body and meeting date for each cited source
  - Active versus superseded labelling for each cited source
  - Flags where cited sources conflict or where current status is uncertain
  - An explicit insufficient-information response when eligible sources do not support a reliable answer
- Current authorization, lifecycle/currentness, sensitivity, AI eligibility, approved or curated corpus status, purpose, and provider-transmission policy are checked before retrieval, before selected evidence enters model context, and before the response and citations are released.
- Permission or policy revocation takes effect immediately. Stale indexes or caches may reduce recall but may not preserve or broaden access.
- **Hard rule:** The AI shall never surface content to a user who lacks permission to see the source. AI synthesis shall never broaden access.

#### Pilot Scope for Search

- Full keyword and filter search is available across all authorized content types from pilot launch.
- AI-assisted Q&A is available over approved protocols, decisions, tasks, and the curated historical document dataset. Advanced legal interpretation and automatic conflict resolution are deferred.

### 7.11 Governance Dashboard

The Community Manager's governance dashboard provides a cross-body operational view of what was decided, what is moving, what is stuck, and what requires attention. The design is attention-oriented and prioritizes items requiring Community Manager action.

#### Five Dashboard Areas

**1. Tasks and Execution**
Overdue tasks, tasks approaching deadline, tasks awaiting update, blocked tasks, tasks awaiting assignment, extension requests pending, and tasks requiring escalation. Grouped by governance body, responsible person or role, and task status.

**2. Protocol Workflow Status**
Meetings with no content uploaded, content currently being processed, drafts awaiting review, protocols awaiting approval, approved protocols awaiting publication, publication/access status, distribution requests, delivery failures requiring attention, and processing failures.

**3. Upcoming Meetings**
Governance body, scheduled date, agenda preparation status, proposed agenda items, missing required documents, and matters flagged to return from previous meetings.

**4. Unresolved and Returning Matters**
Flagged unresolved items, decisions requiring future review, matters whose reporting date has arrived, overdue tasks that may need to return to a governance body, and appointments or contracts approaching expiry. The Community Manager confirms whether a flagged matter is added to a future agenda.

**5. Recent Decisions**
Cross-body summary of recently approved decisions showing governance body, meeting date, decision summary, responsible person, deadline or review date, and links to the approved protocol and source context.

#### Dashboard Behaviour

- Filters: governance body, date range, responsible person, task status, protocol status, urgency, and item type.
- Each summary item links to the underlying meeting, protocol, decision, task, or source document.
- Publication authorization, distribution request, and per-recipient or channel delivery outcomes must remain distinguishable.
- **Pilot boundary:** Predefined dashboard sections only. No custom report builders, complex charts, predictive governance scoring, benchmarking, or configurable layouts.

### 7.12 Notifications and Escalation

#### Delivery Channels (Pilot)

- In-app notifications
- Email

SMS, mobile push notifications, WhatsApp, and other channels are deferred. Email is a critical pilot channel because many task owners may not log in daily.

#### Notification Events

| Event | Recipient | Channel |
|---|---|---|
| Processing complete — draft ready for review | Authorized reviewer | In-app + email |
| Task assigned | Task owner | In-app + email |
| Task approaching deadline | Task owner | In-app + email |
| Task overdue | Task owner | In-app + email |
| Escalation after grace period | Community Manager | In-app + email |
| Critical task — overdue | Responsible person and Community Manager simultaneously | In-app + email |
| Extension request received | Community Manager | In-app + email |
| Unresolved matter flagged for agenda review | Community Manager | In-app |
| Processing failure | Uploading user | In-app + email |

#### Escalation Logic

- Overdue task: responsible person notified first.
- After the configured grace period with no response: Community Manager receives an escalation with full task context.
- Critical tasks: simultaneous notification to the responsible person and Community Manager; no grace period.
- Deadline extensions: require Community Manager confirmation; never automatic.
- Grace period duration is configured at onboarding.
- Notification delivery failure does not reverse the underlying approval, task, extension, or unresolved-matter state. Delivery failure remains visible and actionable.

---

## 8. Data Requirements

### 8.1 Core Data Entities

The dedicated pilot deployment has one explicit **Organization/Deployment** ownership root. All governed information belongs to that root, even though the pilot is single-tenant. The product must preserve relationships among the following information domains:

| Information domain | Required information |
|---|---|
| Organization / Deployment | Organization identity, subscription status, customer-controlled deployment configuration |
| Governance Context | Governance bodies, members and role holders, meetings, dates, participants, agenda items or topics |
| Users and Access | Local accounts, account status, organization authority, per-body permissions, task-context access |
| Content and Provenance | Original files, pasted text, transcripts, AI outputs, human edits, source references, processing history |
| Protocols | Drafts and sections, section-review progress, reviewer confirmation, approval and publication status, immutable official artifacts |
| Decisions and Tasks | Decisions, unresolved matters, task candidates, assignments, deadlines, reporting dates, extensions, escalation, return links |
| Documents and Knowledge | Metadata, versions, lifecycle and amendment relationships, access, sensitivity, AI eligibility, curation activation |
| Notifications and Operations | Notification and delivery status, provider-processing status, backup/restore status, configuration and audit history |

### 8.2 Source Document Provenance Model

At every stage of the processing pipeline, the following layers must remain distinguishable and independently accessible:

| Layer | Description |
|---|---|
| Original source | Uploaded file or pasted text, preserved exactly as received |
| Provider transcript | Text output from a transcription provider, with speaker and timestamp data where available |
| AI-generated draft | Structured extraction output; clearly marked as AI-generated |
| Human-edited content | Reviewer modifications to the AI draft, attributed to the reviewing user |
| Approved official content | Content confirmed by the authorized approver; the only layer that is an official record |

No layer may overwrite or replace a previous layer. Each layer is associated with a timestamp, the actor or system responsible for producing or confirming it, and the relevant processing stage.

### 8.3 Content Item Metadata

For each uploaded or processed item, the system shall record:
- Original file and file type
- Uploading user and upload timestamp
- Organization, governance body, and meeting record
- Meeting date
- Relevant agenda items or topic
- Processing status and status history
- Transcription provider, where applicable
- Generated transcript, where applicable
- Subsequent AI-generated outputs
- Access scope, sensitivity, and AI eligibility
- Lifecycle, version, activation, retention, and deletion status where applicable

### 8.4 Retention and Deletion

Official document and governance-record retention periods are not defined in this pilot specification. Applicable legal obligations, deletion constraints, recording-consent requirements, and customer policy require explicit confirmation before production. See Section 14.

Raw provider responses are optional rather than automatic. Where retained for traceability, quality review, debugging, provider comparison, or source verification, they must follow the customer's sensitivity policy, configured retention rules, and data-minimization requirements. AI audit records should retain identifiers and structured evidence metadata without unnecessarily duplicating transcripts, prompts, or source passages.

---

## 9. Permissions and Access Control

### 9.1 Community Manager as Organization Administrator

The Community Manager is the default Organization Administrator. By default, the Community Manager has:

- Full read and write access across governance bodies, meetings, protocols, decisions, tasks, and documents within the deployment
- Permission-management authority to create and deactivate users, assign and remove permissions, update role holders, and revoke access
- Cross-body dashboard visibility without requiring a body filter for read-only overview

Exceptions — such as a sensitive committee with restricted access or a delegated Organization Administrator — are explicitly configured during onboarding. They are not inferred from role names.

Delegation does not automatically remove the Community Manager's own access unless explicitly configured.

### 9.2 Governance-Body Role Model

Permissions are assigned per governance body. Users may hold different permissions in different bodies.

| Permission level | Capabilities |
|---|---|
| Approver | Formally approve protocols and decisions when granted for the body; does not automatically inherit review/edit capability |
| Reviewer / Editor | Edit AI-generated drafts, confirm tasks, classify documents, and review content |
| Member | Read permitted content within the body |
| Read-only | Read permitted content only; no editorial or approval actions |

Task ownership is assigned to a person or organizational role independently of these permission levels. It grants the task interaction and permitted context needed for the assignment, not a general governance-body role.

### 9.3 Multi-Body Access

- Read access is additive. A user assigned to multiple governance bodies may view permitted content from all assigned bodies in a combined view.
- A user's permissions are evaluated independently per governance body.
- **Material actions require explicit governance-body context.** Before performing any create, edit, approve, publish, assign, classify, status-change, or permission-management action, the user must be acting within a clearly identified governance-body context.
- The interface shall display the active governance body and the actions the user is authorized to perform during active use.
- Organization-level authority does not remove the need to identify governance-body context for a material or official action.

**Material actions that require governance-body context:**
- Creating or editing a meeting record
- Uploading or classifying a document
- Reviewing or editing a protocol draft
- Approving or publishing a protocol
- Confirming or assigning a task
- Changing document lifecycle or knowledge-activation status
- Managing user permissions

### 9.4 AI and Permissions Hard Rules

- AI shall never surface content to a user who lacks permission to see the source.
- AI shall never broaden access through synthesis, summarization, search, or Q&A.
- These rules are enforced against current system state, not only at the interface.
- Local read access and eligibility for AI processing or external transmission are independent policy decisions. Content may be readable locally while remaining ineligible for transmission or AI use.
- These are hard system rules, not configuration options.

### 9.5 Support Staff Access Policy

MeshekOS support staff shall have no standing credentials, permanent VPN access, or routine access to customer content. An authorized customer administrator initiates any remote support session. The session must be time-limited, revocable, diagnostic-only by default, and fully audited. Content access or system changes require separate explicit authorization.

### 9.6 Audit Trail — Permissions

All permission changes shall record:
- Actor who made the change
- Affected user
- Affected governance body or content scope
- Previous permission state
- New permission state
- Timestamp
- Whether the change was made under Community Manager or delegated administrative authority

### 9.7 Authentication and Session Behavior

- The pilot uses locally managed user accounts and server-side sessions; external SSO is deferred.
- Sensitive actions use the user's current account and permission state, not authorization embedded in a stale client token.
- Logout, account disablement, password reset, forced revocation, and removal of a permission or governance-body assignment invalidate affected sessions promptly.
- An account granted privileged access cannot use that access until MFA enrollment is complete. The mapping of customer-specific permissions to the privileged category and the accepted recovery policy require customer confirmation.
- Password recovery supports email where an approved mail path exists and a secure administrator-assisted path for restricted-network deployments.

---

## 10. AI Behavior and Trust Model

### 10.1 Advisory-Only Principle

AI output is advisory at every stage of the product. Nothing becomes an official record without explicit human confirmation from the authorized person for the relevant governance body.

This is a non-negotiable product principle, not a configuration option.

### 10.2 Extraction Output Structure

Each extracted item in the main structured draft shall include:
- Proposed classification
- Exact source evidence reference and direct navigation target
- Speaker attribution, where the provider supplies it
- Timestamp, page, or document location
- Confidence or uncertainty indication
- Clear labelling as AI-generated and subject to human review

### 10.3 Source Trace Interface

- Each protocol section and structured review card shall carry the exact supporting source passage or segment, speaker where available, timestamp or page/location, and a stable pointer to the original context.
- The reviewer shall be able to navigate directly from an item to its position in the draft and to the exact transcript, recording, or document location.
- Desktop review retains a source/draft split. Agenda/topic segmentation, a global flag queue, and sequential small-screen panels organize navigation without changing provenance.
- AI reasoning explanations and cross-item reasoning chains are not included in the pilot.

### 10.4 Confidence and Uncertainty Handling

- High-confidence items appear in the main structured draft.
- Low-confidence items appear in a separate **"Items Requiring Human Review"** section and are not presented as confirmed decisions or tasks.
- Items with missing required fields may appear in the main draft with incomplete fields clearly marked; missing fields must be completed by a human before approval.
- The system never invents values for missing fields.
- Uncertainty is always visible, reviewable, and never presented as an official conclusion.
- The confidence threshold is system-defined for the pilot. Configurable thresholds per customer are deferred.

### 10.5 AI and Permissions — Hard Rules

The following rules apply across all AI capabilities:
- AI shall never surface content to a user who lacks permission to see the source.
- AI shall never broaden access through synthesis.
- These rules are enforced against current system state, not only at the interface.
- Current authorization and AI eligibility are checked before source retrieval, before selected evidence enters model context, and before a generated response and its citations are exposed.
- Before external transmission, the approved purpose, sensitivity policy, and provider destination are also revalidated.
- These rules apply to Q&A, structured extraction, document classification, and task suggestion.

### 10.6 Processing Provider Requirements

MeshekOS must support approved external API providers and local or privately deployed providers through replaceable integrations. Enabling an external provider does not grant access to the MeshekOS business application. The normal external flow is outbound upload, provider processing, and result retrieval; a narrowly scoped completion callback is optional only where provider capability and customer network policy permit it.

Before any external transmission, the system must revalidate the content's current authorization, sensitivity policy, AI eligibility, approved purpose, and provider destination. The transmission and governing policy decision must be auditable without unnecessarily copying sensitive content.

Provider selection for transcription, OCR, extraction, embedding, and Q&A must satisfy the criteria applicable to the capability:
- Hebrew language capability — a hard requirement
- Multi-speaker identification support for transcription
- In-person and online meeting support for transcription
- Outbound API support; callback support is optional
- Commercial redistribution or embedded-product rights compatible with the self-hosted subscription model
- Data retention and deletion terms consistent with MeshekOS commitments
- Explicit prohibition on using customer content to train provider models
- Data Processing Agreement availability
- Security certifications
- Documented hosting location and data residency
- Sensitive-data handling controls
- Export format support
- Acceptable cost at pilot and production scale
- Service availability and vendor-dependency risk

Specific providers remain to be selected under these constraints.

---

## 11. Non-Functional Requirements

### 11.1 Language

- Hebrew is the primary language. This is a hard constraint.
- The full product interface — labels, notifications, error messages, help text, and system-generated content — must be available in Hebrew.
- AI processing, structured extraction, protocol drafting, and Q&A must operate in Hebrew natively.
- Document processing must handle Hebrew-language content, including right-to-left layout where applicable.
- All AI outputs generated by the system must be produced in Hebrew.

### 11.2 Performance

Processing time targets are pilot targets, not contractual SLAs.

| Input type | Target |
|---|---|
| Text-based input | Within a few minutes |
| Audio/video — typical | 10–30 minutes |
| Audio/video — complex | Under 1 hour |

- Processing is asynchronous. Users do not wait on-screen.
- Users are notified by in-app notification and email when the structured draft is ready.
- If processing is expected to exceed one hour, the user shall be warned before leaving the upload screen.

### 11.3 Availability and Backup

- The pilot uses verified scheduled backups. It does not claim PostgreSQL point-in-time recovery.
- The recoverable unit includes PostgreSQL data, file/object storage, and the non-secret configuration information required to restore a consistent deployment.
- The recovery-point objective is no more than 24 hours. The recovery-time objective is provisional until demonstrated by a full restore test and accepted by the customer.
- Backups must support a customer-controlled off-host destination, encryption, integrity verification, visible last-success and failure status, and scheduled restore testing. A backup retained only on the application host is not complete disaster recovery.
- Routine application restart must not perform an upgrade. Version upgrades require an explicit, auditable process with compatibility and health checks, a verified pre-upgrade backup, authorization, controlled data migration where required, failure halt, post-upgrade validation, and a recovery path appropriate to the release.
- The pilot availability and support commitment remains an open commercial and operational decision; no unsupported availability default applies.

### 11.4 Security and Data Sensitivity

- Governance content contains sensitive organizational decisions. Access control is mandatory.
- Access control shall be enforced against current system state, not only at the UI.
- Audit logging is required for permission changes, document status changes, AI-assisted actions, approval events, task confirmations, escalation events, support access, restores, and upgrades.
- Each audit record shall include actor identity, action type, affected entity or content reference, previous state, new state, timestamp, and governance-body context where applicable.
- Permission changes, formal approvals, support-access lifecycle actions, restores, and upgrades are not successful unless their required durable audit record is persisted. If audit persistence fails, the critical action fails or rolls back rather than completing unaudited.
- MeshekOS support staff have no routine access to customer content. Any support access must be explicitly authorized, limited in scope and duration, revocable, and fully logged.

### 11.5 Compliance and Retention

- Applicable legal obligations, official-record retention periods, deletion constraints, and recording-consent requirements have not yet been established and require qualified review before production deployment.
- Provider-output and diagnostic retention must remain configurable and data-minimized.
- Official approved records and their audit provenance remain durable subject to the final retention policy.
- The pilot acceptance process must obtain explicit agreement on retention, deletion, recording consent, provider policy, recovery time, and availability commitments. See Section 14.

### 11.6 Accessibility

- All pilot workflows shall provide a responsive Hebrew interface with correct right-to-left layout and stable bidirectional rendering for mixed Hebrew, numbers, dates, identifiers, and other left-to-right content.
- All actions in pilot workflows shall be operable by keyboard without requiring a pointer-only interaction.
- Controls, navigation, statuses, validation feedback, and errors shall have perceivable labels and state information.
- Keyboard focus shall remain visible and move predictably when users navigate, open or close an overlay, submit an action, or encounter an error.
- Pilot workflows shall remain understandable and operable with assistive technologies, including access to the current context, available actions, status changes, and errors.

---

## 12. Integrations and Provider Dependencies

### 12.1 Vendor-Agnostic Architecture Principle

MeshekOS owns the data model, customer-deployment isolation, permissions, workflows, decisions, tasks, source traceability, and domain logic. Third-party providers handle transcription, speaker identification, OCR, and AI language processing only where enabled.

Providers must be replaceable without redesigning core product behavior. Vendor lock-in is an explicit risk to be managed.

Local hosting does not imply that all AI processing is local. A deployment may enable approved external providers or local/private providers according to customer policy. No customer data is shared with another customer, and data leaves the local deployment only through an explicitly enabled, purpose-limited provider path.

### 12.2 Transcription Provider

The transcription provider handles audio-to-text conversion, multi-speaker identification, and in-person and online meeting formats.

**Selection criteria:**
- Hebrew transcription quality — hard requirement
- Multi-speaker identification
- In-person and online meeting support
- API support for outbound upload, processing-status retrieval, and result retrieval; callback support is optional
- Commercial rights compatible with redistribution and self-hosted deployment
- Data retention and deletion terms consistent with MeshekOS commitments
- No customer-content training use
- Data Processing Agreement availability
- Security certifications
- Documented hosting location and data residency
- Sensitive-data handling controls
- Export format support
- Cost at pilot and production scale
- Service availability and vendor-dependency risk

No provider is selected by this specification.

### 12.3 AI Extraction Provider

The AI provider supports structured extraction from meeting transcripts and documents, classification, uncertainty handling, embeddings where enabled, and Hebrew-language Q&A.

**Selection criteria:**
- Hebrew language capability — hard requirement
- Structured output compatible with MeshekOS product requirements
- Confidence or uncertainty information that MeshekOS can present as qualitative review signals; provider-native numeric scores are optional technical input
- Data Processing Agreement availability
- No customer-content training use
- Security certifications
- Documented hosting location and data residency
- Cost at pilot and production scale
- API availability and response characteristics appropriate to the pilot targets
- Vendor-dependency risk

No provider is selected by this specification.

### 12.4 External System Integrations

Business-system integrations such as calendar, document-management, ERP, and HR systems are deferred. Replaceable transcription, OCR, AI-processing, email-delivery, and authorized support connectivity are provider or operational dependencies and are not excluded by this boundary.

---

## 13. Constraints and Assumptions

### 13.1 Known Constraints

| Constraint | Description |
|---|---|
| Hebrew-first | The full product, AI processing, and document handling must support Hebrew natively. This cannot be deferred. |
| Accompanied implementation | The pilot is not self-service. The MeshekOS team accompanies onboarding, configuration, permissions, document import, training, and adoption. |
| Predefined permission sets | Arbitrary custom-role creation is not supported in the pilot. Organization authority and governance-body action permissions are assigned during onboarding. |
| Pilot dataset scope | The historical document dataset is limited to a curated subset. Full historical migration is not required for the pilot. |
| Advisory AI | AI output is advisory throughout the product. Human confirmation is required for official actions. |
| Dedicated on-premises deployment | Each pilot customer receives a separate deployment and customer-controlled data boundary. The product remains portable rather than customer-forked. |
| Restricted-network operation | The local core must start and preserve governed data without mandatory external services. Provider-dependent capabilities may become unavailable or delayed without compromising local records or unrelated workflows. |
| No business-system integrations | Calendar, ERP, HR, and document-management integrations are deferred; approved processing and operational providers remain in scope. |

### 13.2 High-Risk Assumptions Under Test

The following assumptions represent the pilot's primary learning objectives. If any fail, product direction requires reassessment.

1. **Hebrew transcription quality:** Real kibbutz meetings — in-person or hybrid, with natural speech, multiple speakers, and informal discussion — can be transcribed with sufficient accuracy to support reliable structured extraction.
2. **AI extraction reliability:** AI can reliably distinguish discussion, recommendation, proposed decision, approved decision, task, and unresolved matter in natural governance conversations.
3. **User trust in AI-assisted protocols:** Governance participants will trust AI-assisted protocol drafts with human approval and source-linked verification, and will be willing to approve them as official records.
4. **Permission model effectiveness:** The permission model prevents direct and AI-mediated access to content that a user is not authorized to see under realistic usage conditions.
5. **Workflow adoption:** The connected governance workflow delivers sufficient value to justify changing established management habits and tools.

### 13.3 Scope Assumptions

- The pilot organization is a single kibbutz with one dedicated deployment.
- Governance bodies, users, permission assignments, sensitive-body exceptions, and configurations are set up during accompanied onboarding before the pilot begins.
- Baseline measurements of current process performance are collected before or at the start of the pilot.
- Pilot success thresholds are agreed between MeshekOS and the pilot organization before the pilot begins.

---

## 14. Open Questions

The following product and customer-policy questions remain unresolved for the pilot.

| # | Question |
|---|---|
| OQ-1 | **Retention, deletion, and recording obligations:** Which legal and customer-policy obligations apply, what retention periods and deletion constraints follow, and what recording-consent controls are required? |
| OQ-2 | **Sensitive governance bodies:** Which pilot bodies require restricted-access exceptions, and what access configuration is required for each? |
| OQ-3 | **Pilot success thresholds:** What numerical targets define success across the five success dimensions for the agreed pilot size, duration, and meeting frequency? |
| OQ-4 | **Baseline measurement:** Which current-process measures will be collected, by whom, and before which pilot activity? |
| OQ-5 | **Privileged MFA and recovery:** Which customer-specific permissions are privileged, and what account-recovery process is accepted for restricted-network installations? |
| OQ-6 | **Recovery-time objective:** What RTO is demonstrated by a full restore test, and is that result acceptable to the pilot organization? The RPO is no more than 24 hours and point-in-time recovery is not claimed. |
| OQ-7 | **External-provider policy:** Which content classes may be sent to which approved provider capabilities, under what data-processing, retention, training-use, residency, and sensitivity terms? |
| OQ-8 | **Transcription provider selection:** Which provider will be selected for the pilot under the criteria in Section 12.2? |
| OQ-9 | **AI provider selection:** Which provider or providers will be selected for extraction, embedding, and Q&A under the criteria in Section 12.3? |
| OQ-10 | **Email delivery:** Will the deployment use customer SMTP or an approved external relay, and what user-visible behavior is required while email delivery is unavailable? |
| OQ-11 | **Availability and support commitment:** What pilot availability objective, maintenance window, incident-response commitment, and support boundary apply to the self-hosted subscription? |

---

*This specification is a Draft for Review. Open questions in Section 14 require explicit resolution through the applicable pilot or customer decision process.*
