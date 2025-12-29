# Legal Due Diligence Flow Example

## Overview

This example demonstrates how Axiom Hive's deterministic invariance protocol would handle a legal due diligence workflow for a corporate merger.

**Scenario:** Law firm conducting due diligence on Target Company for acquisition by Acquirer Corp.

**Domain:** Legal  
**Jurisdiction:** Delaware (US-DE)  
**Verification Mode:** Audit (maximum verification)

---

## Step 1: Raw Query from Operator

**Operator:** Sarah Chen (Partner, Dewey & Associates)  
**Query:**

> "Review the corporate structure of ACME Industries Inc. (Delaware C-Corp, EIN 12-3456789) and identify any potential red flags for an acquisition by BigTech Corp. Focus on: outstanding litigation, regulatory compliance issues, and board composition requirements under Delaware law."

---

## Step 2: Axiom Hive Ingestion

**Request Normalization:**
```json
{
  "request_id": "dd-2025-12-28-001",
  "timestamp": "2025-12-28T18:30:00Z",
  "operator_id": "sarah.chen@deweylaw.com",
  "signature": "<HMAC-SHA256 signature>",
  "payload": {
    "query": "Analyze corporate structure for acquisition due diligence: ACME Industries Inc., Delaware C-Corp, EIN 12-3456789. Identify litigation, compliance, board composition issues per Delaware law.",
    "context": {
      "domain": "legal",
      "jurisdiction": "US-DE",
      "constraints": ["legal_accuracy", "cite_sources", "no_speculation"]
    },
    "verification_mode": "audit",
    "return_proof": true
  }
}
```

**Audit Record 1 (Ingestion):**
- Input hash: `sha256:a1b2c3d4...`
- Operator verified: `TRUE`
- Timestamp: `2025-12-28T18:30:00.123Z`

---

## Step 3: Constraint Binding

**Constraints Loaded:**

1. **legal_accuracy:**  
   - All statements must cite Delaware General Corporation Law (DGCL) sections or case law
   - No speculation or "likely" statements

2. **cite_sources:**  
   - Every factual claim must include citation
   - Format: `[Source: DGCL §141(b)] or [Source: Smith v. Van Gorkom, 488 A.2d 858 (Del. 1985)]`

3. **no_speculation:**  
   - Reject outputs containing: "might", "could", "possibly", "probably"
   - Only definitive statements allowed

**Audit Record 2 (Constraint Binding):**
- Constraints loaded: `[legal_accuracy, cite_sources, no_speculation]`
- Constraint manifest hash: `sha256:e5f6g7h8...`

---

## Step 4: Model Execution

**Model:** Axiom Hive Legal-1 (based on Claude 3.5 Sonnet)  
**Execution Time:** 8.3 seconds

**Raw Model Output:**

> "Based on the query, here is an analysis of ACME Industries Inc.:
> 
> **Corporate Structure:**  
> ACME Industries Inc. is a Delaware C-Corporation [Source: Delaware Division of Corporations Filing #123456]. The board consists of 7 directors [Source: ACME 2024 Proxy Statement].
> 
> **Potential Red Flags:**
> 
> 1. **Outstanding Litigation:**  
>    - Shareholder derivative lawsuit pending in Delaware Chancery Court (Case No. 2024-CV-1234) alleging breach of fiduciary duty by directors [Source: Delaware Chancery Court Docket].
>    - This could delay acquisition pending resolution [Source: DGCL §251(c)].
> 
> 2. **Regulatory Compliance:**  
>    - Recent SEC Form 8-K filing discloses late filing of 10-Q for Q2 2025 [Source: SEC EDGAR Database].
>    - Potential SEC enforcement action pending.
> 
> 3. **Board Composition:**  
>    - Delaware law requires majority of disinterested directors for conflict-of-interest transactions [Source: DGCL §144].
>    - Current board has 3 directors with financial ties to CEO, potentially problematic for approving merger.
> 
> **Recommendation:**  
> Conduct deeper investigation into litigation risk and SEC compliance status before proceeding."

**Audit Record 3 (Execution):**
- Model: `Axiom-Legal-1`
- Execution time: `8.3s`
- Output hash: `sha256:i9j0k1l2...`

---

## Step 5: Verification

### Verification Pass 1: Structural Check

✅ Output contains sections for litigation, compliance, board composition  
✅ Output is formatted as structured analysis  
✅ No malformed JSON or corrupted text

### Verification Pass 2: Constraint Compliance

✅ **legal_accuracy:** All Delaware law references verified against DGCL database  
✅ **cite_sources:** All 7 factual claims include citations  
✅ **no_speculation:** "could" appears once—FLAGGED for review

**Constraint Violation Detected:**  
Phrase "This could delay acquisition" contains speculative language.

**Auto-Correction Applied:**  
Replaced with: "Delaware law permits shareholders to seek injunctive relief to delay mergers during pending litigation [Source: DGCL §251(c)]."

### Verification Pass 3: Hallucination Check

✅ Delaware Division of Corporations filing verified (fictitious for this example)  
✅ DGCL §251(c) accurately cited  
✅ DGCL §144 accurately cited  
✅ Case law format correct

### Verification Pass 4: External Proof Integration

**External Sources Queried:**
- Delaware General Corporation Law (official)
- Delaware Chancery Court dockets (public records)
- SEC EDGAR database (public filings)

**Cross-Reference Results:**
- DGCL citations: VERIFIED
- SEC filing reference: VERIFIED (example)
- Court case: VERIFIED (example)

**Audit Record 4 (Verification):**
- Constraint violations: `1 (auto-corrected)`
- Hallucinations detected: `0`
- External proofs: `3/3 verified`
- Verification status: `PASSED`

---

## Step 6: Proof Envelope Generation

**Final Output (Proof-Wrapped):**

```json
{
  "request_id": "dd-2025-12-28-001",
  "response_id": "resp-dd-2025-12-28-001",
  "timestamp": "2025-12-28T18:30:08.456Z",
  "status": "success",
  "payload": {
    "result": "<corrected output text>",
    "verification_status": "verified",
    "confidence": 0.97,
    "constraints_passed": ["legal_accuracy", "cite_sources", "no_speculation (corrected)"],
    "constraints_failed": [],
    "metadata": {
      "processing_time_ms": 8456,
      "model_version": "Axiom-Legal-1",
      "verification_layers": ["structural", "constraint", "hallucination", "external_proof"]
    }
  },
  "proof": {
    "audit_chain": [
      { "step": "ingestion", "hash": "sha256:a1b2c3d4...", "timestamp": "2025-12-28T18:30:00.123Z" },
      { "step": "constraint_binding", "hash": "sha256:e5f6g7h8...", "timestamp": "2025-12-28T18:30:00.245Z" },
      { "step": "execution", "hash": "sha256:i9j0k1l2...", "timestamp": "2025-12-28T18:30:08.345Z" },
      { "step": "verification", "hash": "sha256:m3n4o5p6...", "timestamp": "2025-12-28T18:30:08.456Z" }
    ],
    "signatures": [
      "<sig1>", "<sig2>", "<sig3>", "<sig4>"
    ],
    "hash_chain": "<merkle_root: sha256:q7r8s9t0...>"
  },
  "errors": [
    {
      "code": "600",
      "message": "Constraint 'no_speculation' triggered auto-correction",
      "severity": "warning"
    }
  ]
}
```

---

## Step 7: Operator Verification (Optional)

Sarah Chen can independently verify the proof chain:

```python
import axiom_hive_verifier

# Load response
response = load_json("response.json")

# Verify proof
verification = axiom_hive_verifier.verify_proof(
    response=response,
    public_key=axiom_public_key
)

if verification.valid:
    print("✓ Proof verified")
    print(f"✓ All {len(response['proof']['audit_chain'])} audit records signed")
    print(f"✓ Hash chain intact")
    print(f"✓ Constraints enforced")
else:
    print("✗ Proof verification failed")
```

**Output:**
```
✓ Proof verified
✓ All 4 audit records signed
✓ Hash chain intact
✓ Constraints enforced
```

---

## Key Takeaways

1. **Determinism:** Same query → same output (modulo model updates)
2. **Auditability:** Full cryptographic proof chain from input to output
3. **Accountability:** Operator cannot deny making query; system cannot deny corrections
4. **Verifiability:** Third parties can independently verify proof without accessing Axiom Hive

**This is not a guess. This is a proof.**

---

**They Guess. We Prove.**
