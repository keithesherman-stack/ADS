# Agent Deployment Standard (ADS)

**An open specification for responsibly deploying AI agents into operational business workflows.**

Published by [SAIL Institute](https://sailinstitute.org) · Version 0.1 · Working Draft · May 2026

---

## Status

ADS v0.1 is a **working draft**. It is published openly to invite public review, critique, and contribution from operators, builders, and buyers actively deploying AI agents in production environments. The specification will evolve as field experience accumulates. Breaking changes are expected between v0.x releases. A stable v1.0 is targeted for late 2026.

If you operate agents in production and have deployment patterns, failure modes, or implementation experience to contribute, open an issue or pull request.

---

## Why ADS Exists

The conversation about AI agents has been dominated by two questions: *what can agents do?* and *how do we identify them?* Both matter. Neither addresses the operational reality that determines whether agent deployments succeed or fail.

The harder question — the one operators face the moment an agent talks to a real customer, books a real appointment, or sends a real invoice — is **how is the agent deployed responsibly into the business it serves?**

This is a different question from capability. An agent can be capable of booking calendar appointments and still cause damage if it books outside business hours, double-books known customers, or fails to escalate a medical emergency to a human. Deployment is not capability. Deployment is the discipline of placing a capable agent into a real workflow with the right scope, the right handoff protocols, the right escalation paths, and the right audit trail.

There is currently no shared standard for this discipline. Every deployment is a private trust exercise between the vendor, the buyer, and hope. ADS exists to change that.

---

## Scope

ADS is concerned with the **deployment layer** — the operational contract between an agent and the business workflow it participates in. It is intentionally narrower than identity, credentialing, or capability standards.

**In scope:**

- How an agent declares what it is authorized to do
- How upstream systems hand context to an agent
- How an agent escalates to humans
- How an agent records what it did
- How known entities (returning customers, recurring contexts) are recognized
- How third parties verify what happened

**Out of scope:**

- Agent identity and authentication (see W3C DIDs, Verifiable Credentials)
- Agent capability discovery (see MCP, OpenAPI for agents)
- Model selection, training, or evaluation

ADS assumes the agent exists, is identified, and is capable. ADS specifies how it is *deployed*.

---

## The Six Pillars

ADS v0.1 defines six pillars of responsible agent deployment. A deployment is ADS-conformant when all six are implemented and verifiable.

### 1. Scope Declaration

Before deployment, every agent must publish a machine-readable declaration that defines:

- **Authorized actions** — the specific actions the agent may perform on behalf of the business
- **Forbidden actions** — actions the agent must never perform, regardless of user request
- **Human-approval-required actions** — actions the agent may propose but not execute without explicit human confirmation
- **Operating context** — the business, time windows, jurisdictions, and conditions under which the agent operates

The Scope Declaration is the agent's deployment contract. It is authored by the deploying business (or its representative), not by the agent vendor. It is versioned, signed, and accessible to auditors.

A scope declaration is not a system prompt. System prompts are runtime instructions that can be overridden, ignored, or jailbroken. A scope declaration is a deployment artifact enforced by the runtime — actions outside scope are rejected by the platform, not refused by the model.

### 2. Context Handoff Protocol

When upstream systems (phone networks, CRMs, calendars, ticketing platforms, websites) invoke an agent, they must hand the agent structured context describing:

- **What triggered this interaction** (inbound call, scheduled task, customer-initiated chat, internal escalation)
- **Who the counterparty is**, to the extent known (caller ID, authenticated user, recognized email)
- **What history exists** that the agent should be aware of (prior interactions, open tickets, recent transactions)
- **What constraints apply** to this specific interaction (time-sensitive, regulated, VIP, internal-only)

The context handoff must be authenticated. Unauthenticated context is untrusted context. ADS specifies the minimum authentication primitives (shared-secret headers, HMAC-signed payloads, mutually-authenticated TLS) acceptable for handoff.

The agent must acknowledge receipt of the context and may reject the interaction if context is malformed, expired, or insufficient for the declared scope.

### 3. Escalation Triggers

Every ADS-conformant deployment defines a taxonomy of conditions under which the agent must hand off to a human. These conditions are not optional. The runtime — not the model — enforces them.

ADS v0.1 defines five categories of escalation triggers:

- **Emergency triggers** — keywords, intents, or signals that indicate safety risk (medical, legal, physical danger). Detection requires immediate handoff regardless of conversational state.
- **Scope-boundary triggers** — explicit user requests for actions outside the agent's Scope Declaration. The agent must not refuse silently; it must declare the boundary and route to a human.
- **Confidence triggers** — runtime indicators that the agent's response confidence has fallen below a deployment-defined threshold.
- **Explicit-request triggers** — user requests to speak to a human, regardless of context. Always honored.
- **Repetition triggers** — patterns indicating the conversation is not progressing (the same user question asked multiple times, the same agent response given multiple times).

Each trigger has a defined **handoff destination** (a phone number, a queue, a notification target) and a defined **handoff payload** (what context is transferred to the human).

### 4. Completion Records

Every agent interaction produces a structured Completion Record. The record is generated by the runtime, not the agent, and is cryptographically signed. It contains:

- **Interaction metadata** — start time, end time, duration, channel, agent version, scope declaration version
- **Intent summary** — what the counterparty appeared to want (model-generated)
- **Actions taken** — every action the agent invoked, with timestamps and outcomes
- **Actions refused** — every action the agent declined or could not perform, with reason
- **Escalations** — every escalation trigger that fired, with destination and acknowledgment
- **Outcome classification** — success, partial completion, failure, or handoff
- **Handoff context** — if escalated, what was handed to the human and when

Completion Records are the unit of accountability. They are queryable, exportable, and verifiable without trusting the agent vendor.

### 5. Known-Entity Recognition

When agents interact with returning counterparties — repeat customers, recurring callers, known vendors — they must:

- **Recognize the entity** through deterministic identifiers (phone number, authenticated identity, account ID), not inference
- **Disclose recognition** to the counterparty when relevant ("I see you called us last week about...")
- **Apply context appropriately** — surface relevant history, respect prior decisions, avoid asking previously-answered questions
- **Log the recognition decision** in the Completion Record, including what identifier matched and what context was retrieved

Recognition without disclosure damages trust. Recognition without logging damages accountability. Both are non-negotiable.

ADS does not specify *how* known entities are stored — that is an implementation choice. ADS specifies that recognition decisions are auditable and that recognition events appear in the Completion Record.

### 6. Verifiable Audit Trail

The combination of Scope Declaration, Context Handoff records, Completion Records, and Escalation logs constitutes the Verifiable Audit Trail.

The audit trail must be:

- **Complete** — no agent interaction occurs outside the trail
- **Queryable** — by business owner, auditor, regulator, or counterparty (with appropriate authorization)
- **Tamper-evident** — cryptographic signatures or append-only logs ensure records cannot be silently modified
- **Portable** — exportable in a standard format (ADS specifies JSON Lines as the baseline)
- **Retained** — for a minimum period defined by the deployment's regulatory context

The audit trail is what allows a business to answer the question "what did the agent actually do?" without depending on the agent vendor's good faith.

---

## Conformance

A deployment is ADS v0.1 conformant when:

1. A signed Scope Declaration is published and accessible
2. All upstream invocations use authenticated Context Handoff
3. Escalation Triggers are defined for all five required categories
4. Every interaction produces a signed Completion Record
5. Known-Entity Recognition events are logged when applicable
6. The Verifiable Audit Trail meets all five baseline properties

ADS does not currently certify conformance. Self-attestation is the v0.1 baseline. A formal conformance test suite is planned for v1.0.

---

## Relationship to Other Standards

ADS is designed to compose with, not replace, related standards.

- **Agent identity** is the concern of W3C Decentralized Identifiers (DIDs) and Verifiable Credentials. ADS assumes agents have stable identities; it does not specify how.
- **Capability discovery** is the concern of Anthropic's Model Context Protocol (MCP) and similar specifications. ADS assumes agents have declared capabilities; it specifies how those capabilities are scoped at deployment.

A mature agent deployment composes these concerns. ADS is concerned only with the deployment layer between them.

---

## Authorship

ADS is authored by **Keith Sherman**, founder of SAIL Institute, drawing on direct experience deploying voice and operational AI agents into trades businesses and other operational SMBs. The specification is informed by — and pressure-tested against — production deployments in regulated and customer-facing contexts.

The specification is openly developed. Contributions, critiques, and implementation reports from the agent operator community are welcomed.

---

## License

The Agent Deployment Standard specification and documentation are published under the [Apache 2.0 License](LICENSE). Implementations may be open or proprietary; conformance is determined by behavior, not licensing.

---

## Roadmap

- **v0.1 — May 2026** — Working draft, six pillars defined, open for comment *(current)*
- **v0.2 — Q3 2026** — JSON schemas for Scope Declaration and Completion Record
- **v0.3 — Q4 2026** — Reference implementation patterns, escalation taxonomy refinements
- **v1.0 — Late 2026** — Stable release, conformance test suite, formal review window

---

## Contributing

Schema and specification changes are proposed via pull request to this repository. All proposals require:

1. A description of the deployment problem being addressed
2. The proposed specification change in full
3. At least one production deployment context where the change is motivated by real experience
4. Backward compatibility analysis

Issues are welcomed for questions, deployment reports, and discussion. The specification is intended to be shaped by the people doing the work.

---

## Contact

- **Maintainer:** Keith Sherman ([keith@sailinstitute.org](mailto:keith@sailinstitute.org))
- **Institute:** [sailinstitute.org](https://sailinstitute.org)
- **Repository:** This GitHub repository
- **Issues:** Use GitHub Issues on this repository

---

*Agent Deployment Standard v0.1 · © 2026 SAIL Institute · Published under Apache 2.0*
