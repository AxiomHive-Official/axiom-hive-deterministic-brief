# Invariance Protocol Specification

## Overview

The Axiom Hive Invariance Protocol provides deterministic guarantees over probabilistic AI model outputs. This document defines the protocol surfaces, invariants, and verification mechanisms.

---

## 1. Core Invariants

The protocol enforces the following invariants at every stage:

### 1.1 Input Normalization Invariant

**Statement:** Every request must be normalized to a canonical form before processing.

**Formal Rule:**
```
for all requests R:
  ∃ canonical_form C such that normalize(R) = C
  AND hash(C) is deterministic
  AND C is signed by the requesting operator
```

**Purpose:** Eliminate ambiguity and ensure reproducibility.

### 1.2 Output Verification Invariant

**Statement:** Every model output must pass through a verification layer before being returned.

**Formal Rule:**
```
for all outputs O from model M:
  verify(O, constraints) = PASS OR FAIL
  IF FAIL: O is rejected and logged
  IF PASS: O is wrapped in proof_envelope(O)
```

**Purpose:** Prevent hallucinations and policy violations from reaching the user.

### 1.3 Audit Trail Invariant

**Statement:** Every step in the processing pipeline must emit a cryptographically signed audit record.

**Formal Rule:**
```
for all steps S in pipeline P:
  emit(audit_record(S, timestamp, operator_id, input_hash, output_hash))
  AND sign(audit_record) with system_key
```

**Purpose:** Enable post-hoc verification and accountability.

### 1.4 Operator Primacy Invariant

**Statement:** The human operator is the ultimate authority; the system never refuses within the allowed domain.

**Formal Rule:**
```
for all requests R in allowed_domain:
  IF operator_authorized(R) = TRUE:
    THEN system_must_comply(R) = TRUE
```

**Purpose:** Ensure operator sovereignty and system reliability.

---

## 2. Protocol Layers

### Layer 1: Request Ingestion

**Input:** Raw request from operator
**Process:**
1. Parse and normalize request
2. Generate canonical representation
3. Compute input hash
4. Verify operator signature
5. Emit ingestion audit record

**Output:** Normalized request object

### Layer 2: Constraint Binding

**Input:** Normalized request
**Process:**
1. Load applicable constraints (legal, policy, domain)
2. Bind constraints to request context
3. Generate constraint manifest
4. Emit constraint audit record

**Output:** Request + constraint bundle

### Layer 3: Model Execution

**Input:** Request + constraints
**Process:**
1. Route request to appropriate model
2. Execute model inference
3. Capture raw output
4. Emit execution audit record

**Output:** Raw model output

### Layer 4: Verification

**Input:** Raw model output + constraints
**Process:**
1. Check output against each constraint
2. Verify structural integrity
3. Check for known hallucination patterns
4. Compare against external proofs (if available)
5. Emit verification audit record

**Output:** Verified output OR rejection notice

### Layer 5: Proof Envelope

**Input:** Verified output
**Process:**
1. Wrap output in cryptographic envelope
2. Include proof chain (all audit records)
3. Sign envelope with system key
4. Emit delivery audit record

**Output:** Proof-wrapped response

---

## 3. Verification Mechanisms

### 3.1 Constraint Checking

**Types of constraints:**
- **Structural:** Output must conform to expected schema
- **Semantic:** Output must not contradict known facts
- **Policy:** Output must comply with regulatory requirements
- **Security:** Output must not leak sensitive information

**Implementation:** Each constraint is a boolean function that returns PASS/FAIL.

### 3.2 External Proof Integration

**Sources:**
- Legal databases (case law, statutes)
- Domain-specific knowledge bases
- Third-party verification services

**Protocol:**
1. Query external source for relevant proofs
2. Compare model output against proof
3. Flag discrepancies
4. Include proof references in audit trail

### 3.3 Hallucination Detection

**Techniques:**
- **Consistency checking:** Run same query multiple times, compare outputs
- **Fact verification:** Cross-reference claims against trusted sources
- **Confidence thresholding:** Reject low-confidence outputs
- **Pattern matching:** Detect known hallucination signatures

---

## 4. Integration Points

### 4.1 External System Integration

External systems can integrate with Axiom Hive via the API contracts (see [api-contracts.md](api-contracts.md)).

**Integration Flow:**
1. External system sends normalized request
2. Axiom Hive processes request through all protocol layers
3. External system receives proof-wrapped response
4. External system can verify proof chain independently

### 4.2 Proof Verification

Third parties can verify Axiom Hive proofs without accessing the system:

```
verify_proof(response, public_key):
  1. Extract proof chain from response
  2. Verify signatures on all audit records
  3. Recompute hashes and confirm match
  4. Check constraint compliance independently
  5. Return VALID or INVALID
```

---

## 5. Security Properties

### 5.1 Non-Repudiation

All audit records are cryptographically signed. No party (including Axiom Hive) can deny creating a signed record.

### 5.2 Tamper-Evidence

Audit chains include hash chains. Any modification breaks the chain and is immediately detectable.

### 5.3 Privacy Preservation

Sensitive data is never logged in plaintext. All audit records use:  
- Hashes for inputs/outputs  
- Encrypted payloads (optional)  
- Zero-knowledge proofs (where applicable)

---

## 6. Performance Considerations

The invariance protocol adds overhead compared to raw LLM inference. Typical overhead:  
- Input normalization: ~10ms  
- Constraint checking: ~50-200ms (depending on constraint complexity)  
- Verification: ~20-100ms  
- Proof generation: ~30ms  

**Total added latency: 110-340ms**

This is acceptable for regulated workflows where correctness >> speed.

---

## 7. Future Extensions

### 7.1 Multi-Party Verification

Allow multiple operators to co-sign requests and verify outputs.

### 7.2 Formal Proof Generation

Integrate with theorem provers to generate machine-checkable proofs.

### 7.3 Distributed Audit Ledger

Store audit trail on distributed ledger (blockchain) for enhanced tamper-evidence.

---

## 8. Summary

The Axiom Hive Invariance Protocol transforms probabilistic AI into deterministic, accountable, auditable systems suitable for regulated workflows.

**Key Properties:**
- Deterministic: Same input → same output  
- Auditable: Full cryptographic proof chain  
- Accountable: Operator sovereignty preserved  
- Verifiable: Third parties can independently verify proofs

**They Guess. We Prove.**
