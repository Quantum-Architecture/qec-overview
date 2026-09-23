# QEC — Quantum Excellium Core (public architecture overview)
**Engineering Intelligence. Securing Autonomy.**

QEC is a layered architecture for systems that must be not only capable, but **governable, auditable and provable**.

```
Intent  ->  Policy & Governance  ->  Orchestration  ->  Controlled Execution  ->  Runtime Integrity  ->  Evidence
```

| Layer | Public purpose | Boundary |
|---|---|---|
| Policy & Governance | what is allowed, by whom, under which budget and which class of irreversibility | policy formats and decision points are public; internal thresholds are not |
| Orchestration | routing work to the right engine under resource and energy constraints | interfaces are public; scheduling and scoring logic is not |
| Controlled Execution | fail-closed execution: beyond budget or outside the manifest, the action is refused | the refusal principle is public; enforcement internals are not |
| Runtime Integrity | detecting alteration of models and runtimes, and acting on it | the principle is public; sealing mechanisms are patent-pending |
| Evidence | chained journals, replay capsules, third-party verification | the format and the verifier are public — see [ledger-verify](https://github.com/Quantum-Architecture/ledger-verify) |

## Design rules
1. **Separation of authority** — the component that proposes an action never authorises it.
2. **Fail-closed by default** — when evidence is insufficient, the action does not happen; "insufficient" is a first-class verdict, never silently treated as "yes".
3. **Every decision leaves a proof** — chained, verifiable by a third party, without disclosing the protected content.
4. **Stated limits** — every component documents what it does not guarantee.

## Controlled disclosure
This repository publishes purpose, layers, interfaces and boundaries. It does not publish enabling algorithms, cryptographic parameters, internal thresholds or patent-sensitive implementation.

Licensing, partnerships and controlled evaluation: [quantumexcellium.com](https://quantumexcellium.com)