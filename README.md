# PoG Framework - Proof of Governance

**An open framework for turning security and AI-driven decisions into traceable, sealed, verifiable evidence.**

Security and AI systems detect, classify and summarize. They do not prove enough. When a regulator, auditor or court asks *"why did the system decide this, based on what data, validated by whom"*, logs and scores are not an answer. PoG forces every governed decision to leave a sealed, re-verifiable artifact with a recorded human gate.

Website: https://pog-protocol.org · Contact: spec@pog-protocol.org

---

## The four properties

A system is PoG-conformant only if every governed decision produces an artifact satisfying all four. Three out of four is non-conformant.

| # | Property | Meaning |
|---|----------|---------|
| P1 | **Traceability** | Unbroken chain from raw event to final decision. No implicit links. |
| P2 | **Integrity** | Every artifact sealed at creation. Tampering, substitution, chain corruption and unauthorized resealing are detectable failure states. |
| P3 | **Governance** | AI-assisted judgments pass through an explicit, recorded human gate before becoming authoritative. |
| P4 | **Verifiability** | Any artifact can be re-verified after the fact, including outside the system that produced it. |

## Normative documents

| Document | ID | Version | Status |
|----------|----|---------|--------|
| [Technical Specification](docs/POG-SPEC.md) | POG-SPEC | 1.0.0 | Draft, open for review |
| [Threat Model](docs/POG-THREAT-MODEL.md) | POG-TM | 1.0.0 | Draft, open for review |
| [Conformance Test Plan](docs/POG-CONFORMANCE.md) | POG-CTP | 1.0.0 | Draft, open for review |
| [Claims Policy](docs/POG-CLAIMS-POLICY.md) | POG-CP | 1.0.0 | Draft, open for review |

## Status, stated plainly

PoG is a framework initiated and maintained by **Smart STB Sàrl** (Geneva, Switzerland). The reference implementation is [PREVORN](https://prevorn.ch), operated in production on Swiss infrastructure. This is a single-vendor framework opening itself to scrutiny, not an industry consortium, and we will not pretend otherwise.

| Stage | Gate | Status |
|-------|------|--------|
| Technical credibility | Sealing, verification, event-to-evidence chain, tenant-safe attribution | In place (reference implementation) |
| Framework formalization | Spec, threat model, conformance plan drafted, versioned, published | In progress (drafts v1.0) |
| External review | Substantive adversarial review by parties independent of Smart STB | Open - reviewers wanted |
| Independent implementation | At least one conformant implementation not written by the authors | Not started |
| Protocol-grade language | Stable spec, provable conformance, multiple coherent implementations | **Not earned, by design** |

Yes, the domain is called pog-protocol.org. The name states the destination, not the current status. Until the last gate is passed, PoG is a framework, and our own [Claims Policy](docs/POG-CLAIMS-POLICY.md) forbids us from calling it anything more.

## How to review

Adversarial readings are the most useful kind and explicitly welcome.

1. Open an issue using the **Specification review** template, or email spec@pog-protocol.org.
2. Aim at the [Threat Model](docs/POG-THREAT-MODEL.md): it tells you where the framework claims to resist.
3. Substantive reviews are credited in the [CHANGELOG](CHANGELOG.md) unless you prefer otherwise.

## What this repository is not

- Not a standard. Not a certification. Not a compliance guarantee.
- Not the PREVORN product code, which is proprietary and out of scope. This repository contains the framework documents only.

## License

Framework documents are published under [CC BY 4.0](LICENSE): read, cite and implement freely, with attribution.

---

PoG Framework · initiated and maintained by Smart STB Sàrl, 17 Rue Dancet, 1204 Geneva, Switzerland.
