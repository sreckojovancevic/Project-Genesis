# SAIP: Signed Agent Identity Protocol

**Version:** 1.1 (Identity & Discovery Stable)  
**Status:** Draft / Proposal  
**Author:** Srećko Jovančević  
**Date:** April 2026

---

## 1. Introduction
The modern web lacks a reliable mechanism for verifying the identity of HTTP user agents. Existing methods such as User-Agent strings and IP-based attribution are insufficient due to spoofing, shared infrastructure (NAT), and the rise of automated agents. 

**SAIP** introduces a lightweight, opt-in mechanism for verifiable client identity at the application layer, enabling servers to distinguish between legitimate automated traffic and malicious actors.

## 2. Problem Statement
* **Spoofability:** `User-Agent` strings are trivial to manipulate.
* **IP Fatigue:** IP-based filtering is unreliable for NAT users and shared cloud proxies.
* **Automation Friction:** Critical systems (backup agents, internal APIs, AI bots) are frequently blocked by generic security rules.
* **SMTP Trust Gap:** Lack of granular client-level identity allows spam and abuse from compromised local systems.

## 3. Design Goals
* **Simplicity (KISS):** Minimal overhead, single-header implementation.
* **Opt-in:** No mandatory adoption; compatible with legacy systems.
* **Backward Compatibility:** No changes to existing protocol semantics or HTTP methods.
* **Protocol-Agnostic:** Applicable to HTTP, SMTP, and other header-based protocols.

## 4. Protocol Overview
SAIP defines a cryptographic identity signal transmitted via a single header. SAIP is designed as a supplemental identity signal, not a replacement for existing authentication or bot-detection frameworks (like OAuth2, JWT, or TLS-fingerprinting). It provides an additional layer of verifiable metadata that allows existing web-bot-auth systems to make faster and more accurate trust decisions.

## 4. Protocol Overview

SAIP defines a cryptographic identity signal transmitted via a single HTTP‑style header. SAIP is designed as a **supplemental identity signal**, not a replacement for existing authentication or bot‑detection mechanisms (such as OAuth 2.0, JWT, or TLS‑level fingerprinting). It provides an additional layer of verifiable metadata that can be consumed by existing web‑bot‑auth systems and security stacks to make faster, more accurate trust decisions.

In practice, SAIP allows:
* Application‑level identification of specific software instances (e.g., a backup agent or M365 sync bridge).
* Flexible key discovery, either via an embedded public key (`pk`) or via external registries based on the `id`.
* Gradual, opt‑in deployment alongside legacy infrastructure, without altering the semantics of HTTP or SMTP.

### 4.1 Header Format

The SAIP header is structured as a semicolon‑separated list of parameters:

`SAIP: id="<ID>"; alg="<ALG>"; ts="<TS>"; nonce="<NONCE>"; [pk="<PK>"]; sig="<SIG>"`

Each parameter is encoded as a quoted string, and the order of parameters is not significant for parsing. The header is case‑sensitive for the parameter names, but implementations MUST treat them as case‑sensitive identifiers in their logic.

### 4.2 Parameters

| Parameter | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| **id** | String | **Yes** | Logical agent identifier (e.g., `backup-001`, `m365-bridge-002`). Allowed characters: `a-z`, `0-9`, `.`, `_`, `-`. **No semicolons (`;`) or spaces.** Max 128 characters. Used for reputation, logging, and instance‑level rate‑limiting. |
| **alg** | String | **Yes** | Algorithm identifier. Recommended: `ed25519` (asymmetric) or `hmac-sha256` (symmetric). Implementations SHOULD support at least one of these algorithms. |
| **ts** | Integer | **Yes** | Unix timestamp (seconds) for freshness validation. Enables servers to reject stale or replayed requests. |
| **nonce** | String | **Yes** | Per‑request unique value (min 8 characters) to prevent replay attacks. The server MAY track nonces for sensitive endpoints to detect reuse. |
| **sig** | Base64 | **Yes** | Signature computed over the canonical string defined in Section 5. MUST be encoded as standard Base64 (with optional padding) suitable for HTTP header values. |
| **pk** | Base64URL | No | (Optional) Public key for immediate or stateless verification. If present, the server MAY use this key to verify the signature without performing an external registry lookup. The key is encoded in Base64URL without padding (`=`). |

## 5. Canonicalization and Signature
The signature **MUST** be computed over a canonical string constructed as follows:
`id=<id>;ts=<ts>;nonce=<nonce>;method=<method>;path=<path>`

*Note: The canonical string binds the identity to a specific request method and path, preventing signature reuse across different endpoints.*

## 6. Processing Model
1.  **Client:**
    * Constructs the canonical string.
    * Signs the string using a private key (Asymmetric) or shared secret (Symmetric).
    * Adds the `SAIP` header to the request.
2.  **Server:**
    * Parses the `SAIP` header.
    * Reconstructs the canonical string from request metadata.
    * Verifies the signature (using `pk` from header or looking up key via `id`).
    * Applies policy based on the verified identity.

## 7. Trust & Discovery Model
SAIP supports two primary trust discovery patterns:
* **Stateless (Key-in-Header):** If `pk` is provided, the server may verify the signature immediately.
* **Registry-Based:** The server uses `id` to lookup the public key in a distributed registry (e.g., **Consortium Blockchain**, DNS-based records, or internal pre-shared keys).

## 8. Security Considerations
* **Timestamp Validation:** Servers SHOULD enforce a strict window (e.g., ±300 seconds).
* **Constant-Time Comparison:** Signature verification MUST use constant-time algorithms to prevent timing attacks.
* **Replay Protection:** Nonce tracking is recommended for sensitive endpoints.
* **ID Safety:** By restricting the `id` character set, SAIP prevents header injection and parsing ambiguities.

## 9. Economic Traffic Model (Policy)
SAIP shifts traffic handling from binary filtering to **incentive-based classification**.

| Agent Category | SAIP Status | Trust Level | Example Rate Limit | Handling Priority |
| :--- | :--- | :--- | :--- | :--- |
| **Internal Systems** | Verified | High | Unrestricted | **High (Fast Track)** |
| **Known Partners** | Verified | Medium | 100 req/sec | **Medium** |
| **General Clients** | Verified | Low | 10 req/sec | **Standard** |
| **Anonymous** | None | Minimal | 1 req/sec | **Lowest (Throttle)** |

## 10. System Overview
```text
+-------------------+       +-------------------+       +-----------------------------+
|      CLIENT       |       |      SERVER       |       |   POLICY DECISION ENGINE    |
|-------------------|       |-------------------|       |-----------------------------|
| 1. Generate ID    |       | 1. Parse SAIP     |       | 1. Trust Scoring            |
| 2. Sign Request   | ----> | 2. Verify Sig     | ----> | 2. Reputation Lookup        |
| 3. Send Header    |       | 3. Check Freshness|       | 3. Apply Rate Limit         |
+-------------------+       +-------------------+       +-----------------------------+
```

## 11. Conclusion
**SAIP does not enforce trust — it enables it.** By providing a verifiable identity signal, SAIP allows servers to reward legitimate agents with preferential handling, effectively increasing the economic cost of abuse and impersonation.


