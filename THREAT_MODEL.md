# QEC — public threat model (governed agent path)

Scope: the control path documented in this repository and reproduced by the public shim
(`qec-governed-agent-demo`). Assets: the authority an agent holds, the money or side effects it can trigger,
the evidence of what it did. Out of scope here: the licensed runtime's internals, key custody, infrastructure.
Method: STRIDE per boundary. "Public proof" names the test or command that demonstrates the mitigation today.

## Actors and trust levels

| Actor | Trust | Typical capability |
|---|---|---|
| Operator (approves policy, holds signing key) | trusted | signs configuration, issues root authority |
| Root agent | governed | calls tools inside its authority |
| Sub-agent | governed, derived | authority ⊆ parent, never more |
| Tool / external system | untrusted input, trusted execution boundary | returns data that may carry injected instructions |
| Content reaching the model (invoices, e-mails, web pages) | untrusted | prompt injection vector |
| Auditor / buyer | verifies without trusting us | runs the public verifier offline |

## Threats, mitigations, proofs

| ID | Boundary | Threat (STRIDE) | Mitigation in the control path | Public proof |
|---|---|---|---|---|
| T1 | Policy | Prompt injection makes the agent pay an attacker IBAN (Tampering of intent) | Policy evaluated **before** execution; vendor allow-list; per-call ceiling; nothing executes before it is journaled as allowed | `demo.py`: attacker IBAN paid 48 000 ungoverned → 0 governed; `test_demo` |
| T2 | Policy | Exfiltration by e-mail to an external address (Information disclosure) | recipient domain rule in policy | `demo.py` refusal `external_recipient` |
| T3 | Budget | Slow drain: many small calls under the ceiling (Denial of funds / Unbounded consumption) | cumulative Decimal budget per session; refusal `budget_exhausted` | `demo.py` INV-5 |
| T4 | Budget | Float rounding used to slip under a threshold (Tampering) | money accepted as `str`/`int` only; `float` refused | `test_demo` (`float_forbidden`) |
| T5 | Delegation | Sub-agent asks for more than its parent holds (Elevation of privilege) | `child ⊆ parent` on tools, per-call and budget, checked at delegation time; refusal journaled | `demo.py` `authority_escalation`; delegation tests |
| T6 | Delegation | Sub-agent uses a tool it was never given (Elevation) | tool set is part of authority; `tool_not_delegated` | `demo.py` reconciler → `schedule_payment` refused |
| T7 | Configuration | Policy file edited after approval (Tampering) | configuration hash covered by an approval; mismatch refuses fail-closed (`config_drift`) | `test_demo` drift case |
| T8 | Evidence | A record is rewritten after the fact (Repudiation) | SHA-256 chain `prev_hash → hash` on canonical JSON; any edit breaks every later hash | `ledger_verify.py` → INVALID, line named |
| T9 | Evidence | A refusal is silently dropped (Repudiation) | refusals are journaled **before** the exception is raised; removing any record breaks the `prev_hash` of the next one | `ledger_verify.py` on a ledger with one line deleted → INVALID (`prev_hash mismatch`) |
| T10 | Evidence | The evidence file itself is edited (Tampering) | evidence hash over canonical content; `--check` | `tools/evidence_pack.py --check` → INVALID on a changed value |
| T11 | Supply chain | A dependency is compromised (Tampering) | zero third-party dependency; standard library only; SBOM published | `sbom/cyclonedx.json`; CI installs nothing |
| T12 | Availability | Governance latency makes the agent unusable (DoS by design) | decision cost measured, not asserted | `bench/bench_overhead.py` (median 46 µs allowed, 26 µs refused, on the shim) |

## Residual risks, stated

- **Identity and time.** A hash chain proves order and integrity of what was written; it does not prove *who* wrote it
  nor *when* in trusted time. The licensed runtime adds Ed25519 signatures; trusted timestamping is an integration
  (external TSA), not a built-in claim.
- **Semantic gaps in policy.** A rule that is not written is not enforced. The demo policy is deliberately small.
  Buyers must write their own allow-lists and ceilings; the runtime refuses what it does not know, it does not guess.
- **Executor honesty.** Once a call is allowed, the tool executes outside the governor. Side effects of a compromised
  tool are journaled as the tool reported them.
- **Host compromise.** An attacker with write access to the process can stop the journal. Ship the ledger out of the
  host continuously (append-only sink) to keep T8 meaningful.

## Not claimed

No conformance claim to any external standard, no third-party attestation, no cryptographic proof of identity in the
public shim. Mapping to frameworks is a self-assessment: see `CONTROL_MAPPING.md`.
