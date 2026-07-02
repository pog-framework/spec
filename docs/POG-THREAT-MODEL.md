# PoG Framework Threat Model v1

## Document Control
- Document ID: POG-THREAT-MODEL-V1
- Version: 1.0.0
- Status: Draft, open for review
- Date: 2026-04-06
- Owner: Smart STB SARL
- Audience: Security, Architecture, Backend, Platform, QA, Leadership

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

### 4.2 Adversarial Actors
- External attacker
- Tenant user with malicious intent
- Privileged insider
- Compromised service account
- Rogue admin
- API abuser
- Supply-chain compromised component
- Accidental but destructive operator

## 5. Trust Assumptions

1. Upstream event sources MAY be malicious or malformed.
2. Internal services MAY contain bugs and MUST NOT be blindly trusted.
3. Privileged access is not equivalent to integrity.
4. Public verification consumers MUST be considered untrusted.
5. The database is authoritative for persistence but not beyond integrity checks.
6. UI rendering is not a trusted source of validity.
7. Schema migration is a security-relevant activity.

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

## 9. STRIDE-Oriented Threat Mapping

### Spoofing
- fake verifier service
- fake admin identity
- impersonated service account

**Mitigations**
- strong service authentication
- audited privileged actions
- mTLS or signed service identity where feasible

### Tampering
- payload alteration
- hash changes
- seal metadata edits
- rendering substitution

**Mitigations**
- canonical hash verification
- append-only or immutable history where feasible
- rendering hash generation
- integrity alerts

### Repudiation
- actor denies approval
- admin denies revocation
- system cannot prove state transition

**Mitigations**
- decision records
- audit trail
- decision-to-action linkage
- revocation record

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

## 11.2 Detective Controls
- integrity mismatch alerts
- chain break alerts
- tenant mismatch alerts
- unauthorized access alerts
- repeated verify abuse detection
- unusual reseal/revoke pattern detection
- schema incompatibility alerts

## 11.3 Corrective Controls
- revocation of invalid seals
- superseding corrected artifacts
- incident workflow for integrity events
- migration rollback where feasible
- cache purge mechanisms
- admin review and forensics

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

The platform SHOULD monitor:
- verification latency
- invalid verification rate
- revoked verification lookups
- seal generation failures
- cross-tenant access denials
- chain break rate
- schema mismatch rate

## 13. Residual Risks

Residual risks remain for:
- compromise of privileged administrators,
- corruption before ingestion from trusted but compromised sources,
- sophisticated supply-chain compromise,
- incomplete lineage from legacy migrated data,
- dependence on SHA-256 until stronger or signed model adopted,
- public UI spoofing outside platform control.

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
- threat model has been reviewed by Security.
