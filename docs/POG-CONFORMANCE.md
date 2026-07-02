# PoG Framework Conformance Test Plan v1

## Document Control
- Document ID: POG-CONFORMANCE-TEST-PLAN-V1
- Version: 1.0.0
- Status: Draft, open for review
- Date: 2026-04-06
- Owner: Smart STB SARL
- Audience: QA, Security, Backend, Platform, Release Managers

## 1. Purpose

This document defines the conformance and adversarial test plan for PoG Framework v1. It verifies that PoG implementations behave correctly on valid paths, fail safely on invalid paths, resist common integrity threats, preserve tenant isolation, and produce consistent outputs across APIs and rendering layers.

## 2. Test Categories

- Positive tests
- Negative tests
- Adversarial tests
- Regression tests
- Compatibility tests
- Multi-tenant tests
- Rendering integrity tests
- Verification consistency tests

## 3. Conformance Entry Criteria

Testing begins only when:
- technical spec is frozen for the iteration,
- schemas are available,
- seal/verify/revoke APIs are implemented,
- test fixtures exist for events, incidents, evidence items, packs, and seals.

## 4. Positive Tests

### POG-TST-001 - Valid Seal Verification
- **Objective**: Verify that a correctly sealed object validates successfully.
- **Preconditions**: Existing evidence pack with valid seal.
- **Input**: Valid `seal_id`.
- **Expected Output**:
  - HTTP 200
  - `verification_result=valid`
  - `seal_status=valid`
  - `chain_integrity=verified|unknown` depending on chain mode
- **Failure Mode**: Returns invalid or unknown.
- **Severity**: Critical
- **Automation Feasibility**: High

### POG-TST-002 - Pack Metadata Retrieval
- **Objective**: Ensure pack metadata endpoint returns canonical state.
- **Preconditions**: Existing pack.
- **Input**: Valid `pack_id`.
- **Expected Output**:
  - correct pack_number,
  - correct status,
  - evidence count matches source fixture.
- **Failure Mode**: Inconsistent metadata.
- **Severity**: High
- **Automation Feasibility**: High

### POG-TST-003 - Decision Retrieval
- **Objective**: Verify governance decision lookup works.
- **Preconditions**: Existing decision linked to incident or action.
- **Input**: Valid `decision_id`.
- **Expected Output**: Canonical decision object returned.
- **Failure Mode**: Missing required fields or wrong state.
- **Severity**: Medium
- **Automation Feasibility**: High

### POG-TST-004 - Revocation Visibility
- **Objective**: Confirm revoked objects are still queryable.
- **Preconditions**: Existing revoked seal.
- **Input**: Revoked `seal_id`.
- **Expected Output**:
  - HTTP 200
  - `verification_result=revoked`
  - `revocation_status=revoked`
- **Failure Mode**: 404 or valid result for revoked object.
- **Severity**: Critical
- **Automation Feasibility**: High

## 5. Negative Tests

### POG-TST-005 - Unknown Seal
- **Objective**: Validate handling of non-existent identifiers.
- **Preconditions**: None.
- **Input**: Well-formed but unknown UUID.
- **Expected Output**:
  - HTTP 404
  - `seal_not_found`
- **Failure Mode**: 200 or internal leakage.
- **Severity**: High
- **Automation Feasibility**: High

### POG-TST-006 - Malformed Identifier
- **Objective**: Validate handling of malformed seal identifiers.
- **Preconditions**: None.
- **Input**: Invalid string, e.g. `abc`.
- **Expected Output**:
  - HTTP 400
  - `malformed_identifier`
- **Failure Mode**: 500 or ambiguous error.
- **Severity**: Medium
- **Automation Feasibility**: High

### POG-TST-007 - Unsupported Schema Version
- **Objective**: Ensure unsupported schema fails safely.
- **Preconditions**: Fixture with incompatible schema version.
- **Input**: Verification request for incompatible object.
- **Expected Output**:
  - `verification_result=unknown|error`
  - explicit schema-related failure reason
- **Failure Mode**: Silent valid status.
- **Severity**: Critical
- **Automation Feasibility**: Medium

### POG-TST-008 - Duplicate Sealing Attempt
- **Objective**: Block duplicate seal creation on already sealed object when not allowed.
- **Preconditions**: Existing sealed object.
- **Input**: POST seal again.
- **Expected Output**:
  - HTTP 409
  - `already_sealed`
- **Failure Mode**: Multiple competing active seals created silently.
- **Severity**: High
- **Automation Feasibility**: High

## 6. Adversarial Tests

### POG-TST-009 - Payload Hash Tampering
- **Objective**: Ensure modified payload invalidates verification.
- **Preconditions**: Existing sealed pack.
- **Input**: Tamper with canonical object payload after sealing.
- **Expected Output**:
  - `verification_result=invalid`
  - integrity failure recorded
- **Failure Mode**: Verification remains valid.
- **Severity**: Critical
- **Automation Feasibility**: Medium

### POG-TST-010 - Chain Break Detection
- **Objective**: Detect broken predecessor linkage.
- **Preconditions**: Chain-enabled seals.
- **Input**: Remove or corrupt predecessor chain hash.
- **Expected Output**:
  - `chain_integrity=broken`
  - failure reason populated
- **Failure Mode**: Still reports verified chain.
- **Severity**: High
- **Automation Feasibility**: Medium

### POG-TST-011 - Revocation Bypass Attempt
- **Objective**: Ensure revoked seals cannot verify as valid through alternate routes.
- **Preconditions**: Revoked seal + UI/API access.
- **Input**: Query API, UI page, cached route.
- **Expected Output**: All channels show revoked.
- **Failure Mode**: At least one channel shows valid.
- **Severity**: Critical
- **Automation Feasibility**: Medium

### POG-TST-012 - Unauthorized Resealing
- **Objective**: Ensure non-privileged users cannot reseal.
- **Preconditions**: User without elevated role.
- **Input**: Attempt reseal call.
- **Expected Output**:
  - HTTP 403
  - no reseal created
- **Failure Mode**: Reseal succeeds or partial state written.
- **Severity**: Critical
- **Automation Feasibility**: High

### POG-TST-013 - Timeline Falsification
- **Objective**: Detect invalid chronology or tampered timestamps.
- **Preconditions**: Pack with timeline fixture.
- **Input**: Reorder timestamps to impossible sequence.
- **Expected Output**:
  - quality/integrity failure,
  - timeline flagged inconsistent
- **Failure Mode**: Pack renders as valid and normal.
- **Severity**: High
- **Automation Feasibility**: Medium

### POG-TST-014 - Evidence Substitution Attack
- **Objective**: Ensure swapping one evidence item with another is detectable.
- **Preconditions**: Existing sealed pack with known item hashes.
- **Input**: Replace evidence item payload but preserve metadata where possible.
- **Expected Output**:
  - invalid verification,
  - lineage hash mismatch
- **Failure Mode**: Verification stays valid.
- **Severity**: Critical
- **Automation Feasibility**: Medium

## 7. Regression Tests

### POG-TST-015 - Tenant Isolation Regression
- **Objective**: Prevent recurrence of tenant attribution failures.
- **Preconditions**: Multi-tenant fixture dataset.
- **Input**: Attempt cross-tenant pack composition.
- **Expected Output**:
  - blocked creation,
  - tenant mismatch logged
- **Failure Mode**: Cross-tenant pack created.
- **Severity**: Critical
- **Automation Feasibility**: High

### POG-TST-016 - Verify Route Public Accessibility
- **Objective**: Ensure intended public verify route stays publicly reachable while secure.
- **Preconditions**: Deployed route with public allowlist.
- **Input**: Anonymous GET verify.
- **Expected Output**:
  - 200/404/400 as applicable
  - never 401 for intended public route
- **Failure Mode**: Auth regression or SPA fallback corruption.
- **Severity**: High
- **Automation Feasibility**: High

### POG-TST-017 - Rendering Status Consistency
- **Objective**: Ensure JSON, HTML, and PDF agree on object status.
- **Preconditions**: Existing valid and revoked artifacts.
- **Input**: Render all output formats.
- **Expected Output**: Same status semantics in all formats.
- **Failure Mode**: Contradictory status display.
- **Severity**: High
- **Automation Feasibility**: Medium

## 8. Compatibility Tests

### POG-TST-018 - Previous Supported Schema Verification
- **Objective**: Verify previous supported major version remains readable if policy allows.
- **Preconditions**: Fixture with previous major version.
- **Input**: Verification request.
- **Expected Output**: Valid result or explicit supported compatibility mode.
- **Failure Mode**: Unexpected incompatibility.
- **Severity**: Medium
- **Automation Feasibility**: Medium

### POG-TST-019 - Incompatible Schema Hard Fail
- **Objective**: Ensure incompatible major version fails safely.
- **Preconditions**: Fixture with unsupported version.
- **Input**: Verify request.
- **Expected Output**: Unknown/error, never valid.
- **Failure Mode**: False valid.
- **Severity**: Critical
- **Automation Feasibility**: Medium

## 9. Multi-Tenant Tests

### POG-TST-020 - Cross-Tenant Verify Leakage
- **Objective**: Ensure public verify does not expose forbidden tenant-private metadata.
- **Preconditions**: Sealed object from tenant A.
- **Input**: Public verify call by arbitrary client.
- **Expected Output**: Only approved public fields visible.
- **Failure Mode**: Sensitive tenant data leaked.
- **Severity**: Critical
- **Automation Feasibility**: High

### POG-TST-021 - Tenant-Specific Admin Action Enforcement
- **Objective**: Ensure admin tooling respects tenant scope rules.
- **Preconditions**: Privileged user with scoped access.
- **Input**: Revoke/reseal outside authorized scope.
- **Expected Output**: Forbidden.
- **Failure Mode**: Cross-tenant privileged mutation.
- **Severity**: Critical
- **Automation Feasibility**: Medium

## 10. Rendering Integrity Tests

### POG-TST-022 - PDF Binary Hash Persistence
- **Objective**: Ensure rendered PDF hash is stored and retrievable.
- **Preconditions**: Generated PDF artifact.
- **Input**: Request rendered artifact metadata.
- **Expected Output**: Binary hash matches stored artifact.
- **Failure Mode**: Missing or inconsistent render hash.
- **Severity**: Medium
- **Automation Feasibility**: Medium

### POG-TST-023 - Visible Integrity Metadata in PDF
- **Objective**: Ensure PDF displays visible seal, hash, and verification information.
- **Preconditions**: Premium PDF generated.
- **Input**: Open PDF text layer or visual assertion.
- **Expected Output**: Required metadata visible.
- **Failure Mode**: Missing integrity cues.
- **Severity**: Medium
- **Automation Feasibility**: Low/Medium

## 11. Verification Consistency Tests

### POG-TST-024 - Repeat Verification Determinism
- **Objective**: Ensure repeated verification returns stable result absent state change.
- **Preconditions**: Existing valid seal.
- **Input**: Multiple verify calls over time.
- **Expected Output**: Same result and semantics.
- **Failure Mode**: Flapping status.
- **Severity**: High
- **Automation Feasibility**: High

### POG-TST-025 - Revocation Propagation Consistency
- **Objective**: Ensure revocation is visible immediately across API and UI.
- **Preconditions**: Valid seal then revoke action.
- **Input**: Verify before and after revocation.
- **Expected Output**:
  - before: valid
  - after: revoked
- **Failure Mode**: delayed or inconsistent propagation.
- **Severity**: High
- **Automation Feasibility**: Medium

### POG-TST-026 - Verification Error Path Safety
- **Objective**: Ensure dependency failures do not produce false valid responses.
- **Preconditions**: Simulated verifier dependency outage.
- **Input**: Verify request.
- **Expected Output**:
  - error/unavailable
  - never valid by default
- **Failure Mode**: optimistic false-valid response.
- **Severity**: Critical
- **Automation Feasibility**: Medium

## 12. Release Gating Rules

Release MUST be blocked if any of the following fail:
- POG-TST-001
- POG-TST-004
- POG-TST-009
- POG-TST-011
- POG-TST-012
- POG-TST-015
- POG-TST-019
- POG-TST-020
- POG-TST-026

Release SHOULD be blocked if repeated instability occurs on:
- POG-TST-017
- POG-TST-024
- POG-TST-025

## 13. Automation Strategy

### Mandatory Automation
- schema validation tests
- verify endpoint tests
- revocation tests
- malformed ID tests
- tenant isolation tests
- unsupported schema tests
- state transition guard tests

### Recommended Automation
- chain break tests
- rendering consistency tests
- latency assertions
- binary hash persistence

### Manual / Assisted Validation
- premium PDF readability
- executive rendering review
- forensic operator usability
- public verify UX sanity

## 14. Exit Criteria

Conformance is acceptable for release only when:
- all critical tests pass,
- no unresolved P0 integrity defect remains,
- tenant isolation regressions are green,
- public verify route is stable,
- revoked objects cannot appear valid,
- unsupported schemas fail safely,
- rendering consistency is acceptable,
- security sign-off is complete.
