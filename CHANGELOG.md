# Changelog

All notable changes to the PoG Framework documents are recorded here. Substantive external reviews are credited unless reviewers prefer otherwise.

## [Unreleased]

## [1.1.1-draft] - 2026-09-25

### Fixed
- POG-SPEC 1.1.1: section 16 contradicted section 15.1 on a seal whose signature fails verification. The verification completes with HTTP 200 and `verification_result=invalid`; the codes POG-422-004 and POG-422-005 are withdrawn (13.2, 16).
- POG-CTP 1.1.1: POG-TST-031 and POG-TST-032 expect HTTP 200 with `signature_status=invalid` or `key_unknown`.

### Changed
- POG-CP 1.1.0: "reference implementation" becomes a claim to be earned, conditional on a dated conformance report produced by executing the conformance plan. PREVORN is described as the originating implementation. README aligned.

## [1.1.0-draft] - 2026-09-25

### Added
- POG-SPEC 1.1.0: mandatory Ed25519 signature and optional RFC 3161 timestamp token on the Seal (8.8); External Anchor entity (8.12); Signing Key Record entity (8.13); Boundary K, seal store to external anchoring authority (6.1); signing, timestamping and anchoring rules (12.5 to 12.7); two verification outcomes, integrity verified and independently anchored, reported separately (13.2, 13.4); detached verification bundle (13.5); key, anchor and bundle endpoints (15.7 to 15.9); error codes for signature and key failures (16); transition rules from 1.0 (17.1); Annex A, informative profile for an AI-assisted software change record.
- POG-TM 1.1.0: abuse cases 8.11 to 8.14 (anchor substitution, signing key substitution, timestamp misrepresentation, pending anchor presented as anchored); scenarios G to J; corresponding assets, actors, trust assumptions, controls, logging and residual risks.
- POG-CTP 1.1.0: tests POG-TST-027 to POG-TST-036; five of them added to the release-blocking set.

### Changed
- POG-SPEC 12.3: chaining moves from SHOULD to MUST for evidence packs; `previous_chain_hash` is required for every seal after the first of a chain scope.
- POG-SPEC 3: cryptographic signing is no longer listed as a non-goal; mandating a particular anchoring technology and defining key distribution infrastructure are.
- POG-SPEC 22: digital signatures and the detached verification bundle leave the deferred list.

### Review
- Resolves review issue #2 (Seal narrower than the implemented Seal, external anchoring unspecified). Answers to its open questions are recorded in the pull request.

## [1.0.0-draft] - 2026-07-02

### Added
- POG-SPEC v1.0.0: Technical Specification (canonical data model, state machines, integrity invariants, sealing and verification model, revocation, API contracts, error model, versioning, security and conformance requirements).
- POG-TM v1.0.0: Threat Model (assets, actors, trust assumptions, abuse cases, STRIDE mapping, adversarial scenarios, controls, logging requirements, residual risks, review gates).
- POG-CTP v1.0.0: Conformance Test Plan (26 tests across positive, negative, adversarial, regression, compatibility, multi-tenant, rendering and consistency categories; release gating rules).
- POG-CP v1.0.0: Claims Policy (staged claims, forbidden claims, mandatory transparency statements).
- Repository governance: README with public maturity table, CONTRIBUTING, SECURITY.

### Status
- All documents are drafts, open for review.
