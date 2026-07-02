# Contributing to the PoG Framework

Thank you for taking the time to challenge this framework. Adversarial readings are the most useful contributions and are explicitly welcome.

## Ways to contribute

### 1. Specification review
Read the [Technical Specification](docs/POG-SPEC.md) and open an issue for anything that is ambiguous, contradictory, unimplementable, or insufficiently specified. The bar is: could two independent teams implement this section and end up with incompatible behavior? If yes, that is a defect.

### 2. Threat model attacks
Read the [Threat Model](docs/POG-THREAT-MODEL.md) and try to break it on paper. Missing abuse cases, insufficient controls, unrealistic trust assumptions: open an issue with the attack narrative. You do not need a proof of concept; a coherent argument is enough to open the discussion.

### 3. Conformance plan gaps
Read the [Conformance Test Plan](docs/POG-CONFORMANCE.md) and identify behaviors that the specification requires but no test verifies. Untested requirements are unenforced requirements.

### 4. Editorial fixes
Typos, broken links, unclear phrasing: pull requests welcome directly.

## Process

- Open an issue using the appropriate template. One concern per issue.
- Substantive issues receive a public answer. Accepted changes are recorded in the [CHANGELOG](CHANGELOG.md) with credit to the reporter, unless you prefer otherwise.
- Breaking changes to sealed-artifact semantics require a major version bump and a migration note, per the versioning policy in the specification.

## What we will not merge

- Changes that weaken an integrity invariant for convenience.
- Marketing language. This repository is normative, not promotional.
- Claims inconsistent with the [Claims Policy](docs/POG-CLAIMS-POLICY.md).

## Decision authority

Until an independent governance structure is justified by real multi-party adoption, final editorial decisions rest with the maintainers at Smart STB Sàrl. Every rejection of a substantive issue will state its reasons publicly. We consider this arrangement honest for the current maturity stage; pretending otherwise would violate our own claims policy.
