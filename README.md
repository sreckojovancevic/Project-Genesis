# SAIP: Signed Agent Identity Protocol (v0.1-alpha)

> **"Identity is the new perimeter. If you can't prove who you are, you shouldn't be knocking on the door."**

---

### 🚀 Project Genesis: Human-AI Collaboration
This protocol is a unique architectural experiment born in **Belgrade, Serbia**. It was developed through a real-time, high-intensity brainstorming session between **Srećko Jovančević** (IT Strategy & Systems Expert) and **Gemini** (Google’s AI).

**The Vision:** Create a "KISS-compliant" (Keep It Simple, Stupid) cryptographic trust layer for the modern web, moving beyond the easily spoofed `User-Agent` strings of the past.

---

## 1. The Problem: The "User-Agent" Illusion
In the current HTTP ecosystem, any bot, scraper, or malicious actor can claim to be "Chrome" or "GoogleBot" by simply editing a text string. This leads to:
* **Anonymous Abuse:** Servers cannot distinguish between a legitimate AI agent and a malicious DDoS bot.
* **IP-based Inaccuracy:** Blocking IPs often results in "collateral damage," affecting innocent users on shared networks.
* **Zero Accountability:** There is no mechanism to revoke access for a specific software instance without affecting the entire user base.

## 2. The Solution: SAIP Framework
SAIP introduces a **Cryptographic Identity Layer** at the application level (HTTP/3 compatible). It doesn't replace existing security; it adds a verifiable "Digital License Plate" to every software agent.

### Key Pillars:
* **Vendor-Verified:** Developers register with a Trusted Authority to receive a Master Key.
* **Instance Isolation:** Every installation has a unique ID. If one instance is compromised, the Vendor's reputation remains intact.
* **RKDF (Rolling Key Derivation Function):** Keys rotate with every request or session, making replay attacks nearly impossible.

---

## 3. Technical Proof of Concept (C#)

Following the **KISS Principle**, here is how an agent generates a signed, rolling identity header:

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
        // Concept: Method + URL + Timestamp + Instance
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
To prevent "Reputation Hijacking," SAIP uses a sequence-based rolling key. Even if a malicious actor captures a packet, the key for the *next* request will be different.

**The Formula:**
$$RollingKey_{n+1} = HMAC\_SHA256(MasterKey, InstanceID + Sequence_{n})$$

* **Security:** Provides forward secrecy.
* **Accountability:** Servers can ban specific `InstanceIDs` without blacklisting the entire software (Vendor).

---

## 5. Roadmap & IETF Goals
- [x] **Phase 1:** Concept & Collaboration (Project Genesis).
- [ ] **Phase 2:** Reference Implementations (C# and Python).
- [ ] **Phase 3:** Draft submission to IETF as an HTTP Extension.
- [ ] **Phase 4:** Community-driven Trusted Authority (TA) specifications.

## 6. How to Contribute
We are looking for security researchers, cryptographers, and sysadmins who believe in a more accountable internet. 

**Join the Genesis.** Let's build a web where "Trust" is a cryptographic certainty, not a pinky-promise.

---
*Created by Srećko Jovančević & Gemini AI (2026)*
```

---

