# Spectre

Spectre is a proprietary Construction Spec to Submittal Builder for general contractors. It turns project specifications into a traceable draft register, lets a project manager review and finalize it, and creates approved submittal records in Procore.

## Product and build documents

- [Product requirements document](docs/PRD.md) — confirmed scope, proposed defaults, workflows, permissions, data requirements, evaluation and pricing proposal.
- [Phased build plan](docs/BUILD-PLAN.md) — prioritized backlog, dependencies and acceptance test matrix.
- [Procore setup guide](docs/PROCORE-SETUP.md) — developer registration, sandbox validation and production access dependencies.

The PRD is the current product specification. Proposed settings are identified separately from confirmed decisions. The repository is a planning scaffold, not a working application.

## Intended workflow

1. **Upload specifications** — Provide the project specification documents to be processed.
2. **Extract and classify** — Identify potential submittal requirements and organize them into a structured draft register.
3. **Human review** — Let a reviewer verify, correct, approve, or reject extracted items before anything is sent onward.
4. **Push to Procore** — Send only reviewed and approved submittal data to Procore.

## Model architecture

The MVP uses managed Jev by TypeSafe for initial classification and managed OpenAI Sol/Astra models for extraction and verification. Exact routing, versions and provider storage settings must pass evaluation and integration checks. Classification must not silently exclude requirements from completeness checking.

Provider adapters keep workflow logic separate from models. Customer-supplied model access (BYOM) is a later feature, not an MVP requirement. No credentials are included.

## MVP scope

The first release should prove the complete human-reviewed workflow on representative construction specifications.

### Initial backlog

- [ ] Define the minimum input formats and document-size limits.
- [ ] Define the draft submittal-register schema and required Procore fields.
- [ ] Upload and securely store a project specification.
- [ ] Extract candidate submittal requirements with source references.
- [ ] Classify and normalize extracted requirements into the draft register.
- [ ] Show extraction status, failures, and items that need attention.
- [ ] Provide a review screen for editing, approving, and rejecting candidates.
- [ ] Preserve a clear audit trail from approved items back to source text.
- [ ] Validate approved records before export or push.
- [ ] Authenticate with Procore and push approved records to a selected project.
- [ ] Support a managed-model configuration.
- [ ] Provide managed Jev classification and evaluated Sol/Astra routing through replaceable adapters.
- [ ] Establish a representative evaluation set and baseline extraction-quality measures.
- [ ] Define project isolation, retention, access-control, and deletion requirements.
- [ ] Implement configurable Starter/Pro/Enterprise subscriptions and page usage accounting; obtain approval of proposed commercial terms before live billing.

## Product principles

- Human approval is required before downstream submission.
- Source traceability is part of every extracted requirement.
- Model choice should not change the reviewer workflow or output contract.
- Project data and credentials must not be committed to this repository.
- Extraction quality, failure handling, security, and auditability are MVP concerns.
- Existing Procore records are never overwritten or deleted; duplicate flags can be skipped by a reviewer.
- Each user's Procore permissions constrain their access and actions in Spectre.

## Status

PRD v0.1 drafted on 2026-10-18. Product licensing is proprietary. Implementation stack, external integration validation and final commercial terms remain to be resolved. No implementation or performance/accuracy results are claimed.

## Contributing

Contributor access and guidelines will be defined by the owner. This repository does not grant an open-source license.
