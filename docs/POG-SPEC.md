# PoG Framework Technical Specification v1

## Document Control
- Document ID: POG-TECH-SPEC-V1
- Version: 1.0.0
- Status: Draft, open for review
- Date: 2026-04-06
- Owner: Smart STB SARL
- Product Context: PREVORN / PoG Framework
- Audience: Backend, Frontend, Platform, Security, QA, Product, Architecture

## 1. Purpose

This document defines the technical specification for the PoG Framework as an operational governance and evidence framework. Its purpose is to standardize how real events, incidents, decisions, evidence artifacts, seals, and verification states are represented, processed, rendered, and validated inside the product stack.

This specification is intended to:
- remove ambiguity for engineering teams,
- define canonical entities and behavior,
- reduce inconsistent implementations,
- enable conformance testing,
- establish a defensible technical baseline.

## 2. Scope

This specification covers:
- canonical data model,
- core entities,
- state machines,
- integrity invariants,
- sealing behavior,
- verification behavior,
- revocation behavior,
- API contracts,
- error model,
- versioning policy,
- security requirements,
- conformance requirements.

## 3. Non-Goals

This specification does **not**:
- define an open industry standard,
- provide legal or regulatory certification,
- replace enterprise logging, SIEM, GRC, or case management tooling,
- guarantee compliance with any regulation,
- define cryptographic signing as mandatory in v1,
- define cross-vendor interoperability requirements beyond stable schemas and semantics.

## 4. Normative Language

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHOULD**, **SHOULD NOT**, and **MAY** are to be interpreted as normative requirements.

## 5. Architectural Principles

1. PoG artifacts MUST be derived from real operational objects.
2. PoG artifacts MUST remain traceable to source evidence.
3. PoG seals MUST be tamper-evident.
4. Verification results MUST be independently retrievable from the rendering layer.
5. Revocation MUST be explicit and durable.
6. Multi-tenant attribution ambiguity MUST be treated as a severity-1 integrity issue.
7. Rendering MUST NOT become the source of truth; the canonical record remains the structured data model.
8. State transitions MUST be deterministic and auditable.

## 6. Trust Boundaries

### 6.1 Boundaries
- **Boundary A**: External event source -> ingestion endpoint
- **Boundary B**: Ingestion -> normalization pipeline
- **Boundary C**: Normalization -> correlation engine
- **Boundary D**: Correlation -> incident engine
- **Boundary E**: Incident engine -> evidence collection
- **Boundary F**: Evidence pack generation -> seal generation
- **Boundary G**: Seal store -> verification service
- **Boundary H**: Verification service -> public verify endpoint
- **Boundary I**: API layer -> UI rendering
- **Boundary J**: Tenant-scoped data access -> administrative functions

### 6.2 Trust Assumptions
- Event sources MAY be noisy, incomplete, or malicious.
- The ingestion boundary MUST validate format, attribution, and source metadata.
- Internal services are NOT implicitly trusted to bypass integrity checks.
- Verification endpoints MUST NOT rely on UI rendering to establish validity.
- Database contents MAY be corrupted by logic errors or privileged misuse; integrity checks MUST detect inconsistencies.

## 7. Core Entities

PoG Framework v1 defines the following canonical entities:
- Event
- Normalized Event
- Correlation
- Incident
- Governance Decision
- Evidence Item
- Evidence Pack
- Seal
- Verification Record
- Revocation Record
- Lineage Record
- Policy Reference
- Rendered Artifact

## 8. Canonical Data Model

## 8.1 Event
```json
{
  "event_id": "uuid",
  "tenant_id": "uuid",
  "event_source": "string",
  "event_type": "string",
  "event_time": "ISO8601 UTC",
  "received_at": "ISO8601 UTC",
  "raw_ref": "string",
  "source_ip": "string|null",
  "host_id": "string|null",
  "identity_id": "string|null",
  "severity": "low|medium|high|critical|unknown",
  "schema_version": "string"
}
```

### Requirements
- `event_id` MUST be globally unique.
- `tenant_id` MUST be present and valid.
- `event_time` MUST represent observed time when available.
- `received_at` MUST represent system reception time.
- `raw_ref` MUST reference raw source material or an immutable locator.

## 8.2 Normalized Event
```json
{
  "normalized_event_id": "uuid",
  "event_id": "uuid",
  "tenant_id": "uuid",
  "normalized_type": "string",
  "normalized_at": "ISO8601 UTC",
  "parser_version": "string",
  "fields": {},
  "schema_version": "string"
}
```

### Requirements
- A normalized event MUST link to exactly one source event.
- Normalization MUST preserve tenant attribution.
- Parser version MUST be stored.

## 8.3 Correlation
```json
{
  "correlation_id": "uuid",
  "tenant_id": "uuid",
  "rule_id": "string",
  "rule_version": "string",
  "correlated_at": "ISO8601 UTC",
  "confidence_score": 0.0,
  "severity_score": 0.0,
  "source_event_ids": ["uuid"],
  "kill_chain_stage": "string|null",
  "schema_version": "string"
}
```

### Requirements
- `source_event_ids` MUST contain at least one event.
- `confidence_score` MUST be bounded between 0 and 1.
- `rule_id` and `rule_version` MUST be stored for traceability.

## 8.4 Incident
```json
{
  "incident_id": "uuid",
  "tenant_id": "uuid",
  "incident_type": "string",
  "created_at": "ISO8601 UTC",
  "updated_at": "ISO8601 UTC",
  "status": "open|in_progress|contained|closed|reopened",
  "severity": "low|medium|high|critical",
  "confidence_score": 0.0,
  "summary": "string",
  "narrative": "string",
  "mitre_tactics": ["string"],
  "mitre_techniques": ["string"],
  "ioc_refs": ["uuid"],
  "correlation_ids": ["uuid"],
  "affected_assets": ["string"],
  "affected_identities": ["string"],
  "timeline_refs": ["uuid"],
  "schema_version": "string"
}
```

### Requirements
- `incident_id` MUST be globally unique.
- `narrative` SHOULD be fact-based and structured.
- MITRE mappings SHOULD be present where applicable.
- Timeline references SHOULD be ordered chronologically.

## 8.5 Governance Decision
```json
{
  "decision_id": "uuid",
  "tenant_id": "uuid",
  "related_incident_id": "uuid|null",
  "decision_type": "allow|block|challenge|defer|require_human_approval",
  "decision_reason": "string",
  "policy_ref": "string|null",
  "approval_mode": "auto|human_gate|hybrid",
  "human_gate_status": "not_required|pending|approved|rejected|expired",
  "decided_at": "ISO8601 UTC",
  "decider_identity": "string",
  "schema_version": "string"
}
```

### Requirements
- Every consequential automated action MUST reference a governance decision.
- `approval_mode` MUST be explicit.
- If `approval_mode=human_gate`, then `human_gate_status` MUST NOT be `not_required`.

## 8.6 Evidence Item
```json
{
  "evidence_item_id": "uuid",
  "tenant_id": "uuid",
  "source_type": "event|incident|artifact|note|finding|external_reference",
  "source_ref": "string",
  "collected_at": "ISO8601 UTC",
  "observed_at": "ISO8601 UTC|null",
  "summary": "string",
  "classification": "raw|parsed|derived|analyst_authored|system_authored",
  "confidence_score": 0.0,
  "lineage_hash": "sha256:...",
  "schema_version": "string"
}
```

### Requirements
- Each evidence item MUST have a `source_ref`.
- `classification` MUST indicate whether the item is raw, parsed, derived, or authored.
- `lineage_hash` MUST be stable for the item payload.

## 8.7 Evidence Pack
```json
{
  "evidence_pack_id": "uuid",
  "pack_number": "string",
  "tenant_id": "uuid",
  "created_at": "ISO8601 UTC",
  "status": "draft|packed|sealed|revoked|superseded",
  "incident_id": "uuid|null",
  "decision_ids": ["uuid"],
  "evidence_item_ids": ["uuid"],
  "evidence_count": 0,
  "source_count": 0,
  "generator_version": "string",
  "schema_version": "string",
  "quality_score": 0.0
}
```

### Requirements
- A pack MUST contain one or more evidence items.
- `pack_number` MUST be unique within the product environment.
- `quality_score` SHOULD be derived from completeness and consistency checks.

## 8.8 Seal
```json
{
  "seal_id": "uuid",
  "object_type": "evidence_pack|incident_report|decision_record",
  "object_id": "uuid",
  "tenant_id": "uuid",
  "sealed_at": "ISO8601 UTC",
  "payload_hash": "sha256:...",
  "hash_algorithm": "SHA-256",
  "previous_chain_hash": "sha256:...|null",
  "chain_position": 0,
  "seal_status": "valid|invalid|revoked|superseded",
  "schema_version": "string"
}
```

### Requirements
- A seal MUST reference exactly one sealed object.
- `payload_hash` MUST be computed from canonical serialized payload.
- `chain_position` MUST be monotonic within the applicable chain.
- `seal_status` MUST NOT be inferred from UI state.

## 8.9 Verification Record
```json
{
  "verification_id": "uuid",
  "seal_id": "uuid",
  "verified_at": "ISO8601 UTC",
  "verification_result": "valid|invalid|unknown|revoked|error",
  "chain_integrity": "verified|broken|unknown",
  "verifier_version": "string",
  "failure_reason": "string|null",
  "schema_version": "string"
}
```

### Requirements
- Verification MUST produce a durable record for internal audit.
- `failure_reason` MUST be populated for invalid or error results.

## 8.10 Revocation Record
```json
{
  "revocation_id": "uuid",
  "seal_id": "uuid",
  "revoked_at": "ISO8601 UTC",
  "revoked_by": "string",
  "revocation_reason": "string",
  "replacement_seal_id": "uuid|null",
  "schema_version": "string"
}
```

### Requirements
- Revocation MUST be explicit, durable, and queryable.
- A revoked seal MUST remain visible as revoked.

## 8.11 Rendered Artifact
```json
{
  "rendered_artifact_id": "uuid",
  "object_type": "evidence_pack",
  "object_id": "uuid",
  "render_format": "json|html|pdf",
  "render_version": "string",
  "generated_at": "ISO8601 UTC",
  "binary_hash": "sha256:...",
  "schema_version": "string"
}
```

### Requirements
- Rendering MUST NOT modify the canonical source data.
- Rendered outputs SHOULD include visible integrity metadata.

## 9. Canonical Serialization Rules

- Canonical payload serialization MUST use stable field order.
- Timestamps MUST be normalized to UTC in ISO8601 format.
- Null and empty semantics MUST be consistent across services.
- Hash input generation MUST be deterministic.
- Schema version MUST be included in all sealable objects.

## 10. State Machines

## 10.1 Evidence Lifecycle
`collected -> normalized -> correlated -> incident_created -> enriched -> packed -> sealed -> verified -> revoked|superseded`

### Transition Requirements
- An object MUST NOT move from `sealed` back to `draft`.
- `verified` MAY occur multiple times, but only the latest verification result is current.
- `revoked` is terminal for a seal.
- `superseded` indicates replacement by a newer valid sealed object.

## 10.2 Decision Lifecycle
`proposed -> evaluated -> approved|blocked|challenged|deferred -> executed -> sealed -> verified -> revoked`

### Transition Requirements
- `blocked` MUST NOT transition to `executed`.
- `require_human_approval` MUST pass through `pending` and then `approved|rejected|expired`.
- Executed actions MUST reference the decision that authorized them.

## 11. Integrity Invariants

The following invariants are REQUIRED:

1. A sealed object's payload hash MUST match its canonical payload.
2. A revoked seal MUST NOT verify as valid.
3. A tenant mismatch between sealed object and source lineage MUST fail verification.
4. A broken chain link MUST downgrade `chain_integrity` to `broken`.
5. An evidence pack without traceable sources MUST NOT be marked PoG-complete.
6. An executed action without a linked decision MUST NOT be PoG-compliant.
7. A decision requiring human approval MUST NOT be treated as approved without explicit approval evidence.
8. A rendering hash mismatch MUST raise an integrity alert.
9. A schema version mismatch MAY render an object unverifiable unless explicitly supported.
10. Verification MUST NOT rely on client-side logic to establish validity.

## 12. Sealing Model

### 12.1 Sealing Preconditions
Before sealing, the system MUST verify:
- object exists,
- object is tenant-attributed,
- canonical payload is serializable,
- source lineage is resolvable,
- required fields are present,
- object state is eligible for sealing.

### 12.2 Canonical Hashing
- Payload hashing MUST use SHA-256 in v1.
- The canonical payload MUST exclude transient UI-only metadata.
- The hash input specification MUST remain stable for the schema version.

### 12.3 Chain Linking
- `previous_chain_hash` SHOULD be stored when chaining is enabled.
- Chain linking MUST be deterministic for the configured chain scope.
- Chain corruption MUST be detectable by verification.

### 12.4 Sealing Output
Sealing MUST produce:
- seal record,
- payload hash,
- timestamp,
- status,
- chain position,
- internal audit event.

## 13. Verification Model

### 13.1 Verification Inputs
The verifier MUST process:
- seal_id,
- sealed object metadata,
- canonical payload,
- current revocation state,
- chain position and predecessor metadata where applicable.

### 13.2 Verification Result Semantics
- `valid`: payload, lineage, and revocation state are consistent.
- `invalid`: one or more integrity checks failed.
- `unknown`: object cannot be resolved or schema unsupported.
- `revoked`: object was valid historically but has been revoked.
- `error`: verification failed due to processing issue or unavailable dependency.

### 13.3 Public Verification
The public verification endpoint MUST return:
- protocol_version,
- seal_id,
- object_type,
- seal_status,
- verification_result,
- revocation_status,
- sealed_at,
- payload_hash,
- hash_algorithm,
- chain_integrity,
- chain_position,
- verified_by.

## 14. Revocation Model

### 14.1 Revocation Triggers
Revocation MAY occur for:
- incorrect source attribution,
- integrity failure,
- evidence substitution,
- duplicate sealing error,
- legal removal requirement,
- superseding corrected artifact.

### 14.2 Revocation Behavior
- Revoked objects MUST remain queryable.
- Revocation reason MUST be recorded.
- Replacement seal MAY be referenced if applicable.
- Public verification MUST reveal revocation status.

## 15. API Contracts

## 15.1 GET /api/v1/pog/verify/{seal_id}
### Success Response
```json
{
  "protocol_version": "1.0",
  "seal_id": "uuid",
  "object_type": "evidence_pack",
  "seal_status": "valid",
  "verification_result": "valid",
  "revocation_status": "not_revoked",
  "sealed_at": "ISO8601 UTC",
  "payload_hash": "sha256:...",
  "hash_algorithm": "SHA-256",
  "chain_integrity": "verified",
  "chain_position": 47,
  "previous_chain_hash": "sha256:...|null",
  "verified_by": {
    "organization": "Smart STB SARL",
    "platform": "PREVORN",
    "framework": "PoG Framework"
  }
}
```

## 15.2 GET /api/v1/pog/packs/{pack_id}
Returns pack metadata and object state.

## 15.3 GET /api/v1/pog/packs/{pack_id}/lineage
Returns source references and lineage metadata.

## 15.4 GET /api/v1/pog/decisions/{decision_id}
Returns governance decision metadata.

## 15.5 POST /api/v1/pog/packs/{pack_id}/seal
Creates a new seal if preconditions are satisfied.

## 15.6 POST /api/v1/pog/seals/{seal_id}/revoke
Revokes an existing seal.

## 16. Error Codes

| Code | Meaning | HTTP |
|---|---|---|
| POG-400-001 | malformed_identifier | 400 |
| POG-400-002 | invalid_request_payload | 400 |
| POG-401-001 | unauthorized | 401 |
| POG-403-001 | forbidden | 403 |
| POG-404-001 | seal_not_found | 404 |
| POG-404-002 | object_not_found | 404 |
| POG-409-001 | invalid_state_transition | 409 |
| POG-409-002 | already_sealed | 409 |
| POG-409-003 | already_revoked | 409 |
| POG-422-001 | integrity_check_failed | 422 |
| POG-422-002 | tenant_attribution_mismatch | 422 |
| POG-422-003 | unsupported_schema_version | 422 |
| POG-500-001 | verification_processing_error | 500 |
| POG-503-001 | verifier_dependency_unavailable | 503 |

## 17. Versioning Policy

- All canonical PoG objects MUST include `schema_version`.
- API endpoints MUST be versioned in the path.
- Breaking schema changes MUST increment the major version.
- Non-breaking additive changes SHOULD increment the minor version.
- Deprecated fields MUST remain readable for at least one supported major cycle unless security risk forbids it.

## 18. Compatibility Policy

- Readers SHOULD support at least the current major version and one previous supported major version.
- Seal verification MUST fail safely on unknown incompatible schemas.
- Rendering SHOULD degrade gracefully for unsupported optional fields.
- Migration tooling SHOULD be provided for persisted core objects.

## 19. Performance Expectations

- Public verification SHOULD respond within 500 ms p95 under normal load.
- Sealing SHOULD complete within 2 seconds p95 for standard evidence packs.
- Integrity mismatch alerts SHOULD be emitted within 60 seconds of detection.
- Timeline rendering SHOULD support packs containing at least 500 evidence items without corruption.
- Verification endpoints MUST fail closed on integrity uncertainty.

## 20. Security Requirements

1. Canonical payload hashing MUST occur server-side.
2. Verification MUST occur server-side.
3. Public verification MUST expose only approved metadata.
4. Tenant-scoped objects MUST NOT leak across tenants.
5. Administrative resealing MUST be access-controlled and audited.
6. Seal revocation MUST require elevated authorization.
7. Integrity-related failures MUST be logged as security-relevant events.
8. Client-side rendering MUST NOT determine object validity.
9. Replayed stale payloads MUST NOT overwrite current valid objects.
10. Unsupported schema payloads MUST fail safely.

## 21. Conformance Requirements

An implementation is PoG-conformant at v1 only if it can demonstrate:
- canonical objects present,
- stable serialization,
- deterministic hashing,
- seal lifecycle support,
- verification support,
- revocation support,
- tenant attribution enforcement,
- failure-safe behavior,
- version awareness,
- audit logging for integrity events.

## 22. Open Items for v1.1+
The following are deferred:
- digital signatures,
- external attestation export format,
- vendor-neutral profile definitions,
- cryptographic transparency log integration,
- detached verification bundles,
- delegated verification tokens.

## 23. Definition of Done for Spec Adoption

This specification is considered adopted only when:
- schemas are implemented,
- API contracts are live,
- state transition guards are enforced,
- tests cover invariants,
- public verification works,
- revocation is functional,
- documentation is published,
- security review is completed.
