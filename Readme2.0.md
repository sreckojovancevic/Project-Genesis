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

### Key Pillars

- **Vendor-Verified:** Developers register with a Trusted Authority to receive a Master Key.
- **Instance Isolation:** Every installation has a unique ID. If one instance is compromised, the Vendor's reputation remains intact.
- **RKDF (Rolling Key Derivation Function):** Keys rotate with every request or session, making replay attacks nearly impossible.
- **Wide Applicability:** Usable for any service instance — backup agents, AI agents, monitoring systems, internal integrations, desktop/mobile clients, SMTP servers, and more.
- **Early Verification:** Checks can happen before request processing or data transfer begins.
- **Economic Model:** Verified agents receive priority handling; anonymous traffic is throttled, not blocked.

### The DKIM Analogy

If **DKIM** is a cryptographic signature for email *messages* (proving a domain takes responsibility), then **SAIP** is a cryptographic signature for **agents and bots** — a simpler, lighter, and more broadly applicable approach to verifying the identity of automated clients.

---

## 3. Protocol Overview: Header Format

SAIP defines a cryptographic identity signal transmitted via a single HTTP-style header:

```
SAIP: id="<ID>"; alg="<ALG>"; ts="<TS>"; nonce="<NONCE>"; [pk="<PK>"]; sig="<SIG>"
```

| Param | Type      | Required | Description                                              |
|-------|-----------|----------|----------------------------------------------------------|
| id    | String    | Yes      | Logical agent identifier. Max 128 chars: a-z, 0-9, ., _, - |
| alg   | String    | Yes      | Algorithm: `ed25519` or `hmac-sha256`                   |
| ts    | Integer   | Yes      | Unix timestamp for freshness validation                  |
| nonce | String    | Yes      | Per-request unique value (min 8 chars) to prevent replay |
| sig   | Base64    | Yes      | Signature over the canonical string                      |
| pk    | Base64URL | No       | Optional public key for stateless verification           |

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

## 6. Trust & Discovery Model

SAIP supports two primary trust discovery patterns:

**Stateless (Key-in-Header):**
If `pk` is provided, the server verifies the signature immediately — no external lookup required. Ideal for small entities and internal systems.

**Registry-Based:**
The server uses `id` to look up the public key via a distributed registry. SAIP uses a model inspired by **DNS root servers**:

- Any qualified organization (Microsoft, Amazon, Google, Cloudflare, GoDaddy, etc.) can become a **Registration Entity (RE)**.
- RE metadata is replicated via a distributed ledger (blockchain-like mechanism — used purely as a synchronization layer, not a financial system).
- **Dynamic rotation:** The top 5–7 strongest REs are active replicators. The system self-balances — new dominant players enter, the weakest exit.

This enables true decentralization and broad participation from major players without any single entity owning the "identity switch."

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

| Concern                 | SAIP Mitigation                                                              |
|-------------------------|------------------------------------------------------------------------------|
| Replay attacks          | Nonce per request + timestamp window (±300s)                                |
| Timing attacks          | Signature verification MUST use constant-time comparison                    |
| Header injection        | Restricted `id` character set (a-z, 0-9, `.`, `_`, `-`)                    |
| Key theft / cloning     | TPM hardware attestation (future work)                                      |
| Reputation hijacking    | Instance-level RKDF — revoke one instance without affecting the vendor      |

---

## 9. Economic Traffic Model

SAIP shifts traffic handling from binary filtering to **incentive-based classification**:

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

## 11. Open Challenges — Join the Discussion

We are actively iterating on these architectural problems and welcome community input:

1. **The Trusted Authority Model** — How do we decentralize vendor registration so no single entity controls the internet's "identity switch"?
2. **Privacy vs. Accountability** — Can we use **Zero-Knowledge Proofs (ZKP)** to prove an agent is "Certified" without revealing the unique `InstanceID` to every website?
3. **Hardware Attestation** — Binding the `MasterKey` to the machine's **TPM (Trusted Platform Module)** so the identity cannot be exported or cloned.

---

## 12. Roadmap & IETF Goals

| Phase   | Description                                                                 | Status       |
|---------|-----------------------------------------------------------------------------|--------------|
| Phase 1 | Concept & Human-AI Collaboration (Project Genesis)                          | ✅ Complete  |
| Phase 2 | Reference Implementations (C# + Python) + SMTP module                      | 🔄 In Progress |
| Phase 3 | Active participation in IETF **web-bot-auth** Working Group                 | 🔄 In Progress |
| Phase 4 | Community-driven RE ecosystem & broad adoption                              | 📋 Planned   |

IETF Draft: **[draft-jovancevic-saip-00](https://datatracker.ietf.org/doc/draft-jovancevic-saip/)**
IANA Registration requested for `SAIP` HTTP header field.

---

## 13. Conclusion

SAIP does not enforce trust — **it enables it**.

By providing a verifiable identity signal, SAIP allows servers to reward legitimate agents with preferential handling, effectively increasing the economic cost of abuse and impersonation. It is a lightweight, KISS-compliant, protocol-agnostic layer that complements — never replaces — existing standards (OAuth2, JWT, TLS, SPF/DKIM/DMARC, RFC 9421).

The modern web is flooded with automated agents. It is time they had a passport.

---

*Created by Srećko Jovančević & Gemini AI — Belgrade, Serbia, 2026*
*Based on 20+ years of network security research, from DDoS analysis (2000) to agent identity (2026)*
Ovaj README.md je sada prava, zaokružena celina. Uspeo si da spojiš sirovu energiju "v1.0" (sa onim legendarnim citatom iz Munja koji mu daje lokalni šmek) sa ozbiljnim, tehničkim autoritetom koji donosi status IETF drafta. 

Posebno je snažan deo o **SMTP integraciji** — to SAIP-u daje dubinu koja prevazilazi običan HTTP header i pozicionira ga kao fundamentalni sigurnosni sloj.

Evo kompletno spakovanog teksta spremanog za tvoj `README.md`:

---

# Project Genesis v2.0  
**SAIP: Signed Agent Identity Protocol**

> "Identity is the new perimeter. If you can't prove who you are, you shouldn't be knocking on the door."  
> "Rizlu imaš a ličnu kartu nemaš... Možda ćeš večeras ti u stanici da dremaš" — Munje

**Derived from 20 years of research into network stress testing and DDoS mitigation, SAIP is the defensive evolution of infrastructure management.**

---

### Project Genesis: Human-AI Collaboration

Ovaj protokol je jedinstveni arhitektonski eksperiment rođen u **Beogradu, Srbija**. Razvijen je kroz intenzivnu real-time saradnju između **Srećka Jovančevića** (IT Strategy & Systems Expert) i **Gemini AI**-ja.

**Vizija:** Kreirati **KISS-compliant** (Keep It Simple, Stupid) kriptografski trust layer za moderni web, koji prevazilazi lako spoof-ovane `User-Agent` stringove.

SAIP je sada i formalni IETF individualni draft:  
**[draft-jovancevic-saip-00](https://datatracker.ietf.org/doc/draft-jovancevic-saip/)**

---

## 1. The Problem: The "User-Agent" Illusion

U današnjem HTTP ekosistemu bilo koji bot, scraper ili zlonamerni akter može da se predstavi kao "Chrome" ili "GoogleBot" samo promenom teksta. To dovodi do:

- **Anonymous Abuse** — serveri ne mogu da razlikuju legitimni agent od DDoS bota.
- **IP-based Inaccuracy** — blokiranje IP adresa često pogađa nevine korisnike na shared mrežama.
- **Zero Accountability** — nema načina da se revocira pristup samo jednoj instanci softvera.

Problem je mnogo širi od velikih crawlera — postoji ogroman saobraćaj od automatizovanih agenata (backup, monitoring, AI skripte, interna integracija...) koji trenutno nemaju jak identitet.

---

## 2. The Solution: SAIP Framework

SAIP uvodi **kriptografski identitet agenta** na nivou aplikacije (HTTP/3 kompatibilan). Ne zamenjuje postojeću sigurnost — dodaje **verifikabilnu "Digital License Plate"** svakom softverskom agentu.

**SAIP je protokol-agnostičan** — radi kao jedan header (`SAIP:`) i može se koristiti na HTTP-u, SMTP-u i drugim protokolima.

### Ključne prednosti SAIP-a

- **Široka primenjivost** — može se koristiti za **instancu bilo kog servisa**: backup agenti, AI agenti, monitoring sistemi, interna integracija, desktop/mobilni klijenti, SMTP serveri itd.
- **Rana verifikacija** — provera se može desiti pre obrade zahteva ili transfera podataka.
- **Fina granularnost** — Vendor ID + Instance ID omogućava revocaciju samo jedne kompromitovane instance.
- **Economic model** — verifikovani agenti dobijaju prioritet i bolji tretman, dok se anonimni saobraćaj ograničava.

**Analogija:** Ako je **DKIM** kriptografski potpis za email poruke (da domen preuzima odgovornost), onda je **SAIP** kriptografski potpis za **agente i botove** — jednostavniji, lakši i šire primenjiv pristup verifikaciji identiteta automatizovanih klijenata.

---

## 3. Technical Proof of Concept (C#) – v0.1 (zadržano iz v1.0)

```csharp
using System;
using System.Security.Cryptography;
using System.Text;

public class SaipAgent
{
    private string vendorId = "Srecko_Dev_01";
    private string instanceId = "Agent_BG_77"; // Unique to this install
    private byte[] rollingKey = Encoding.UTF8.GetBytes("Initial_Seed_123"); 

    public string GetSecureHeader(string method, string url)
    {
        string timestamp = DateTimeOffset.UtcNow.ToUnixTimeSeconds().ToString();
        string payload = $"{method.ToUpper()}:{url}:{timestamp}:{instanceId}";
        
        using (var hmac = new HMACSHA256(rollingKey))
        {
            byte[] hash = hmac.ComputeHash(Encoding.UTF8.GetBytes(payload));
            string signature = Convert.ToBase64String(hash);
            
            return $"SAIP-V1 {vendorId}:{instanceId}:{timestamp}:{signature}";
        }
    }
}
```

---

## 4. The Rolling Key Logic (RKDF)

SAIP koristi **Rolling Key Derivation Function (RKDF)** da spreči replay napade i "reputation hijacking".

**Formula:** Rolling Key(n+1) = HMAC-SHA256(Master Key, Instance ID + Sequence n)

- **Forward secrecy**
- **Accountability** — možeš banovati samo jednu instancu bez uticaja na ceo vendor

---

## 5. Decentralizovani Trust Model (novo u v2.0)

SAIP koristi model inspirisan **DNS root serverima** (mali broj logičkih autoriteta sa ogromnom distribucijom):

- Bilo koja kvalifikovana organizacija (Microsoft, Amazon, Google, GoDaddy, Cloudflare...) može postati **Registration Entity (RE)**.
- Metadata RE-ova se replicira preko distribuiranog ledgera (blockchain-like tehnologija — samo kao mehanizam sinhronizacije).
- **Dinamička rotacija** — top 5–7 najjačih RE-ova su aktivni replicatori. Sistem se samobalansira: novi dominantni igrač ulazi, najslabiji izlazi.
- Podržan je i **stateless** mod (`pk=` parametar) za male entitete.

Ovo omogućava decentralizaciju i široko učešće velikih igrača bez jednog centralnog vlasnika.

---

## 6. SMTP Integration – Early Authentication Module (novo u v2.0)

SAIP se može koristiti kao **dodatni security modul** u SMTP-u (slično DKIM-u, ali na nivou agenta):

- Šaljući server može poslati `SAIP:` već u **EHLO** fazi.
- Receiving server prvo proveri potpis.
- Ako nije validan → **odmah zatvara konekciju** (npr. `550 SAIP verification failed`) — **pre nego što počne transfer poruke** (MAIL FROM / DATA).

**MX** kaže „ovo je SMTP server“.  
**SAIP** kaže „ovo je **verifikovan i legitiman** SMTP server“.

Ovo je prirodna nadogradnja SPF + DKIM + DMARC lanca sa ranom zaštitom.

---

## 7. Open Challenges (Join the Discussion) – zadržano iz v1.0

1. **The Trusted Authority Model** — kako decentralizovati registraciju vendora.
2. **Privacy vs. Accountability** — mogućnost Zero-Knowledge Proofs.
3. **Hardware Attestation** — vezivanje ključa za TPM.

---

## 8. Roadmap & IETF Goals – prošireno

- **Phase 1:** Concept & Collaboration (završeno)
- **Phase 2:** Reference Implementations (C# + Python) + SMTP modul (u toku)
- **Phase 3:** Aktivno učešće u IETF **web-bot-auth** Working Group (interim sastanak 13. april 2026.)
- **Phase 4:** Community-driven RE ekosistem i široka adopcija

**SAIP Alignment** SAIP je lagani, KISS pristup unutar prostora verifikacije identiteta agenata. Cilj je da bude komplementaran postojećim standardima (RFC 9421, SPF/DKIM/DMARC) i da omogući široku primenu — od velikih cloud provajdera do malih servisa.

*Created by Srećko Jovančević & Gemini AI (Belgrade, April 2026)*
