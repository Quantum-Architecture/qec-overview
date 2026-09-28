# QEC — control mapping (self-assessment, not conformance)

This table maps the public control path to the frameworks buyers ask about first. It is a **self-assessment written by
the vendor**: it says where a QEC control is *relevant* to a framework item and how to reproduce it. It is **not** a
conformance, certification or attestation claim, and it has not been reviewed by an auditor or counsel. Framework
editions cited: OWASP Top 10 for LLM Applications (2025 list), NIST AI RMF 1.0 (core functions), ISO/IEC 42001:2023
(clause level only), EU AI Act (articles cited apply to high-risk systems; classification is the deployer's analysis).

| QEC control (public path) | OWASP LLM Top 10 (2025) | NIST AI RMF 1.0 | ISO/IEC 42001:2023 | EU AI Act | Public proof |
|---|---|---|---|---|---|
| Policy evaluated before any tool execution (allow-list, ceilings, recipients) | LLM01 Prompt Injection · LLM06 Excessive Agency | Manage (risk treatment at run time) | Clause 8 Operation (operational planning and control) | Art. 14 human oversight — technical measures enabling oversight | `qec-governed-agent-demo`: refusals `amount_over_ceiling`, `vendor_not_approved`, `external_recipient` |
| Exact Decimal budget, floats refused, cumulative ceiling | LLM10 Unbounded Consumption | Measure (quantitative limits) | Clause 8 | Art. 9 risk management (if high-risk) | `test_demo`: `float_forbidden`, `budget_exhausted` |
| Non-escalating delegation (child ⊆ parent), tool set bound to authority | LLM06 Excessive Agency | Govern (roles and authority) · Manage | Clause 5 Leadership / roles; Clause 8 | Art. 14 | `authority_escalation`, `tool_not_delegated` |
| Configuration hash covered by an approval; drift refused fail-closed | LLM03 Supply Chain (config integrity) | Govern · Manage | Clause 8; Clause 7.5 documented information control | Art. 9 | `config_drift` test |
| SHA-256 chained ledger of every decision and refusal, verifiable offline by a third party | LLM05 Improper Output Handling (traceability of what was executed) | Measure (evidence) · Manage (incident reconstruction) | Clause 9 Performance evaluation (monitoring, measurement); Clause 7.5 | Art. 12 record-keeping (automatic logging) · Art. 26 deployers keep logs | `ledger-verify` → VALID / INVALID with line number |
| Evidence pack with content hash; SBOM with zero third-party dependency | LLM03 Supply Chain | Map (inventory) · Govern | Clause 7.5; Clause 8 | Art. 11 technical documentation (as input) | `tools/evidence_pack.py --check`; `sbom/cyclonedx.json` |
| Measured governance overhead | — | Measure | Clause 9 | — | `bench/bench_overhead.py` |

## What is deliberately not mapped

- LLM02 Sensitive Information Disclosure, LLM07 System Prompt Leakage, LLM08 Vector and Embedding Weaknesses,
  LLM09 Misinformation: these concern model content and retrieval, which the governance kernel does not inspect.
  QEC bounds *what an agent may do*, not *what a model says*.
- LLM04 Data and Model Poisoning: out of scope (training pipeline).
- Identity of the writer and trusted time: not provided by the public chain (see `THREAT_MODEL.md`, residual risks).

## How to use this table in due diligence

1. Run the public proofs in the last column (30 minutes, `EVALUATION_GUIDE.md`).
2. Ask for the evaluation licence of the runtime: the same controls with Ed25519-signed approvals and delegations,
   and the full automated test suite executed on your side.
3. Have your own counsel or auditor decide classification and applicability; this document gives them the map,
   not the verdict.
