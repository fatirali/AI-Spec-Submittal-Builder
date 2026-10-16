# Spectre

Spectre is a Construction Spec to Submittal Builder. It turns project specifications into a draft submittal register, keeps a human in control of the result, and prepares approved data for Procore.

## Intended workflow

1. **Upload specifications** — Provide the project specification documents to be processed.
2. **Extract and classify** — Identify potential submittal requirements and organize them into a structured draft register.
3. **Human review** — Let a reviewer verify, correct, approve, or reject extracted items before anything is sent onward.
4. **Push to Procore** — Send only reviewed and approved submittal data to Procore.

## Model-agnostic approach

Spectre is intended to keep the workflow separate from the AI model used for extraction and classification. It should support two operating modes:

- **Managed models:** Spectre provides and operates the model access, with usage included or metered as part of the service.
- **Bring your own model (BYOM):** A customer connects a supported model provider under its own account and usage arrangement.

The supported providers, model-quality requirements, data-handling controls, and commercial packaging are still to be validated. This repository does not contain credentials or prescribe a specific model vendor.

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
- [ ] Support a BYOM configuration without storing credentials in source control.
- [ ] Establish a representative evaluation set and baseline extraction-quality measures.
- [ ] Define project isolation, retention, access-control, and deletion requirements.
- [ ] Add usage visibility suitable for subscription, credit, or outcome-based packaging experiments.

## Product principles

- Human approval is required before downstream submission.
- Source traceability is part of every extracted requirement.
- Model choice should not change the reviewer workflow or output contract.
- Project data and credentials must not be committed to this repository.
- Extraction quality, failure handling, security, and auditability are MVP concerns.

## Status

Early product scaffold. Architecture, implementation stack, integrations, licensing, and pricing remain open decisions.

## Contributing

Contribution guidelines will be added after the project scope and licensing approach are decided.
