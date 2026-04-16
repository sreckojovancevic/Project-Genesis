# Verifiable Identity Claims and Delegation Model (VICDM)

## Status of This Document

This document extends the conceptual framework defined in Project Genesis 2.1 and aligns with the SAIP (Signed Agent Identity Protocol) architecture.

It formalizes rules for identity assertions and delegation in application-layer communication.

---

## Abstract

This document defines a minimal rule for identity handling on the Internet:

> Identity is optional.
> Identity assertion is allowed.
> False identity assertion is not.

It introduces a verifiable model where identity claims MUST be provable through mechanisms such as DNS or cryptographic signatures (e.g., SAIP).

It further defines a delegation model that allows third-party infrastructure to operate on behalf of a domain, provided that such delegation is explicitly and verifiably authorized.

---

## 1. Introduction

Modern network systems increasingly rely on identity signals for trust decisions, prioritization, and abuse prevention.

However, most identity signals (e.g., User-Agent strings) are trivially spoofable.

This creates a fundamental problem:

> There is no enforced relationship between identity claims and verifiable ownership.

VICDM addresses this by introducing a simple constraint:

> Any asserted identity MUST be verifiable.

---

## 2. Terminology

* **Anonymous Client**
  A client that does not assert identity.

* **Claiming Client**
  A client that asserts a domain, service, or organizational identity.

* **Verified Client**
  A client whose identity claim is cryptographically or operationally verifiable.

* **Identity Claim**
  Any application-layer assertion linking a client to a domain or entity.

* **Delegated Infrastructure**
  Third-party systems authorized to act on behalf of a domain.

---

## 3. Core Principle

Systems MUST allow anonymous interaction.

However:

> A client asserting a domain-based identity MUST provide verifiable proof of association with that domain.

Unverifiable identity claims MUST be treated as invalid.

---

## 4. Verification Mechanisms

Verification MAY be performed using one or more of the following:

* DNS-based validation (forward/reverse consistency, authorization records)
* Cryptographic identity proofs (e.g., SAIP signatures)
* Transport-layer validation (TLS SNI consistency)
* Explicit delegation records

DNS is RECOMMENDED as the baseline verification mechanism due to its global availability and decentralization.

---

## 5. Delegation Model

Domains MAY authorize third-party infrastructure to act on their behalf.

However:

> Delegation MUST be explicitly declared and verifiable.

Recommended mechanism:

* DNS-based authorization records

Examples:

```
_saip-auth.example.com TXT "v=saip1; agent=agent01; pubkey=..."
_delegate.example.com CNAME proxy.trusted-cdn.com
```

Delegation that cannot be verified MUST NOT be trusted.

---

## 6. Relationship with SAIP

VICDM defines **policy and identity rules**.
SAIP provides **cryptographic enforcement** of identity at the agent level.

Together:

* VICDM answers: *Who is allowed to claim identity?*
* SAIP answers: *Can this specific instance prove it?*

---

## 7. Policy Model

Systems SHOULD apply adaptive trust instead of binary decisions.

Example:

* Verified identity → full trust
* Delegated but valid → medium trust
* Unverified claim → reduced trust / throttling
* False claim → rejection

---

## 8. Security Considerations

Unverified identity claims enable:

* Impersonation of trusted services
* Reputation abuse
* Bypass of filtering systems

VICDM mitigates these risks by requiring verifiability without removing anonymity.

---

## 9. Non-Goals

This document does NOT:

* Eliminate anonymous access
* Require universal identity systems
* Replace existing standards such as SPF, DKIM, TLS, or HTTP Message Signatures

---

## 10. Design Philosophy

* Identity is optional
* Trust is conditional
* Claims require proof
* Delegation requires transparency

---

## 11. Summary

> Anonymous interaction is permitted.
> Identity assertion is permitted.
> False identity assertion is not.

This model preserves the openness of the Internet while introducing accountability where identity is claimed.

---
