# Spectre — Phased Build Plan

Date: 2026-10-18. Companion to [PRD v0.1](PRD.md). This is an ordered implementation backlog, not a report of completed software. All tickets begin in Backlog. Priorities: P0 = release blocker or core path; P1 = supporting MVP requirement. No delivery dates are assigned.

## Implementation handoff

Read the PRD and applicable repository instructions before choosing a stack. Preserve confirmed decisions. Proposed technical defaults can be implemented in a prototype and evaluated; prices and commercial terms must not become live billing without owner approval. Never invent provider credentials, Procore IDs, production permissions or passed quality results.

Build vertical slices with meaningful tests. Keep a decision log, integration capability notes and actual test evidence. External writes in automated tests use sandbox fixtures and expressly selected test projects. No application feature may update/delete a Procore record. Do not commit customer documents or secrets.

## Phase 0 — Feasibility and evaluation foundation

| Ticket | Priority | Scope | Completion evidence |
|---|---|---|---|
| SP-001 | P0 | Establish technical design and decision log: regional storage, durable jobs, identity, adapters, deployment and cost. | Stack decision ties back to PRD constraints; no unexplained infrastructure assumptions. |
| SP-002 | P0 | Procore sandbox investigation: individual auth, company/project discovery, tool/granular permissions, required fields, create and duplicate reconciliation. | Capability matrix and captured sanitized results for allowed/denied paths, hidden records, custom required fields, ambiguous create outcomes. |
| SP-003 | P0 | Jev/OpenAI account and regional-storage validation. | Verified access/limits/version policy and content-retention map; unsuitable region/provider combinations blocked. |
| SP-004 | P0 | Assemble benchmark manifest, annotation rules and adjudication workflow. | Public seed linked with provenance; reference-register exclusion; project-level train/dev/holdout split; domain reviewer assigned. |
| SP-005 | P0 | Evaluate a thin PDF → OCR → Jev → Sol/Astra → evidence-linked register path. | Real clean/scanned example results, omission analysis and stage cost report; compare routing enabled/disabled. |

Gate: resolve feasibility issues or document a revised design without changing confirmed scope silently. A mock can unblock UI work, but cannot satisfy SP-002/SP-003/SP-005.

## Phase 1 — Workspace and durable run foundation

Dependencies: SP-001; supported Procore authentication from SP-002.

| Ticket | Priority | Scope | Completion evidence |
|---|---|---|---|
| SP-010 | P0 | Signup, company region selection, invitations, roles and per-user Procore identity. | Role/access matrix tested; region persists independently of user IP. |
| SP-011 | P0 | Workspace/project mapping and server authorization. | Cross-company/project access denied on list, detail, export and background paths. |
| SP-012 | P0 | Resumable PDF upload, original storage, parser isolation, preflight and page manifest. | Corrupt/locked/over-limit fixtures handled; mixed PDF accounts for every page. |
| SP-013 | P0 | Durable jobs, work units, leases, checkpoints, retries and cancellation. | Worker kill/restart retains completed outputs; no duplicate work settlement. |
| SP-014 | P1 | Project history and processing monitor with actionable errors. | UI distinguishes retrying, action required, incomplete and ready; browser close loses no server job. |

## Phase 2 — Extraction and quality

Dependencies: SP-003–005, SP-012–013.

| Ticket | Priority | Scope | Completion evidence |
|---|---|---|---|
| SP-020 | P0 | Text/OCR pipeline, transformation lineage and section map. | Clean, rotated, multi-column, image-only and mixed-page fixtures; no invented text used as source. |
| SP-021 | P0 | Jev multi-label classification and versioned routing. | Unknown/ambiguous categories escalate; classifier failures are visible; no hard omission by negative class alone. |
| SP-022 | P0 | Sol extraction/Astra escalation, candidate contract, source citations and inference labels. | Distinct deliverables preserved; cross-page/cross-reference cases; unknown trade/date values remain explicit. |
| SP-023 | P0 | Independent source coverage pass and duplicate normalization. | Seeded omissions in low-ranked sections recovered; distinct deliveries not merged; every substantive section checked. |
| SP-024 | P0 | Automated evaluation report and model/prompt regression gate. | Held-out recall/precision/correction metrics, document/cohort breakdowns and cost per completed book. |

Gate: do not call an extraction ready if units are unprocessed or evidence/coverage checks are unresolved. Production quality claims wait for the full held-out corpus.

## Phase 3 — Review, export, and history

Dependencies: candidate/evidence contract from SP-022; authorization and checkpoint persistence.

| Ticket | Priority | Scope | Completion evidence |
|---|---|---|---|
| SP-030 | P0 | Register grid/source viewer, filters, manual additions and edits. | Page citations open original source; reviewers can correct fields; extract-only users cannot mutate. |
| SP-031 | P0 | Approval/rejection, inferred acceptance, edit conflicts and final snapshots. | Editing invalidates approval; stale updates conflict; unresolved gaps block finalization. |
| SP-032 | P1 | CSV/XLSX approved export. | Evidence/inference markers preserved; formula-like text exported safely; revoked access blocks download. |
| SP-033 | P1 | New-run history, audit events, recycling, restoration and local purge. | Old provenance retained; restore never publishes; Procore untouched; deletion policy includes derived artifacts/backups. |

## Phase 4 — Procore publication

Dependencies: SP-002, SP-011, SP-031. Complete sandbox behavior before production access.

| Ticket | Priority | Scope | Completion evidence |
|---|---|---|---|
| SP-040 | P0 | Required field mapping, payload preview and fresh user permission checks. | Missing/invalid configured fields block affected writes; no fabricated IDs or assignments. |
| SP-041 | P0 | Duplicate checks across current run, receipts and visible Procore records. | Skip creates nothing; false-positive resolution is recorded; hidden records are not disclosed. |
| SP-042 | P0 | Create-only worker, publish intents/receipts, partial results and unknown-outcome reconciliation. | Timeout/retry/concurrent-click scenarios; known creates not repeated; no update/delete operation. |
| SP-043 | P0 | Production/pilot access process and operational runbook. | Owner obtains applicable Procore access; tested installation and consent flow; exact environment recorded. |

## Phase 5 — Billing, operations, and pilot release

Billing sandbox and UI can be developed alongside Phases 2–4. Live activation follows owner-approved plan terms and feasibility evidence.

| Ticket | Priority | Scope | Completion evidence |
|---|---|---|---|
| SP-050 | P0 | Configurable plan entitlements, page reservations/ledger, trials, top-ups and payment events. | Duplicate webhook/retry tests; failed jobs do not double-charge; cap enforcement is race-safe. |
| SP-051 | P1 | Billing administration, usage notices, cancellation and recovery flows. | Prices/currency/renewal displayed consistently; no unexpected overage; no Procore cleanup on cancellation. |
| SP-052 | P0 | Regional storage, backup/restore, secret management and tenant isolation verification. | Traceable evidence across all content stores/providers; restore/deletion tests. |
| SP-053 | P0 | Pilot load, fairness and provider-rate-limit tests. | 100-active-user workload documented; 1,000-page stress fixture; queue behavior and worker recovery measured. |
| SP-054 | P1 | Notifications, accessibility and support diagnostics. | Action-required messages usable without secrets/source content leakage; keyboard review works. |
| SP-055 | P0 | End-to-end quality/product acceptance and launch decision. | Domain-adjudicated held-out gates pass; owner reviews pricing; Procore/provider access ready; known limitations recorded. |

## Acceptance test matrix

| Test | Related PRD | Scenario and expected outcome |
|---|---|---|
| AT-01 | FR-01 | 1,000-page supported PDF: all pages accounted for, no truncation. |
| AT-02 | FR-01 | Unreadable page: exact page flagged, finalization blocked; no guessed source text. |
| AT-03 | FR-02 | Jev labels a requirement-bearing section unlikely: independent coverage pass still detects obligation. |
| AT-04 | FR-03 | Multiple deliverables in one section: separate supported rows with correct timing/type. |
| AT-05 | FR-03/04 | Inferred/manual item: evidence basis visible; explicit human disposition required. |
| AT-06 | §4 / FR-04 | Extract-only/admin without Procore access: neither role can bypass operation-level checks. |
| AT-07 | FR-04 | Concurrent reviewers and edit after approval: conflict surfaced, approval invalidated. |
| AT-08 | FR-05 | Existing visible match: reviewer skips; no create, update or delete. |
| AT-09 | FR-05 | Procore create succeeds but response is lost: unknown outcome and reconciliation, no blind retry. |
| AT-10 | FR-05 | Partial publish and repeated click: preserve successful receipts, retry eligible items only. |
| AT-11 | FR-06 | Crash/provider outage/browser close: resume from saved compatible units, retain human edits. |
| AT-12 | FR-07 | Recycle/permanent-delete/cancel subscription: local lifecycle only; no Procore mutation. |
| AT-13 | FR-08 | Canadian workspace: originals, derivatives, logs, backups and provider retention meet storage policy. |
| AT-14 | FR-09 | Concurrent starts/payment events/retries: one usage settlement, no silent overage. |
| AT-15 | §9 | Frozen held-out books: recall ≥95%, precision ≥90%, material correction ≤10%, cohort report provided. |
| AT-16 | §10 | 100-user pilot workload: interactive response and queue/recovery targets measured, not assumed. |
| AT-17 | FR-02 / §4 | Malicious instructions inside a PDF: treated as source content, never authorization or executable instructions. |
| AT-18 | §4 / FR-05 | Access revoked while queued: content access and writes blocked; no substitution of an admin identity. |
| AT-19 | FR-05 | A private Procore record is not visible: duplicate UI reveals no hidden content and states check scope. |
| AT-20 | FR-05 | Custom required field/spec-section mapping missing: actionable preflight error, no invented value. |
| AT-21 | FR-04/08 | Export/download link after revocation or expiry: denied; no public source URLs. |
| AT-22 | §9 | Embedded reference register: excluded from model input so scoring cannot copy the answers. |

## Definition of release-ready

Core functional criteria and held-out quality gates pass with evidence. Regional storage and Procore permission/write behavior are verified in the intended environment. Pricing is approved before live billing. Providers are actually available to the deployed account. Restore/retry/unknown-publish runbooks work. No unfinished mocks are represented as integrations. The owner reviews remaining risks and the documented limits before pilot activation.
