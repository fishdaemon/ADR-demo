# ADR-0003: Use server-side session cookies for API authentication

- **Status:** Superseded by [ADR-0007](ADR-0007-api-authentication.md)
- **Date:** 2024-09-15
- **Deciders:** Platform team (A. Lincoln, T. Roosevelt)

## Context

The product runs in a **single region** and is accessed almost entirely through the browser.
We need a simple, well-understood way to authenticate users and to be able to revoke a session
immediately (e.g. on logout or a security event).

## Decision

Authenticate users with **server-side sessions**: on login we create a session record in a
shared Redis store and return an opaque session cookie to the browser. Each request looks the
session up in Redis to authenticate and authorize the user.

## Consequences

**Positive**

- Simple and familiar; well-supported by our web framework.
- Instant revocation — deleting the session record immediately invalidates the cookie.

**Negative**

- Every request depends on a shared, low-latency session store that all application instances
  must reach.
- Cookies are awkward for non-browser clients (mobile apps, third-party integrators).
- Per-request authorization decisions are hard to trace for audit purposes.
