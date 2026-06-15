# ADR-0015: Choose an access solution for non-web internal resources

- **Status:** Draft
- **Date:** 2026-06-01
- **Deciders:** Platform team (A. Lindqvist, R. Okafor), Security (M. Sato), IT Operations (D. Berg)
- **Related:** [ADR-0014](ADR-0014-internal-website-access.md) (IAP for internal web access), [ADR-0009](ADR-0009-iap-tcp-forwarding-private-access.md) (IAP TCP forwarding for engineering access)

> **Draft** — this is a proposed decision, not yet accepted. It is the prerequisite for fully
> decommissioning the corporate VPN (see [ADR-0014](ADR-0014-internal-website-access.md)). Open questions are listed at the end and must
> be resolved before it moves to Accepted.

## Context

[ADR-0014](ADR-0014-internal-website-access.md) moved our internal **web** applications behind IAP and removed the VPN requirement for
them. The corporate VPN ([ADR-0005](ADR-0005-corporate-vpn-internal-apps.md)) still stands, because **non-web** internal resources — direct
SSH, database connections, and thick-client applications — are not covered by that decision. We
cannot decommission the VPN until non-web access has a replacement.

This space is **partially covered already**: [ADR-0009](ADR-0009-iap-tcp-forwarding-private-access.md) established IAP TCP forwarding via `gcloud`
for **engineering** access to non-HTTP systems (SSH, databases on GCP VMs), scoped per the
least-privilege rules in [ADR-0010](ADR-0010-least-privilege-tunnel-scoping.md). So any decision here must reconcile with [ADR-0009](ADR-0009-iap-tcp-forwarding-private-access.md) rather than
ignore it — the open question is whether IAP TCP forwarding is extended to cover the remaining
non-web cases and audiences, or whether a different tool is adopted alongside or in place of it.

## Decision (proposed)

No decision is locked in yet. [ADR-0014](ADR-0014-internal-website-access.md) named **Tailscale** as the leading candidate for general
non-web access. This draft records that lean while explicitly flagging that it must be squared
with [ADR-0009](ADR-0009-iap-tcp-forwarding-private-access.md)'s existing IAP TCP forwarding before acceptance.

## Options considered

1. **Keep the VPN for non-web resources (status quo).** Blocks the goal of decommissioning the
   VPN and preserves broad network-level access. Acceptable only as an interim state, not as the
   end state.
2. **Extend IAP TCP forwarding ([ADR-0009](ADR-0009-iap-tcp-forwarding-private-access.md)) to all non-web access.** Reuses tooling we already run,
   keeps a single audit trail in Cloud Audit Logs, and stays consistent with [ADR-0009](ADR-0009-iap-tcp-forwarding-private-access.md)/[ADR-0010](ADR-0010-least-privilege-tunnel-scoping.md). But
   it depends on `gcloud` tunnels per resource, which is awkward for non-technical staff, weak for
   thick-client UX, and does not naturally cover non-GCP or on-prem resources.
3. **Adopt Tailscale (WireGuard mesh).** Identity-based access with SSO, device-posture checks,
   fine-grained ACLs, any protocol, and reach across GCP, other clouds, and on-prem. It requires a
   client agent on every device, and — importantly — overlaps with [ADR-0009](ADR-0009-iap-tcp-forwarding-private-access.md), so adopting it means
   deciding whether Tailscale **replaces** IAP TCP forwarding for engineering or **coexists** with
   it (which would leave two non-web access paths to maintain and audit).

## Consequences (to be completed once a direction is chosen)

- Whichever option wins, the outcome must let us retire the VPN and must preserve a clean,
  per-resource, identity-tied audit trail for SOC 2 / ISO 27001.

## Open questions (to resolve before Accepted)

- **Relationship to [ADR-0009](ADR-0009-iap-tcp-forwarding-private-access.md):** does the chosen solution replace IAP TCP forwarding for
  engineering, or coexist with it? Two parallel non-web paths would be a maintenance and audit
  cost we should avoid without a clear reason.
- Which non-web resources are in scope (GCP-only, multi-cloud, on-prem, thick clients)?
- Device-posture / MDM requirements and how they integrate with our identity provider.
- Cost and operational ownership of any new agent-based tooling.
