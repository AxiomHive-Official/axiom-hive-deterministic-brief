# Axiom Hive – Deterministic Invariance Protocol (Public Brief)

They guess. We prove.

This repository is a **public technical brief** of Axiom Hive's deterministic AI architecture:
- What guarantees we provide.
- How the invariance protocol is structured.
- How external systems can integrate with a verified "They Guess. We Prove." layer.

No proprietary kernel or optimization code is present here. This repo is safe to share with:
- Investors
- Technical partners
- General public

---

## 1. Problem

LLMs are:
- Non-deterministic.
- Non-auditable.
- Non-accountable.

Regulated workflows (law, finance, compliance, medicine) need:
- Deterministic answers.
- Cryptographic audit trails.
- Operator sovereignty over every decision.

---

## 2. Axiom Hive approach

Axiom Hive wraps probabilistic models with a **deterministic invariance protocol**:

- **Input constraints:** Every request is normalized and signed.
- **Verification layer:** Outputs are checked against invariants and external proofs.
- **Proof chain:** Every step emits a cryptographic record for later verification.
- **Operator primacy:** The human operator is the single source of truth; the system never "refuses" within the allowed domain.

This repository documents:
- The invariance protocol surfaces (`/spec`).
- Example flows (`/examples`).
- High-level architecture diagrams (`/diagrams`).

---

## 3. What is here (and what is not)

Included:
- High-level protocol specification.
- Integration contracts (how third parties would talk to Axiom Hive).
- Conceptual examples and traces.

Not included:
- Internal kernels.
- Optimization strategies.
- Production deployment details.

For partnership, enterprise, or investment discussions:
- Visit [https://www.axiomhive.net](https://www.axiomhive.net)
- Or contact: [your contact email]

---

## 4. Repository structure

```
axiom-hive-deterministic-brief/
├─ README.md
├─ /spec
│  ├─ invariance-protocol.md
│  ├─ api-contracts.md
│  └─ threat-model.md
├─ /examples
│  ├─ legal_due_diligence_flow.md
│  ├─ audit_trail_demo.md
│  └─ toy_protocol_trace.json
├─ /diagrams
│  ├─ architecture-overview.png
│  └─ verification-flow.png
└─ /meta
   └─ disclaimer.md
```

---

## 5. Getting started

1. Read the [Invariance Protocol specification](spec/invariance-protocol.md)
2. Review the [API Contracts](spec/api-contracts.md)
3. Explore the [example flows](examples/)
4. Check the [threat model](spec/threat-model.md)

---

## License

This documentation is released under MIT License. The actual Axiom Hive implementation remains proprietary.

---

**They Guess. We Prove.**
