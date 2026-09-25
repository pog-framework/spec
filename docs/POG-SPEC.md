# PoG Framework Technical Specification v1

## Document Control
- Document ID: POG-TECH-SPEC-V1
- Version: 1.1.0
- Status: Draft, open for review
- Date: 2026-09-25
- Supersedes: 1.0.0 (draft, 2026-04-06)
- Owner: Smart STB SARL
- Product Context: PREVORN / PoG Framework
- Audience: Backend, Frontend, Platform, Security, QA, Product, Architecture

### Changes in 1.1.0
Version 1.1.0 answers review issue #2 (Seal narrower than the implemented Seal, external anchoring unspecified). It adds a mandatory signature and an optional timestamp token to the Seal (8.8), a new External Anchor entity (8.12), a new trust boundary (6.1, Boundary K), the signing, timestamping and anchoring rules (12.5 to 12.7), two distinct verification outcomes (13.2 and 13.4), the detached verification bundle (13.5), a key distribution endpoint (15.7), and the compatibility rules for seals produced under 1.0.0 (17). Annex A, informative, maps the PoG entities onto an AI-assisted software change record. The change is additive to the data model; see section 17 for migration.

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
- mandate a particular anchoring technology or anchoring authority,
- define the key distribution infrastructure itself (it is referenced in 12.5 and 15.7 and deserves its own document),
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
9. Verification MUST distinguish what is established from material the producer publishes from what is established against a reference the producer cannot revise after the fact.

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
- **Boundary K**: Seal store -> external anchoring authority

The relevant property of Boundary K is direction. Material crosses outward only. The producing system has no privileged channel to revise what the authority has recorded, and the authority's record MUST remain retrievable by a party other than the producer.

### 6.2 Trust Assumptions
- Event sources MAY be noisy, incomplete, or malicious.
- The ingestion boundary MUST validate format, attribution, and source metadata.
- Internal services are NOT implicitly trusted to bypass integrity checks.
- Verification endpoints MUST NOT rely on UI rendering to establish validity.
- Database contents MAY be corrupted by logic errors or privileged misuse; integrity checks MUST detect inconsistencies.
- Anchoring authorities MAY be unavailable. Their unavailability MUST NOT block sealing and MUST NOT be hidden from verification output (12.7).
- The signing key distribution channel is a trust dependency of every verifier. It MUST be documented and MUST NOT depend on the producer's user interface (12.5).

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
- External Anchor (since 1.1.0)
- Signing Key Record (since 1.1.0)

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
  "chain_scope": "string",
  "signature": {
    "algorithm": "Ed25519",
    "public_key_id": "string",
    "value": "base64"
  },
  "timestamp_token": {
    "profile": "RFC3161",
    "authority": "string",
    "qualified": false,
    "value": "base64"
  },
  "seal_status": "valid|invalid|revoked|superseded",
  "schema_version": "string"
}
```

`timestamp_token` MAY be null. `signature` MUST be present on every seal produced under schema version 1.1 or later; see section 17 for seals produced under 1.0.

### Requirements
- A seal MUST reference exactly one sealed object.
- `payload_hash` MUST be computed from canonical serialized payload.
- `chain_position` MUST be monotonic within the applicable chain, and `chain_scope` MUST identify that chain (12.3).
- `previous_chain_hash` MUST be null when `chain_position` is 0 and MUST be present otherwise.
- `signature.value` MUST be computed over the canonical serialized payload of the seal, never over a rendering of it (12.5).
- `signature.public_key_id` MUST resolve to a Signing Key Record (8.13) through the key distribution channel defined in 12.5.
- `timestamp_token.qualified` MUST reflect whether the issuing authority is qualified under the regime applicable to the tenant. A verifier MUST be able to read this flag without interpreting the authority name.
- An implementation MUST NOT present a non-qualified timestamp as qualified in any rendering or verification output.
- `seal_status` MUST NOT be inferred from UI state.

## 8.9 Verification Record
```json
{
  "verification_id": "uuid",
  "seal_id": "uuid",
  "verified_at": "ISO8601 UTC",
  "verification_result": "valid|invalid|unknown|revoked|error",
  "chain_integrity": "verified|broken|unknown",
  "signature_status": "valid|invalid|unsigned|key_unknown",
  "anchor_status": "not_anchored|pending|anchored|superseded",
  "anchor_id": "uuid|null",
  "verifier_version": "string",
  "failure_reason": "string|null",
  "schema_version": "string"
}
```

### Requirements
- Verification MUST produce a durable record for internal audit.
- `failure_reason` MUST be populated for invalid or error results.
- `signature_status` and `anchor_status` MUST be recorded separately from `verification_result`; neither MAY be folded into it (13.2).

## 8.10 Revocation Record
```json
{
  "revocation_id": "uuid",
  "seal_id": "uuid|null",
  "anchor_id": "uuid|null",
  "revoked_at": "ISO8601 UTC",
  "revoked_by": "string",
  "revocation_reason": "string",
  "replacement_seal_id": "uuid|null",
  "replacement_anchor_id": "uuid|null",
  "schema_version": "string"
}
```

### Requirements
- Revocation MUST be explicit, durable, and queryable.
- A revoked seal MUST remain visible as revoked.
- Exactly one of `seal_id` or `anchor_id` MUST be set. A revocation record with `anchor_id` records the supersession of an External Anchor (12.7) and MUST be surfaced to the tenant.

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

## 8.12 External Anchor
```json
{
  "anchor_id": "uuid",
  "tenant_id": "uuid",
  "anchor_type": "opentimestamps|rfc3161_qualified|external_registry",
  "authority": "string",
  "regime": "eIDAS|ZertES|none",
  "submitted_at": "ISO8601 UTC",
  "confirmed_at": "ISO8601 UTC|null",
  "chain_scope": "string",
  "anchored_chain_position": 0,
  "anchored_digest": "sha256:...",
  "external_reference": "string|null",
  "status": "pending|confirmed|failed|superseded",
  "schema_version": "string"
}
```

### Coverage semantics
An anchor commits to one chain position: `anchored_digest` is the seal hash (12.3) of the seal at `anchored_chain_position` in `chain_scope`. Because every seal hash commits to its predecessor, a confirmed anchor covers that seal and every seal before it in the same scope. It covers nothing after it. A seal is anchored when at least one confirmed anchor in its scope has `anchored_chain_position` greater than or equal to the seal's `chain_position`, the chain between the two positions is intact, and the recomputed seal hash at the anchored position equals `anchored_digest`. Continuous coverage therefore requires repeated anchoring; adding an anchor at a later position is normal operation, not supersession.

### Requirements
- An implementation MUST support at least one `anchor_type` whose confirmation is observable outside the producing system.
- An implementation MUST declare its anchoring cadence per chain scope: per seal, or periodic with a maximum interval in seals and in time. Evidence packs MUST be anchored per seal or at an interval no longer than the declared target confirmation latency (12.7).
- `anchored_digest` MUST equal the seal hash of the seal at `anchored_chain_position`, and `submitted_at` MUST be the time at which it was handed to the authority.
- `external_reference` MUST be populated when `status` is `confirmed` and MUST allow a party other than the producer to retrieve the authority's record.
- Where the applicable regime requires a qualified authority, `regime` MUST record which regime was applied for the tenant at enrolment.
- An anchor MUST NOT be superseded silently. Superseding an anchor MUST produce a Revocation Record (8.10) carrying the superseded `anchor_id` and MUST be surfaced to the tenant.

## 8.13 Signing Key Record
```json
{
  "public_key_id": "string",
  "algorithm": "Ed25519",
  "public_key": "base64",
  "valid_from": "ISO8601 UTC",
  "valid_to": "ISO8601 UTC|null",
  "status": "active|retired|compromised",
  "compromised_at": "ISO8601 UTC|null",
  "schema_version": "string"
}
```

### Requirements
- Every `public_key_id` referenced by a seal MUST resolve to exactly one Signing Key Record.
- A retired key MUST remain published with its validity period. Seals signed during that period MUST continue to verify.
- A key marked `compromised` MUST carry `compromised_at`. Seals whose `sealed_at` is at or after `compromised_at` MUST return `signature_status=invalid`. Seals sealed before it MUST return `signature_status=valid`, and the verification output MUST state that the key was later reported compromised. Where a trusted timestamp token exists, the token time takes precedence over `sealed_at` for this comparison.

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
11. A seal whose signature does not verify against the Signing Key Record for its `public_key_id` MUST NOT verify as valid.
12. A seal MUST NOT be presented as independently anchored unless its chain position is covered by a confirmed External Anchor in the same chain scope.
13. A non-qualified timestamp MUST NOT be presented as qualified.
14. The supersession of an External Anchor without a Revocation Record MUST be treated as an integrity failure.

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
- Chaining MUST be enabled for evidence packs. It SHOULD be enabled for incident reports and decision records. An implementation MUST declare which sealed object types belong to each chain scope.
- The seal hash of a seal is SHA-256 over its canonical seal payload as defined in 12.5 (the fields that are signed). It commits to the object's `payload_hash` and to the seal's own `previous_chain_hash`, so each seal hash commits to the whole chain before it.
- `previous_chain_hash` MUST be stored for every seal whose `chain_position` is greater than 0, and MUST equal the seal hash of the immediately preceding seal in the same chain scope. Linking on the object's `payload_hash` alone is not conformant, because it would leave the predecessor's own link outside the commitment.
- Chain linking MUST be deterministic for the configured chain scope.
- Chain corruption MUST be detectable by verification.

### 12.4 Sealing Output
Sealing MUST produce:
- seal record,
- payload hash,
- signature,
- timestamp,
- timestamp token when a timestamping authority is configured,
- status,
- chain position,
- internal audit event.

### 12.5 Signing
- Every seal MUST be signed server-side, over the canonical serialized seal payload, with the algorithm declared in `signature.algorithm`. Ed25519 is REQUIRED in 1.1; other algorithms MAY be added in a later version.
- The canonical seal payload for signing MUST include `object_type`, `object_id`, `tenant_id`, `sealed_at`, `payload_hash`, `hash_algorithm`, `previous_chain_hash`, `chain_position`, `chain_scope` and `schema_version`, in that order, and MUST exclude `signature`, `timestamp_token` and `seal_status`.
- Signing keys MUST be distributed through a documented channel that does not depend on the producer's user interface. The endpoint in 15.7 is the minimum; publication of the same records outside the producer's infrastructure is RECOMMENDED.
- Key rotation MUST NOT invalidate seals produced under a previous key. Retired keys MUST remain published with their validity period (8.13).
- Private keys MUST NOT be exportable through any product interface.

### 12.6 Timestamping
- When a timestamping authority is configured, the seal MUST carry a `timestamp_token` obtained over the `payload_hash`.
- The token profile in 1.1 is RFC 3161. The `authority` field MUST identify the issuer. The `qualified` flag MUST be set from the authority's status under the regime recorded on the tenant's External Anchor (`regime`), never from the authority's name.
- Timestamping authority unavailability MUST NOT block sealing. The seal MUST then carry `timestamp_token=null`, the condition MUST be logged as an integrity-relevant event, and verification output MUST show the absence.

### 12.7 Anchoring
- Anchoring MAY be asynchronous. While an anchor has `status=pending`, seals in its coverage MUST NOT be presented as anchored, and verification MUST return `anchor_status=pending` rather than failing.
- Anchoring authority unavailability MUST NOT block sealing. It MUST produce an explicit pending state, visible in verification output, and MUST NOT be silently retried into a different `anchor_type`.
- An implementation MUST declare a target confirmation latency for each `anchor_type`. An anchor that remains `pending` beyond that target MUST raise an integrity alert; the pending state itself is not a defect, the silent overrun is.
- Superseding an anchor MUST produce a Revocation Record referencing the superseded `anchor_id`, MUST record the replacement `anchor_id`, and MUST be surfaced to the tenant. The superseded anchor MUST remain queryable.
- An anchor's `external_reference` MUST be included in the public verification output (13.3) once the anchor is confirmed.
- Anchors accumulate. A new anchor at a later position does not supersede an earlier one; supersession is the replacement of an existing anchor record and follows the rule above.

## 13. Verification Model

### 13.1 Verification Inputs
The verifier MUST process:
- seal_id,
- sealed object metadata,
- canonical payload,
- current revocation state,
- chain position and predecessor metadata where applicable,
- the Signing Key Record for `signature.public_key_id`,
- the timestamp token when present,
- the External Anchor covering the seal's chain scope and position, when one exists.

### 13.2 Verification Result Semantics
`verification_result` keeps the 1.0 semantics:
- `valid`: payload, lineage, signature, and revocation state are consistent.
- `invalid`: one or more integrity checks failed.
- `unknown`: object cannot be resolved or schema unsupported.
- `revoked`: object was valid historically but has been revoked.
- `error`: verification failed due to processing issue or unavailable dependency.

Two further fields qualify the result and MUST NOT be collapsed into it:
- `signature_status`: `valid`, `invalid`, `unsigned` (seal produced under schema 1.0, section 17), or `key_unknown` (the `public_key_id` does not resolve). `invalid` and `key_unknown` MUST force `verification_result=invalid`. `unsigned` MUST NOT force it.
- `anchor_status`: `not_anchored` (no anchor at or beyond this position), `pending` (an anchor at or beyond this position exists, not yet confirmed), `anchored` (a confirmed anchor at or beyond this position exists, the chain up to it is intact, and the recomputed seal hash at the anchored position equals `anchored_digest`; see 8.12, coverage semantics), `superseded` (every anchor that would cover this position was superseded; the Revocation Record MUST be referenced).

The two outcomes an implementation MUST distinguish are therefore:
- **Integrity verified**: `verification_result=valid` and `signature_status=valid`. Digest, signature and chain are internally consistent with material published by the producer.
- **Independently anchored**: integrity verified and, in addition, `anchor_status=anchored`. An undetected full-chain rewrite is excluded for the covered range because the reference point is held by a party the producer cannot revise.

### 13.3 Public Verification
The public verification endpoint MUST return:
- protocol_version,
- seal_id,
- object_type,
- seal_status,
- verification_result,
- signature_status,
- public_key_id,
- timestamp (authority and qualified flag, or null),
- revocation_status,
- sealed_at,
- payload_hash,
- hash_algorithm,
- chain_integrity,
- chain_position,
- chain_scope,
- anchor_status,
- anchor (type, authority, external_reference, anchored_chain_position, confirmed_at, or null),
- verified_by.

### 13.4 Verification With Producer-Published Material Only
A verifier that holds only material published by the producer (seals, payloads, key records, chain predecessors) can establish at most **Integrity verified**. Presenting that outcome as **Independently anchored** is a conformance failure. An implementation's rendering, API and documentation MUST use the two terms with these meanings and MUST NOT use "independent" for the first outcome.

### 13.5 Detached Verification Bundle
To make P4 (re-verification outside the producing system) executable rather than asserted, an implementation MUST be able to export, for any sealed object, a detached verification bundle: a single JSON document containing
- the seal record (8.8), including signature and timestamp token,
- the canonical payload of the sealed object,
- the Signing Key Record (8.13) for the seal's `public_key_id`,
- the seal hash (12.3) of each seal in the scope from the seal under examination up to the anchored position, in order, so that the verifier can recompute the chain,
- the External Anchor record (8.12) covering the seal, when one exists,
- the current Revocation Record(s), when any exist,
- the version of the verification procedure the bundle was produced for.

A verifier outside the producing system, holding the bundle and the authority's record referenced by `external_reference`, MUST be able to reach the same `verification_result`, `signature_status` and `anchor_status` as the producer's endpoint. The bundle format is versioned with `schema_version` and MUST NOT contain tenant-private fields beyond those approved for public verification (section 20, item 3).

## 14. Revocation Model

### 14.1 Revocation Triggers
Revocation MAY occur for:
- incorrect source attribution,
- integrity failure,
- evidence substitution,
- duplicate sealing error,
- legal removal requirement,
- superseding corrected artifact,
- supersession of an External Anchor (12.7),
- signing key compromise (8.13).

### 14.2 Revocation Behavior
- Revoked objects MUST remain queryable.
- Revocation reason MUST be recorded.
- Replacement seal MAY be referenced if applicable.
- Public verification MUST reveal revocation status.
- A Revocation Record for an anchor MUST reference the replacement anchor when one exists and MUST be returned by the anchor endpoint (15.8) for the superseded `anchor_id`.

## 15. API Contracts

## 15.1 GET /api/v1/pog/verify/{seal_id}
### Success Response
```json
{
  "protocol_version": "1.1",
  "seal_id": "uuid",
  "object_type": "evidence_pack",
  "seal_status": "valid",
  "verification_result": "valid",
  "signature_status": "valid",
  "public_key_id": "string",
  "timestamp": {
    "profile": "RFC3161",
    "authority": "string",
    "qualified": false
  },
  "revocation_status": "not_revoked",
  "sealed_at": "ISO8601 UTC",
  "payload_hash": "sha256:...",
  "hash_algorithm": "SHA-256",
  "chain_integrity": "verified",
  "chain_position": 47,
  "chain_scope": "string",
  "previous_chain_hash": "sha256:...|null",
  "anchor_status": "anchored",
  "anchor": {
    "anchor_type": "opentimestamps",
    "authority": "string",
    "external_reference": "string",
    "anchored_chain_position": 52,
    "confirmed_at": "ISO8601 UTC"
  },
  "verified_by": {
    "organization": "Smart STB SARL",
    "platform": "PREVORN",
    "framework": "PoG Framework"
  }
}
```

`timestamp` and `anchor` are null when absent. A response for a seal produced under schema 1.0 carries `signature_status=unsigned`, `public_key_id=null` and `protocol_version=1.1`.

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

## 15.7 GET /api/v1/pog/keys/{public_key_id}
Returns the Signing Key Record (8.13). This endpoint MUST be publicly reachable without authentication, MUST return retired and compromised keys, and MUST NOT be the only distribution channel (12.5).

## 15.8 GET /api/v1/pog/anchors/{anchor_id}
Returns the External Anchor record (8.12), its current status, and, when superseded, the Revocation Record and the replacement `anchor_id`. Publicly reachable without authentication.

## 15.9 GET /api/v1/pog/seals/{seal_id}/bundle
Returns the detached verification bundle (13.5). Publicly reachable without authentication for seals whose object is approved for public verification.

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
| POG-422-004 | signature_invalid | 422 |
| POG-422-005 | signing_key_unknown | 422 |
| POG-404-003 | anchor_not_found | 404 |
| POG-404-004 | signing_key_not_found | 404 |
| POG-500-001 | verification_processing_error | 500 |
| POG-503-001 | verifier_dependency_unavailable | 503 |

Signature and key errors apply to verification of a seal; they are never returned by the sealing endpoint, which MUST fail closed with `POG-500-001` if it cannot sign.

## 17. Versioning Policy

- All canonical PoG objects MUST include `schema_version`.
- API endpoints MUST be versioned in the path.
- Breaking schema changes MUST increment the major version.
- Non-breaking additive changes SHOULD increment the minor version.
- Deprecated fields MUST remain readable for at least one supported major cycle unless security risk forbids it.

### 17.1 Transition from 1.0 to 1.1
Version 1.1 adds fields to the Seal, the Verification Record and the Revocation Record, and adds two entities. No 1.0 field changes meaning. The transition is therefore a minor version, with the following rules:
- `schema_version` discriminates. A 1.1 verifier MUST accept seals produced under 1.0 and MUST report them with `signature_status=unsigned` and `anchor_status=not_anchored`, never as `invalid` on those grounds alone.
- A 1.1 implementation MUST NOT produce new unsigned seals.
- Re-sealing an existing artifact to add a signature MUST NOT be silent. It is a new seal, at a new chain position, recorded as superseding the original; the original seal MUST be preserved and MUST remain verifiable under the 1.0 rules.
- The tightening of chaining from SHOULD to MUST (12.3) applies to seals produced under 1.1. Chains that begin at the first 1.1 seal are conformant; an implementation MAY bring earlier 1.0 seals under coverage by re-sealing them under 1.1 as predecessors of the first 1.1 seal, recorded as such (this section, third rule), so that a later anchor commits to them.

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
11. Signing MUST occur server-side, and private keys MUST NOT be exportable through any product interface.
12. Signing key publication, retirement and compromise declaration MUST be access-controlled and audited.
13. Anchor supersession MUST require elevated authorization and MUST produce a Revocation Record.

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
- audit logging for integrity events,
- since 1.1: signature on every new seal, verifiable through the published key records,
- since 1.1: at least one anchor type observable outside the producing system,
- since 1.1: the two verification outcomes of 13.2 reported separately and never collapsed,
- since 1.1: detached verification bundle export (13.5).

## 22. Open Items for v1.2+
The following are deferred:
- vendor-neutral profile definitions (Annex A is a first, informative, profile),
- cryptographic transparency log integration beyond the anchor types of 8.12,
- delegated verification tokens,
- signature algorithms other than Ed25519,
- a normative key distribution document.

Resolved in 1.1.0: digital signatures (12.5), external attestation export format and detached verification bundles (13.5).

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

## Annex A (informative). Profile: AI-assisted software change record

This annex is informative. It does not add requirements. It shows how the entities of this specification carry the evidence record that European guidance on AI-assisted software development is converging on: what was changed, why, which checks were performed, what the results were, who reviewed or approved, and how a third party can verify all of it after the fact. The mapping below was used in the PoG contribution to the ENISA consultation on the Technical Advisory on AI-assisted software development (September 2026). A normative version is a candidate for 1.2.

| Evidence needed | PoG carrier | Notes |
|---|---|---|
| Change identifier, affected artefacts and their versions | Evidence Pack (8.7) `pack_number`; Evidence Items (8.6) with `source_type=artifact`, `source_ref` pointing at the artefact and version | One pack per change; the pack is the record. |
| AI-assisted origin: tool, model or workflow identifier; hash of the prompt or tool action | Evidence Item with `classification=system_authored`; `source_ref` carries the tool or model identifier; `lineage_hash` carries the hash of the prompt or action | Provenance is provable without storing the prompt in the clear. |
| Checks performed, results, tooling identifier | Evidence Items with `classification=derived`; one item per check, `source_ref` = tooling identifier and version | Results are part of the item payload and therefore of the pack hash. |
| Decision taken, role of the approver, time of decision | Governance Decision (8.5): `decision_type`, `approval_mode`, `human_gate_status`, `decider_identity`, `decided_at` | A change accepted without human line-by-line review is `approval_mode=auto` with an explicit `policy_ref`; it is recorded, not hidden. |
| SBOM delta when a dependency is added, removed or changed | Evidence Item with `source_type=artifact` and a payload listing added, removed and changed components with versions | The delta is what the reporting timeline of the Cyber Resilience Act needs first when a component turns out to be vulnerable. |
| Integrity: content hash, algorithm, chain position, trusted timestamp | Seal (8.8): `payload_hash`, `hash_algorithm`, `chain_position`, `previous_chain_hash`, `signature`, `timestamp_token` | Captured at the time of sealing, not reconstructed. |
| Verification by a party other than the producer | Public verification (13.3), detached verification bundle (13.5), External Anchor (8.12) | Integrity verified with producer-published material; independently anchored when a confirmed anchor covers the position. |

Maturity, as used in that contribution and consistent with the Claims Policy: documented (a written practice exists), logged (records exist and are retained), evidenced (records are captured at the time of the action and integrity-protected), verifiable (a third party can check the records with a published procedure). An implementation of this specification operates at the fourth level for the objects it seals; it says nothing about objects it does not seal.
