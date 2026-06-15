# ADR-0008: Rotate JWT signing keys on a schedule with an overlap window

- **Status:** Draft
- **Date:** 2026-06-01
- **Deciders:** Platform team (A. Lindqvist, R. Okafor), Security review (M. Sato)
- **Related:** [ADR-0007](ADR-0007-api-authentication.md) (API authentication)

> **Draft** — this is a proposed decision, not yet accepted. Open questions are listed at the
> end and need resolving before it moves to Accepted.

## Context

[ADR-0007](ADR-0007-api-authentication.md) adopted asymmetric (RS256) JWTs whose public keys are published via a JWKS endpoint
so each region can verify tokens locally. That decision left us owning the signing keys, and a
signing key that never changes is a long-lived secret: if the private key leaks, every token
ever issued is forgeable until we notice and react. We need a defined rotation process so that
keys have a bounded lifetime and so that rotating a key does **not** invalidate tokens that are
still within their (15-minute) lifetime.

Without a defined process, rotation becomes a risky manual event that teams avoid, which is the
opposite of what we want.

## Decision (proposed)

- Each signing key has a key ID (`kid`) included in the JWT header so verifiers select the
  correct public key from JWKS.
- We rotate signing keys on a fixed schedule **and** immediately on suspected compromise.
- Rotation uses an **overlap window**: when a new key is introduced, both the old and new
  public keys are published in JWKS. New tokens are signed with the new key; tokens already
  signed with the old key continue to validate until they expire. The old key is removed from
  JWKS only after the overlap window, which must exceed the maximum access-token lifetime
  (15 minutes per [ADR-0007](ADR-0007-api-authentication.md)) plus a safety margin.
- Key material is managed through a managed service rather than hand-rolled storage, and
  rotation is automated rather than performed by hand.

## Options considered

1. **No rotation.** Simplest, but leaves an unbounded-lifetime secret — rejected by the
   security review.
2. **Manual rotation.** Possible, but error-prone and easy to defer; a botched manual rotation
   risks signing-key downtime.
3. **Automated scheduled rotation with an overlap window (proposed).** Bounded key lifetime,
   zero-downtime rotation, and a ready path for emergency rotation on compromise.

## Consequences

**Positive**

- Bounded blast radius: a leaked key is only useful until the next rotation.
- Zero-downtime rotation thanks to the JWKS overlap window.
- A rehearsed path for emergency rotation if a key is suspected compromised.

**Negative / trade-offs**

- Adds operational machinery (key management + automation + monitoring).
- Verifiers must honour the `kid` header and refresh their JWKS cache within the overlap
  window, or they will reject newly signed tokens. The JWKS cache TTL must be shorter than the
  overlap window.

## Open questions (to resolve before Accepted)

- Rotation cadence — 30, 60, or 90 days?
- Where key material lives and how rotation is automated (e.g. Cloud KMS vs. a secrets manager).
- The exact JWKS cache TTL clients should use, and how we communicate it to integrators.
