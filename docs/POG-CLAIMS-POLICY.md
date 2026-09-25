# PoG Claims Policy v1

## Document Control
- Document ID: POG-CLAIMS-POLICY-V1
- Version: 1.1.0
- Status: Draft for review
- Date: 2026-09-25
- Supersedes: 1.0.0
- Owner: Smart STB SARL
- Audience: Anyone communicating about PoG, internally or externally

### Changes in 1.1.0
"Reference implementation" moves from the claims acceptable now to the claims that must be earned. It requires a dated conformance report produced by executing the conformance plan. Section 2 now says "originating implementation", section 5 asks for the conformance status to be stated, and section 4 forbids the reference-implementation claim in any document of this repository without such a report.

## 1. Purpose

This document governs what may and may not be said about the PoG Framework externally, at each maturity stage. It exists so that PoG's own marketing can be held to a verifiable standard, by anyone, including our critics.

Ambition is not the enemy of credibility. Unearned claims are. This policy stages the language so that every public claim is either currently true or explicitly labeled as a target.

## 2. Claims acceptable now

The following claims are authorized today:

- PoG is a governance and evidence framework initiated by Smart STB (Geneva).
- PoG is designed to produce traceable, sealed, and auditable artifacts.
- PoG is formalized through a technical specification, a threat model, and a conformance test plan, published as versioned drafts open for review.
- PREVORN uses PoG as its trust and evidence layer and is the originating implementation, the system the framework was extracted from.
- PoG conformance is defined by executable tests, not declarations.

## 3. Claims acceptable only when earned

The following claims require the corresponding gate to be passed, as tracked in the public maturity table:

| Claim | Required gate |
|-------|---------------|
| "PoG has a stable, protocol-grade specification" | Spec frozen across a major version with no breaking drift |
| "PoG provides formal verification workflows" | Verification procedure independently exercised end to end |
| "PoG has conformance-tested implementations" | Conformance plan executed and results published |
| "PREVORN is the reference implementation of PoG" or "PREVORN is conformant to POG-SPEC x.y" | Dated conformance report for that version, produced by executing the conformance plan, published in this repository |
| "PoG has independent implementations" | At least one conformant implementation not authored by Smart STB |
| "PoG has survived adversarial review" | Substantive external reviews received, answered publicly, and incorporated |

## 4. Claims that are forbidden until fully earned

The following claims MUST NOT be made in any external material while unearned:

- PoG is a standard, an industry standard, or a global standard.
- PoG is a recognized or universal protocol.
- PoG guarantees compliance with any law or regulation.
- PoG is institutionally established, endorsed, or certified by any third party.
- Any implication that a consortium, working group, or multi-vendor body governs PoG.
- Calling any implementation, PREVORN included, a reference implementation or a conformant implementation, in any document of this repository or any public material, without a dated conformance report for the version claimed.

## 5. Mandatory transparency statements

Any substantial public presentation of PoG (website, deck, whitepaper) MUST state:

1. that PoG is initiated and maintained by Smart STB Sàrl;
2. that PREVORN is the originating implementation, and whether a dated conformance report exists for the specification version in force;
3. the current maturity stage, consistent with the public maturity table.

## 6. Enforcement

- Public materials inconsistent with this policy are treated as defects and corrected with priority.
- Third parties who identify a claim inconsistent with this policy are invited to report it to spec@pog-protocol.org. Confirmed reports are acknowledged in the changelog.

## 7. Rationale

A framework whose core promise is "proof over declaration" cannot allow its own communications to be declarative. This policy applies PoG's discipline to PoG itself.
