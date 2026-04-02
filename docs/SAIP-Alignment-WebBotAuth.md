Može, i to bi izgledalo izuzetno profesionalno. Formatiranje u **Markdown (.md)** obezbeđuje preglednost, automatsko generisanje tabela i onaj "inženjerski" izgled koji korisnici na GitHub-u očekuju od ozbiljnih protokola.

Evo kako bi tvoj tekst trebalo da bude formatiran u samom fajlu `docs/SAIP-Alignment-WebBotAuth.md` da bi se maksimalno iskoristila moć Markdown-a:

---

# SAIP: Signed Agent Identity Protocol

**Version:** 1.0  
**Status:** Draft / Proposal  
**Author:** Srećko Jovančević  
**Date:** April 2026

---

## 1. Introduction
The modern web lacks a reliable mechanism for verifying the identity of HTTP user agents. Existing methods such as User-Agent strings and IP-based attribution are insufficient due to spoofing and shared infrastructure. **SAIP** introduces a lightweight, opt-in mechanism for verifiable client identity at the application layer.

## 2. Problem Statement
* **Spoofability:** User-Agent strings are easily manipulated.
* **IP Fatigue:** IP-based filtering is unreliable (NAT, shared proxies).
* **Automation Friction:** Legitimate systems (backup, API, AI agents) are frequently blocked.
* **SMTP Trust Gap:** Lack of client-level identity allows abuse from compromised systems.

## 3. Design Goals
* **Simplicity (KISS):** Minimal overhead, single-header implementation.
* **Opt-in:** No mandatory adoption.
* **Backward Compatibility:** No changes to existing protocol semantics.
* **Protocol-Agnostic:** Applicable to HTTP, SMTP, and similar protocols.

## 4. Protocol Overview
SAIP defines a cryptographic identity signal transmitted via a single header.

### 4.1 Header Format
`SAIP: id="<ID>"; alg="<ALG>"; ts="<TS>"; nonce="<NONCE>"; sig="<SIG>"`

### 4.2 Parameters
| Parameter | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| **id** | String | Yes | Agent or instance identifier |
| **alg** | String | Yes | Algorithm identifier (e.g., `hmac-sha256`) |
| **ts** | Integer | Yes | Unix timestamp |
| **nonce** | String | Yes | Unique per-request value |
| **sig** | Base64 | Yes | Base64-encoded signature |

## 5. Canonicalization and Signature
The signature **MUST** be computed over a canonical string:
`id=<id>;ts=<ts>;nonce=<nonce>;method=<method>;path=<path>`

This binds identity to a specific request and prevents replay across endpoints.

## 6. Processing Model
1.  **Client:** Constructs canonical string, signs using local key, and sends SAIP header.
2.  **Server:** Parses header, reconstructs string, verifies signature, and applies policy.

## 7. Trust Model
SAIP does not mandate a specific trust infrastructure. A globally accessible identity registry (e.g., **Consortium Blockchain** or **DNS-based**) MAY be used to associate `id` values with cryptographic keys.

## 8. Multi-Layer Identity Model
SAIP operates as part of a layered system:
* Network layer (IP, TLS)
* Domain layer (DNS)
* **Application layer (SAIP)**
* Instance layer (InstanceID)

## 9. Deployment Model
* **Optional:** Clients MAY include headers; servers MAY ignore them.
* **Incremental:** Adoption can be policy-driven.

## 10. Security Considerations
* Timestamp validation (±300 seconds).
* Constant-time signature comparison is **REQUIRED**.
* Replay protection via nonce tracking.

## 11. Policy Model
SAIP shifts traffic handling from binary filtering to **incentive-based classification**.
* **Verified SAIP:** Preferential handling ("Fast Track").
* **Anonymous:** Standard throttling/challenges.

## 12. Example Policy Implementation
```c
if (has_saip_header(request)) {
    if (verify_signature(request) &&
        is_within_time_window(request.ts) &&
        is_nonce_valid(request.nonce) &&
        get_reputation(request.id) > TRUST_THRESHOLD) {

        allow_immediate_processing(request); // Fast Track
    } else {
        apply_strict_rate_limit(request); // Penalty
    }
} else {
    apply_standard_throttling(request); // Anon Zone
}
```

## 13. Economic Traffic Model
| Agent Category | SAIP Status | Trust Level | Rate Limit | Priority |
| :--- | :--- | :--- | :--- | :--- |
| **Internal Systems** | Verified | High | Unrestricted | High |
| **Known Partners** | Verified | Medium | Moderate | Medium |
| **General Clients** | Verified | Low | Limited | Standard |
| **Anonymous** | None | Minimal | Strict | Lowest |

## 14. System Overview
```text
+-------------------+       +-------------------+       +-----------------------------+
|      CLIENT       |       |      SERVER       |       |   POLICY DECISION ENGINE    |
|-------------------|       |-------------------|       |-----------------------------|
| Generate Signature| ----> | Verify Signature  | ----> | Trust Scoring               |
| Add SAIP Header   |       | Check TS/Nonce    |       | Rate Limiting (Fast Track)  |
+-------------------+       +-------------------+       +-----------------------------+
```

## 15. Conclusion
**SAIP does not enforce trust — it enables it.** It provides a minimal, composable identity signal that allows servers to make informed decisions in an automated environment.
