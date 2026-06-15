# ADR-0007: Use stateless JWT access tokens with short-lived refresh tokens for API authentication

- **Status:** Accepted
- **Date:** 2026-05-28
- **Deciders:** Platform team (A. Lincoln, T. Roosevelt), Security review (J. Madison)
- **Supersedes:** [ADR-0003](ADR-0003-server-side-session-cookies.md) (Server-side session cookies)
- **Related:** [ADR-0008](ADR-0008-jwt-signing-key-rotation.md) (JWT signing-key rotation — follow-up covering rotation mechanics)

## Context

Our public API currently authenticates clients with server-side session cookies backed by a
shared Redis store. This worked when we ran a single region, but it now causes problems:

- We are expanding to three regions, and the shared session store has become a cross-region
  latency and consistency bottleneck.
- Third-party integrators and our mobile apps want a token-based scheme rather than cookies,
  which are awkward outside a browser.
- Our SOC 2 auditors asked us to demonstrate how access is scoped and revoked; the current
  setup makes per-request authorization decisions hard to trace.

We need an authentication approach that works across regions without a shared session store,
is friendly to non-browser clients, and gives us a clear, auditable authorization story.

## Decision

We will use **stateless JWT access tokens** signed with an asymmetric key (RS256), valid for
**15 minutes**, carrying the user ID, scopes, and tenant claim. Clients obtain a new access
token using a **refresh token** valid for **7 days**, stored server-side so it *can* be
revoked. Public keys are exposed via a JWKS endpoint so each region can verify tokens locally
without a network round-trip.

## Options considered

1. **Keep server-side session cookies (status quo).** Simple and easy to revoke instantly,
   but requires a shared, low-latency session store across regions — the exact thing we are
   trying to remove.
2. **Stateless JWTs with no refresh token.** Fully stateless and fast, but we lose any ability
   to revoke a token before it expires, which the security review rejected outright.
3. **Stateless JWT access tokens + revocable refresh tokens (chosen).** Keeps the hot path
   (verifying an access token) stateless and region-local, while preserving a revocation lever
   through the refresh token.

## Consequences

**Positive**

- No shared session store on the request hot path; each region verifies tokens independently.
- Works cleanly for mobile and third-party clients.
- Scopes and tenant are encoded in the token, so authorization decisions are explicit and
  loggable — directly useful as SOC 2 evidence.

**Negative / trade-offs**

- An access token cannot be revoked before it expires. We accept a worst-case 15-minute window
  and mitigate by keeping that lifetime short.
- We now own signing-key management and rotation (mitigated via the JWKS endpoint and a
  documented rotation runbook — see [ADR-0008](ADR-0008-jwt-signing-key-rotation.md)).
- Clients must implement refresh-token handling, which adds integration complexity. We will
  publish an SDK to reduce this burden.

**Follow-ups**

- [ADR-0008](ADR-0008-jwt-signing-key-rotation.md) will cover signing-key rotation.
- Update the integrator documentation and threat model before rollout.
