# PoG Framework Threat Model v1

## Document Control
- Document ID: POG-THREAT-MODEL-V1
- Version: 1.1.0
- Status: Draft, open for review
- Date: 2026-09-25
- Supersedes: 1.0.0 (draft, 2026-04-06)
- Owner: Smart STB SARL
- Audience: Security, Architecture, Backend, Platform, QA, Leadership

### Changes in 1.1.0
Adds the assets, actors, abuse cases, scenarios, controls and residual risks introduced by signatures, timestamp tokens and external anchoring (POG-SPEC 1.1.0, sections 8.8, 8.12, 8.13, 12.5 to 12.7 and 13). Answers review issue #2, section 6.

## 1. Purpose

This document defines the threat model for the PoG Framework. Its purpose is to identify the assets PoG protects, the trust assumptions it depends on, the attack surfaces it exposes, the abuse cases it must withstand, and the controls required to preserve the integrity and defensibility of PoG artifacts.

## 2. Security Objectives

PoG security aims to preserve:
- integrity of evidence,
- correctness of tenant attribution,
- traceability of lineage,
- stability of seal semantics,
- trustworthiness of verification outcomes,
- durability of revocation state,
- non-ambiguity of state transitions,
- auditability of consequential actions.

## 3. Assets

### 3.1 Primary Assets
- Canonical evidence item payloads
- Evidence packs
- Seals
- Verification records
- Revocation records
- Governance decisions
- Incident narratives
- Timeline ordering
- Tenant attribution metadata
- Hash values
- Chain linking metadata
- Schema version metadata
- Seal signatures and signing private keys
- Signing Key Records (public keys, validity periods, retirement and compromise state)
- Timestamp tokens
- External Anchor records and their external references
- Detached verification bundles

### 3.2 Secondary Assets
- Rendering templates
- Verification APIs
- Public verification page
- Audit logs
- Alerting rules
- Access control policies
- Migration scripts
- Admin tooling
- Observability telemetry
- Key distribution endpoint and any out-of-band key publication
- Anchoring authority integration

## 4. Actors

### 4.1 Legitimate Actors
- SOC analyst
- Security engineer
- Platform engineer
- Product backend service
- Verification service
- Pack generation service
- Administrator
- Auditor
- External customer verifier
- Timestamping authority
- Anchoring authority

### 4.2 Adversarial Actors
- External attacker
- Tenant user with malicious intent
- Privileged insider
- Compromised service account
- Rogue admin
- API abuser
- Supply-chain compromised component
- Accidental but destructive operator
- Compromised or malicious anchoring or timestamping authority
- Actor able to publish under a trusted `public_key_id`

## 5. Trust Assumptions

1. Upstream event sources MAY be malicious or malformed.
2. Internal services MAY contain bugs and MUST NOT be blindly trusted.
3. Privileged access is not equivalent to integrity.
4. Public verification consumers MUST be considered untrusted.
5. The database is authoritative for persistence but not beyond integrity checks.
6. UI rendering is not a trusted source of validity.
7. Schema migration is a security-relevant activity.
8. The producer has no privileged channel to revise what an anchoring authority has recorded (POG-SPEC Boundary K). If this assumption fails for a given authority, that authority provides no independence and MUST NOT be presented as an anchor.
9. Verifiers trust the key distribution channel. The channel, not the signature algorithm, is the weakest link of signature verification.
10. Anchoring and timestamping authorities MAY be unavailable at any time; the framework degrades to explicit pending states, never to false validity.

## 6. Threat Boundaries

- External connectors and webhook ingress
- Tenant-to-platform access boundary
- Service-to-service internal APIs
- Admin-to-platform operations
- Database persistence layer
- Seal generation path
- Verification path
- Rendering path
- Export/download path
- Monitoring and audit path
- Seal store to anchoring authority (outbound only)
- Key publication path

## 7. Attack Surfaces

- Webhook ingestion endpoints
- Event normalization services
- Correlation rule execution
- Incident enrichment logic
- Evidence collection jobs
- Seal creation endpoints
- Verification API
- Revocation endpoint
- Admin reseal flows
- PDF/HTML render generation
- Database migrations
- Cross-tenant queries
- Internal object lookup APIs
- Signing service and key storage
- Key distribution endpoint
- Anchor submission and confirmation polling
- Detached bundle export

## 8. Key Abuse Cases

### 8.1 Tenant Confusion
An attacker or logic flaw causes evidence, incidents, or packs to be linked to the wrong tenant.

**Impact**
- catastrophic trust failure,
- cross-tenant data leakage,
- invalid proof artifacts,
- regulatory and contractual exposure.

**Required Controls**
- hard tenant_id validation,
- no fallback system tenant,
- lineage-level tenant consistency checks,
- negative tests across all pack/seal/verify flows.

### 8.2 Evidence Substitution
An attacker replaces or swaps evidence items after pack generation or before verification.

**Impact**
- fake proof,
- false blame,
- audit contamination,
- reputational damage.

**Required Controls**
- item-level lineage hash,
- canonical payload hashing,
- pack-level payload hash,
- verification against canonical persisted payload.

### 8.3 Hash Tampering
Stored hash or payload hash generation logic is manipulated.

**Impact**
- false validity,
- silent integrity compromise.

**Required Controls**
- deterministic server-side hashing,
- hash recomputation during verify,
- sealed hash input specification,
- audit event on mismatch.

### 8.4 Revocation Bypass
A revoked seal is still displayed or returned as valid by outdated logic or alternate path.

**Impact**
- false trust,
- legal and audit failure.

**Required Controls**
- central revocation lookup,
- verification response MUST include revocation status,
- UI MUST consume backend verification result,
- tests for revoked objects on every endpoint.

### 8.5 Verifier Spoofing
An attacker serves fake verification responses or a spoofed verification UI.

**Impact**
- third-party deception,
- forged trust.

**Required Controls**
- canonical public verify endpoint,
- TLS hardening,
- verifier version visible,
- signed/hashed rendered artifacts planned for future version,
- direct API verification path documented.

### 8.6 Chain Breakage
Chain linking is broken, reordered, or partially missing.

**Impact**
- damaged integrity story,
- false continuity claim.

**Required Controls**
- predecessor lookup,
- chain integrity state,
- broken chain detection,
- alerting on chain discontinuity.

### 8.7 Stale Schema Replay
A stale or incompatible object schema is replayed to bypass newer validation rules.

**Impact**
- downgraded integrity checks,
- outdated semantics,
- inconsistent verification.

**Required Controls**
- schema_version in all core objects,
- fail-safe on incompatible versions,
- compatibility policy enforcement,
- migration audit records.

### 8.8 Unauthorized Resealing
A user or service reseals modified content without proper authority.

**Impact**
- forged proof refresh,
- concealment of tampering.

**Required Controls**
- reseal restricted to privileged roles,
- explicit reseal reason,
- previous seal retention,
- reseal audit event,
- chain/replacement semantics.

### 8.9 Timeline Falsification
Event ordering, incident sequencing, or key timestamps are altered.

**Impact**
- narrative distortion,
- false causal chain,
- analyst deception.

**Required Controls**
- preserve observed_at and received_at,
- prohibit destructive rewriting,
- store timeline generation version,
- detect impossible chronology.

### 8.10 Privilege Misuse
Privileged insider manipulates packs, revokes seals improperly, or edits metadata.

**Impact**
- insider fraud,
- accountability collapse.

**Required Controls**
- separation of duties,
- immutable audit trail,
- admin activity logging,
- high-risk action alerts,
- review workflow for revocation and resealing.

### 8.11 Anchor Substitution
An actor with sufficient privilege supersedes an External Anchor and re-anchors a rewritten chain, so that a full rewrite becomes indistinguishable from an intact history.

**Impact**
- loss of the only reference point the producer cannot revise,
- false "independently anchored" outcome,
- collapse of P4 for the covered range.

**Required Controls**
- anchor supersession MUST produce a Revocation Record (POG-SPEC 8.10, 12.7) and be surfaced to the tenant,
- superseded anchors remain queryable with their original `external_reference`,
- the authority's record MUST remain independently retrievable; an anchor type whose record can be withdrawn by the producer is not an anchor,
- elevated authorization plus reason plus review for supersession,
- alert on any supersession.

### 8.12 Signing Key Substitution
An actor publishes a key under a `public_key_id` that verifiers already trust, or introduces a new key record that verifiers accept without noticing, and signs rewritten seals with it.

**Impact**
- forged seals that verify,
- repudiation of legitimate seals by declaring the genuine key compromised.

**Required Controls**
- key distribution through a documented channel independent of the producer UI (POG-SPEC 12.5, 15.7), with RECOMMENDED publication outside the producer's infrastructure,
- Signing Key Records are append-only; a `public_key_id` MUST NOT be reassigned,
- retirement and compromise declarations are access-controlled, audited, and carry a time that verifiers apply (POG-SPEC 8.13),
- alert on any key record change.

### 8.13 Timestamp Misrepresentation
A non-qualified timestamp is presented as qualified, or the absence of a timestamp token is hidden in rendering.

**Impact**
- legal value of the artifact overstated,
- Claims Policy violation.

**Required Controls**
- `qualified` flag set from the anchor's recorded regime, never from the authority name (POG-SPEC 12.6),
- rendering and API MUST show `timestamp_token=null` as absent,
- conformance test on the rendering of both cases.

### 8.14 Pending Anchor Presented as Anchored
Verification or rendering reports a chain position as anchored while the only anchors at or beyond it are pending or failed, or while the seal lies after the last confirmed anchor.

**Impact**
- false independence claim for the affected range.

**Required Controls**
- `anchor_status` computed from `anchored_chain_position`, `status` and chain recomputation only (POG-SPEC 8.12, 13.2),
- integrity alert when an anchor stays pending beyond the declared target latency,
- conformance tests POG-TST-027 to POG-TST-030.

## 9. STRIDE-Oriented Threat Mapping

### Spoofing
- fake verifier service
- fake admin identity
- impersonated service account
- forged signing key record

**Mitigations**
- strong service authentication
- audited privileged actions
- mTLS or signed service identity where feasible
- append-only key records, out-of-band key publication

### Tampering
- payload alteration
- hash changes
- seal metadata edits
- rendering substitution
- full-chain rewrite with re-anchoring

**Mitigations**
- canonical hash verification
- seal signatures over the canonical payload
- append-only or immutable history where feasible
- external anchor with revocation record on supersession
- rendering hash generation
- integrity alerts

### Repudiation
- actor denies approval
- admin denies revocation
- system cannot prove state transition
- producer denies having sealed an artifact, or claims a different sealing time

**Mitigations**
- decision records
- audit trail
- decision-to-action linkage
- revocation record
- signature and timestamp token on every seal, anchor for the sealing time

### Information Disclosure
- cross-tenant evidence leakage
- excessive verification payload
- internal metadata leakage to public verify

**Mitigations**
- payload minimization
- tenant scoping
- public/private schema separation

### Denial of Service
- verify endpoint abuse
- pack generation overload
- verification dependency exhaustion

**Mitigations**
- rate limiting
- caching safe metadata
- backpressure controls
- graceful degradation without false validity

### Elevation of Privilege
- unauthorized reseal
- unauthorized revoke
- cross-tenant object access
- bypass of approval gates

**Mitigations**
- RBAC/ABAC
- route-level authorization
- server-side enforcement
- security review of admin paths

## 10. Adversarial Scenarios

### Scenario A - Cross-Tenant Seal Poisoning
Attacker submits malformed event payload causing tenant mismatch but successful pack creation.

**Expected Control Outcome**
- pack creation blocked,
- tenant mismatch logged,
- no seal created,
- alert emitted.

### Scenario B - Evidence Swap After Sealing
Attacker modifies stored evidence item after pack sealed.

**Expected Control Outcome**
- verification returns invalid,
- payload hash mismatch recorded,
- alert emitted.

### Scenario C - Revoked Seal Publicly Appears Valid
Cached layer serves old valid status for revoked seal.

**Expected Control Outcome**
- cache invalidation required,
- verification endpoint authoritative,
- revoked status returned,
- stale cache incident created.

### Scenario D - Malformed Identifier Enumeration
Automated actor probes verify endpoint with malformed or guessed IDs.

**Expected Control Outcome**
- 400 for malformed,
- 404 for unknown,
- rate limits,
- no leakage of internal identifiers.

### Scenario E - Unauthorized Reseal by Insider
Admin reseals corrected pack without recording prior invalidity.

**Expected Control Outcome**
- blocked unless role + reason + approval,
- old seal retained,
- superseded relation stored.

### Scenario F - Chain Link Corruption
Chain predecessor hash missing or inconsistent after migration.

**Expected Control Outcome**
- chain_integrity=broken,
- object MAY remain inspectable but not chain-verified,
- migration incident raised.

### Scenario G - Full-Chain Rewrite Behind an Unchanged Anchor
Privileged actor rewrites every seal of a chain scope, re-signs them with the active key, and leaves the confirmed anchor untouched.

**Expected Control Outcome**
- verification recomputes the seal hash at `anchored_chain_position` and finds it differs from `anchored_digest`,
- chain_integrity=broken for the covered range,
- anchor_status never reported as anchored for that range,
- integrity alert emitted.

### Scenario H - Anchor Superseded Without Revocation Record
Privileged actor replaces the anchor record in place, with a new `anchored_digest` matching the rewritten chain.

**Expected Control Outcome**
- blocked: the anchor endpoint MUST refuse an in-place change,
- if it nevertheless occurs, verification detects the missing Revocation Record for the previously published `anchor_id` and returns an integrity failure,
- alert emitted, tenant notified.

### Scenario I - Key Rotation Presented as Compromise
Actor declares the genuine key compromised with a backdated `compromised_at` to invalidate legitimate seals.

**Expected Control Outcome**
- compromise declaration requires elevated authorization, reason and review,
- seals carrying a trusted timestamp token earlier than `compromised_at` keep `signature_status=valid`,
- audit event and alert emitted.

### Scenario J - Anchoring Authority Outage
The anchoring authority is unreachable for longer than the declared target latency.

**Expected Control Outcome**
- sealing continues,
- anchors stay `pending`, verification reports `anchor_status=pending`,
- integrity alert on overrun,
- no silent switch to another anchor type.

## 11. Controls

## 11.1 Preventive Controls
- strict schema validation
- tenant attribution enforcement
- server-side hashing
- authenticated service calls
- role-based access control
- state transition guards
- canonical serialization
- public/private schema split
- migration controls
- explicit revocation workflow
- server-side signing with non-exportable private keys
- append-only signing key records
- anchor supersession only through the revocation workflow

## 11.2 Detective Controls
- integrity mismatch alerts
- chain break alerts
- tenant mismatch alerts
- unauthorized access alerts
- repeated verify abuse detection
- unusual reseal/revoke pattern detection
- schema incompatibility alerts
- signature verification failures and unknown key references
- anchor supersession and anchor pending overrun alerts
- key record changes (publication, retirement, compromise)
- `anchored_digest` mismatch on recomputation

## 11.3 Corrective Controls
- revocation of invalid seals
- superseding corrected artifacts
- incident workflow for integrity events
- migration rollback where feasible
- cache purge mechanisms
- admin review and forensics
- key retirement and compromise declaration with recorded time
- re-anchoring with a Revocation Record for the superseded anchor

## 12. Logging and Monitoring Requirements

The platform MUST log:
- seal creation
- verification requests
- verification failures
- revocations
- reseals
- tenant attribution mismatches
- chain integrity failures
- unsupported schema verification attempts
- admin actions on PoG objects
- signature verification failures
- key record changes
- anchor submissions, confirmations, failures and supersessions
- detached bundle exports

The platform SHOULD monitor:
- verification latency
- invalid verification rate
- revoked verification lookups
- seal generation failures
- cross-tenant access denials
- chain break rate
- schema mismatch rate
- anchor confirmation latency against the declared target
- share of seals without timestamp token

## 13. Residual Risks

Residual risks remain for:
- compromise of privileged administrators,
- corruption before ingestion from trusted but compromised sources,
- sophisticated supply-chain compromise,
- incomplete lineage from legacy migrated data,
- dependence on SHA-256 and Ed25519 until further algorithms are adopted,
- public UI spoofing outside platform control,
- trust in the key distribution channel: a verifier who obtains a forged key record from a compromised channel cannot detect forged seals; the control is organizational (out-of-band publication) rather than technical,
- trust in the anchoring authority: independence holds only as long as the authority's record stays retrievable and unrevisable by the producer; a compromised authority removes independence for the covered range without removing integrity verification,
- seals produced under 1.0 remain unsigned and unanchored unless re-sealed, and re-sealing is itself a producer action.

Each residual risk MUST be documented, accepted, reduced, or scheduled.

## 14. Security Review Gates

### Gate 1 - Design Review
Before implementation:
- trust boundaries approved,
- abuse cases reviewed,
- controls assigned.

### Gate 2 - Pre-Release Review
Before shipping:
- threat model updated,
- key negative tests passing,
- access control verified,
- logs and alerts implemented.

### Gate 3 - Post-Release Review
After release:
- verify telemetry reviewed,
- incidents analyzed,
- cache/revocation consistency checked,
- drift from spec assessed.

## 15. Minimum Security Definition of Done

PoG security work is not done unless:
- tenant confusion tests pass,
- integrity mismatch tests pass,
- revocation consistency tests pass,
- privileged actions are audited,
- public verification is rate-limited and safe,
- unsupported schema handling fails safely,
- chain break detection exists,
- signature verification failures and unknown keys fail closed,
- anchor supersession without revocation record is detected,
- pending anchors are never reported as anchored,
- threat model has been reviewed by Security.
