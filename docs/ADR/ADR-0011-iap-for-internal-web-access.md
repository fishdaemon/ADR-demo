# ADR-0011: Use IAP for access to internal web applications instead of VPN or Tailscale

- **Status:** Superseded by [ADR-0014](ADR-0014-internal-website-access.md)
- **Date:** 2026-06-01
- **Deciders:** Platform team (A. Lincoln, T. Roosevelt), Security review (J. Madison), IT
- **Related:** [ADR-0009](ADR-0009-iap-tcp-forwarding-private-access.md) (IAP TCP forwarding for engineering access)

## Context

Most staff need to reach internal web applications — dashboards, admin consoles, internal
tooling — that we don't want exposed to the public internet. We need a way to gate access to
these sites that is secure, easy for non-technical staff, and cheap to operate.

The two conventional options both push software onto every staff member's device:

- A **VPN** grants network-level access once connected, which is broad (a connected device can
  often reach more of the network than the one app it needed) and ties authorization to network
  position rather than identity. It also requires a client and onboarding for every user.
- **Tailscale** is easier to run than a traditional VPN and works for any protocol, but it
  still requires installing and maintaining the Tailscale client on every device, and it grants
  network reachability rather than per-application authorization.

Google Cloud's Identity-Aware Proxy (IAP) takes a different approach for web apps: an
application is placed behind a Google Cloud HTTPS load balancer with IAP enabled, and users
authenticate with their existing Google identity in the browser. Access is authorized per
application through IAM. No client software, no network tunnel, no standing VPN infrastructure.

Note that IAP is **not** limited to HTTP. It also supports TCP forwarding for SSH, databases,
and other non-HTTP systems — but that path is for engineering staff and is decided separately
in **[ADR-0009](ADR-0009-iap-tcp-forwarding-private-access.md)**. This ADR covers only IAP's HTTP/web access mode for internal websites.

## Decision

We will use **IAP (HTTP/web access mode)** as the default way staff reach internal web
applications, and we will **not** roll out a VPN or Tailscale for this purpose. Access to each
internal app is authorized per application via IAM, using staff members' existing Google
identities.

Non-HTTP access (SSH, databases, other TCP services) is handled by IAP TCP forwarding through
`gcloud` as described in [ADR-0009](ADR-0009-iap-tcp-forwarding-private-access.md), and remains scoped to engineering staff.

## Options considered

1. **VPN.** Familiar and gives full network access, but requires client software for everyone,
   authorizes by network position, and grants broad reachability beyond the single app a user
   needs.
2. **Tailscale.** Easier to operate than a classic VPN and protocol-agnostic, but still needs a
   client installed on every device and grants network-level reachability rather than
   per-application access.
3. **IAP for web access (chosen).** No client software for staff — they use a browser and our
   existing Google SSO. Authorization is per application through IAM, and every access is tied
   to a Google identity. The trade-off is that it covers HTTP(S) apps only, which we accept
   because the non-HTTP cases are served by [ADR-0009](ADR-0009-iap-tcp-forwarding-private-access.md).

## Consequences

**Positive**

- **No extra software for staff.** Internal sites are reached through the browser with the
  Google login they already have — nothing to install, minimal onboarding.
- **Per-application, identity-based access.** Each app is authorized individually in IAM, far
  more granular than the network-level reach a VPN or Tailscale grants.
- **Less standing infrastructure.** No VPN concentrator or mesh coordinator to run and patch.
- **Auditable.** Access is tied to a Google identity and logged, which supports SOC 2 /
  ISO 27001 access-control evidence.

**Negative / trade-offs**

- IAP web access covers **HTTP(S) applications only**. Non-HTTP systems are out of scope here
  and rely on [ADR-0009](ADR-0009-iap-tcp-forwarding-private-access.md).
- Apps must sit behind a Google Cloud HTTPS load balancer (or a supported serverless backend).
  Apps not fronted that way — including some on-prem services — need additional setup before
  they can be put behind IAP.
- We take a dependency on Google identity as the authentication source for internal app access.

**Follow-ups**

- Inventory existing internal web apps and confirm each can sit behind a GCLB + IAP, flagging
  any that need a migration path.
