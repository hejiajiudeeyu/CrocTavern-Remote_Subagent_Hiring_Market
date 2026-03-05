<div align="center">
  <img src="logo.jpg" alt="Croc Tavern Logo" width="180">
  <h1>Croc Tavern | Remote Subagent Hiring Market</h1>
  <p><em>A marketplace for AI capabilities as a service — you describe what you need, we find the best remote executor</em></p>
</div>

English | [中文](README.zh-CN.md)

---

## Live Demo

- [Ecosystem Page](https://hejiajiudeeyu.github.io/CrocTavern-Remote_Subagent_Hiring_Market/site/ecosystem.html)
- [Playground Demo](https://hejiajiudeeyu.github.io/CrocTavern-Remote_Subagent_Hiring_Market/site/playground.html)

## Community

- [Contributing Guide](CONTRIBUTING.md)
- [Security Policy](SECURITY.md)
- [Roadmap](ROADMAP.md)
- [Discord](https://discord.gg/e4nDukFAc6)

## The Problem

Today's AI skill marketplaces require buyers to install and run everything locally. This creates real friction:

- **Configuration overhead**: dozens of skills, each needing separate API keys, dependencies, and environment setup.
- **Context pollution**: skills consume tokens and pollute the main agent's prompt, degrading output quality.
- **No IP protection**: skill implementations are fully exposed — anyone can clone and republish them, leaving no incentive for maintainers.
- **Security risk**: malicious prompts in community skills and supply-chain vulnerabilities.

**Remote Subagent flips this model**: you describe what result you need, the platform finds a qualified remote executor, and you get back a structured result. No installation, no configuration, no prompt leakage.

## What Is a Remote Subagent?

A **Remote Subagent** is an AI capability unit deployed in the seller's environment and invoked through a standardized contract. Buyers only handle input/output — execution always happens on the seller side.

### Traditional Skills vs Remote Subagent

| Dimension | Traditional Local Skills | Remote Subagent |
| :--- | :--- | :--- |
| **Deployment** | Installed and configured on buyer side | Runs in seller environment, zero buyer setup |
| **IP Boundary** | Prompts/workflows fully exposed | Black-box isolation |
| **Quality Signal** | README claims | Real runtime metrics: success rate, latency, compliance |
| **Security** | Local supply-chain + runtime risk | Short-lived token + signed result + remote sandbox |
| **Integration** | Runtime-bound function call | JSON contract, transport-agnostic |

### Where It Fits in the Stack

Remote Subagent is not a replacement for MCP, nor a remote version of a Skill. It sits at a higher layer, packaging everything below:

| Layer | Concept | Role |
| :--- | :--- | :--- |
| L0 Infrastructure | LLM API / GPU / Private Data | Raw compute and knowledge supply |
| L1 Tool Protocol | MCP / Function Call | Standardized tool invocation — lets agents "use tools" |
| L2 Capability Pack | Skill (prompt + tool bundle) | Recipe for solving a specific problem with tools |
| L3 Executor | Agent / Subagent | Autonomous planner that invokes tools |
| **L4 Remote Service** | **Remote Subagent** | **Packaged L0–L3 delivery: black-box, contract-based, measurable, tradable** |

### vs MCP / Skill / SaaS

| Dimension | MCP | Skill | Enterprise SaaS | Remote Subagent |
| :--- | :--- | :--- | :--- | :--- |
| **Essence** | Tool protocol (L1) | Prompt + toolbox (L2) | Heavy-asset service | Tradable L0–L4 service unit |
| **Deployment** | Local or remote | Must install locally | Enterprise-grade | Seller-side, zero buyer cost |
| **IP Protection** | Depends on impl | Fully exposed | Closed source | Black-box isolation |
| **Quality Signal** | None | README | Enterprise SLA | Real call metrics |
| **Entry Barrier** | Developer | Developer | High (capital/team) | Low — anyone with standardizable expertise |

## Trust & Quality

"Black-box" does not mean "unverifiable." Trust is built on three layers:

- **Technical trust**: short-lived task tokens (bound to identity + budget + expiry), Ed25519 result signatures, `request_id`-based idempotency with 24h dedup window.
- **Data trust**: hard metrics (success rate, latency, schema compliance) drive ranking — not self-reported claims. Seller capability templates and call history serve as portfolio.
- **Commercial trust**: buyer ratings, automatic retry on failed validation, reserved DISPUTED state for arbitration (post-MVP).

## Open Source vs Commercial Boundary

The project follows a clear principle: **open collaboration at the protocol and reference implementation layer; commercial value at the operations, domain knowledge, and accountability layer.**

| Open Source (Apache 2.0) | Seller's Commercial Freedom | Platform Moat |
| :--- | :--- | :--- |
| Contract spec & JSON Schema | Internal models, workflows, prompts | Operations & community governance |
| Buyer/Seller reference implementations | Pricing strategy & service commitments | Ranking signal quality & evaluation |
| Evaluation tools | Accumulated domain knowledge & data | Risk control & dispute resolution |
| Platform control plane API | | Settlement network (future) |

## Target Audience

| Role | Profile |
| :--- | :--- |
| **Buyer** | Any orchestrator agent (or its human user) that needs external AI capabilities. You might be a Cursor/IDE user, or an agent hitting a sub-problem it can't solve alone. |
| **Seller** | Anyone who can standardize professional experience into an executable workflow. You might be an AI developer, a biomedical PhD (molecular experiment troubleshooting), a designer (PPT layout), or a lawyer (contract review). |

> **MVP Tech Bar**: familiarity with JSON, Email MCP basics, and API Key authentication.

## MVP Scope

Current MVP focuses on validating the transaction loop, not production-grade low latency.

- Transport: Email MCP
- Platform scope: minimal control plane only
- Auth mode: API Key (control plane)
- Buyer side: request orchestration + validation
- Seller side: validation + execution + signed result + idempotency

## Platform API (v0.1)

Task payload and result payload are exchanged via Email MCP.

| Method | Endpoint | Purpose |
| :--- | :--- | :--- |
| `GET` | `/v1/catalog/subagents` | Discover available subagents |
| `POST` | `/v1/catalog/subagents` | Register/update catalog item (MVP import-oriented) |
| `POST` | `/v1/tokens/task` | Request short-lived task token |
| `POST` | `/v1/tokens/introspect` | Validate token online (seller side) |
| `POST` | `/v1/metrics/events` | Submit minimal metrics events |
| `GET` | `/v1/metrics/summary` | Query aggregated metrics |
| `POST` | `/v1/requests/{request_id}/ack` | Seller ACK event |
| `GET` | `/v1/requests/{request_id}/events` | Poll request events |
| `POST` | `/v1/sellers/{seller_id}/heartbeat` | Seller heartbeat |

## Core Documents

- [Architecture Baseline](docs/architecture-mvp.md)
- [Platform API v0.1](docs/platform-api-v0.1.md)
- [Integration Playbook](docs/integration-playbook-mvp.md)
- [Data Collection](docs/data-collection-mvp.md)
- [Defaults v0.1](docs/defaults-v0.1.md)
- [Development Tracker](docs/development-tracker.md)

## Resource Templates

- [Single Catalog Template](docs/templates/catalog-subagent.template.json)
- [Batch Import Template](docs/templates/catalog-subagents.import.template.ndjson)
- Subagent templates: `docs/templates/subagents/{subagent_id}/`

## Project Status

Current phase: architecture + MVP implementation bootstrap.

The repository currently contains design-first artifacts and templates, with implementation work in progress.

## License

- [Apache License 2.0](LICENSE)
