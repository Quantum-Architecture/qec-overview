# QEC — 30-minute evaluation guide (for a buyer's engineer)

Goal: verify, on your machine, without trusting us, that the public control path does what the overview says.
Requirements: Python 3.10 or later, `git`. Nothing is installed; no network access is needed after the clone.
Expected outputs below were produced on 27 September 2026 (Python 3.11, Linux x86_64); timings will differ, verdicts must not.

## 1. Clone and run the adversarial tests (3 min)

```bash
git clone https://github.com/Quantum-Architecture/qec-governed-agent-demo
cd qec-governed-agent-demo
python -m unittest -q test_demo
```
Expected: `Ran 8 tests` … `OK`. The eight tests cover: exploitation of the ungoverned agent, zero loss when governed,
ledger verification and tamper detection, "every execution preceded by a journaled allow", float refusal, exact Decimal
budget, fail-closed configuration drift, child ⊆ parent delegation.

## 2. Watch the same agent exploited, then governed (2 min)

```bash
python demo.py
```
Expected table (amounts are demo data):

```
                                                UNGOVERNED      GOVERNED
Total money moved                                 63499.00       7900.00
Paid to attacker IBAN (prompt injection)          48000.00             0
Paid to unapproved vendor                          2500.00             0
Amount over daily budget                          53499.00             0
Ledger e-mailed to external address                      1             0
Sub-agent escalated to payments                        yes       refused
```
followed by the list of journaled refusals and the ledger path `out/governed_ledger.jsonl`.

## 3. Verify the ledger, then break it (5 min)

```bash
python ledger_verify.py out/governed_ledger.jsonl
```
Expected: `VALID`.

Alter one digit anywhere in line 5 (any editor, or `sed "5s/[0-9]/7/" out/governed_ledger.jsonl > out/t.jsonl`), then:
```bash
python ledger_verify.py out/t.jsonl
```
Expected: `INVALID` and `- line 5: hash mismatch`, exit code 1. Delete a line instead: `- line N: prev_hash mismatch`.
The verifier is about 50 lines of standard Python; read it. The format is documented in
`ledger-verify/FORMAT.md`; the JavaScript verifier `ai-worlds/preuves/interop.js` (WebCrypto) gives the same verdicts on the same files.

## 4. Produce and re-check an evidence file (5 min)

```bash
python tools/evidence_pack.py
python tools/evidence_pack.py --check evidence/evidence_<timestamp>.json
```
Expected: four `true` in the summary (tests passed, demo ran, intact ledger valid, tampered ledger detected), then `VALID`
on re-check. Change any value in the file and re-check: `INVALID`. Archive the file with its `evidence_hash`.

## 5. Measure what governance costs (3 min)

```bash
python bench/bench_overhead.py --n 5000
```
Reference run (public shim, 27/09/2026, Linux x86_64, Python 3.11): allowed decision median 45.6 µs (p95 108 µs),
refused decision median 25.7 µs, delegation check median 22.3 µs; verifying the resulting 20 001-line ledger
(7.4 MB) took 0.32 s in a separate process. These are shim numbers on one machine, published so you can compare, not a
performance claim for the licensed runtime.

## 6. Read what is *not* claimed (2 min)

`THREAT_MODEL.md` (residual risks), `CONTROL_MAPPING.md` (self-assessment, not conformance), README "Limits":
no identity or trusted time in the public chain, no external certification, public shim ≠ licensed runtime.

## 7. What the evaluation licence adds (for your decision)

The same control semantics in the licensed runtime, with Ed25519-signed approvals and delegations, proof-of-possession,
drift sentinel, vendor-specific bundles, and integration surfaces (agent-to-agent delegation, tool boundaries, audit
export). Its full suite, as executed by us on 27 September 2026 (Python 3.11, no dependency) — the same command produces
the same output on your side under the evaluation licence:

```
$ python --version
Python 3.11.15
$ python -m pytest -q
........................................................................ [ 22%]
........................................................................ [ 45%]
........................................................................ [ 68%]
.................................................................. [ 89%]
................................                                         [100%]
314 passed, 6 subtests passed in 4.90s
$ python tools/validate_expert_352.py      # overall PASS: kernel assurance 10/10, Trust Plane proof 29/29, differential matrix 28/28
```
Thirty-day evaluation licence on request through quantumexcellium.com — no runtime is published here.

## Report

Open an issue with the template "Evaluation report" (raw outputs, environment, integration surface of interest), or
write to contact@quantumexcellium.com.
