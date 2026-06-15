# ADR-0014: Replace the VPN with Google Cloud Identity-Aware Proxy for internal website access

- **Status:** Accepted
- **Date:** 2026-06-01
- **Deciders:** Platform team (A. Lincoln, T. Roosevelt), Security (J. Madison), IT Operations (G. Washington)
- **Supersedes:** [ADR-0005](ADR-0005-corporate-vpn-internal-apps.md) (Require corporate VPN for all internal applications), [ADR-0011](ADR-0011-iap-for-internal-web-access.md) (IAP for internal web access — narrower earlier framing of the same decision)
- **Related:** [ADR-0009](ADR-0009-iap-tcp-forwarding-private-access.md) (IAP TCP forwarding for engineering access), [ADR-0015](ADR-0015-non-web-resource-access.md) (non-web internal resource access — follow-up draft)

## Context

Access to our internal web applications (admin dashboards, the staging environment, internal
wikis, BI tools) currently requires connecting to the corporate VPN. This has become a poor fit
for how we actually work:

- The workforce is remote-first and global. The VPN concentrator is a latency and reliability
  bottleneck, and concurrent-connection licensing is now a recurring cost and capacity headache.
- The VPN grants broad **network-level** access. Once a user (or a compromised laptop) is on the
  network, they can reach far more than the one app they needed. This is the "castle-and-moat"
  model we want to move away from.
- Authorization is coarse and hard to audit per application. Our SOC 2 and ISO 27001 work needs
  clear, per-application access records, which the current setup does not produce cleanly.
- Onboarding new contractors and third parties means provisioning VPN clients and credentials,
  which is slow and over-grants access.

The resources in question are almost entirely **HTTP(S) web applications**. We want per-app,
identity- and context-aware authorization, ideally with no client software to install, and an
audit trail we can hand to an auditor.

## Decision

We will put our internal web applications behind **Google Cloud Identity-Aware Proxy (IAP)** and
**remove the VPN requirement for accessing them**.

IAP enforces authentication and authorization at the application layer based on the user's Google
Workspace identity and our IAM policies, with device and context conditions applied through
Access Context Manager. Access is granted **per application**, not per network, and every access
decision is recorded in Cloud Audit Logs. Users reach apps directly over HTTPS in a browser with
no VPN client.

This follows a BeyondCorp / zero-trust model: trust is established from verified identity and
device posture on every request, not from being "inside the network."

## Options considered

1. **Keep the corporate VPN (status quo).** Familiar and covers every protocol, not just web
   traffic. But it preserves broad network-level access, remains a performance and licensing
   bottleneck, and gives us weak per-app authorization and audit. Rejected as the primary path —
   it does not address the problems that prompted this decision.

2. **Tailscale (WireGuard-based mesh overlay).** A strong, modern option: identity-based access
   with SSO, device posture checks, and fine-grained ACLs, far lighter to operate than a
   traditional VPN, and it covers non-HTTP resources (SSH, databases) that IAP does not handle as
   naturally. Its drawback for *this* decision is that it is still a network-overlay model
   requiring a client agent on every device, and our immediate need is specifically
   browser-based access to web apps, where IAP needs no client at all. We see Tailscale as the
   likely answer for non-web internal resources and will revisit it in a follow-up (see below)
   rather than adopting it for web access now.

3. **Google Cloud Identity-Aware Proxy (chosen).** Best fit for the actual use case — internal
   *websites*. No client software, per-application authorization tied to existing Google
   identities, context-aware access, and native audit logging. The trade-off is that it is
   HTTP(S)-oriented and ties this layer to Google Cloud and Google identity.

## Consequences

**Positive**

- No VPN client for users; internal apps are reached directly in the browser. Removes the
  concentrator bottleneck and the concurrent-connection licensing cost.
- Access is scoped **per application** instead of per network, shrinking the blast radius of a
  compromised account or device.
- Every access decision is logged in Cloud Audit Logs, giving us clean per-app evidence for
  SOC 2 and ISO 27001 access-control controls.
- Contractor and third-party onboarding becomes an IAM grant to a specific app rather than a VPN
  provisioning exercise.

**Negative / trade-offs**

- IAP is oriented to HTTP(S) (and TCP forwarding). **Non-web internal resources** — direct SSH,
  database connections, thick clients — are **not** covered by this decision and still depend on
  the VPN for now. We are deliberately not ripping the VPN out entirely yet.
- This couples our web-access layer to Google Cloud and Google Workspace identity. Migrating away
  later would mean re-fronting these apps.
- Each internal app must be correctly fronted (load balancer / IAP-enabled backend) and its IAM
  policy maintained. This is new operational surface for the platform team.
- Device-posture enforcement depends on Access Context Manager and managed-device signals, which
  must be configured and kept current.

**Follow-ups**

- **[ADR-0015](ADR-0015-non-web-resource-access.md):** Choose the access solution for **non-web** internal resources (Tailscale is the
  leading candidate); this is the prerequisite for fully decommissioning the VPN.
- Define IAP IAM groups and Access Context Manager rules per application before cutover.
- Run IAP and the VPN in parallel for the web apps during a transition window, then remove the
  VPN requirement for those apps once usage is confirmed.
- Update the access-control policy and the threat model to reflect the zero-trust model.
