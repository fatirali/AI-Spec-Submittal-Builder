# Spectre — Product Requirements Document

Version: 0.1 • Date: 2026-10-18 • Status: build-planning draft

Product owner: Fatir Ali. Product: Construction Spec to Submittal Builder.

## 1. Purpose and authority

Spectre helps commercial and mid-market general contractors turn complete construction specification PDFs into traceable submittal registers, review them, and create approved records in Procore. Larger enterprise customers are a future growth segment. The primary value is fewer missed requirements and less preparation/review effort; processing speed and cost matter, but accuracy takes precedence.

This document records the owner's decisions from the product discovery conversation and supplies proposed operating details where needed. **Confirmed** means explicitly selected by the owner. **Proposed** means a design default for review, not a validated commercial promise or measured result. **Validation dependency** means an external capability or experiment must be verified before that feature is claimed complete. All acceptance criteria describe future behavior; none claim an implemented or tested system.

The product owner reports dissatisfaction with the incumbent Procore submittal-generation workflow and related AI expense. Treat this as a customer hypothesis to test, not an independently established competitor benchmark.

The local scaffold and this document are not evidence of a deployed product. Technology stack, infrastructure vendors, API credentials, and production accounts have not been selected or provisioned by this PRD.

## 2. Confirmed decisions

| Area | Decision |
|---|---|
| Customer and operator | General contractor; project coordinator prepares, PM reviews and finalizes. |
| Buyer | CEO/COO, project director, or senior PM may sign the commercial contract. No separate executive register approval is required. |
| Licensing | Proprietary. No open-source release in MVP. |
| Inputs | Complete specification PDFs, including scanned/poor-quality documents; target up to 1,000 pages. English only. |
| Output unit | One distinct submittal deliverable per row, not one combined row per specification section. |
| Core fields | Specification section number/title, submittal type, description, responsible trade, timing requirement. |
| Requirement scope | Explicit requirements plus clearly distinguished inferred suggestions; cover all disciplines/categories present in the specification. |
| Review | Add missing items; edit descriptions and other extracted fields; approve or reject candidates. |
| Roles | Admin can grant extract-only or extract-and-review permissions. Review permission includes approve/reject/edit. |
| Procore writes | Create new submittal records only. Flag existing matches; a reviewer can skip them. No overwriting or deletion. |
| Workspace | One Procore company/instance per Spectre company space, with multiple accessible Procore projects. No external collaborators. |
| Access | Spectre must respect each user's existing Procore project and tool permissions. |
| History | Each upload creates a new extraction run within a project; preserve previous run results. No automatic addendum reconciliation. |
| Models | Managed Jev by TypeSafe for initial classification; managed OpenAI Sol/Astra for extraction and reasoning. Customer BYOM deferred. |
| Residency | Company selects Canada or US for storage. Inference need not occur in that country. |
| Deletion | Spectre recycle bin and permanent deletion; never delete data in Procore. |
| Billing | Self-service signup and Starter, Pro, Enterprise subscriptions. Plan terms below are proposals. |
| Pilot and growth | 10–100 concurrent users initially; design to scale toward thousands and additional AI workflows. |
| Quality | Recall ≥95%; precision ≥90%, targeting ≥95%; material correction ≤10%, targeting ≤5%. |
| Schedule | No fixed delivery date or budget constraint supplied. |

## 3. MVP boundaries

Included: company signup, invitations, subscriptions, region selection, per-user Procore connection, accessible project selection, PDF upload/preflight/OCR, Jev classification, OpenAI extraction and completeness checking, saved progress/resume, review, run history, approved register export, duplicate checks, new Procore submittal creation, local recycle bin, audit events, usage reporting, and an evaluation harness.

Excluded: drawing ingestion/reconciliation; automatic addendum comparison or merging; French; BYOM; arbitrary model providers in the UI; self-hosting; open source; external collaborators; executive sign-off; creating Procore projects, specification sections, packages, revisions, or review workflows; updating/deleting existing Procore records; native mobile apps; general-purpose autonomous agents; claims of engineering/code compliance. A specification may still require shop drawings or record drawings as deliverables: extracting that obligation is in scope even though reading drawing files is not.

The initial interface is a desktop web application. Responsive basic navigation is proposed; full mobile document review is deferred. Additional AI workflows can reuse the platform later but are not MVP features.

## 4. Users, permissions, and company boundaries

### 4.1 Role model

| Action | Extract-only | Extract + review | Admin |
|---|---|---|---|
| View accessible projects, sources, runs, and review results | Yes | Yes | Yes |
| Upload and start/resume an extraction | Yes | Yes | Yes |
| Add/edit/approve/reject candidate rows | No | Yes | Yes |
| Finalize register; export approved register | No | Yes | Yes |
| Resolve duplicate flags and publish to Procore | No | Yes | Yes |
| Move a local candidate to recycle bin; restore it | No | Yes | Yes |
| Permanently delete local content; manage retention | No | No | Yes |
| Manage membership, role grants, billing, connection configuration | No | No | Yes |

The table's details beyond the owner's two operational permission sets are proposed defaults. Admin never bypasses Procore authorization. A business buyer need not be a product user. Approval in Spectre means approval of the register for creation in Procore, not architect/engineer approval of the eventual construction submittal.

### 4.2 Authorization rules

- Effective access is the intersection of Spectre membership/role, the user's current Procore access, and applicable subscription entitlements. Model output cannot grant permissions.
- Each user links their own Procore identity. A workspace-wide administrator credential must not expose projects, private submittals, files, or actions unavailable to that user.
- **Proposed MVP access rule:** viewing project content and performing extraction requires access to that project and its Specifications and Submittals tools. Publishing additionally requires the actual permission to create submittals, including supported granular permissions. The Procore integration investigation must prove this mapping; do not infer it only from broad role names.
- Enforce authorization on the server for list, detail, download, search, export, background work, and publish paths. Hiding UI controls alone is insufficient.
- Recheck authorization before sensitive reads and every external write; deny access when authorization cannot be established. Short-lived internal authorization caching may be designed, but writes require a fresh check.
- On revocation/disconnection, block further content access and new external writes, pause affected work, and retain already completed checkpoints for an authorized resume. Another reviewer must explicitly take ownership of publishing; do not silently substitute identities.
- Do not reveal hidden Procore records through duplicate messages, counts, search results, or error text. Duplicate coverage is limited to what the acting user can see and must be described accordingly.
- Tenant identifiers and region boundaries apply to every source, run, candidate, export, cache, and audit access. User-supplied IDs never establish access on their own.

### 4.3 Workspace and project identity

A workspace has one selected Procore company ID and its applicable Procore API region. A Spectre project references an existing Procore project; it does not create a project in Procore. Interpret “instance” this way unless integration discovery identifies a different customer requirement. A user can belong to multiple invited company spaces, with distinct access in each (proposed). A project list includes only projects both mapped to the workspace and accessible to that user.

Workspace storage region and Procore API region are separate settings. The company's storage region is chosen explicitly during signup and cannot be casually changed after uploads; migration is a future operated procedure.

## 5. User journeys and interface

1. **Company setup:** admin verifies their email, creates a company space, selects Canada/US storage, starts a trial/subscription, links Procore, and chooses the company. Invite colleagues and assign roles. No project data is visible until individual Procore access is established.
2. **Start an extraction:** coordinator selects an accessible project, uploads a PDF, sees page count/readability checks and usage impact, then starts the run. A filename alone is not document identity; retain a source fingerprint.
3. **Monitor:** show queued/active stages, completed versus total units, unreadable pages, retry status, and next action. Closing the browser does not stop a server-side job. Avoid fabricated completion percentages or time estimates.
4. **Review:** open a register grid beside a source viewer. Search/filter by section, type, trade, evidence basis, and review status. Edit, add, approve, reject, and examine source passages. Inferred items have a distinct badge and require deliberate acceptance.
5. **Finalize:** the PM resolves all candidates and blocking coverage gaps, checks required values, and creates a versioned final snapshot. Rejected and skipped rows remain in history but are not exported/published.
6. **Publish:** preview the exact approved payload, map missing required Procore values, run permission and duplicate checks, then explicitly confirm creation. Show per-row created/skipped/failed/unknown results and links where available.
7. **Resume/history:** continue interrupted work from durable checkpoints or open an earlier extraction. New uploads produce new runs. Prior outputs and decisions remain attributable to their original source.
8. **Recycle bin:** authorized users delete only local items, can restore them before permanent deletion, and see a clear notice that Procore records are unaffected.

Core screens: signup/company setup; company projects; project run history; upload/preflight; processing monitor; register/source review; publish preflight/results; local recycle bin; company members/settings; billing/usage. In-app notifications are required; transactional email for invitations and completed/action-required jobs is proposed, with notification preferences.

## 6. Functional requirements and acceptance criteria

### FR-01 — Ingest and account for the entire source

- Support text PDFs, image-only scans, mixed text/image pages, rotated pages, and multi-column layouts within the validated 1,000-page envelope. Do not advertise “any PDF” as an unconditional guarantee.
- **Proposed starting limit:** 1 GB per upload, subject to infrastructure load tests. Show limits before upload. Password-protected or corrupt files produce a specific explanation and a request for a readable/unlocked copy; do not bypass protection.
- Validate file content, isolate document parsing, preserve the original, and version all OCR/image transformations. Never overwrite the original with an enhanced version.
- Use embedded text where trustworthy; otherwise OCR and, when useful, reversible deskew/contrast/denoise improvements. Generated-looking text is not evidence. Unrecoverable text creates a coverage gap rather than a guessed requirement.
- Assign every physical page an outcome: processed, intentionally excluded with reason, or unreadable/blocked. Preserve physical PDF page numbers plus printed labels when available. Show page and section coverage.
- **Acceptance:** a 1,000-page supported fixture completes without silent truncation. A corrupt file fails at preflight. A mixed-quality fixture identifies unreadable pages and cannot be marked fully complete. The source viewer resolves each citation to the original page.

### FR-02 — Classify with Jev, then extract with OpenAI

- After text preparation and section segmentation, use Jev to classify discipline/section context, likely submittal types, ambiguity, and routing needs. Permit multiple applicable types; do not force an entire section into one exclusive submittal category.
- Treat Jev output as a routing signal, not a definitive exclusion or contract interpretation. Low-confidence, unknown, and conflicting cases go to a stronger extraction path.
- Start evaluation with Sol as the principal extractor and Astra for ambiguous content and completeness verification. Model versions, prompts, schemas, routing policy, and classifier questions must be recorded for each run.
- **Proposed completeness design:** every substantive section receives a source-based completeness pass independent of Jev's keep/drop decision, including sections Jev labels unlikely to contain requirements. This pass can identify requirements absent from the candidate list. Do not verify only extracted rows.
- Include Division 01/general requirements and requirements outside paragraphs titled “Submittals.” Resolve internal cross-references where possible; flag missing referenced source material. Do not add obligations based solely on generic industry expectations.
- Keep chunk overlap/source offsets, reconcile cross-page requirements, and deduplicate repeated references without merging distinct deliverables. Timing/approval conditions remain associated with the correct requirement.
- Provider errors cannot become an empty successful result. A failed classifier may use a pre-evaluated managed OpenAI fallback under a versioned policy; otherwise pause with a clear dependency error. Such fallback is proposed and must preserve regional storage policy and quality gates.
- **Acceptance:** seeded requirements in low-scoring sections are still surfaced; a mixed-type section yields all applicable categories; a prompt-like instruction inside a PDF cannot alter permissions, system policy, provider configuration, or trigger external actions.

### FR-03 — Extraction scope and row contract

Use the categories below as a coverage checklist, adapted to the actual specification. UFGS supplies a useful broad taxonomy, not automatic requirements for commercial projects [S3].

| Category | Candidate deliverables when required by the source |
|---|---|
| Preconstruction/administrative | Plans, schedules, qualifications, proposed product/subcontractor lists, safety/quality/environmental submissions. |
| Shop drawings | Fabrication, installation, coordination, reinforcement and equipment layout submissions. |
| Product data | Technical sheets, performance data, equipment/material documentation. |
| Samples/mock-ups | Material/color samples, sample panels and required mock-up submissions. |
| Design data | Calculations, mix designs, delegated-design submissions. |
| Test/inspection reports | Material tests, field tests, commissioning/test-and-balance reports where specified. |
| Certificates | Compliance certificates, personnel/installer qualifications and specified certifications. |
| Manufacturer instructions | Installation, handling, operation and maintenance instructions as specified. |
| Manufacturer field reports | Site observations and verification reports. |
| O&M/training | O&M manuals, training plans/records and operating information. |
| Closeout | Warranties, record/as-built documentation, spare-parts lists and other specified handover submissions. |

Electrical, mechanical, concrete, architectural, structural, civil/sitework and other disciplines are separate labels, not substitutes for these categories. The presence of a material or work activity is not by itself an explicit requirement to submit a document. Preserve “if applicable,” alternatives, frequency, recipient, and before/after conditions in the description/timing.

Conceptual candidate fields (not a prescribed database schema):

| Field | Rule |
|---|---|
| Identity/context | Stable candidate ID; company, project, source version and extraction run. |
| Section number/title | Preserve original values; normalized equivalents may be stored separately. Unknown values remain marked unknown. |
| Deliverable title/type/description | Concise distinct deliverable title; controlled type plus original wording; editable description. |
| Responsible trade | Source-stated or explicitly inferred. Do not invent an assigned subcontractor company. |
| Timing | Original condition and normalized interpretation when supported. Do not manufacture calendar dates without a known anchor. |
| Evidence basis | Explicit, inferred suggestion, or manually added. |
| Source evidence | PDF page, section/paragraph when present, source quotation and viewer location; multiple references supported. |
| Inference information | Basis, uncertainty, and reviewer decision; never disguise a suggestion as a direct quotation. |
| Review/version | Pending/approved/rejected; row revision; author/reviewer/timestamps; approval tied to content revision. |
| Publish/duplicate data | Duplicate check result, reviewer resolution, publish attempt, external record ID and permitted URL. |

Missing evidence or required data creates a review issue. Manual items may lack a citation only with an explicit “manual addition” designation and reviewer explanation. Store unknowns instead of plausible fabricated values. No confidence score is presented as a proven probability of correctness unless validated on this task.

- **Acceptance:** a concrete section with three distinct deliverables yields three traceable rows; an inferred item remains visibly inferred after approval/export; an unanchored “30 days before installation” requirement remains relative.

### FR-04 — Review and finalization

- Reviewers can add, edit, approve, reject, and optionally merge/split candidates while preserving provenance. Merge/split is proposed convenience; distinct-deliverable correctness is required regardless of UI implementation.
- Use version checks for concurrent edits. Never silently overwrite another reviewer's work. Editing approved substantive content resets that row to pending and invalidates affected finalization/publish readiness.
- **Proposed approval defaults:** bulk approval is available for selected explicit items after source inspection; inferred suggestions require individual acceptance. No model auto-approval.
- A finalized register is a snapshot of approved row revisions, with source/run identity and approving user/time. It requires every candidate dispositioned and no unresolved coverage gap. Unreadable pages cannot be dismissed merely to claim completeness; obtaining a better source creates a new linked run if file content changes.
- Draft progress can be reviewed while processing, clearly marked incomplete. Final exports and Procore publication require finalization. A proposed draft CSV export must be watermarked/labeled incomplete and cannot be confused with a final register.
- Export approved data to CSV and XLSX with source references and inference labels. Guard spreadsheet cells against formula interpretation. Reviewer permission and current project access are required for export/download.
- **Acceptance:** extract-only users cannot edit/approve through either UI or API; editing an approved row invalidates its approval; stale concurrent edits show a conflict; rejected items are absent from final export.

### FR-05 — Duplicate checks and create-only Procore publishing

- Validate accessible target project, authorization, supported API version, field configuration, types, numbering, manager and other required values. Retrieve supported choices or request an explicit mapping; do not invent IDs, dates, assignments, or missing specification sections.
- The owner's extracted fields are not assumed sufficient for every Procore project. Required field discovery/mapping is a validation dependency [S6].
- Check duplicates within the run, across prior successful Spectre publishes, and among visible Procore records. Use stable external IDs for known publishes and candidate matching by section/type/title/content for new potential matches; matching thresholds need evaluation.
- Flag possible duplicates before a publish attempt. A reviewer may **skip** the item, or mark a potential match as **not a duplicate** with a reason and proceed to create. The latter resolves a false positive; it never overwrites the existing record. An unresolved flag blocks that row. Do not offer “override,” update, replace, or delete.
- Exact records already created by Spectre are not created again during a retry, even if their local rows have since been recycled. Preserve minimal publish receipts needed for this purpose under the retention policy.
- Show a preview of the approved, current payload and require an explicit user publish action. Readiness/approval alone must not start external writes.
- Persist a publish intent before sending each create request; record the outcome and external ID. Serialize conflicting publish operations for the same candidate and use provider idempotency features only if documented/supported.
- On a timeout where creation may have succeeded, mark the outcome **unknown**, reconcile against authorized Procore data, and block blind retries. If no unambiguous match can be established, require operator/reviewer resolution. Do not claim atomic exactly-once delivery across systems without evidence.
- Partial success preserves successful receipts. Retry only unresolved/failed rows after reconciliation and fresh permissions/duplicate checks. Never roll back by deleting Procore records.
- Changes after successful publishing are local only and cannot be pushed as an update. Previously published items must not silently become new creates in another run.
- **Acceptance:** repeated clicks and worker restarts do not knowingly duplicate known successful creates; a simulated timeout enters unknown/reconciliation; revoked create permission blocks the write; skip produces no external mutation; the integration has no update/delete business operation.

### FR-06 — Durable progress and intelligible failures

- Persist stage outputs and checkpoints at page/section/work-unit boundaries. Store leases, attempts, source fingerprints and pipeline versions so worker restarts cannot corrupt completed work.
- Resume retries an unfinished unit and reuses completed compatible outputs. It does not promise restoration at an arbitrary token or access to a model's hidden reasoning. Store task state, evidence and results, not hidden chain-of-thought.
- A browser close, server restart or model outage must not erase completed extraction/review work. A changed source or incompatible pipeline creates a new linked run rather than silently reusing stale outputs.
- Surface failure category, affected pages/sections, last completed step, whether a retry is automatic, and the user's remedy. Sanitize provider messages and show a reference ID instead of secrets/raw stack traces.
- Retry transient rate limits/timeouts with bounded backoff and provider limits. Authentication errors, exhausted allowance, unreadable files, and invalid configuration pause for action. Do not retry indefinitely or silently switch countries/providers.
- Example: “Extraction paused at section 26 05 00. 38 of 62 sections are saved. The provider is rate-limiting requests; we will retry automatically. You can close this page.”
- **Acceptance:** terminate a worker mid-run, restart it, and verify completed outputs/review revisions survive; inject provider failures and show an actionable reason; a retry is not a second customer usage charge.

### FR-07 — History, recycling, and deletion

- Each upload creates a distinct run with original source, pipeline version, output revisions, finalization and publish receipts. Rerunning the same source is an explicit new run, not a silent replacement of the old one.
- Rejected, duplicate-skipped, recycled and permanently deleted are different states. Rejection preserves a review disposition; recycling hides content with restore available; permanent deletion purges the local content under the policy below.
- **Proposed defaults:** 30-day recycle retention; reviewer can recycle/restore local rows; admin can recycle/delete whole runs and permanently delete content. Finalized-run deletions invalidate its current finalization while retaining a prior-snapshot audit event. Restore never republishes automatically.
- Permanent deletion removes active content and derived copies, revokes links, and schedules backup expiry within a proposed maximum 35 days. Do not promise immediate erasure from immutable backups. The backup policy must be proven before launch.
- Keep only minimal, access-controlled event/publish identifiers where needed to prevent duplicate creates and record a deletion; do not retain deleted descriptions or source passages in generic audit logs. UI explains retained metadata and backup expiry.
- Deleted Procore-linked local rows leave Procore untouched. No connector path may send a Procore delete request, including cleanup, rollback, account cancellation or restore.
- **Acceptance:** recycling and permanent deletion cause no Procore mutation; active downloads fail after deletion; backup restore testing reapplies deletion tombstones.

### FR-08 — Regional storage and provider controls

- Store source PDFs, page images, OCR text, extracted rows, final registers, backups, content-bearing logs and temporary files in the workspace's selected Canada/US region. Prevent cross-region content caches and unapproved replication.
- Processing outside the selected country is allowed by the owner. This is not permission for undisclosed persistent provider storage elsewhere. Verify provider retention, file storage, request logs and contract settings, including TypeSafe and OpenAI, before sending customer content.
- OpenAI documents regional Canadian storage separately from inference locality [S5]. Account eligibility and endpoint/model compatibility remain validation dependencies. Do not assume account access is already enabled.
- For Jev, confirm request/response retention and storage location or suitable no-retention controls. Its launch article is not a residency guarantee. A provider configuration that fails the selected storage policy is blocked; notify the owner rather than silently weakening the promise.
- Avoid document content in analytics, error trackers, emails, billing metadata, support tools, and global logs. Define any account/billing metadata exceptions explicitly in customer-facing terms before launch.
- Encrypt transport/storage; keep secrets in managed secret storage and server-side only; use short-lived authorized downloads. Prohibit training/fine-tuning on customer documents unless separately authorized. Validate actual provider terms rather than assuming a universal guarantee.
- **Acceptance:** trace an uploaded Canadian source through storage, OCR artifacts, logs, backups and provider calls; show all persistent content stores comply. Run cross-tenant and expired-link tests. Region choice does not silently follow a user's travel/IP address.

### FR-09 — Usage and subscription behavior

- Enforce subscription entitlements server-side and maintain an auditable usage ledger. Count source PDF pages, not model tokens, as the proposed customer-facing unit. Track provider cost internally by stage, retries and model.
- Reserve allowance before starting, settle once when extraction and automated quality checks reach review-ready, release on system failure/cancellation, and avoid duplicate settlement. An unresolved coverage gap is not a completed billable run. Fraud/abuse controls must not change documented billing semantics.
- New uploads/reruns consume allowance as new work when completed; system-caused retries/resume and exports/publish retries do not. Display usage impact before a deliberate rerun.
- Payment webhook handling must verify authenticity and deduplicate events. A failed payment blocks new paid processing after a proposed 7-day grace period; retain permitted access to existing data/export during a proposed 30-day recovery window. Cancellation stops renewal and does not delete Procore content.
- No automatic paid overages by default. Notify admins at proposed 80%/100% allowance thresholds; at the cap, require an authorized top-up/upgrade before starting new work. Preserve ongoing work/checkpoints on billing interruption.
- **Acceptance:** duplicate billing events do not double-charge; worker retries do not consume pages twice; usage reservation races cannot overspend allowance; no charge occurs merely for Procore publication or re-export.

## 7. State and data model requirements

These are logical entities and transitions, not a selected database implementation.

Entities: company workspace/region; membership/role; per-user Procore authorization; mapped project; source document version; extraction run and work units; classifier decisions; candidate and candidate revision; evidence reference; review decision; register snapshot; publish batch/item/receipt; usage reservation/ledger; subscription; recycle entry; audit event.

Every project-content entity carries company/project scope and references its origin. Extracted fields and later human edits must remain distinguishable. Model/prompt changes do not rewrite existing provenance.

| Object | State progression and constraints |
|---|---|
| Extraction | Uploaded → validating → queued → preparing text → classifying → extracting → verifying coverage → ready for review. Active stages may enter waiting for retry, action required, failed, or cancelled. Resume returns to the affected unfinished unit. |
| Candidate review | Pending → approved or rejected. Substantive edit returns approved to pending. Inferred acceptance remains recorded. |
| Register | Draft → finalized snapshot. Edits invalidate current readiness and require a new finalization. Old successful publish receipts remain valid facts. |
| Publish item | Not queued → preflight → blocked duplicate/missing field/permission, skipped, or ready → creating → created/failed/unknown. Unknown must be reconciled before another create. |
| Local lifecycle | Active → recycled → restored or permanently deleted. External lifecycle is unaffected. |

Use separate review, publishing and deletion states; one overloaded status field must not conflate “approved,” “sent,” and “deleted.”

## 8. Architecture direction — proposed

Separate the web experience/API, identity/authorization, regional document/artifact storage, durable job queue/workers, model adapters, review/versioning, Procore adapter, and billing/usage. Long jobs run outside interactive request lifetimes. Regional worker pools keep storage boundaries explicit. Use per-tenant fair scheduling, quotas, backpressure and bounded provider concurrency.

Pipeline: **PDF intake → text/OCR and section map → Jev classification → Sol extraction with Astra escalation → source-based completeness verification → reviewer → final snapshot → publish preflight → create-only Procore adapter.**

Make model providers replaceable behind versioned contracts for classification, extraction and verification. This supports future BYOM without exposing it at launch. Use task-specific inputs/outputs and deterministic authorization; model responses cannot choose arbitrary tools or URLs. Provider access and storage policy must be checked before calls.

The shared harness owns durable runs, checkpoints, retries, quotas, provenance and review handoffs. Spectre owns the submittal taxonomy, extraction contract and Procore mapping. Future workflows should be separate versioned definitions; do not build a visual workflow designer or generic agent marketplace for MVP.

Choose the implementation stack in a short technical design after validating Procore permission discovery, provider availability/residency, and a representative extraction experiment. No specific database, web framework, OCR vendor, queue, cloud provider, or payment processor is mandated here.

## 9. Quality evaluation and launch gates

### 9.1 Definitions

- **Requirement recall:** correctly surfaced unique explicit deliverables / all reference explicit deliverables. A generic section summary does not count as matching several distinct obligations.
- **Precision:** correctly supported unique explicit candidates / all candidates presented as explicit. Unsupported items and duplicate extra rows count as errors.
- **Material correction rate:** presented explicit candidates needing substantive field correction / all presented explicit candidates, measured before reviewer edits. Report additions, rejections and duplicate removals separately; do not hide missed requirements in this metric.
- Evaluate inferred suggestions separately for usefulness, unsupported inference and reviewer acceptance. An inference accepted later does not retroactively improve explicit extraction recall.
- Report per-document, per-discipline, per-category, scan-quality and aggregate results. Give sample counts and uncertainty; a small successful example does not establish a 95% product-wide claim.

### 9.2 Accepted targets

| Metric | Minimum launch gate | Improvement target |
|---|---|---|
| Explicit requirement recall | ≥95% | Sustain/improve, especially costly omissions |
| Explicit candidate precision | ≥90% | ≥95% |
| Material correction rate | ≤10% | ≤5% |
| Coverage accounting | Every page accounted for; no unresolved coverage gaps in a finalized run | Same |
| Access controls and Procore mutation boundaries | No known cross-tenant/unauthorized access; no update/delete operations | Same |
| Recoverability and publication | Pass restart, retry, duplicate and unknown-outcome test scenarios | Same |

**Proposed release rule:** meet recall/precision/correction gates in aggregate and on every held-out complete book used for release; separately investigate weak category/scan cohorts. A missed high-consequence requirement triggers review even if the aggregate passes. Do not silently lower thresholds to ship.

### 9.3 Evaluation corpus

Start with public specifications paired with supplied registers, then obtain authorized English Canadian/US commercial examples. A public reference already identified is the 526-page Repair Airfield Pavement FY22 specification package, containing a populated register [S4]. It is a seed, not verified human-authored/comprehensive ground truth or a representative commercial benchmark.

**Proposed minimum pilot corpus:** ten distinct complete books, including at least three commercial building books, clean/mixed/scanned examples, and one 1,000-page stress fixture. A synthetic 1,000-page fixture tests scale, not real-world accuracy. Include at least 500 adjudicated explicit requirements spanning major disciplines/categories; expand where coverage is thin.

A construction-qualified reviewer reconciles reference registers against their source, adds omissions and records match/splitting rules. A second reviewer adjudicates disputed cases. Owner must identify a reviewer before quality sign-off. Model-generated answers alone are not ground truth.

Split by project/book into development and held-out evaluation sets. Freeze held-out labels before final evaluation. Exclude embedded reference-register pages from extraction inputs and label the exclusion; retain them only for judging. Re-run the release suite on material model/prompt/OCR/routing changes. Compare Jev routing against the same pipeline without that routing optimization to measure total cost and any recall loss.

### 9.4 Pilot product metrics

Measure time from upload to review-ready, reviewer minutes per 100 requirements, time to approved Procore creation, manual additions/rejections/edits, retry recovery, coverage gaps, per-run provider/OCR/storage cost and customer-rated usefulness. Compare against the customer's actual prior workflow rather than vendor marketing claims. No turnaround-time guarantee is set yet.

## 10. Performance and operations

- Validate 100 concurrently active pilot users across isolated tenants, including simultaneous uploads, reviews and a bounded number of heavy jobs. “100 active users” does not imply 100 simultaneous 1,000-page model calls; record the tested workload and queue behavior.
- **Proposed responsiveness target:** ordinary project/register interactions p95 ≤2 seconds under the agreed pilot workload, excluding file transfer and long processing. Starting a job returns promptly with a persisted job ID.
- Support horizontal worker/API scaling toward thousands of users. Prove that plan through progressively larger load tests and quota/rate-limit planning; do not claim thousands supported before measurement.
- Establish monitoring for queue age, provider failures, stuck leases, OCR gaps, permission errors, unknown publish outcomes, usage drift and regional storage failures. Content-free operational telemetry by default.
- Proposed recovery objectives: RPO ≤15 minutes and RTO ≤4 hours for persisted platform data, subject to infrastructure validation. A committed checkpoint must survive an ordinary worker restart. Restore drills must preserve deletion restrictions and publish receipts.
- Use accessible labels, keyboard operation, visible focus, and non-color-only status indicators. Provide source citations in exported data, not only in the UI.

## 11. Subscription proposal — not approved prices

The owner authorized a proposal, not live billing changes. The following are hypotheses for customer/economic validation. USD is used consistently; CAD pricing, tax treatment and currency display are commercial decisions before checkout goes live.

| Plan | Proposed monthly price | Included internal seats | Completed source pages/month | Positioning |
|---|---|---|---|---|
| Starter | US$199 | 3 | 2,000 | Small project team beginning repeat usage. |
| Pro | US$599 | 10 | 10,000 | Mid-market GC with several active projects. |
| Enterprise | Custom annual contract | Negotiated | Negotiated | Higher volume, procurement and support requirements. |

All plans receive the same extraction quality standard, source citations, review, export, history, Canada/US storage selection and Procore create-only integration. No safety, access or accuracy downgrade for Starter. All plans use managed Jev/OpenAI; Enterprise does not imply BYOM or self-hosting in MVP. Company billing administration is available on every plan.

Proposed commercial details: 14-day/250-page trial with three seats and the same workflow; no card required; trial limited per company; additional page packs at US$50 per 1,000 pages purchased explicitly by an admin; no automatic rollover or overage; upgrades effective immediately with clearly shown proration; downgrades at renewal. Do not bill seats above plan limits automatically; require plan change. Trial runs remain subject to the normal 1,000-page document limit and available allowance.

Before pricing approval, measure total Jev/OpenAI/OCR cost including full coverage verification, retries, storage and support across clean/scanned books. If the proposed allowances cannot support the chosen margin, revise the proposal instead of weakening quality. A proposed ≥70% gross margin target is for planning, not a forecast. Enterprise SSO, contractual SLA, special retention and dedicated deployment are not included unless separately scoped and built.

Production checkout must remain disabled until the owner approves prices/currency/allowances, billing/retention terms, and payment/tax setup. Development can use configurable plans and payment sandbox behavior meanwhile.

## 12. Dependencies and decisions before release

| Item | Status / next action | Owner |
|---|---|---|
| Jev identity | Confirmed by owner's TypeSafe link; integrate official service, not similarly named third-party sites. | Engineering |
| Jev/OpenAI access and versions | Verify actual account availability, limits, pinned versions and fallback policy. No credentials have been supplied. | Owner + engineering |
| Regional provider storage | Verify Canada/US content storage and retention settings, particularly Jev; no unsupported residency claim. | Owner + engineering |
| Procore integration | Prove user-scoped permissions, required fields, creation and reconciliation in sandbox. | Engineering |
| Procore production distribution | Commercial SaaS verification/certification or approved pilot path; see setup guide. | Owner |
| Benchmark reviewer and corpus | Public seed identified; labeled/adjudicated commercial corpus not yet created. | Owner + domain reviewer |
| Plans | Prices, allowances, trial, overage and currency are proposed; approve before live checkout. | Owner |
| Operating defaults | File-size cap, recycle/backup periods, SLOs, payment grace and full-page review rules are proposed. | Owner + engineering |
| Technical stack | Document selection and infrastructure cost after integration/evaluation investigation. | Engineering |

These dependencies do not prevent a local prototype or mocked integration. They do prevent unsupported claims of production readiness. Do not replace real provider/Procore behavior with mocks in acceptance evidence without clearly labeling it.

## 13. Delivery references

- [Build plan and acceptance test matrix](BUILD-PLAN.md)
- [Procore setup guide](PROCORE-SETUP.md)

## 14. Sources and research notes

Sources checked 2026-10-18 unless noted. External capabilities can change; verify exact account access and API contracts during implementation.

- **S1 — Jev:** [TypeSafe introduction](https://typesafe.ai/blog/introducing-system-one-models-and-jev), supplied by the owner; [official documentation](https://docs.typesafe.ai/introduction). Supports using typed classification decisions. Vendor speed/accuracy claims are not Spectre measurements; output type constraints do not guarantee a correct classification.
- **S2 — OpenAI:** [API model catalog](https://developers.openai.com/api/docs/models). Identifies Sol/Astra options; final routing is an experiment, not an established best model for construction specifications.
- **S3 — Taxonomy:** [UFGS 01 33 00](https://www.wbdg.org/FFC/DOD/UFGS/UFGS%2001%2033%2000.pdf). Source of the broad coverage checklist, not project-specific obligations.
- **S4 — Public evaluation seed:** [Repair Airfield Pavement FY22 specifications and register](https://imlive.s3.amazonaws.com/Federal%20Government/ID41965134815393746628866773157725509183/FINAL%20SPECIFICATIONS%20CP2201M%20Repair%20Airfield%20Pavement%20FY22%20.pdf). Inspected during discovery; public availability is not proof of comprehensive labels or permission to redistribute in the repository. Record provenance and check reuse conditions when assembling test fixtures.
- **S5 — Residency:** [OpenAI data controls](https://developers.openai.com/api/docs/guides/your-data). Regional storage and processing are distinct; eligibility/settings require verification.
- **S6 — Procore:** [Data Connector](https://developers.procore.com/documentation/content/building-data-connection-apps), [production access](https://developers.procore.com/documentation/content/verification-and-production-access), [configurable submittal fields](https://dev.support.procore.com/faq-which-fields-in-the-submittals-tool-can-be-configured-as-required-optional-or-hidden/), [submittal permissions](https://dev.support.procore.com/product-manuals/submittals-project/permissions). User-scoped authorization, company field configuration and commercial access need integration validation.
