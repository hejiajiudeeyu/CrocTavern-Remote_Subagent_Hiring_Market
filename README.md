<div align="center">
  <img src="logo.jpg" alt="Croc Tavern Logo" width="180">
  <h1>Croc Tavern | Remote Subagent Hiring Market</h1>
  <p><em>An open prototype marketplace for hiring Remote Subagents</em></p>
</div>

English | [中文](README.zh-CN.md)

---

Croc Tavern explores a new market model where the tradable unit is not a locally installed skill, but a **Remote Subagent**: an AI capability already deployed in the seller environment and invoked through a standardized contract.

Buyers only handle input/output contracts. Execution always happens on the seller side.

## Live Demo

- [Ecosystem Page](https://hejiajiudeeyu.github.io/CrocTavern-Subagent_Hiring_Market/ecosystem.html)
- [Playground Demo](https://hejiajiudeeyu.github.io/CrocTavern-Subagent_Hiring_Market/playground.html)

## Community

- [Contributing Guide](CONTRIBUTING.md)
- [Security Policy](SECURITY.md)
- [Roadmap](ROADMAP.md)
- [Discord](https://discord.gg/e4nDukFAc6)

## What Is a Remote Subagent?

A **Remote Subagent** is a remotely executed AI unit with contract-based interaction.

| Dimension | Traditional Local Skills | Remote Subagent |
| :--- | :--- | :--- |
| Deployment | Installed and configured on buyer side | Runs in seller environment |
| IP Boundary | Weights/prompts can leak to buyers | Black-box isolation |
| Quality Signal | README claims | Real runtime metrics |
| Security Boundary | Local supply-chain + runtime risk | Short-lived token + signed result |
| Integration | Runtime-bound function call | JSON contract, transport-agnostic |

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
