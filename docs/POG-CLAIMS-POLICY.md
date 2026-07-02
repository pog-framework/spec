# PoG Claims Policy v1

## Document Control
- Document ID: POG-CLAIMS-POLICY-V1
- Version: 1.0.0
- Status: Draft for review
- Owner: Smart STB SARL
- Audience: Anyone communicating about PoG, internally or externally

## 1. Purpose

This document governs what may and may not be said about the PoG Framework externally, at each maturity stage. It exists so that PoG's own marketing can be held to a verifiable standard, by anyone, including our critics.

Ambition is not the enemy of credibility. Unearned claims are. This policy stages the language so that every public claim is either currently true or explicitly labeled as a target.

## 2. Claims acceptable now

The following claims are authorized today:

- PoG is a governance and evidence framework initiated by Smart STB (Geneva).
- PoG is designed to produce traceable, sealed, and auditable artifacts.
- PoG is formalized through a technical specification, a threat model, and a conformance test plan, published as versioned drafts open for review.
- PREVORN uses PoG as its trust and evidence layer and serves as the reference implementation.
- PoG conformance is defined by executable tests, not declarations.

## 3. Claims acceptable only when earned

The following claims require the corresponding gate to be passed, as tracked in the public maturity table:

| Claim | Required gate |
|-------|---------------|
| "PoG has a stable, protocol-grade specification" | Spec frozen across a major version with no breaking drift |
| "PoG provides formal verification workflows" | Verification procedure independently exercised end to end |
| "PoG has conformance-tested implementations" | Conformance plan executed and results published |
| "PoG has independent implementations" | At least one conformant implementation not authored by Smart STB |
| "PoG has survived adversarial review" | Substantive external reviews received, answered publicly, and incorporated |

## 4. Claims that are forbidden until fully earned

The following claims MUST NOT be made in any external material while unearned:

- PoG is a standard, an industry standard, or a global standard.
- PoG is a recognized or universal protocol.
- PoG guarantees compliance with any law or regulation.
- PoG is institutionally established, endorsed, or certified by any third party.
- Any implication that a consortium, working group, or multi-vendor body governs PoG.

## 5. Mandatory transparency statements

Any substantial public presentation of PoG (website, deck, whitepaper) MUST state:

1. that PoG is initiated and maintained by Smart STB Sàrl;
2. that PREVORN is the reference implementation;
3. the current maturity stage, consistent with the public maturity table.

## 6. Enforcement

- Public materials inconsistent with this policy are treated as defects and corrected with priority.
- Third parties who identify a claim inconsistent with this policy are invited to report it to spec@pog-protocol.org. Confirmed reports are acknowledged in the changelog.

## 7. Rationale

A framework whose core promise is "proof over declaration" cannot allow its own communications to be declarative. This policy applies PoG's discipline to PoG itself.
