# ADR-0010: Scope IAP tunnel access to specific instances and ports with IAM conditions

- **Status:** Accepted
- **Date:** 2026-06-01
- **Deciders:** Platform team (A. Lincoln, T. Roosevelt), Security review (J. Madison)
- **Related:** [ADR-0009](ADR-0009-iap-tcp-forwarding-private-access.md) (IAP TCP forwarding for engineering access)

## Context

[ADR-0009](ADR-0009-iap-tcp-forwarding-private-access.md) established IAP TCP forwarding as the way engineers reach private VMs and databases,
authorized through the IAP-secured Tunnel User role (`roles/iap.tunnelResourceAccessor`).
That decision deliberately left *how broadly* the role is granted to this ADR.

Granting the role at the project level lets a principal tunnel to **any** port on **any** VM in
the project. That is more access than most engineers need — someone who only debugs the
application database does not need SSH to every production host — and it weakens the
least-privilege and audit story that justified IAP in the first place. We need a consistent
rule for narrowing tunnel grants.

## Decision

Tunnel access is granted at the **smallest scope that fits the need**, in this order of
preference:

1. **Per-instance grants** rather than project-wide, so a principal can only tunnel to the
   specific VMs they need.
2. **IAM conditions on the destination port**, so a grant can be limited to, say, the database
   port without also permitting SSH.

Project-wide grants of the tunnel role are reserved for break-glass roles and require security
sign-off.

## Example

Limit a user to the PostgreSQL port (5432) on a single database VM using an IAM condition:

```bash
gcloud compute instances add-iam-policy-binding db-vm \
  --zone=europe-north1-a \
  --member="user:engineer@company.com" \
  --role="roles/iap.tunnelResourceAccessor" \
  --condition='expression=destination.port == 5432,title=db-port-only,description=Tunnel to PostgreSQL port only'
```

With this binding the engineer can open a tunnel to 5432 on `db-vm` but cannot SSH to it or
reach any other host.

## Consequences

**Positive**

- Access matches need; a compromised credential grants a much smaller blast radius.
- Strengthens the least-privilege evidence for SOC 2 / ISO 27001 access-control reviews —
  grants are demonstrably scoped, not blanket.

**Negative / trade-offs**

- More IAM bindings to manage than a single project-wide grant. We mitigate this by managing
  bindings as code (Terraform) and grouping common access patterns.
- IAM conditions add a small amount of complexity to grant requests; the example above is kept
  in the internal access-request template to reduce friction.
