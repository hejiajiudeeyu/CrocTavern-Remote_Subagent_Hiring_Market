<div align="center">
  <img src="logo.jpg" alt="Croc Tavern Logo" width="180">
  <h1>Croc Tavern | Remote Subagent Hiring Market</h1>
  <p><em>一个面向 Remote Subagent 的雇佣市场原型</em></p>
</div>

---

**核心思想**：市场交易对象不是本地可安装的 skills，而是 **Remote Subagent** —— 已部署在卖家环境中、通过标准化合约接口远程调用的 AI 能力单元。买家只处理输入输出，执行始终发生在卖家侧环境。

**在线演示**：
- [Ecosystem 介绍](https://hejiajiudeeyu.github.io/CrocTavern-Subagent_Hiring_Market/ecosystem.html)
- [Playground 演示](https://hejiajiudeeyu.github.io/CrocTavern-Subagent_Hiring_Market/playground.html)

## 什么是 Remote Subagent？(Core Concept)

**Remote Subagent** 是本项目定义的核心交易对象：一个已部署在卖家环境中、通过结构化合约接口远程调用的 AI 能力单元。

| 维度 | 传统本地 Skills | Remote Subagent |
| :--- | :--- | :--- |
| **部署位置** | 下载到买家本地，手动安装与配置 | 始终运行在卖家环境，买家零部署 |
| **知识产权** | 模型权重、Prompt 完全暴露 | 黑盒隔离 —— 买家看不到模型，卖家看不到原始数据 |
| **质量评估** | 依赖 README 自述 | 真实调用硬指标：成功率、延迟、合规率 |
| **安全边界** | 供应链攻击风险，本地执行无隔离 | 远程沙箱执行 + 短期 Token + 结果签名 |
| **交互方式** | 函数调用，强依赖本地运行时 | 结构化 JSON 合约驱动，传输通道可替换 |

**五大核心特征**：
- **远程执行**：买家只处理输入输出，无需本地安装或配置
- **合约驱动**：交互通过结构化 JSON 合约进行，输入输出 schema 预定义
- **黑盒隔离**：买家看不到模型实现，卖家看不到原始数据上下文
- **可度量**：真实调用产生硬指标（成功率、延迟、合规率），支撑评测与排序
- **传输无关**：通过 TransportAdapter 抽象，不绑定具体传输通道

## 为什么孵化这个项目？(Why This Project)

传统技能分发模式在真实使用中常遇到以下痛点：
- **质量无法保证**：skills 质量稳定性差。
- **配置繁琐**：本地安装与依赖配置极其复杂。
- **安全隐患**：供应链与执行环境存在较高的安全风险。

本项目提出 **Remote Subagent** 模式 —— 通过远程执行与合约交互，大幅降低上述成本，并为后续的评测、排序和商业化打下坚实基础。

## 目标受众 (Target Audience - MVP)

| 角色 | 画像 |
| :--- | :--- |
| **买家 (Buyer)** | 使用 Cursor / IDE 等工具的 AI 应用开发者。需要调用外部 AI 能力（分类、提取、生成等），但不想管理模型部署和复杂依赖。 |
| **卖家 (Seller)** | 已有可运行 AI 能力（模型、Pipeline、工具链）的独立开发者或小团队。希望将自身能力标准化，并对外提供服务。 |

> **MVP 技术门槛**：熟悉 JSON 数据格式、邮箱 MCP 基础操作、API Key 认证即可快速接入！

## 项目状态 (Project Status)

当前阶段：**架构设计 / MVP 构思阶段**  
目前仓库以设计文档为主，尚未进入完整工程实现。

## MVP 边界与范围 (MVP Scope)

MVP 的核心目标是 **验证交易闭环**，而非追求低延迟的生产可用性：
- **传输通道**：邮箱 MCP（低接入成本，适配面广）
- **平台控制面**：平台只做最小控制面，包括目录、Token 和指标。
- **控制面鉴权**：初期采用 API Key 方式。
- **买家侧主控 (Buyer Skill)**：负责任务拆分、合约发起、结果验收。
- **卖家侧模板 (Seller Template)**：负责校验、执行、结果回传与幂等处理。

## 目录检索策略 (Catalog Retrieval Strategy)

**当前阶段（Skills 数量较少时的策略）**：
- 优先采用 **"遍历 + 分类过滤"**（按 `capability/task_type` 过滤）。
- 暂不做复杂的检索能力开发（无联想、模糊搜索或复杂的查询排序）。

**后续增强方向（已在架构中预留）**：
- 联想检索 (Query Suggestion)
- 模糊搜索 (拼写容错 / 近义词)
- 细分领域策略 (不同领域使用不同的排序配置)

**兼容原则**：
- 保留基础目录接口，检索增强能力将以 **新增参数** 或 **新端点** 方式平滑演进。
- Buyer 侧仅消费候选结果，不与底层的具体检索实现耦合。

---

## 核心角色与职责 (Roles and Responsibilities)

### 买家端 (Buyer Controller Skill)
**核心职责**："发起任务并验收结果"，不负责卖家真实的执行环境。
- **任务拆分**：将上层目标拆解为可执行的子任务。
- **合约生成**：构造标准任务合约（含 `request_id`、输入参数、约束条件、`output_schema`）。
- **目录检索**：从平台目录中选取合适的卖家 subagent。
- **Token 申请**：向平台申请短期 task token。
- **发信投递**：通过邮箱 MCP 发送任务邮件。
- **结果轮询**：按 `request_id` 拉取同线程的执行结果。
- **最小验收**：执行 Schema 校验、签名验签及最小业务规则校验。
- **状态管理**：维护 `request_id` 状态机，并处理重试、超时、去重。
- **指标上报**：上报买家观测指标（成功率、端到端耗时、验签失败率等）。

### 卖家端 (Seller Agent Template)
**核心职责**："安全执行并返回可验证结果"。
- **收信解析**：从邮箱 MCP 拉取任务，解析收到的任务合约。
- **入站校验**：校验字段完整性、版本兼容性及输入合法性。
- **Token 校验**：验证 `aud/sub/request_id/subagent_id/exp` 等 Claims 信息。
- **护栏检查**：进行预算、超时及策略约束检查（有权拒绝不合规任务）。
- **幂等去重**：按 `request_id` 去重；遇到重复请求则直接回放历史结果。
- **执行调用**：调用本地或远端执行函数，完成具体的子任务。
- **结果封装**：按标准结果包格式返回 `status/output/error/timing/usage`。
- **结果签名**：对最终结果包签名，以供买家验签保证防篡改。
- **回信发送**：在同邮件线程回复结果。
- **指标上报**：上报执行侧最小观测指标（成功/超时/Schema 失败/P95耗时等）。

### 平台服务端 (Platform Minimal Service)
**核心职责**：提供"控制面能力"，绝对 **不转发任务正文**，也 **不托管长期密钥**。
- **目录服务 (Catalog)**
  - Subagent 注册（MVP 阶段支持手工导入，采用表单提交+CLI审核的流程）。
  - Subagent 查询（基于能力、标签、版本、卖家进行筛选）。
  - 分发可用性状态（`healthy / degraded / offline`）。
  - 分发卖家公钥与元数据（用于后续的结果验签）。
- **Token 服务 (Token)**
  - 签发短期 task token（绑定 `request_id / buyer_id / seller_id / subagent_id / budget / exp`）。
  - 提供在线 token 校验接口（供卖家统一通过 introspect 校验）。
- **请求事件服务 (Events)**
  - 接收卖家 ACK 事件（确认已接单）。
  - 提供按 `request_id` 的事件查询接口（供买家轮询，避免无效死等）。
  - *预留：轻量级的任务进度事件同步*。
- **卖家心跳服务 (Heartbeat)**
  - 接收 Seller Heartbeat，计算并在目录中更新其实时在线状态，辅助买家选路。
- **指标服务 (Metrics)**
  - 接收双端（买家与卖家）最小维度的指标事件上报。
  - 数据聚合，并提供基础评测展示接口（调用量、成功率、超时率、Schema合规率）。
- **基础治理能力 (Governance)**
  - 最小化限流控制及基础风控钩子。
  - 管理合约/结果包的版本升级与兼容窗口。
  - 处理主体的注册流程和 API Key 发放。

---

## 平台核心 API 概览 (Platform API Overview - MVP)

> 以下为控制面的最小接口集合。注意：任务指令正文与执行结果正文完全通过 **邮箱 MCP** 传输，不经过平台转发。详见 [docs/platform-api-v0.1.md](docs/platform-api-v0.1.md)。

| 方法 | 路径 | 核心用途 |
| :--- | :--- | :--- |
| `GET` | `/v1/catalog/subagents` | 买家检索可用 subagent |
| `POST` | `/v1/catalog/subagents` | 注册/更新目录项（MVP 阶段倾向手工导入） |
| `POST` | `/v1/tokens/task` | 买家申请短期授权任务 Token |
| `POST` | `/v1/tokens/introspect` | 卖家在线查验 Token 合法性 |
| `POST` | `/v1/metrics/events` | 买卖双方上报各自的可观测指标事件 |
| `GET` | `/v1/metrics/summary` | 查询聚合后的硬指标（用于榜单生成） |
| `POST` | `/v1/requests/{request_id}/ack` | 卖家上报 ACK 事件（声明已接单） |
| `GET` | `/v1/requests/{request_id}/events` | 买家轮询请求的最新事件状态 |
| `POST` | `/v1/sellers/{seller_id}/heartbeat` | 卖家上报探活心跳 |

## 核心交互时序图 (Core Interaction Flow)

1. **合约生成**：买家主控 Skill 生成包含 `request_id`、约束和 Token 的任务合约。
2. **服务检索**：买家在目录中寻找并选择适配的目标卖家 Subagent。
3. **任务派发**：通过邮箱 MCP 发送任务邮件。
4. **安全校验**：卖家读取邮件，校验合约、Token 以及自身的护栏策略。
5. **确认接单**：卖家通过服务端发送 ACK（声明已接单，并可附带预期 ETA）。
6. **执行与回传**：卖家执行完毕后，回传结构化的结果包（同邮件线程回复）。
7. **结果验收**：买家轮询请求事件+邮箱收件箱，进行 Schema 验证及基础验收校验。
8. **指标上报**：买卖双方各自向平台上报最小指标以供后续的评测展示。

> **备注**：卖家在全生命周期内会周期性上报 Heartbeat；买家选路时，可优先路由至 `healthy` 状态的卖家。

## 安全模型 (Security Model - MVP)

- 项目**绝不**使用长期的明文密钥做任务级授权。
- 全面采用**短期 Task Token**（强绑定 `request_id` / 买卖双方身份 / 预算额度 / 过期时间）。
- 卖家必须严格基于 `request_id` 实施**幂等去重**，杜绝重复执行引发的侧漏或资损。
- 收到任务后，卖家需尽快发出 ACK 事件；买家逾期未收到 ACK（`ack_deadline`）则可止损并判定超时。
- 结果包建议**强制签名**认证，买家验签无误后才纳入自身的成功统计体系。
- 平台始终恪守"通道与身份"原则：**不转发业务正文、不托管任何长期密钥**。

## 计费与定价 (Pricing Model - MVP)

- **v0.1 版本为免费试用期**，不涉及任何真实世界的资金清结算流程。
- 合约中的 `pricing_hint` 和 `budget_cap` 字段目前**仅作为参考提示**，不触发实际计费流程。
- `cost_estimate` （卖家自报成本估算）将作为后续正式定价的参考值，平台此刻**不做对账与结算**。
- 商业化计费、分账及抽成方案将在 MVP 跑通后进行专项设计。

## MVP 拆解模块 (MVP Modules)

1. 任务合约与结果包数据规范
2. 买家侧主控 Skill 工程原型
3. 卖家侧接入模板工程原型
4. 平台最小控制面服务（目录 / Token / 事件追踪 / 指标 / 基础治理）
5. 最小评测与榜单模块（统计调用量、成功率、超时率、Schema合规率）

## 评测与验收方案 (Evaluation Plan)

- **通道 A**：影子评测 (Shadow Evaluation, 后置离线评测)
- **通道 B**：基准评测 (Benchmarking, 在 MVP 中后期接入，先选取每类 10-20 道公开测试题)
- **通道 C**：真实链路验收贡献度 (后置评测)
- **人工评测**：作为"竞技场"形式的补充，主要用于项目传播与主观感受校准，**不主导核心排序系统的打分**。

---

## 详细架构文档 (Architecture Document)

MVP 架构的深入设计与基线定义请查阅以下文档：
- [docs/architecture-mvp.md](docs/architecture-mvp.md) - 系统架构基线
- [docs/data-collection-mvp.md](docs/data-collection-mvp.md) - 数据采集与指标定义
- [docs/defaults-v0.1.md](docs/defaults-v0.1.md) - v0.1 默认配置说明
- [docs/platform-api-v0.1.md](docs/platform-api-v0.1.md) - API 接口规范
- [docs/integration-playbook-mvp.md](docs/integration-playbook-mvp.md) - 接入指南手册
- [docs/development-tracker.md](docs/development-tracker.md) - 研发进度追踪

**这些文档涵盖了**：
- 完整的系统边界与信任安全模型
- 合约及结果包的具体字段定义与 JSON 参考实例
- 全局状态机、错误码归类、重试策略及幂等语义抽象
- 分层嵌套的超时控制模型及监控指标口径
- `TransportAdapter` 传输层抽象设计（为未来从邮箱通道零成本切换至 HTTP API / Webhook 提前铺路）

**资源包导入模板**：
- [docs/templates/catalog-subagent.template.json](docs/templates/catalog-subagent.template.json) - 单条记录模板
- [docs/templates/catalog-subagents.import.template.ndjson](docs/templates/catalog-subagents.import.template.ndjson) - 批量导入模板

**交互式演示页面**：
- [playground.html](playground.html) - 分步演示 Buyer / Seller / Platform 操作链路并预填示例

## 代码库结构 (Repository Structure)

```text
.
├── docs/
│   ├── architecture-mvp.md
│   ├── data-collection-mvp.md
│   ├── defaults-v0.1.md
│   ├── development-tracker.md
│   ├── integration-playbook-mvp.md
│   ├── platform-api-v0.1.md
│   ├── issues/                          # 开发决策记录
│   │   └── playground-review-issues-2026-03-03.md
│   └── templates/
│       ├── catalog-subagent.template.json
│       └── catalog-subagents.import.template.ndjson
├── ecosystem.html
├── playground.html
├── logo.jpg
├── LICENSE
└── README.md
```

## 发展路线图 (Roadmap)

- [ ] **Milestone 1**: 冻结 v0.1 版本的跨端合约规范及统一流转状态机
- [ ] **Milestone 2**: 编码完成平台最小服务 (Platform Minimal Service) 的原型
- [ ] **Milestone 3**: 完成一套首尾呼应的 买方/卖方 原型，并全自动化跑通 4 类核心的 E2E 测试用例
- [ ] **Milestone 4**: 串联上线最小维度的评测看板和基础排序榜单
- [ ] **Milestone 5**: 启动新通道技术评估，准备将底层通道从纯邮箱平滑演进到更具实时性的 API / Webhook 方案

## 开源协议与声明 (Open Source & License)

- **许可协议**：[Apache License 2.0](LICENSE) 
- **拥抱开源**：项目的 核心合约规范、双端参考实现框架、基础验证评测工具 均首选完全开源！
- **平台护城河**：我们将发力构建深度运营体系、高质量的排序信号源、坚如磐石的风控系统及未来的清结算网络。

## 参与贡献 (Contribution)

Croc Tavern 市场原型处于高度活跃的设计阶段，我们热忱欢迎在以下方向进行深度的探讨：
- 合约字段定义及其 Schema 的演进策略
- Token鉴权、防篡改签名机制与各种弱网环境下的幂等语义设计
- 适用于不同开发语言生态的买家/卖家接入模板接口封装设计
- 新型指标口径的定义及如何做到最小可解释的打分和排序算法

如需提交变更建议或参与讨论，**请随时向我们发起 Issue 或直接提交 PR**！（官方贡献模板即将上线）。

## 常见问题解答 (FAQ)

**Q: 为什么 MVP 版本采用「邮箱」而非传统的 HTTP API 去传输任务？**
- **零端点暴漏**：邮箱 MCP 已有极其广泛的生态支持，卖家全程无需额外部署服务器或将自身端点暴露在公网上，接入成本趋近于零。
- **天然异步特质**：AI 分析和生成任务耗时通常在数秒到数十分钟不等，传统的 HTTP 同步保持极易超时崩盘，而邮件队列天然匹配这种纯异步调用。
- **架构已做抽象解耦**：我们在底层设计了无状态的 `TransportAdapter` 接口，剥离了通信协议，后续切换或者增加 HTTP / Webhook 通道易如反掌。

**Q: 邮箱通道作为主力干道，性能够用吗？**
- **聚焦验证闭环**：MVP 的首要法则是验证"这套商业模式的供需闭环"，暂不追求企业级的高并发和超低延迟毫秒级生产应用。
- **非实时场景极佳**：邮箱通道极度契合大规模批处理 (batch-style)、异步回调、或长周期的复杂推理型 AI 任务系统。
- **针对性调优**：我们已基于实际的邮件投递物理延迟基线设计了合理的超时参数与回退机制（例如：`ack_deadline_s=120` 已经涵盖了极限网络下的邮件流转预算）。

**Q: 接入 MVP 阶段需要真金白银地付费吗？**
- **完全免费接入**：v0.1 为纯粹的**免费对接和试跑期**，现阶段绝对不会发生真实资金的清结或充值。
- **前瞻性占坑设计**：目前所有合约请求与返回负载中的 `pricing_hint` 与 `budget_cap` 等涉费字段，仅作为"未来账单"的一场预演，绝不触发实际扣费。
