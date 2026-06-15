# ADR-0009: Use IAP TCP forwarding for engineering access to private resources

- **Status:** Accepted
- **Date:** 2026-06-01
- **Deciders:** Platform team (A. Lincoln, T. Roosevelt), Security review (J. Madison)
- **Related:** [ADR-0007](ADR-0007-api-authentication.md) (API authentication), [ADR-0010](ADR-0010-least-privilege-tunnel-scoping.md) (least-privilege scoping of tunnel access), [ADR-0014](ADR-0014-internal-website-access.md) (IAP for internal web access — supersedes [ADR-0011](ADR-0011-iap-for-internal-web-access.md), which this ADR originally referenced)

## Context

Engineers occasionally need direct access to private infrastructure — SSHing into VMs,
connecting to a database for a debugging session, or reaching an internal web UI. Our VMs and
databases have no public IP addresses by design, so today this access depends on a bastion
host and, for some engineers, a VPN. That arrangement has drawbacks:

- The bastion is standing infrastructure we have to patch, monitor, and treat as a high-value
  target.
- Access is gated by SSH keys and network position rather than by identity, which makes it
  hard to answer "who connected to the production database, and when?" — exactly the kind of
  question our SOC 2 auditors ask.
- VPN onboarding is friction for new engineers and contractors.

Google Cloud's Identity-Aware Proxy (IAP) supports TCP forwarding: `gcloud` opens an
authenticated tunnel from an engineer's laptop to a port on a private VM, brokered by IAP.
It is not limited to SSH — any TCP port can be forwarded, including database ports.

Granting tunnel access broadly would undermine the audit and least-privilege goals above, so
*how* we scope that access down — restricting principals to specific instances and ports using
IAM conditions — is decided separately in **[ADR-0010](ADR-0010-least-privilege-tunnel-scoping.md)**. This ADR assumes that scoping is in
place; it does not redefine it.

## Decision

We will use **IAP TCP forwarding via `gcloud compute start-iap-tunnel`** as the standard
mechanism for engineer access to private VMs and databases. Access is authorized through IAM
using the **IAP-secured Tunnel User** role (`roles/iap.tunnelResourceAccessor`), granted at
the instance level where possible rather than project-wide. We will **decommission the bastion
host** once teams have migrated.

## Options considered

1. **Keep the bastion host + VPN (status quo).** Familiar, but it is standing attack surface,
   ties access to SSH keys and network location instead of identity, and gives a weak audit
   trail.
2. **Cloud VPN / interconnect for everyone.** Heavier to operate and still authorizes by
   network position rather than identity.
3. **IAP TCP forwarding (chosen).** No public IPs, no bastion, no VPN. Access is authenticated
   against the engineer's Google identity, authorized through IAM, and every tunnel connection
   is recorded in Cloud Audit Logs.

## Consequences

**Positive**

- No bastion host and no VPN concentrator to maintain — less standing infrastructure and
  attack surface.
- Access is per-identity and per-resource via IAM, and can be scoped to specific instances or
  ports with IAM conditions.
- Every connection is logged in Cloud Audit Logs, giving us a clean, identity-tied access trail
  that directly supports SOC 2 / ISO 27001 access-control evidence.

**Negative / trade-offs**

- IAP TCP forwarding is not meant for bulk data transfer; Google may rate-limit heavy use, so
  this is for interactive access, not data exports.
- Idle tunnels are dropped after about an hour of inactivity, so long-running sessions need to
  handle reconnection.
- Engineers need `gcloud` installed and authenticated, and the project needs a firewall rule
  allowing IAP's source range.

## Runbook — how engineering staff use it

**One-time setup (per project, done by an admin)**

Allow IAP's source range to reach the target ports through the firewall:

```bash
gcloud compute firewall-rules create allow-iap-tcp \
  --network=default \
  --direction=INGRESS \
  --action=ALLOW \
  --rules=tcp:22,tcp:5432 \
  --source-ranges=35.235.240.0/20
```

Grant a user the tunnel role — prefer scoping it to a single instance:

```bash
# Per-instance (preferred)
gcloud compute instances add-iam-policy-binding db-vm \
  --zone=europe-north1-a \
  --member="user:engineer@company.com" \
  --role="roles/iap.tunnelResourceAccessor"

# Project-wide (use sparingly)
gcloud projects add-iam-policy-binding my-project \
  --member="user:engineer@company.com" \
  --role="roles/iap.tunnelResourceAccessor"
```

**SSH into a private VM**

```bash
gcloud compute ssh app-vm \
  --zone=europe-north1-a \
  --tunnel-through-iap
```

(If the VM has no public IP, gcloud uses IAP automatically; the flag just makes it explicit.)

**Reach a private database (PostgreSQL example)**

Open a tunnel that maps the remote DB port to a local port, then connect your client to
localhost:

```bash
# In one terminal: open the tunnel
gcloud compute start-iap-tunnel db-vm 5432 \
  --local-host-port=localhost:5432 \
  --zone=europe-north1-a

# In another terminal: connect as if the DB were local
psql -h localhost -p 5432 -U app_user app_db
```

The same pattern works for any TCP service — internal web UIs, RDP (3389), etc. Just change the
remote port and the local port.

**Follow-ups**

- Decommission the bastion host once all teams have migrated (tracked separately).
