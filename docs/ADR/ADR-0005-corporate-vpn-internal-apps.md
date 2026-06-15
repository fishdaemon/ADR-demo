# ADR-0005: Require the corporate VPN for all internal applications

- **Status:** Superseded by [ADR-0014](ADR-0014-internal-website-access.md)
- **Date:** 2024-11-20
- **Deciders:** Platform team (A. Lindqvist, R. Okafor), IT Operations (D. Berg)

## Context

Our internal applications — admin dashboards, the staging environment, internal wikis, and BI
tools — must not be exposed to the public internet. We need a single, well-understood way for
staff to reach all of them, covering both web apps and non-web services (SSH, databases,
internal thick clients). Most staff work from company-managed laptops.

## Decision

Require the **corporate VPN** to reach any internal application. Internal apps and services are
bound to the private network and are not publicly routable; staff connect to the VPN first and
then reach resources over the internal network. The VPN client is provisioned on all
company-managed devices.

## Consequences

**Positive**

- A single, familiar control that IT already operates.
- Protocol-agnostic: the same mechanism covers web apps, SSH, databases, and thick clients, not
  just HTTP.
- Resources stay on the private network with no public exposure.

**Negative**

- Once connected, a user (or a compromised device) has broad **network-level** reach, not access
  scoped to the one application they need.
- Authorization is tied to network position rather than per-application identity, which makes
  per-app access hard to audit.
- Requires client software and per-user provisioning, which adds onboarding friction for staff,
  contractors, and third parties.
- The VPN concentrator is a single chokepoint for performance and availability, and concurrent
  connections are licensed.
