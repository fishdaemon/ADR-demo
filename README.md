# ADR-demo — Architecture Decision Records, in practice

A small demo repo for teaching **Architecture Decision Records (ADRs)**: short
markdown files that capture *why* a meaningful technical decision was made, not
just *what* was done.

This README is meant to be presentable in about 15 minutes. The ADRs in
[`docs/ADR/`](docs/ADR/) are the deep dive — open them after the talk.

## What is an ADR?

An ADR is a one-page Markdown file that records a single architectural decision
together with its **context**, the **options considered**, the **decision**
itself, and the **consequences** (positive *and* negative) of choosing it. ADRs
live in the same git repo as the code, so the decision trail sits next to the
thing it explains.

## Why ADRs?

- **A durable decision trail.** Six months from now nobody remembers why the
  refresh token is 7 days. The ADR does.
- **Architecture-level documentation.** Code shows *what*. ADRs show *why this
  shape and not the other shape* — the level of context the code itself never
  captures.
- **Onboarding and handover.** New hires (and the next team) can read the ADRs
  and absorb years of accumulated reasoning without booking a single call.
- **Decisions are made together.** Authoring an ADR forces the team and the
  relevant stakeholders (security, ops, product) to land the trade-offs
  *before* the code lands. No more "wait, why did they do it like that and
  not the obvious way?"
- **Compliance evidence.** SOC 2 / ISO 27001 auditors ask *why* a control was
  chosen. Hand them the relevant ADR.
- **Great context for AI code gen.** LLMs are good at *what*. They are bad at
  *the constraint you settled on six months ago*. Feeding an ADR to a model —
  especially the "we picked this **and not** that because…" parts — turns
  vibe-coding into vibe-coding-that-respects-your-architecture. Combine ADRs
  with PRDs and specs and the quality jump is real.

## Lifecycle

```
   ┌─────────┐         ┌─────────────┐         ┌──────────────────────┐
   │  Draft  │ ──────► │  Accepted   │ ──────► │ Superseded by ADR-N  │
   └─────────┘         └─────────────┘         └──────────────────────┘
       │                     │                          │
  open questions       body is now              body stays frozen;
  still listed         IMMUTABLE — only         the new ADR is the
  in the file          metadata in the          live decision
                       header may change
```

Superseding is how decisions evolve. You **never** rewrite an Accepted ADR to
reflect a new direction — you write a **new** ADR that supersedes it, and
update the old one's `Status:` to point at the replacement. The history stays
honest. The chains currently in this repo:

```
  Chain 1 — API authentication
  ────────────────────────────
  ADR-0003 (cookies, Superseded)
       │ superseded by
       ▼
  ADR-0007 (JWT, Accepted) ──related──► ADR-0008 (key rotation, Draft)


  Chain 2 — Internal access
  ─────────────────────────
  ADR-0005 (VPN, Superseded) ──┐
                                │ superseded by
  ADR-0011 (IAP web, ──────────┴──► ADR-0014 (IAP web, Accepted)
            Superseded)                     │ follow-up
                                            ▼
                                       ADR-0015 (non-web, Draft)
```

## How to use them

- **Immutable once Accepted.** If the world changes, write a new ADR — don't
  edit the old one. The only field that may change on an Accepted ADR is the
  metadata header, typically the `Status:` line when something supersedes it.
- **One decision per ADR.** Small, focused, numbered. If you find yourself
  writing two decisions, split it.
- **Be honest about trade-offs.** The *Negative / trade-offs* section is the
  most valuable part of the document. If an ADR has no downsides it is
  probably hiding them.
- **Pick a format and stick to it.** There is no canonical ADR schema. See the
  existing files in [`docs/ADR/`](docs/ADR/) for the convention this repo
  uses (status, date, deciders, context, decision, options considered,
  consequences). Decide a shape with your team and apply it consistently —
  the consistency matters more than the specific shape.

## The ADRs in this repo

- [ADR-0003](docs/ADR/ADR-0003-server-side-session-cookies.md) — Server-side session cookies *(Superseded)*
- [ADR-0005](docs/ADR/ADR-0005-corporate-vpn-internal-apps.md) — Corporate VPN for internal apps *(Superseded)*
- [ADR-0007](docs/ADR/ADR-0007-api-authentication.md) — Stateless JWT access tokens *(Accepted)*
- [ADR-0008](docs/ADR/ADR-0008-jwt-signing-key-rotation.md) — JWT signing-key rotation *(Draft)*
- [ADR-0009](docs/ADR/ADR-0009-iap-tcp-forwarding-private-access.md) — IAP TCP forwarding for engineering access *(Accepted)*
- [ADR-0010](docs/ADR/ADR-0010-least-privilege-tunnel-scoping.md) — Least-privilege tunnel scoping *(Accepted)*
- [ADR-0011](docs/ADR/ADR-0011-iap-for-internal-web-access.md) — IAP for internal web access *(Superseded)*
- [ADR-0014](docs/ADR/ADR-0014-internal-website-access.md) — Replace VPN with IAP for internal websites *(Accepted)*
- [ADR-0015](docs/ADR/ADR-0015-non-web-resource-access.md) — Non-web internal resource access *(Draft)*
