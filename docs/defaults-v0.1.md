# Defaults v0.1（建议冻结参数）

用途：在编码前一次性冻结关键参数，避免买家端、卖家端、服务端实现分叉。

状态说明：
- `FROZEN`：已确认并冻结（v0.1）

更新时间：2026-03-02

## 1) 请求与超时

| 参数 | 建议值 | 状态 | 说明 |
|---|---:|---|---|
| `ack_deadline_s` | `120` | FROZEN | 买家发单后等待 ACK 的最大时长（含邮件投递延迟） |
| `soft_timeout_s` | `90` | FROZEN | 软超时，触发告警或降级 |
| `hard_timeout_s` | `300` | FROZEN | 硬超时，买家终止等待并记超时 |
| `max_retry_attempts` | `2` | FROZEN | 最大重试次数（总尝试数=3） |
| `retry_backoff` | `exponential + jitter` | FROZEN | 重试退避策略 |
| `email_delivery_budget_s` | `60` | FROZEN | 邮件投递预算，用于超时分层计算 |

## 2) Token 与安全

| 参数 | 建议值 | 状态 | 说明 |
|---|---:|---|---|
| `token_ttl_seconds` | `900` | FROZEN | 任务 token 有效期（15 分钟，覆盖邮件投递延迟） |
| `token_min_ttl_seconds` | `600` | FROZEN | 最短建议有效期 |
| `token_max_ttl_seconds` | `1200` | FROZEN | 最长建议有效期 |
| `result_signature_algorithm` | `Ed25519` | FROZEN | 结果包签名算法 |
| `seller_token_validation_mode` | `online_introspect_required` | FROZEN | 卖家校验 token 统一走 `POST /v1/tokens/introspect` |
| `idempotency_window_hours` | `24` | FROZEN | `request_id` 去重窗口 |
| `introspect_sla_p99_ms` | `200` | FROZEN | introspect 接口 P99 延迟目标 |
| `introspect_cache_ttl_s` | `30` | FROZEN | introspect 结果缓存 TTL |

## 3) 心跳与可用性

| 参数 | 建议值 | 状态 | 说明 |
|---|---:|---|---|
| `heartbeat_interval_s` | `30` | FROZEN | 卖家心跳上报间隔 |
| `degraded_threshold_s` | `90` | FROZEN | 超过该值进入 `degraded` |
| `offline_threshold_s` | `180` | FROZEN | 超过该值进入 `offline` |
| `catalog_health_cache_ttl_s` | `60` | FROZEN | 买家读取健康状态缓存 TTL |

## 4) 目录与路由

| 参数 | 建议值 | 状态 | 说明 |
|---|---:|---|---|
| `catalog_cache_ttl_s` | `300` | FROZEN | 买家目录缓存 TTL |
| `catalog_default_status_filter` | `active` | FROZEN | 默认过滤激活条目 |
| `catalog_default_availability_filter` | `healthy` | FROZEN | 默认只选健康卖家 |
| `routing_fallback_policy` | `retry_once_then_switch_seller` | FROZEN | ACK 超时后路由策略 |
| `catalog_import_mode` | `on_demand_immediate` | FROZEN | 目录按需即时导入 |
| `seller_subagent_binding_mode` | `platform_import_association` | FROZEN | subagent 与 seller 关系由平台导入时建立 |

## 5) 指标与展示

| 参数 | 建议值 | 状态 | 说明 |
|---|---:|---|---|
| `metrics_windows` | `24h,7d` | FROZEN | 默认指标窗口 |
| `ranking_min_samples` | `30` | FROZEN | 排序最小样本量门槛 |
| `mvp_display_metrics` | `call_volume,success_rate,timeout_rate,schema_compliance_rate,p95_exec_ms` | FROZEN | MVP 对外展示硬指标 |
| `buyer_event_required` | `request_sent,request_acked,request_succeeded,request_timeout,result_schema_invalid,result_signature_invalid` | FROZEN | 买家最小事件集 |
| `seller_event_required` | `request_received,request_accepted,request_succeeded,request_timeout` | FROZEN | 卖家最小事件集 |

## 6) 定价与结算

| 参数 | 建议值 | 状态 | 说明 |
|---|---:|---|---|
| `pricing_mode` | `reference_only` | FROZEN | pricing 字段仅作参考提示，不触发计费 |
| `settlement_enabled` | `false` | FROZEN | MVP 不启用结算 |

## 7) 版本与兼容

| 参数 | 建议值 | 状态 | 说明 |
|---|---:|---|---|
| `contract_version` | `0.1.0` | FROZEN | 合约版本 |
| `result_version` | `0.1.0` | FROZEN | 结果包版本 |
| `api_version_prefix` | `/v1` | FROZEN | 控制面 API 路径版本 |
| `compat_policy` | `additive-only` | FROZEN | 仅追加字段，不破坏旧语义 |
| `request_event_scope_v0_1` | `ACKED_only` | FROZEN | v0.1 仅实现 ACK 事件，不实现进度事件 |
| `platform_storage_backend` | `PostgreSQL` | FROZEN | 服务端主存储选型 |
| `api_auth_mode` | `api_key` | FROZEN | 控制面 API 鉴权方式 |
| `identity_onboarding_mode` | `pre_register_then_issue_key` | FROZEN | 买卖双方先注册，再发 API Key |
| `catalog_submission_mode` | `form_submit_then_cli_review_import` | FROZEN | 表单收集后由 CLI 审核导入 |

## 8) 核心参数确认结果

以下 8 项已确认并冻结（可直接进入实现）：
1. `ack_deadline_s=120`
2. `token_ttl_seconds=900`
3. `soft_timeout_s=90`, `hard_timeout_s=300`
4. `max_retry_attempts=2`, `retry_backoff=exponential+jitter`
5. `result_signature_algorithm=Ed25519`
6. `heartbeat_interval_s=30`, `degraded_threshold_s=90`, `offline_threshold_s=180`
7. `catalog_default_availability_filter=healthy`
8. `ranking_min_samples=30`

补充实现决议：
- `seller_token_validation_mode=online_introspect_required`
- `request_event_scope_v0_1=ACKED_only`
- `catalog_import_mode=on_demand_immediate`
- `platform_storage_backend=PostgreSQL`
- `seller_subagent_binding_mode=platform_import_association`
- `api_auth_mode=api_key`
- `identity_onboarding_mode=pre_register_then_issue_key`
- `catalog_submission_mode=form_submit_then_cli_review_import`
