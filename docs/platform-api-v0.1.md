# Platform API v0.1（MVP）

本文档定义平台最小控制面 API：目录、token、请求事件、指标。  
MVP 传输通道仍为邮箱 MCP，任务/结果正文不经过平台转发。

## 1. 设计边界

- 平台职责：目录索引、授权签发、指标聚合。
- 非职责：任务正文代理转发、卖家执行编排、长期密钥托管。
- 版本策略：`/v1` 路径版本 + 字段向后兼容扩展。

## 2. 通用约定

## 2.1 Content-Type
- 请求与响应均使用 `application/json; charset=utf-8`

## 2.2 鉴权（v0.1 冻结）
- 统一使用 `API Key` 鉴权。
- 建议请求头：`Authorization: Bearer <API_KEY>`。
- 买家与卖家使用不同主体 key，服务端按 key 解析主体身份与权限范围。

## 2.3 时间与 ID
- 时间：ISO8601 UTC（如 `2026-03-02T12:00:00Z`）
- `request_id`：UUIDv7 推荐
- 分页：使用 `next_page_token`

## 2.4 通用错误响应

```json
{
  "error": {
    "code": "CONTRACT_SCHEMA_INVALID",
    "message": "output_schema is required",
    "retryable": false,
    "request_id": "018f9d5e-8bb2-7bc1-a4a3-1a8d9d8a2f41"
  }
}
```

错误码分域建议：
- `AUTH_*`
- `CONTRACT_*`
- `EXEC_*`
- `RESULT_*`
- `DELIVERY_*`
- `PLATFORM_*`

## 3. 目录 API

## 3.1 查询 subagents

- 方法：`GET /v1/catalog/subagents`
- 用途：买家检索可调用 subagent

Query 参数：
- `capability`（可选）
- `seller_id`（可选）
- `status`（可选，默认 `active`）
- `availability_status`（可选，`healthy|degraded|offline`）
- `page_size`（可选，默认 20，最大 100）
- `page_token`（可选）

200 响应示例：
```json
{
  "items": [
    {
      "subagent_id": "foxlab.text.classifier.v1",
      "seller_id": "seller_foxlab",
      "display_name": "FoxLab Text Classifier",
      "capabilities": ["classification", "customer_support"],
      "version": "1.0.0",
      "status": "active",
      "availability_status": "healthy",
      "last_heartbeat_at": "2026-03-02T11:59:50Z",
      "sla_hint": {
        "p95_exec_ms": 3500,
        "timeout_rate_7d": 0.02
      },
      "pricing_hint": {
        "currency": "USD",
        "per_request": 0.02
      },
      "eta_hint": {
        "queue_p50_s": 8,
        "exec_p50_s": 35,
        "exec_p95_s": 120,
        "sample_size_7d": 340,
        "updated_at": "2026-03-02T12:00:00Z"
      },
      "seller_public_key": "-----BEGIN PUBLIC KEY-----...",
      "template_ref": "docs/templates/subagents/foxlab.text.classifier.v1/"
    }
  ],
  "next_page_token": null
}
```

说明（可扩展性）：
- 当前建议优先使用遍历/分类过滤。
- 后续可新增 `GET /v1/catalog/search`，支持联想、模糊匹配与领域策略。
- 为保持兼容，`GET /v1/catalog/subagents` 长期保留，不因搜索增强而下线。

## 3.2 注册/更新 subagent（MVP 可手工导入）

- 方法：`POST /v1/catalog/subagents`
- 用途：新增或更新目录记录
- 说明：MVP 可不对外开放该 API，先采用线下手工导入模板。
- 说明：v0.1 中 seller 不自行维护 subagent 列表，平台在导入时完成 `seller_id -> subagent_id` 关联。

请求字段（Body）：
- `subagent_id`（必填）
- `seller_id`（必填）
- `display_name`（必填）
- `description`（可选）
- `capabilities`（必填，字符串数组）
- `version`（必填）
- `status`（必填，`active|inactive|blocked`）
- `contact.email`（可选）
- `endpoint_hint`（可选，仅元数据，不用于任务转发）
- `supported_task_types`（必填，字符串数组）
- `constraints.max_budget_cap`（可选）
- `constraints.max_hard_timeout_s`（可选）
- `eta_hint.queue_p50_s`（可选）
- `eta_hint.exec_p50_s`（可选）
- `eta_hint.exec_p95_s`（可选）
- `eta_hint.sample_size_7d`（可选）
- `eta_hint.updated_at`（可选）
- `seller_public_key`（必填）
- `template_ref`（可选，指向能力声明模板目录路径，如 `docs/templates/subagents/{subagent_id}/`）
- `updated_at`（可选）

201 响应示例：
```json
{
  "subagent_id": "foxlab.text.classifier.v1",
  "status": "active",
  "registered_at": "2026-03-02T12:00:00Z"
}
```

## 4. Token API

## 4.1 任务 token 签发

- 方法：`POST /v1/tokens/task`
- 用途：买家为单次任务申请短期授权

请求字段（Body）：
- `request_id`
- `buyer_id`
- `seller_id`
- `subagent_id`
- `budget_cap`
- `ttl_seconds`（建议 600-1200）

200 响应示例：
```json
{
  "task_token": "<JWT_OR_EQUIVALENT>",
  "token_type": "Bearer",
  "expires_at": "2026-03-02T12:05:00Z",
  "claims_preview": {
    "iss": "croc-platform",
    "aud": "seller_foxlab",
    "sub": "buyer_acme",
    "request_id": "018f9d5e-8bb2-7bc1-a4a3-1a8d9d8a2f41",
    "subagent_id": "foxlab.text.classifier.v1"
  }
}
```

## 4.2 token introspect（v0.1 必做）

- 方法：`POST /v1/tokens/introspect`
- 用途：卖家在线查询 token 是否有效（v0.1 统一校验模式）

请求字段（Body）：
- `task_token`

200 响应示例：
```json
{
  "active": true,
  "claims": {
    "iss": "croc-platform",
    "aud": "seller_foxlab",
    "sub": "buyer_acme",
    "request_id": "018f9d5e-8bb2-7bc1-a4a3-1a8d9d8a2f41",
    "subagent_id": "foxlab.text.classifier.v1",
    "exp": 1770005100
  }
}
```

## 5. Metrics API

## 5.1 事件上报

- 方法：`POST /v1/metrics/events`
- 用途：买家/卖家提交最小观测事件

请求字段（Body）：
- `source`：`buyer|seller`
- `event_type`：如 `request_succeeded|request_timeout|schema_invalid|signature_invalid`
- `request_id`
- `buyer_id`
- `seller_id`
- `subagent_id`
- `timestamp`
- `payload`（可选，扩展）

202 响应示例：
```json
{
  "accepted": true,
  "ingested_at": "2026-03-02T12:10:00Z"
}
```

## 5.2 聚合查询

- 方法：`GET /v1/metrics/summary`
- 用途：为榜单和评测展示提供聚合硬指标

Query 参数：
- `window`（如 `24h|7d|30d`）
- `seller_id`（可选）
- `subagent_id`（可选）
- `min_samples`（可选）

200 响应示例：
```json
{
  "window": "7d",
  "subagent_id": "foxlab.text.classifier.v1",
  "sample_size": 120,
  "call_volume": 120,
  "success_rate": 0.94,
  "timeout_rate": 0.03,
  "schema_compliance_rate": 0.98,
  "p95_exec_ms": 4100
}
```

## 6. Request Events API（ACK/状态事件）

该组接口用于长任务的轻量状态回传，避免买家无效等待。  
注意：只传事件摘要，不传任务正文与结果正文。
v0.1 实现范围：仅 `ACKED` 事件。

## 6.1 卖家 ACK（已接单）

- 方法：`POST /v1/requests/{request_id}/ack`
- 用途：卖家通过校验并开始处理后，快速确认“已接单”

Path 参数：
- `request_id`

请求字段（Body）：
- `seller_id`（必填）
- `subagent_id`（必填）
- `accepted_at`（必填）
- `ack_deadline_s`（可选，默认 120）
- `estimated_finish_at`（可选）
- `queue_position`（可选）
- `message`（可选，简短说明）

约束：
- 对同一 `request_id` 幂等。
- 平台校验调用方身份与 `seller_id` 一致。
- 可校验是否与已签发 token 的 `aud/subagent_id` 对齐。

200 响应示例：
```json
{
  "request_id": "018f9d5e-8bb2-7bc1-a4a3-1a8d9d8a2f41",
  "event_type": "ACKED",
  "accepted": true,
  "accepted_at": "2026-03-02T12:00:20Z"
}
```

## 6.2 卖家状态事件上报（后续，不在 v0.1）

- 方法：`POST /v1/requests/{request_id}/events`
- 用途：卖家上报 `RUNNING/PROGRESS/FAILED` 等轻量状态（后续扩展）

请求字段（Body）：
- `seller_id`
- `subagent_id`
- `event_type`（`RUNNING|PROGRESS|FAILED|COMPLETED`）
- `timestamp`
- `progress`（可选，0-100）
- `message`（可选）

202 响应示例：
```json
{
  "accepted": true,
  "ingested_at": "2026-03-02T12:01:00Z"
}
```

## 6.3 买家查询请求事件

- 方法：`GET /v1/requests/{request_id}/events`
- 用途：买家轮询 ACK/状态事件，减少盲等

Query 参数：
- `since`（可选，ISO8601 或游标）
- `limit`（可选，默认 50）

200 响应示例：
```json
{
  "request_id": "018f9d5e-8bb2-7bc1-a4a3-1a8d9d8a2f41",
  "events": [
    {
      "event_type": "ACKED",
      "seller_id": "seller_foxlab",
      "subagent_id": "foxlab.text.classifier.v1",
      "timestamp": "2026-03-02T12:00:20Z",
      "estimated_finish_at": "2026-03-02T12:01:10Z"
    }
  ],
  "next_cursor": null
}
```

## 7. Seller Heartbeat API

心跳用于反映卖家在线状态与基础负载，不替代单请求 ACK。

## 7.1 上报心跳

- 方法：`POST /v1/sellers/{seller_id}/heartbeat`
- 用途：卖家周期性上报在线状态

Path 参数：
- `seller_id`

请求字段（Body）：
- `seller_id`（必填，需与 path 一致）
- `timestamp`（必填）
- `agent_version`（可选）
- `queue_depth`（可选）
- `est_exec_p95_s`（可选）
- `active_subagents`（可选）

200 响应示例：
```json
{
  "accepted": true,
  "seller_id": "seller_foxlab",
  "received_at": "2026-03-02T12:00:30Z",
  "availability_status": "healthy"
}
```

## 7.2 可用性判定（建议默认）

- `heartbeat_interval_s = 30`
- `degraded_threshold_s = 90`
- `offline_threshold_s = 180`

状态规则：
- `healthy`：`now - last_heartbeat_at <= 90s`
- `degraded`：`90s < now - last_heartbeat_at <= 180s`
- `offline`：`now - last_heartbeat_at > 180s`

## 8. 手工导入目录模板

MVP 目录注册采用手工导入，模板文件见：
- `docs/templates/catalog-subagent.template.json`（单条模板）
- `docs/templates/catalog-subagents.import.template.ndjson`（批量模板）

能力声明模板（详见 `architecture-mvp.md` §4.5）：
- 每个 subagent 在 `docs/templates/subagents/{subagent_id}/` 下维护 `input.schema.json`、`output.schema.json`、`example-contract.json`、`example-result.json`、`README.md`。
- 目录条目通过 `template_ref` 字段指向对应目录，买家选定 subagent 后可拉取模板了解输入输出要求。

建议导入流程：
1. seller 先完成主体注册（获取 `seller_id` 和 API Key）。
2. seller 通过表单提交 subagent 信息与能力说明。
3. 平台管理员在模板中填写/修订 `subagent_id/seller_id/capabilities/supported_task_types` 并建立关联。
4. 使用 CLI 执行校验与审核导入。
5. 平台记录导入批次号与审计信息。
6. 导入后通过 `GET /v1/catalog/subagents` 抽样核对。

## 9. 检索增强预留（后续规划，不在 v0.1 实现）

为支持搜索引擎式能力，建议预留以下查询参数/字段：
- `q`：自由文本查询
- `fuzzy`：模糊匹配开关
- `domain`：领域策略（如 `legal|finance|biomed`）
- `ranking_profile`：排序策略标识

建议预留以下响应字段：
- `score`：综合相关性分
- `match_reasons`：命中原因（关键词、同义词、标签）
- `score_breakdown`：分项得分（文本匹配、质量、可用性、成本）

兼容原则：
- 新字段仅追加，不破坏旧字段语义。
- 新参数默认关闭，不影响现有 buyer 行为。
