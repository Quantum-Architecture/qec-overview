# QEC — Public Architecture Overview

**Governed execution before action. Evidence after action.**

This repository documents the public architecture of QEC without publishing the licensed runtime.

## Control path

```text
Intent
  -> Policy Decision
  -> Authority / Delegation Check
  -> Budget Check
  -> Tool Boundary
  -> Execution or Denial
  -> Evidence Record
```

## Public trust boundaries

### Policy boundary
The agent does not define its own authority at execution time.

### Delegation boundary
A child agent cannot receive more authority than its parent holds:

```text
child_authority ⊆ parent_authority
```

This applies to tool scope, per-call limits and budget constraints.

### Tool boundary
Side-effecting operations cross a governed integration boundary where policy, budget and authority can be checked before execution.

### Evidence boundary
Decisions are journaled. Public examples use a hash-chained JSONL ledger that can be checked with `ledger-verify`.

## Public evidence

| Surface | Evidence |
|---|---|
| Pre-execution policy checks | qec-governed-agent-demo |
| Budget enforcement | qec-governed-agent-demo |
| Non-escalating delegation | public delegation tests |
| Explicit denials | demo ledger |
| Ledger tampering detection | ledger-verify |

## Integration surfaces

- agent-to-agent delegation;
- MCP-style tool boundaries;
- enterprise APIs;
- audit export;
- observability / evidence pipelines.

These are integration surfaces, not claims of external-standard conformance.

## Reproduce the public evidence

```bash
git clone https://github.com/Quantum-Architecture/qec-governed-agent-demo
cd qec-governed-agent-demo
python demo.py
```

Verifier:
https://github.com/Quantum-Architecture/ledger-verify

## Limits

- no SOC 2 claim;
- no ISO 27001 claim;
- no OWASP Agent Control Standard conformance claim;
- public shim ≠ licensed runtime;
- hash chaining alone does not prove identity or trusted time.

## Disclosure boundary

No Local Core runtime, proprietary thresholds, private prompts, secret parameters or patent-enabling implementation detail belongs in this public repository.

## Due diligence, in one sitting

| Question a buyer asks | Document |
|---|---|
| How does one decision flow, and where is it journaled? | `ARCHITECTURE.md` (figure `docs/qec_architecture.svg`) |
| What attacks does it stop, and what remains? | `THREAT_MODEL.md` |
| Where does it sit against OWASP LLM Top 10, NIST AI RMF, ISO/IEC 42001, EU AI Act? | `CONTROL_MAPPING.md` — self-assessment, not conformance |
| What does governance cost? | `BENCHMARK.md` — measured on the public shim, reproducible |
| Can my engineer check all this in 30 minutes? | `EVALUATION_GUIDE.md` |
| Can I archive proof that the checks passed? | `qec-governed-agent-demo/tools/evidence_pack.py` (content-hashed evidence file, re-checkable) and `sbom/cyclonedx.json` |
