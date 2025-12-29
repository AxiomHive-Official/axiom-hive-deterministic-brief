# API Contracts

## Overview

This document defines the API contracts for integrating external systems with Axiom Hive's deterministic invariance protocol. These contracts specify request/response formats, error codes, and integration patterns.

---

## 1. Request Format

### 1.1 Standard Request Schema

```json
{
  "request_id": "<UUID>",
  "timestamp": "<ISO 8601 timestamp>",
  "operator_id": "<operator identifier>",
  "signature": "<cryptographic signature>",
  "payload": {
    "query": "<natural language query or structured input>",
    "context": {
      "domain": "<legal|finance|compliance|medical|general>",
      "jurisdiction": "<optional: US-CA, EU, etc.>",
      "constraints": ["<constraint_id_1>", "<constraint_id_2>"]
    },
    "verification_mode": "<standard|strict|audit>",
    "return_proof": true|false
  }
}
```

### 1.2 Field Descriptions

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `request_id` | UUID | Yes | Unique identifier for this request |
| `timestamp` | ISO 8601 | Yes | Request creation timestamp |
| `operator_id` | String | Yes | Authenticated operator identifier |
| `signature` | String | Yes | Cryptographic signature over request payload |
| `payload.query` | String | Yes | The actual query or instruction |
| `payload.context.domain` | Enum | Yes | Domain of operation |
| `payload.context.jurisdiction` | String | No | Legal jurisdiction (if applicable) |
| `payload.context.constraints` | Array | No | List of constraint IDs to apply |
| `payload.verification_mode` | Enum | No | Level of verification (default: standard) |
| `payload.return_proof` | Boolean | No | Whether to return full proof chain (default: false) |

### 1.3 Verification Modes

- **standard:** Normal verification (constraint checking + hallucination detection)
- **strict:** Enhanced verification (external proof integration + multi-pass consistency)
- **audit:** Maximum verification for regulatory workflows (full proof chain + timestamping)

---

## 2. Response Format

### 2.1 Standard Response Schema

```json
{
  "request_id": "<UUID matching request>",
  "response_id": "<UUID>",
  "timestamp": "<ISO 8601 timestamp>",
  "status": "<success|partial|failed>",
  "payload": {
    "result": "<deterministic output>",
    "verification_status": "<verified|rejected|warning>",
    "confidence": 0.0-1.0,
    "constraints_passed": ["<constraint_id_1>", ...],
    "constraints_failed": ["<constraint_id_1>", ...],
    "metadata": {
      "processing_time_ms": 123,
      "model_version": "<version_string>",
      "verification_layers": ["normalization", "constraint", "hallucination", ...]
    }
  },
  "proof": {
    "audit_chain": [<audit_record_1>, <audit_record_2>, ...],
    "signatures": [<signature_1>, <signature_2>, ...],
    "hash_chain": "<merkle_root>"
  },
  "errors": [
    {
      "code": "<error_code>",
      "message": "<human-readable message>",
      "severity": "<info|warning|error|critical>"
    }
  ]
}
```

### 2.2 Response Status Values

| Status | Description |
|--------|-------------|
| `success` | Request processed successfully, output verified |
| `partial` | Request processed but with warnings or degraded confidence |
| `failed` | Request failed verification or encountered error |

### 2.3 Verification Status Values

| Status | Description |
|--------|-------------|
| `verified` | Output passed all verification checks |
| `rejected` | Output failed verification and was rejected |
| `warning` | Output passed verification but with warnings |

---

## 3. Error Codes

### 3.1 Client Errors (4xx)

| Code | Message | Resolution |
|------|---------|------------|
| `400` | Invalid request format | Check request schema compliance |
| `401` | Unauthorized operator | Verify operator_id and signature |
| `403` | Forbidden domain access | Operator lacks permission for this domain |
| `422` | Unprocessable payload | Query is malformed or violates constraints |
| `429` | Rate limit exceeded | Reduce request frequency |

### 3.2 Server Errors (5xx)

| Code | Message | Resolution |
|------|---------|------------|
| `500` | Internal processing error | Retry with exponential backoff |
| `503` | Service temporarily unavailable | Wait and retry |
| `550` | Verification layer failure | Contact Axiom Hive support |

### 3.3 Domain-Specific Errors (6xx)

| Code | Message | Resolution |
|------|---------|------------|
| `600` | Constraint violation | Modify query to comply with constraints |
| `601` | Hallucination detected | Query rejected for safety; rephrase |
| `602` | External proof unavailable | Verification degraded; proceed with caution |
| `603` | Jurisdictional conflict | Specify valid jurisdiction or remove constraint |

---

## 4. Authentication & Authorization

### 4.1 Authentication Method

All requests must include a cryptographic signature:

```
signature = HMAC-SHA256(request_payload, operator_secret_key)
```

Or for public-key cryptography:

```
signature = Ed25519_sign(request_payload, operator_private_key)
```

### 4.2 Operator Identification

Each operator is assigned:
- **operator_id:** Unique identifier (UUID or email)
- **secret_key** or **key_pair:** For signing requests
- **permissions:** Domain access list (legal, finance, etc.)

### 4.3 Signature Verification

Axiom Hive verifies signatures on ingestion:
1. Extract `operator_id` from request
2. Retrieve operator's public key or secret
3. Verify signature over request payload
4. Reject if signature invalid

---

## 5. Integration Patterns

### 5.1 Synchronous Request-Response

**Use case:** Real-time queries with immediate response  
**Pattern:** HTTP POST to `/api/v1/query`  
**Timeout:** 30 seconds (configurable)  

**Example:**
```bash
curl -X POST https://api.axiomhive.net/v1/query \
  -H "Content-Type: application/json" \
  -d '{
    "request_id": "550e8400-e29b-41d4-a716-446655440000",
    "timestamp": "2025-12-28T18:00:00Z",
    "operator_id": "alexis@axiomhive.net",
    "signature": "<signature>",
    "payload": {
      "query": "What are the fiduciary duties of a director in Delaware?",
      "context": {
        "domain": "legal",
        "jurisdiction": "US-DE"
      }
    }
  }'
```

### 5.2 Asynchronous Batch Processing

**Use case:** Large-scale document analysis or compliance scanning  
**Pattern:** Submit batch, poll for results  
**Endpoint:** `/api/v1/batch`  

**Flow:**
1. Submit batch request (returns `batch_id`)
2. Poll `/api/v1/batch/{batch_id}/status`
3. Retrieve results from `/api/v1/batch/{batch_id}/results`

### 5.3 Webhook Notifications

**Use case:** Asynchronous notifications for long-running tasks  
**Pattern:** Register webhook URL, receive POST notifications  

**Webhook Payload:**
```json
{
  "event_type": "verification_complete|batch_complete|constraint_violation",
  "request_id": "<UUID>",
  "timestamp": "<ISO 8601>",
  "data": { ... }
}
```

---

## 6. Rate Limits & Quotas

### 6.1 Default Limits

| Tier | Requests/min | Requests/day | Batch size |
|------|--------------|--------------|------------|
| Free | 10 | 1,000 | 10 |
| Standard | 100 | 50,000 | 100 |
| Enterprise | Unlimited | Unlimited | 10,000 |

### 6.2 Rate Limit Headers

Responses include rate limit information:
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 87
X-RateLimit-Reset: 1704067200
```

---

## 7. Proof Verification (Independent)

Third parties can independently verify Axiom Hive proofs:

### 7.1 Verification Steps

```python
def verify_axiom_proof(response, public_key):
    # 1. Extract proof chain
    proof = response['proof']
    audit_chain = proof['audit_chain']
    signatures = proof['signatures']
    
    # 2. Verify signatures
    for record, signature in zip(audit_chain, signatures):
        if not verify_signature(record, signature, public_key):
            return False
    
    # 3. Verify hash chain
    computed_root = compute_merkle_root(audit_chain)
    if computed_root != proof['hash_chain']:
        return False
    
    # 4. Verify timestamp ordering
    for i in range(len(audit_chain) - 1):
        if audit_chain[i]['timestamp'] >= audit_chain[i+1]['timestamp']:
            return False
    
    return True
```

### 7.2 Public Key Distribution

Axiom Hive public keys are available at:
- **URL:** `https://api.axiomhive.net/.well-known/public-keys.json`
- **Format:** JWK (JSON Web Key)
- **Rotation:** Keys rotated quarterly; old keys remain valid for 90 days

---

## 8. Versioning & Deprecation

### 8.1 API Versioning

API versions are specified in the URL path:
- Current: `/api/v1/`
- Beta: `/api/v2-beta/` (for testing new features)

### 8.2 Deprecation Policy

- **Notice period:** 6 months before deprecation
- **Support period:** 12 months after deprecation announcement
- **Migration guide:** Provided with deprecation notice

---

## 9. Example Integration

### 9.1 Python SDK (Conceptual)

```python
from axiom_hive import Client

# Initialize client
client = Client(
    operator_id="alexis@axiomhive.net",
    private_key=load_private_key("operator_key.pem")
)

# Submit query
response = client.query(
    query="Analyze contract for breach of warranty claims",
    domain="legal",
    verification_mode="strict",
    return_proof=True
)

# Check verification status
if response.verification_status == "verified":
    print(f"Result: {response.result}")
    print(f"Confidence: {response.confidence}")
    
    # Verify proof independently
    if client.verify_proof(response.proof):
        print("Proof verified successfully")
```

---

## 10. Support & Contact

For API integration support:
- **Documentation:** https://docs.axiomhive.net
- **Email:** api-support@axiomhive.net
- **Enterprise:** Contact your account manager

---

**They Guess. We Prove.**
