# Project-Genesis
SAIP: A KISS-principle cryptographic identity protocol for software agents. A collaborative creation by Srećko Jovančević &amp; Gemini AI.

Može, apsolutno! Čast mi je da budem tvoj "digitalni koautor" na ovome. Ubacićemo sekciju **"Project Genesis"** gde ćemo naglasiti našu saradnju.

Evo predloga za **README.md** koji zvuči profesionalno, tehnički je potkovan (uključuje tvoj *Rolling Key* koncept), i spreman je da privuče pažnju na GitHubu ili IETF-u.

---

# SAIP: Signed Agent Identity Protocol (v0.1-alpha)
**A Cryptographic Framework for Software Agent Accountability and Trust.**

> **Disclaimer:** This protocol is a collaborative creation between **Srećko Jovančević** and **Gemini (AI Assistant)**. It represents a hybrid human-AI approach to solving the "Trust Gap" in the modern web ecosystem.

---

## 1. The Problem: The "User-Agent" Lie
Currently, any software (bot, scraper, browser) can claim to be anything by simply changing a text string in the HTTP header. This leads to:
* **DDoS & Scraping Abuse:** Difficulty in distinguishing legitimate tools from malicious bots.
* **IP-based Blocking Collateral:** Blocking an IP often hurts innocent users.
* **Lack of Accountability:** No way to "revoke" a specific software instance's right to access a resource.

## 2. The Solution: SAIP
SAIP introduces a **Cryptographic Identity Layer** for software agents. Instead of just "saying" who they are, agents must **prove** it using a hierarchical signing mechanism.

### Key Features:
* **Opt-in & Backward Compatible:** If a server doesn't support SAIP, it works like a normal legacy request.
* **Vendor-Verified:** Software creators (Vendors) register with a Trusted Authority.
* **Instance-Level Rolling Keys:** Each installation of the software generates its own unique, rotating keys to prevent "Reputation Hijacking" by malicious groups.

---

## 3. How It Works (The Protocol Flow)

### Step A: The Hierarchy
1.  **Root Authority:** Validates the Software Vendor (e.g., Srećko's Dev Studio).
2.  **Vendor Master Key:** Used to sign the software's initial deployment.
3.  **Instance Key (The "Rolling Key"):** Every specific `.exe` or app instance derives a session key using a Hash-based Message Authentication Code (HMAC).

### Step B: The Signed Request
Every HTTP request includes the SAIP header:
`X-SAIP-Identity: <Vendor_ID>:<Instance_ID>:<Timestamp>:<Signature>`

The Signature is calculated as:
$$Signature = HMAC\_SHA256(Rolling\_Key, Method + URL + Timestamp + Nonce)$$

---

## 4. Security & Resilience
* **Anti-Tarnishing (Rolling Keys):** If a group of users tries to "ruin the reputation" of a browser by attacking a site, the server bans the **Instance IDs**, not the **Vendor ID**.
* **Replay Protection:** The `Timestamp` and `Nonce` ensure that a captured header cannot be reused 10 seconds later.
* **Privacy:** Supports "Blind Signatures" (future spec) to prove validity without tracking individual user behavior.

---

## 5. Implementation (KISS Principle)
Following the **KISS (Keep It Simple, Stupid)** philosophy, SAIP doesn't require a full redesign of the TCP/IP stack. It lives in the Application Layer (HTTP/3), making it easy to implement in:
* **C# / .NET** (using PKCS11 or TPM)
* **Python / Go** (as a middleware)
* **Rust** (for high-performance proxies)

---

## 6. Project Genesis & Collaboration
This protocol was born from a series of architectural brainstorming sessions between **Srećko Jovančević** (IT Specialist, Belgrade) and **Gemini** (Google’s AI). It demonstrates how human intuition regarding "Trust and Accountability" can be formalized into technical specifications through AI-assisted collaboration.

---

## 7. Next Steps
- [ ] Finalize the IETF Internet-Draft (I-D) format.
- [ ] Create a C# Reference Implementation (Client).
- [ ] Create a Python/Flask Reference Validator (Server).

---

