# Project Genesis — SAIP: Signed Agent Identity Protocol

> **"Identity is the new perimeter. If you can't prove who you are, you shouldn't be knocking on the door."**

> *"Rizlu imaš a ličnu kartu nemaš... Možda ćeš večeras ti u stanici da dremaš"* — Munje

> *"Derived from 20 years of research into network stress testing and DDoS mitigation, SAIP is the defensive evolution of infrastructure management."*

---

## 🚀 Project Genesis: Human-AI Collaboration

This protocol is a unique architectural experiment born in **Belgrade, Serbia**. It was developed through a real-time, high-intensity brainstorming session between **Srećko Jovančević** (IT Strategy & Systems Expert) and **Gemini** (Google's AI).

**The Vision:** Create a "KISS-compliant" (Keep It Simple, Stupid) cryptographic trust layer for the modern web, moving beyond the easily spoofed `User-Agent` strings of the past.

SAIP is now a formal IETF individual Internet-Draft:
**[draft-jovancevic-saip-00](https://datatracker.ietf.org/doc/draft-jovancevic-saip/)**

---

## 1. The Problem: The "User-Agent" Illusion

In the current HTTP ecosystem, any bot, scraper, or malicious actor can claim to be "Chrome" or "GoogleBot" by simply editing a text string. This leads to:

- **Anonymous Abuse:** Servers cannot distinguish between a legitimate agent and a malicious DDoS bot.
- **IP-based Inaccuracy:** Blocking IPs often results in "collateral damage," affecting innocent users on shared networks.
- **Zero Accountability:** There is no mechanism to revoke access for a specific software instance without affecting the entire user base.
- **Automation Friction:** Critical systems — backup agents, internal APIs, AI bots — are frequently blocked by generic security rules.
- **SMTP Trust Gap:** Lack of granular client-level identity allows spam and abuse from compromised local systems.

The problem extends far beyond large crawlers. There is a massive volume of traffic from automated agents (backup, monitoring, AI scripts, internal integrations) that currently have no strong identity mechanism.

---

## 2. The Solution: SAIP Framework

SAIP introduces a **Cryptographic Identity Layer** at the application level (HTTP/3 compatible). It doesn't replace existing security — it adds a verifiable **"Digital License Plate"** to every software agent.

**SAIP is protocol-agnostic** — it works as a single header (`SAIP:`) and can be applied to HTTP, SMTP, and other header-based protocols.

### The DKIM Analogy

If **DKIM** is a cryptographic signature for email *messages* (proving a domain takes responsibility), then **SAIP** is a cryptographic signature for **agents and bots** — a simpler, lighter, and more broadly applicable approach to verifying the identity of automated clients.

### Relationship to RFC 9421 (HTTP Message Signatures)

RFC 9421 signs **what an agent says** — the HTTP message, headers, and body.
SAIP establishes **who is knocking** — the identity of the agent itself.

These are complementary, not competing:

> RFC 9421 is the **notarized statement**.
> SAIP is the **identity card**.

SAIP operates as the missing identity layer *beneath* RFC 9421 — first establishing who the agent is, then letting RFC 9421 verify message integrity. No conflict, full interoperability.

### Key Pillars

- **Vendor-Verified:** Developers register with a Trusted Authority to receive a Master Key.
- **Instance Isolation:** Every installation has a unique ID. If one instance is compromised, the Vendor's reputation remains intact.
- **RKDF (Rolling Key Derivation Function):** Keys rotate with every request or session, making replay attacks nearly impossible.
- **Wide Applicability:** Usable for any service instance — backup agents, AI agents, monitoring systems, internal integrations, desktop/mobile clients, SMTP servers, and more.
- **Early Verification:** Checks can happen before request processing or data transfer begins.
- **Economic Model:** Verified agents receive priority handling; anonymous traffic is throttled, not blocked.

---

## 3. Protocol Overview: Header Format

SAIP defines a cryptographic identity signal transmitted via a single HTTP-style header:

```
SAIP: id="<ID>"; alg="<ALG>"; ts="<TS>"; nonce="<NONCE>"; [pk="<PK>"]; sig="<SIG>"
```

| Param | Type      | Required | Description                                                   |
|-------|-----------|----------|---------------------------------------------------------------|
| id    | String    | Yes      | Logical agent identifier. Max 128 chars: a-z, 0-9, ., _, -  |
| alg   | String    | Yes      | Algorithm: `ed25519` or `hmac-sha256`                        |
| ts    | Integer   | Yes      | Unix timestamp for freshness validation                       |
| nonce | String    | Yes      | Per-request unique value (min 8 chars) to prevent replay      |
| sig   | Base64    | Yes      | Signature over the canonical string                           |
| pk    | Base64URL | No       | Optional public key for stateless verification                |

**Canonical string (what is signed):**
```
id=<id>;ts=<ts>;nonce=<nonce>;method=<method>;path=<path>
```

This binds the identity to a specific HTTP method and path, preventing signature reuse across endpoints.

---

## 4. Technical Proof of Concept (C#)

Following the **KISS Principle**, here is how an agent generates a signed, rolling identity header:

```csharp
using System;
using System.Security.Cryptography;
using System.Text;

public class SaipAgent
{
    private string vendorId   = "Srecko_Dev_01";
    private string instanceId = "Agent_BG_77";  // Unique to this install
    private byte[] rollingKey = Encoding.UTF8.GetBytes("Initial_Seed_123");

    public string GetSecureHeader(string method, string url)
    {
        string timestamp = DateTimeOffset.UtcNow.ToUnixTimeSeconds().ToString();
        string payload   = $"{method.ToUpper()}:{url}:{timestamp}:{instanceId}";

        using (var hmac = new HMACSHA256(rollingKey))
        {
            byte[] hash      = hmac.ComputeHash(Encoding.UTF8.GetBytes(payload));
            string signature = Convert.ToBase64String(hash);

            return $"SAIP-V1 {vendorId}:{instanceId}:{timestamp}:{signature}";
        }
    }
}
```

---

## 5. The Rolling Key Logic (RKDF)

To prevent "Reputation Hijacking" by malicious groups trying to discredit a legitimate tool, SAIP uses a sequence-based rolling key.

**The Formula:**

$$RollingKey_{n+1} = HMAC\_SHA256(MasterKey,\ InstanceID + Sequence_n)$$

- **Forward Secrecy:** Compromising one key does not expose past or future keys.
- **Fine-Grained Accountability:** Servers can ban a specific `InstanceID` without blacklisting the entire vendor or software product.

---

## 6. Trust & Discovery Model — Decentralized by Design

SAIP supports two primary trust discovery patterns:

**Stateless (Key-in-Header):**
If `pk` is provided, the server verifies the signature immediately — no external lookup required. Ideal for small entities, internal systems, and early adopters. Zero dependency on any registry.

**Registry-Based:**
The server uses `id` to look up the public key via a distributed registry. Here SAIP makes a deliberate and fundamental architectural choice:

> **No single entity should be the gatekeeper for agent identity on the web.**

SAIP's registry model is inspired by **DNS root servers** — a small number of logical authorities with massive geographic and organizational distribution, where no single participant owns or controls the whole.

The largest, most trusted infrastructure players become **Registration Entities (REs)** — not owners, but **equal participants**:

- Organizations such as Google, Microsoft, Amazon, and Cloudflare are natural RE candidates — they already operate global infrastructure at the required scale and are trusted by the wider internet ecosystem.
- RE metadata is synchronized via a **distributed registry protocol** — no central owner, no single point of failure, no single point of control.
- **Dynamic balancing:** The top 5–7 most active REs are primary replicators. As the ecosystem evolves, new strong players enter and the weakest rotate out — automatically, based on participation and reliability.
- Any entity, regardless of size, can participate via the **stateless `pk=` mode** without joining any registry at all.

This architecture ensures the web's agent identity infrastructure remains **open, interoperable, and resistant to capture** — by any single company, regardless of market position or size.

---

## 7. SMTP Integration — Early Authentication Module

SAIP can function as an **additional security module in SMTP** (analogous to DKIM, but at the agent level):

- The sending server includes `SAIP:` already in the **EHLO** phase.
- The receiving server verifies the signature immediately.
- If invalid → **connection is closed at once** (e.g., `550 SAIP verification failed`) — **before the message transfer begins** (before MAIL FROM / DATA).

**MX** says: *"this is an SMTP server."*
**SAIP** says: *"this is a **verified and legitimate** SMTP server."*

This is a natural upgrade to the SPF + DKIM + DMARC chain, adding early protection before any data is exchanged.

---

## 8. Security Considerations

| Concern               | SAIP Mitigation                                                         |
|-----------------------|-------------------------------------------------------------------------|
| Replay attacks        | Nonce per request + timestamp window (±300s)                           |
| Timing attacks        | Signature verification MUST use constant-time comparison               |
| Header injection      | Restricted `id` character set (a-z, 0-9, `.`, `_`, `-`)               |
| Key theft / cloning   | TPM hardware attestation (future work)                                 |
| Reputation hijacking  | Instance-level RKDF — revoke one instance without affecting the vendor |

---

## 9. Granular Policy Engine — Three-Level Identity Control

This is the core operational advantage of SAIP. Because every request carries a cryptographically verified identity at three distinct levels, the receiving server's policy engine can act with **surgical precision** at any level — independently or in combination.

### The Identity Hierarchy

```
Vendor (e.g. OpenAI)
  └── Bot Type (e.g. GPTBot)
        └── Instance (e.g. Agent_NYC_042)  ← SAIP verifies HERE
```

### Policy Actions — At Any Level

The server can **block**, **throttle**, or **shape** traffic at whichever level makes sense:

| Target Level | Example Action                                                              |
|--------------|-----------------------------------------------------------------------------|
| **Instance** | Block only `Agent_NYC_042` — all other OpenAI instances continue normally  |
| **Bot Type** | Throttle all `GPTBot` instances to 5 req/sec — other OpenAI bots unaffected|
| **Vendor**   | Degrade all OpenAI traffic to minimal trust — full vendor-level action      |
| **Combined** | Vendor OK + Bot Type OK + one specific instance blocked                     |

This is **firewall logic applied to cryptographic identity** — not IP addresses, not User-Agent strings, but verified identities at three levels of granularity.

### Why This Matters

Today, if one bot misbehaves, the only options are: do nothing, block the IP (collateral damage), or block the entire User-Agent string (blocks all legitimate instances too). SAIP eliminates this false choice:

- One compromised or misbehaving instance → **block that instance only**
- A bot type that systematically ignores rate limits → **throttle that bot type only**
- A vendor whose entire fleet starts acting aggressively → **degrade the whole vendor**

No collateral damage. No blunt instruments. Precise, revocable, cryptographically grounded control.

### Economic Traffic Classification

| Agent Category   | SAIP Status | Trust Level | Example Rate Limit |
|------------------|-------------|-------------|-------------------|
| Internal Systems | Verified    | High        | Unrestricted       |
| Known Partners   | Verified    | Medium      | 100 req/sec        |
| General Clients  | Verified    | Low         | 10 req/sec         |
| Anonymous        | None        | Minimal     | 1 req/sec          |

---

## 10. System Overview

```
+-------------------+       +-------------------+       +-----------------------------+
|      CLIENT       |       |      SERVER       |       |   POLICY DECISION ENGINE    |
|-------------------|       |-------------------|       |-----------------------------|
| 1. Generate ID    |       | 1. Parse SAIP     |       | 1. Trust Scoring            |
| 2. Sign Request   | ----> | 2. Verify Sig     | ----> | 2. Reputation Lookup        |
| 3. Send Header    |       | 3. Check Freshness|       | 3. Apply Rate Limit         |
+-------------------+       +-------------------+       +-----------------------------+
```

---

## 11. SAIP as a Universal Wrap / Middleware Layer

SAIP is deliberately designed as a **thin, non-intrusive cryptographic wrapper** (middleware layer) that can be added on top of existing protocols without modifying their core specifications.

It does **not** replace, extend, or break HTTP, SMTP, or any other protocol. It simply adds one additional identity signal that supporting servers can check early in the connection lifecycle.

### Why This Approach Matters

- **Zero protocol changes** — Legacy servers that do not understand SAIP will simply ignore the SAIP line and continue normally.
- **Early verification** — Rejection can happen before expensive operations (data transfer, full message processing, spam filtering, etc.).
- **Easy adoption** — SAIP can be implemented as a lightweight middleware, plugin, or policy service in popular software (Nginx, Caddy, Postfix, Haraka, Stalwart, Exim, etc.).
- **Universal applicability** — The same SAIP header format and logic works across HTTP, SMTP, and other header-capable protocols.

SAIP acts as the missing **"who is calling?"** layer, while all your existing security mechanisms (OAuth, mTLS, RFC 9421, DKIM, SPF, DMARC, rate limiting, etc.) continue to work unchanged.

### Implementation Model

SAIP is inserted at the earliest practical point after the initial handshake:

- **HTTP** → as a standard request header (`SAIP: ...`)
- **SMTP** → as an additional line sent immediately after the EHLO/HELO response (before MAIL FROM)
- **Other protocols** → wherever a custom header or command line can be added without breaking compatibility

---

## 12. SMTP Integration — Early Rejection at EHLO Phase

Here is a complete example of how SAIP works with SMTP **without touching the SMTP protocol itself**:

```
Client: EHLO backup-agent.example.com
Server: 250-server.example.com Hello backup-agent.example.com
        250-SIZE 52428800
        250-8BITMIME
        250-ENHANCEDSTATUSCODES
        250-PIPELINING
        250-CHUNKING
        250 STARTTLS

Client: SAIP: id="Srecko_Backup_Agent_01"; alg="ed25519"; ts="1744101234"; nonce="7f3k9p2m"; pk="base64publickey..."; sig="base64signature..."

Server: (SAIP middleware checks signature, timestamp, nonce, and registry or pk)

        If invalid:
        550 5.7.1 SAIP verification failed: Invalid agent identity

        If valid:
        (continue normally)

Client: MAIL FROM:<backup@company.com>
...
```

**Key points:**
- The SAIP line is sent **after** the server replies to EHLO.
- No new SMTP command or extension is required.
- If the server does not support SAIP, it will treat the line as an unknown command and continue — or ignore it. For strict environments, a future `SAIP` SMTP capability advertised in the EHLO response is recommended.
- Supporting servers can reject very early, saving resources.

**Recommended rejection message:**
`550 5.7.1 SAIP verification failed: Invalid or missing agent identity`

---

## 13. Canonical String Definition — Protocol-Agnostic

To ensure consistent signing and prevent replay attacks across different protocols, SAIP uses a protocol-specific **canonical string**.

**Base format (always included):**
```
id=<id>;ts=<ts>;nonce=<nonce>
```

**Protocol-specific extensions:**

- **For HTTP / HTTP/3:**
  ```
  id=<id>;ts=<ts>;nonce=<nonce>;method=<METHOD>;path=<path>
  ```
  *(e.g. `method=GET;path=/api/backup`)*

- **For SMTP:**
  ```
  id=<id>;ts=<ts>;nonce=<nonce>;phase=EHLO;helo=<helo_name>
  ```
  *(e.g. `phase=EHLO;helo=backup-agent.example.com`)*

- **For other protocols:**
  Use a short, meaningful descriptor:
  - `phase=CONNECT` (for generic TCP / custom protocols)
  - `command=AUTH` (when used in authentication phase)

The exact canonical string used for signing **must** be clearly documented by the implementing software. This design keeps SAIP simple while binding the signature to the specific context of the request.

---

## 14. Middleware / Policy Service — Pseudocode

A simple example of how a SAIP middleware or policy service works in practice:

```python
def saip_middleware(request_or_command):
    if not has_saip_header(request_or_command):
        # Optional: allow with lower priority / throttle
        # or reject in strict mode
        return "allow_anonymous"

    saip = parse_saip_header(request_or_command)

    if not is_fresh(saip.ts) or not is_unique_nonce(saip.nonce):
        return "550 SAIP verification failed: Replay or stale signature"

    if saip.pk:  # stateless mode
        valid = verify_signature(saip.canonical_string, saip.sig, saip.pk, saip.alg)
    else:
        pubkey = lookup_in_saip_registry(saip.id)  # fast cached lookup
        valid = verify_signature(saip.canonical_string, saip.sig, pubkey, saip.alg)

    if not valid:
        return "550 5.7.1 SAIP verification failed: Invalid agent identity"

    # Attach verified identity to session for logging / prioritization
    mark_session_as_verified(saip.vendor_id, saip.bot_type, saip.instance_id)

    return "continue"
```

This logic can be implemented as:
- Nginx / Caddy / Apache module (HTTP)
- Postfix policy daemon or Haraka / Stalwart / Exim plugin (SMTP)
- Custom proxy or sidecar service

---

## 15. Open Challenges — Join the Discussion

We are actively iterating on these architectural problems and welcome community input:

1. **The Trusted Authority Model** — How do we decentralize vendor registration so no single entity controls the internet's "identity switch"?
2. **Privacy vs. Accountability** — Can we use **Zero-Knowledge Proofs (ZKP)** to prove an agent is "Certified" without revealing the unique `InstanceID` to every website?
3. **Hardware Attestation** — Binding the `MasterKey` to the machine's **TPM (Trusted Platform Module)** so the identity cannot be exported or cloned.

---

## 16. Roadmap & IETF Goals

| Phase   | Description                                                            | Status          |
|---------|------------------------------------------------------------------------|-----------------|
| Phase 1 | Concept & Human-AI Collaboration (Project Genesis)                     | ✅ Complete     |
| Phase 2 | Reference Implementations (C# + Python) + SMTP module                 | 🔄 In Progress  |
| Phase 3 | Active participation in IETF **web-bot-auth** Working Group            | 🔄 In Progress  |
| Phase 4 | Community-driven RE ecosystem & broad adoption                         | 📋 Planned      |

IETF Draft: **[draft-jovancevic-saip-00](https://datatracker.ietf.org/doc/draft-jovancevic-saip/)**
IANA Registration requested for `SAIP` HTTP header field.

---

## 17. Conclusion

SAIP does not enforce trust — **it enables it**.

By providing a verifiable identity signal, SAIP allows servers to reward legitimate agents with preferential handling, effectively increasing the economic cost of abuse and impersonation. It is a lightweight, KISS-compliant, protocol-agnostic layer that complements — never replaces — existing standards (OAuth2, JWT, TLS, SPF/DKIM/DMARC, RFC 9421).

The modern web is flooded with automated agents. It is time they had a passport.

---

*Created by Srećko Jovančević & Gemini AI — Belgrade, Serbia, 2026*
*Based on 20+ years of network security research, from DDoS analysis (2000) to agent identity (2026)*
