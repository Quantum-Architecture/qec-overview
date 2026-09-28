# QEC — governance overhead, measured (public shim)

Question buyers ask: *what does it cost to check policy, budget and delegation, and to journal every decision?*
Answer: measured, not asserted, with `qec-governed-agent-demo/bench/bench_overhead.py`. Reproduce it yourself.

## Reference run — 27 September 2026

Environment: Linux x86_64, Python 3.11.15, public demonstration shim (pure Python, JSON ledger written to disk on every
decision, no batching). 5 000 iterations of three operations. Raw file: `docs/bench_result_2026-09-27.json`.

| Operation | Median | p95 |
|---|---|---|
| Allowed decision: policy + ceiling + budget + journal "allowed" + execute + journal "executed" | 45.6 µs | 108.3 µs |
| Refused decision: policy refusal, journaled | 25.7 µs | 62.4 µs |
| Delegation check (child ⊆ parent), refusal journaled | 22.3 µs | 54.8 µs |

Ledger produced by the run: 20 001 lines, 7.38 MB. Verification by the public verifier, in a separate process as a
buyer would run it: **0.317 s**, verdict `VALID`.

## How to read these numbers

- An LLM call is measured in hundreds of milliseconds; a governed decision in tens of microseconds. On the shim the
  governance overhead per tool call is three to four orders of magnitude below the model latency it protects.
- Two disk writes per allowed decision are included (the "allowed" record before execution and the "executed" record
  after). That is the design: nothing executes before it is journaled.
- These are shim numbers on one machine. The licensed runtime adds signature verification (Ed25519) on approvals and
  delegations; its cost is reported in the evaluation package, on your hardware, by the same method.

## Reproduce

```bash
git clone https://github.com/Quantum-Architecture/qec-governed-agent-demo
cd qec-governed-agent-demo
python bench/bench_overhead.py --n 5000 --json my_run.json
```
The script exits 0 only if the ledger it produced verifies `VALID`.
