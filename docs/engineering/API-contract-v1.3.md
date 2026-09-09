---
doc_id: TONGPIN-B2-API
title: 同频 B2 前后端交互接口文档
version: 1.3
date: 2026-09-09
status: draft-for-review
architecture_baseline: MySQL-8-only-initial-implementation
backend_stack: Node.js NestJS TypeScript
ai_provider: DeepSeek
ai_default_model: deepseek-v4-flash
ai_provider_docs: https://api-docs.deepseek.com/zh-cn/
base_url: /api/v1
audience: [前端, 后端, 测试, AI Agent]
source_prd: docs/product/AI-friendly-PRD-v1.0.md
ai_reading_rules:
  - 本文档只定义接口契约，不授权模型执行临床判断或外部联络。
  - 涉及安全、隐私、未成年人、外部转介的接口必须保留人工确认。
  - 所有 `to-build` 接口在真实上线前必须通过权限、审计和机构评审。
  - 用户端在设备离线、AI 不可用、资源缺失时仍必须可发起主动求助。
  - 初期实现不引入消息队列和缓存组件，MySQL 8 是唯一状态事实来源。
  - 后端推荐使用 Node.js + NestJS + TypeScript 实现。
  - DeepSeek API Key 只保存在服务端，前端和管理端接口不得返回明文密钥。
  - OpenAPI/Swagger 支持 local 与 remote 两套 server，切换环境后可直接 Try it out。
openapi_file: docs/engineering/openapi-v1.3.yaml
---

# 同频 B2 前后端交互接口文档（V1.3）

## HoldU V0.4 对齐说明

本文档定义 HoldU 前后端接口契约，配套 [OpenAPI V1.3](openapi-v1.3.yaml) 和 [后端技术方案](backend-architecture-v1.2.md)。产品需求与数据规则以 [PRD V0.4](../product/PRD-v0.4.md)、[数据字典](../rules/data-dictionary-v0.4.md) 和 [触发规则](../rules/trigger-rules-v0.4.md) 为准。

## 1. 范围与目标

本文档面向黑客松原型和后续机构试点，定义用户端、志愿者端、管理端、AI 辅助端与后端之间的主要接口。接口设计目标：

1. 支持三端联动：用户、志愿者、值班负责人、专业督导。
2. 支持服务闭环：发起请求、接单、陪伴、交接、专业接管、结束、复盘。
3. 支持安全边界：即时安全优先、人工确认、最小必要数据、权限隔离。
4. 支持失败回退：设备离线、AI 不可用、督导未响应、资源过期时不阻断求助。
5. 支持数据库化基础设施：初期不使用消息队列和缓存，所有状态、幂等、事件和 AI 配置均持久化在 MySQL 8。
6. 支持可配置 AI：默认调用 DeepSeek `deepseek-v4-flash`，模型参数、提示词、skills、tools 可在管理端配置并版本化。

本文档不承诺已完成真实机构对接。涉及真实热线、医院、地图、身份核验的接口均标记为 `to-build`。

初期部署形态为“Node.js NestJS 单体后端 + MySQL 8 + SSE”。消息队列、Redis、Kafka 等组件暂不引入；后续如引入，只能作为性能优化，不得改变数据库事实来源和安全边界。

## 1.1 Swagger API 调试入口

本版本同时提供 OpenAPI 3.0 规范和 Swagger UI 页面：

| 产物 | 文件 | 用途 |
|---|---|---|
| OpenAPI 规范 | `docs/engineering/openapi-v1.3.yaml` | 导入 Apifox、Postman、Stoplight、代码生成器 |
| Swagger UI | `backend-service/public/index.html` | 双击打开，选择环境后点击 Try it out 调用接口 |

### 环境切换

Swagger UI 的 `Servers` 下拉框内置两套环境：

| 环境 | 默认地址 | 说明 |
|---|---|---|
| 本地 | `http://localhost:8080/api/v1` | 本地启动 NestJS 后端 |
| 远程 | `https://{remoteHost}/api/v1` | 将 `remoteHost` 改成实际远程 API 域名 |

远程环境必须启用 HTTPS 和 CORS。不要将数据库密码、DeepSeek API Key 写入 Swagger 页面、接口文档或前端配置。

### 调试步骤

1. 打开 `backend-service/public/index.html`。
2. 在 `Servers` 中选择本地或远程环境。
3. 先调用 `POST /auth/user-session` 创建匿名用户会话，或使用测试账号登录。
4. 点击右上角 `Authorize`，粘贴 `token`。
5. 展开目标接口，点击 `Try it out`。
6. 修改请求参数，点击 `Execute` 获取接口结果。
7. 写接口可填写 `Idempotency-Key`，重复提交同一请求应返回同一结果。

### Swagger 限制

- `GET /events` 是 SSE 流式接口，Swagger UI 只能尝试连接；建议用浏览器、curl 或前端页面验证。
- Try it out 依赖后端已实现对应接口和 CORS 配置。
- 生产环境建议关闭公开 Swagger UI，或仅对测试账号开放。
- Swagger 中的示例均为合成数据，不得使用真实危机用户数据。

---

## 2. 通用约定

### 2.1 基础信息

| 项 | 约定 |
|---|---|
| Base URL | `/api/v1` |
| 协议 | HTTPS |
| 编码 | UTF-8 |
| Content-Type | `application/json` |
| 时间格式 | ISO 8601，含时区，例如 `2026-09-09T10:20:30+08:00` |
| ID | 服务端生成的 UUID 或带前缀短 ID |
| 分页 | `page`、`page_size`，默认 `page=1`、`page_size=20` |
| 幂等 | 写接口可携带 `Idempotency-Key` |
| 请求追踪 | 服务端返回 `X-Request-Id` |
| 错误结构 | 统一使用 `error` 对象 |

### 2.2 认证

| 端 | 认证方式 |
|---|---|
| 用户端 | 匿名会话 Token |
| 志愿者端 | 志愿者账号 Token |
| 管理端 | 值班负责人或专业督导 Token |
| AI 辅助接口 | 服务端内部 Token，或志愿者 Token 加显式人工确认状态 |
| AI 配置页面 | AI 配置管理员或技术管理员 Token |

初期不使用 Redis 保存会话；Token、会话和撤销状态保存在 MySQL。请求头：

```http
Authorization: Bearer <token>
Idempotency-Key: <uuid>
```

### 2.3 角色与权限

后端必须执行以下校验：

1. 身份真实有效。
2. 角色权限匹配。
3. 辖区授权匹配。
4. 个案归属或待分配状态匹配。
5. 用户授权范围匹配。
6. 未成年人、日记、设备数据、聊天全文分别校验。

前端隐藏按钮不能作为安全隔离。所有敏感接口必须服务端鉴权。

### 2.4 状态枚举

#### 支持分层 `support_need_level`

```text
stable
attention
sustained_support
immediate_safety
unverified
```

#### 支援进度 `service_progress`

```text
no_request
waiting_support
in_progress
awaiting_transfer
professional_taken_over
completed
```

#### 响应路径 `response_path`

```text
R0
R1
R2
R3
```

#### 服务工单状态 `case_status`

```text
created
waiting_assignment
assigned
in_progress
awaiting_transfer
professional_takeover_requested
professional_taken_over
external_referral_pending
completed
cancelled
```

#### 交接状态 `transfer_status`

```text
requested
approved
rejected
receiver_confirmed
cancelled
superseded
```

#### 志愿者状态 `volunteer_status`

```text
offline
checked_in
available
assigned
paused
resting
self_check
suspended
```

### 2.5 统一错误结构

```json
{
  "error": {
    "code": "CAPACITY_EXCEEDED",
    "message": "今日接待已达上限，请先申请调配人手。",
    "request_id": "req_01H8YQ",
    "details": {
      "daily_limit": 6,
      "current_daily_count": 6
    }
  }
}
```

### 2.6 常用错误码

| HTTP | code | 场景 |
|---|---|---|
| 400 | `VALIDATION_ERROR` | 请求参数不合法 |
| 401 | `AUTH_REQUIRED` | 未登录或 Token 失效 |
| 403 | `PERMISSION_DENIED` | 角色或辖区权限不足 |
| 403 | `SCOPE_DENIED` | 请求访问的数据超出授权范围 |
| 404 | `NOT_FOUND` | 对象不存在或不可见 |
| 409 | `CASE_STATE_CONFLICT` | 工单状态已变化 |
| 409 | `TRANSFER_SUPERSEDED` | 专业接管后普通交接失效 |
| 422 | `CAPACITY_EXCEEDED` | 志愿者当日或同时接待上限已满 |
| 422 | `IMMEDIATE_SAFETY_REQUIRED` | 普通志愿者不能独立承接即时安全请求 |
| 422 | `RECEIVER_CAPACITY_EXCEEDED` | 接班者满额，不能确认交接 |
| 422 | `AI_CONFIG_INVALID` | AI 配置不合法、缺失或未发布 |
| 422 | `AI_TOOL_FORBIDDEN` | 模型请求调用未启用或未授权的工具 |
| 423 | `ACCOUNT_LOCKED` | 资格过期或账号暂停 |
| 429 | `RATE_LIMITED` | 请求频率过高 |
| 502 | `AI_PROVIDER_ERROR` | DeepSeek 返回 500、503 或协议错误 |
| 503 | `AI_UNAVAILABLE` | AI 服务不可用、超时或被停用 |
| 503 | `RESOURCE_UNAVAILABLE` | 资源库不可用，需返回人工渠道 |

### 2.7 数据库化基础设施约定

初期实现不引入 Redis、Kafka、RabbitMQ 等组件。以下能力全部由 MySQL 8 承担：

1. **状态事实来源**：工单、志愿者、交接、专业接管、授权、AI 配置等均以数据库记录为准。
2. **会话与撤销**：Token、匿名会话和撤销状态保存在数据库表；可使用短有效期 JWT，但撤销必须查库。
3. **幂等**：写接口的 `Idempotency-Key` 保存到 `idempotency_keys` 表，并保存响应摘要或响应体。
4. **事件**：领域事件先写入 `events` 表，再由事件服务轮询并通过 SSE 推送；事件事实仍以数据库为准。
5. **AI 配置**：模型参数、提示词、skills、tools、发布状态和审计均持久化。
6. **限流**：初期可不实现复杂分布式限流；如需要，使用数据库计数表或进程内令牌桶，并以数据库兜底。
7. **并发一致性**：接单、交接、专业接管必须使用数据库事务、行锁或唯一约束，不能依赖缓存或消息队列顺序。
8. **降级原则**：数据库不可用时拒绝写操作；用户端仍必须显示本地现实求助渠道，不能因基础设施故障阻断求助入口。

---

## 3. 核心对象模型

### 3.1 User

```json
{
  "id": "user_01H8YQ",
  "display_name": "小屿",
  "age_band": "14-17",
  "district_id": "district_shanghai_a",
  "created_at": "2026-09-09T10:00:00+08:00",
  "consents": {
    "device_metrics": false,
    "activity_reminder": false,
    "trend_summary_share": false,
    "diary_share": false
  }
}
```

### 3.2 DeviceMetric

```json
{
  "metric_type": "heart_rate",
  "value": 72,
  "unit": "bpm",
  "source": "standard_ble",
  "sampled_at": "2026-09-09T10:10:00+08:00",
  "quality": "valid",
  "context": "resting"
}
```

设备缺失时必须返回：

```json
{
  "metric_type": "heart_rate",
  "value": null,
  "unit": "bpm",
  "source": "unavailable",
  "quality": "missing",
  "sampled_at": null
}
```

禁止把缺失值写成 `0`。

### 3.3 DiaryEntry

```json
{
  "id": "diary_01H8YQ",
  "user_id": "user_01H8YQ",
  "feeling": "low",
  "content": "今天有点累，也不想说话。",
  "status": "draft",
  "confirmed_at": null,
  "shared_scope": null,
  "version": 1
}
```

修改日记后，`version` 递增，原确认和分享状态立即撤销。

### 3.4 ServiceCase

```json
{
  "id": "case_01H8YQ",
  "user_id": "user_01H8YQ",
  "district_id": "district_shanghai_a",
  "support_need_level": "attention",
  "service_progress": "waiting_support",
  "response_path": "R3",
  "assigned_volunteer_id": null,
  "created_at": "2026-09-09T10:20:00+08:00",
  "consent_scope": ["chat_text"]
}
```

### 3.5 VolunteerProfile

```json
{
  "id": "volunteer_01H8YQ",
  "display_name": "林然",
  "role": "volunteer",
  "district_ids": ["district_shanghai_a", "district_shanghai_b"],
  "status": "available",
  "capacity": {
    "daily_limit": 6,
    "current_daily_count": 2,
    "concurrent_limit": 2,
    "current_concurrent_count": 1
  },
  "qualifications": [
    {
      "type": "peer_support_basic",
      "status": "valid",
      "expires_at": "2027-01-01T00:00:00+08:00"
    }
  ]
}
```

### 3.6 TransferRequest

```json
{
  "id": "transfer_01H8YQ",
  "case_id": "case_01H8YQ",
  "type": "shift_transfer",
  "from_volunteer_id": "volunteer_01H8YQ",
  "to_volunteer_id": "volunteer_02H8YQ",
  "status": "requested",
  "summary_scope": ["main_request", "safety_facts", "actions_taken"],
  "receiver_confirmed_at": null
}
```

### 3.7 Resource

```json
{
  "id": "resource_shanghai_12356",
  "name": "上海心理援助热线 12356 / 962525",
  "category": "mental_hotline",
  "region": "shanghai",
  "phone": "12356",
  "available_time": "24h",
  "official_source": "https://www.shanghai.gov.cn/",
  "verified_at": "2026-09-01T00:00:00+08:00",
  "verified_by": "resource_admin",
  "status": "verified"
}
```

---

# 4. 接口总览

## 4.0 全接口总览表

下表按端到端调用视角汇总全部接口。`本地` 表示本地后端默认地址 `http://localhost:8080/api/v1`；`远程` 表示部署后的后端地址，需要在 Swagger 页面或环境变量中配置。所有接口均可通过 Swagger UI 的 Try it out 按钮直接调用，但敏感接口需要先完成登录或匿名会话创建。

| 编号 | 方法 | 路径 | 端 | 用途 | 认证 | 环境 |
|---|---|---|---|---|---|---|
| H01 | GET | `/health` | 服务端 | 健康检查，确认 API 与数据库可用 | 无 | 本地 / 远程 |
| A01 | POST | `/auth/user-session` | 用户端 | 创建匿名用户会话 | 无 | 本地 / 远程 |
| A02 | POST | `/auth/volunteer-login` | 志愿者端 | 志愿者登录并获取会话 Token | 志愿者账号 | 本地 / 远程 |
| A03 | POST | `/auth/manager-login` | 管理端 | 值班负责人或专业督导登录 | 管理端账号 | 本地 / 远程 |
| A04 | POST | `/auth/refresh` | 全端 | 刷新访问 Token | Refresh Token | 本地 / 远程 |
| A05 | POST | `/auth/logout` | 全端 | 退出登录并撤销当前会话 | 用户 / 志愿者 / 管理端 | 本地 / 远程 |
| U01 | GET | `/me` | 用户端 | 获取当前用户信息 | 用户 | 本地 / 远程 |
| U02 | PATCH | `/me` | 用户端 | 更新昵称、年龄段、偏好 | 用户 | 本地 / 远程 |
| U03 | GET | `/me/consents` | 用户端 | 获取授权状态 | 用户 | 本地 / 远程 |
| U04 | PATCH | `/me/consents` | 用户端 | 更新设备、提醒、摘要、日记授权 | 用户 | 本地 / 远程 |
| U05 | GET | `/me/device/metrics` | 用户端 | 获取设备指标摘要 | 用户 | 本地 / 远程 |
| U06 | GET | `/me/device/trends` | 用户端 | 获取近 7 天设备趋势 | 用户 | 本地 / 远程 |
| U07 | POST | `/me/device/care-responses` | 用户端 | 回应活动关怀提醒 | 用户 | 本地 / 远程 |
| U08 | GET | `/me/diary` | 用户端 | 获取日记列表 | 用户 | 本地 / 远程 |
| U09 | POST | `/me/diary/drafts` | 用户端 | 生成或创建日记草稿 | 用户 | 本地 / 远程 |
| U10 | PATCH | `/me/diary/{diary_id}` | 用户端 | 编辑日记 | 用户 | 本地 / 远程 |
| U11 | POST | `/me/diary/{diary_id}/confirm` | 用户端 | 确认日记 | 用户 | 本地 / 远程 |
| U12 | POST | `/me/diary/{diary_id}/share` | 用户端 | 分享日记给当前服务人员 | 用户 | 本地 / 远程 |
| U13 | DELETE | `/me/diary/{diary_id}/share` | 用户端 | 撤回日记分享 | 用户 | 本地 / 远程 |
| U14 | GET | `/me/chat/messages` | 用户端 | 获取小频聊天消息 | 用户 | 本地 / 远程 |
| U15 | POST | `/me/chat/messages` | 用户端 | 发送聊天消息 | 用户 | 本地 / 远程 |
| U16 | POST | `/me/chat/needs` | 用户端 | 提交当前需要与安全状态 | 用户 | 本地 / 远程 |
| U17 | POST | `/me/safety/escalate` | 用户端 | 发起即时安全求助 | 用户 | 本地 / 远程 |
| U18 | POST | `/me/exercises/{type}/start` | 用户端 | 开始舒缓跟练 | 用户 | 本地 / 远程 |
| U19 | PATCH | `/me/exercises/{exercise_id}` | 用户端 | 暂停、继续、停止跟练 | 用户 | 本地 / 远程 |
| U20 | POST | `/me/exercises/{exercise_id}/complete` | 用户端 | 结束跟练并提交感受 | 用户 | 本地 / 远程 |
| U21 | GET | `/me/questionnaires/{type}` | 用户端 | 获取可选量表 | 用户 | 本地 / 远程 |
| U22 | POST | `/me/questionnaires/{type}/responses` | 用户端 | 提交量表回答 | 用户 | 本地 / 远程 |
| U23 | GET | `/me/support-cases/current` | 用户端 | 获取当前支援状态 | 用户 | 本地 / 远程 |
| U24 | POST | `/me/support-cases` | 用户端 | 发起人工陪伴请求 | 用户 | 本地 / 远程 |
| U25 | GET | `/resources` | 用户端 | 获取现实求助渠道 | 用户 | 本地 / 远程 |
| U26 | GET | `/events` | 用户端 | 订阅事件流 | 用户 | 本地 / 远程 |
| U27 | GET | `/events/poll` | 全端 | 事件断线兜底轮询 | 用户 / 志愿者 / 管理端 | 本地 / 远程 |
| V01 | GET | `/volunteer/me` | 志愿者端 | 获取志愿者个人信息 | 志愿者 | 本地 / 远程 |
| V02 | GET | `/volunteer/cases` | 志愿者端 | 获取本人个案与待分配摘要 | 志愿者 | 本地 / 远程 |
| V03 | POST | `/volunteer/cases/{case_id}/accept` | 志愿者端 | 承接支援请求 | 志愿者 | 本地 / 远程 |
| V04 | GET | `/volunteer/cases/{case_id}` | 志愿者端 | 获取个案详情 | 当前承接者 | 本地 / 远程 |
| V05 | POST | `/volunteer/cases/{case_id}/messages` | 志愿者端 | 发送文字陪伴消息 | 当前承接者 | 本地 / 远程 |
| V06 | POST | `/volunteer/cases/{case_id}/close` | 志愿者端 | 结束本次陪伴 | 当前承接者 | 本地 / 远程 |
| V07 | POST | `/volunteer/rest` | 志愿者端 | 申请休息或暂停新接单 | 志愿者 | 本地 / 远程 |
| V08 | POST | `/volunteer/self-check` | 志愿者端 | 提交重新接单自检 | 志愿者 | 本地 / 远程 |
| V09 | GET | `/volunteer/schedule` | 志愿者端 | 获取排班 | 志愿者 | 本地 / 远程 |
| V10 | GET | `/volunteer/capacity` | 志愿者端 | 获取容量 | 志愿者 | 本地 / 远程 |
| V11 | POST | `/volunteer/transfer-requests` | 志愿者端 | 发起普通交接 | 当前承接者 | 本地 / 远程 |
| V12 | POST | `/volunteer/professional-requests` | 志愿者端 | 发起专业求助 | 当前承接者 | 本地 / 远程 |
| V13 | POST | `/volunteer/review-requests` | 志愿者端 | 发起支持分层复核申请 | 当前承接者 | 本地 / 远程 |
| M01 | GET | `/manager/overview` | 管理端 | 获取辖区总览 | 值班负责人 | 本地 / 远程 |
| M02 | GET | `/manager/users` | 管理端 | 获取辖区用户摘要 | 值班负责人 | 本地 / 远程 |
| M03 | GET | `/manager/volunteers` | 管理端 | 获取志愿者与负荷 | 值班负责人 | 本地 / 远程 |
| M04 | GET | `/manager/schedules` | 管理端 | 获取排班 | 值班负责人 | 本地 / 远程 |
| M05 | PATCH | `/manager/schedules/{schedule_id}` | 管理端 | 调整排班 | 值班负责人 | 本地 / 远程 |
| M06 | GET | `/manager/transfer-requests` | 管理端 | 获取交接队列 | 值班负责人 | 本地 / 远程 |
| M07 | POST | `/manager/transfer-requests/{id}/confirm` | 管理端 | 确认普通交接 | 值班负责人 | 本地 / 远程 |
| M08 | GET | `/manager/professional-requests` | 管理端 | 获取专业接管请求 | 专业督导 | 本地 / 远程 |
| M09 | POST | `/manager/professional-requests/{id}/confirm` | 管理端 | 确认专业接管 | 专业督导 | 本地 / 远程 |
| M10 | POST | `/manager/support-reviews` | 管理端 | 记录支持分层复核 | 专业督导 | 本地 / 远程 |
| M11 | GET | `/manager/audit-logs` | 管理端 | 查询审计日志 | 授权管理员 | 本地 / 远程 |
| M12 | POST | `/manager/resources` | 管理端 | 新增或更新资源 | 资源管理员 | 本地 / 远程 |
| AI01 | POST | `/ai/diary/draft` | AI | 生成日记草稿 | 用户 | 本地 / 远程 |
| AI02 | POST | `/ai/chat/suggestion` | AI | 生成小频回复候选 | 用户或志愿者 | 本地 / 远程 |
| AI03 | POST | `/ai/transfer-summary` | AI | 生成最小交接摘要 | 当前承接者 | 本地 / 远程 |
| AI04 | POST | `/ai/safety-check` | 服务端内部 | 检查建议是否越界 | 内部 | 本地 / 远程 |
| AI05 | POST | `/ai/resource-recommendation` | AI | 检索资源候选 | 志愿者或专业督导 | 本地 / 远程 |
| AI06 | GET | `/ai/health` | 服务端内部 | 获取 AI 服务可用性 | 内部 | 本地 / 远程 |
| C01 | GET | `/manager/ai/config` | AI 配置 | 获取当前 AI 配置 | AI 配置管理员 | 本地 / 远程 |
| C02 | POST | `/manager/ai/config` | AI 配置 | 创建 AI 配置草稿 | AI 配置管理员 | 本地 / 远程 |
| C03 | POST | `/manager/ai/config/{id}/publish` | AI 配置 | 发布 AI 配置 | AI 配置管理员 | 本地 / 远程 |
| C04 | POST | `/manager/ai/config/{id}/test` | AI 配置 | 使用合成输入测试配置 | AI 配置管理员 | 本地 / 远程 |
| C05 | GET | `/manager/ai/prompts` | AI 配置 | 获取提示词库 | AI 配置管理员 | 本地 / 远程 |
| C06 | POST | `/manager/ai/prompts` | AI 配置 | 新增提示词版本 | AI 配置管理员 | 本地 / 远程 |
| C07 | PATCH | `/manager/ai/prompts/{id}` | AI 配置 | 启用、停用或修改草稿 | AI 配置管理员 | 本地 / 远程 |
| C08 | GET | `/manager/ai/skills` | AI 配置 | 获取技能库 | AI 配置管理员 | 本地 / 远程 |
| C09 | POST | `/manager/ai/skills` | AI 配置 | 新增技能版本 | AI 配置管理员 | 本地 / 远程 |
| C10 | PATCH | `/manager/ai/skills/{id}` | AI 配置 | 启用、停用或修改技能 | AI 配置管理员 | 本地 / 远程 |
| C11 | GET | `/manager/ai/tools` | AI 配置 | 获取工具注册表 | AI 配置管理员 | 本地 / 远程 |
| C12 | POST | `/manager/ai/tools` | AI 配置 | 新增工具版本 | AI 配置管理员 | 本地 / 远程 |
| C13 | PATCH | `/manager/ai/tools/{id}` | AI 配置 | 启用、停用或修改工具 | AI 配置管理员 | 本地 / 远程 |
| C14 | GET | `/manager/ai/audit-logs` | AI 配置 | 查询 AI 配置和调用审计 | 授权管理员 | 本地 / 远程 |

---

## 4.1 用户端接口

| 方法 | 路径 | 说明 | 权限 |
|---|---|---|---|
| POST | `/auth/user-session` | 创建匿名用户会话 | 公开 |
| GET | `/me` | 获取当前用户信息 | 用户 |
| PATCH | `/me` | 更新昵称、年龄段、偏好 | 用户 |
| GET | `/me/consents` | 获取授权状态 | 用户 |
| PATCH | `/me/consents` | 更新设备、提醒、摘要、日记授权 | 用户 |
| GET | `/me/device/metrics` | 获取设备指标摘要 | 用户 |
| GET | `/me/device/trends` | 获取近 7 天趋势 | 用户 |
| POST | `/me/device/care-responses` | 回应活动关怀提醒 | 用户 |
| GET | `/me/diary` | 获取日记列表 | 用户 |
| POST | `/me/diary/drafts` | 生成或创建日记草稿 | 用户 |
| PATCH | `/me/diary/{diary_id}` | 编辑日记 | 用户 |
| POST | `/me/diary/{diary_id}/confirm` | 确认日记 | 用户 |
| POST | `/me/diary/{diary_id}/share` | 分享日记给当前服务人员 | 用户 |
| DELETE | `/me/diary/{diary_id}/share` | 撤回日记分享 | 用户 |
| GET | `/me/chat/messages` | 获取小频聊天消息 | 用户 |
| POST | `/me/chat/messages` | 发送聊天消息 | 用户 |
| POST | `/me/chat/needs` | 提交当前需要 | 用户 |
| POST | `/me/safety/escalate` | 发起即时安全求助 | 用户 |
| POST | `/me/exercises/{type}/start` | 开始舒缓跟练 | 用户 |
| PATCH | `/me/exercises/{exercise_id}` | 暂停、继续、停止跟练 | 用户 |
| POST | `/me/exercises/{exercise_id}/complete` | 结束跟练并提交感受 | 用户 |
| GET | `/me/questionnaires/{type}` | 获取可选量表 | 用户 |
| POST | `/me/questionnaires/{type}/responses` | 提交量表回答 | 用户 |
| GET | `/me/support-cases/current` | 获取当前支援状态 | 用户 |
| POST | `/me/support-cases` | 发起人工陪伴请求 | 用户 |
| GET | `/resources` | 获取现实求助渠道 | 用户 |
| GET | `/events` | 订阅用户端事件流 | 用户 |
| GET | `/events/poll` | 事件断线兜底轮询 | 用户 / 志愿者 / 管理端 |

## 4.2 志愿者端接口

| 方法 | 路径 | 说明 | 权限 |
|---|---|---|---|
| GET | `/volunteer/me` | 获取志愿者个人信息 | 志愿者 |
| GET | `/volunteer/cases` | 获取本人个案与待分配摘要 | 志愿者 |
| POST | `/volunteer/cases/{case_id}/accept` | 承接支援请求 | 志愿者 |
| GET | `/volunteer/cases/{case_id}` | 获取个案详情 | 当前承接者 |
| POST | `/volunteer/cases/{case_id}/messages` | 发送文字陪伴消息 | 当前承接者 |
| POST | `/volunteer/cases/{case_id}/close` | 结束本次陪伴 | 当前承接者 |
| POST | `/volunteer/rest` | 申请休息或暂停新接单 | 志愿者 |
| POST | `/volunteer/self-check` | 提交重新接单自检 | 志愿者 |
| GET | `/volunteer/schedule` | 获取排班 | 志愿者 |
| GET | `/volunteer/capacity` | 获取容量 | 志愿者 |
| POST | `/volunteer/transfer-requests` | 发起普通交接 | 当前承接者 |
| POST | `/volunteer/professional-requests` | 发起专业求助 | 当前承接者 |
| POST | `/volunteer/review-requests` | 发起支持分层复核申请 | 当前承接者 |
| GET | `/events` | 订阅志愿者事件流 | 志愿者 |

## 4.3 管理端接口

| 方法 | 路径 | 说明 | 权限 |
|---|---|---|---|
| GET | `/manager/overview` | 获取辖区总览 | 值班负责人 |
| GET | `/manager/users` | 获取辖区用户摘要 | 值班负责人 |
| GET | `/manager/volunteers` | 获取志愿者与负荷 | 值班负责人 |
| GET | `/manager/schedules` | 获取排班 | 值班负责人 |
| PATCH | `/manager/schedules/{schedule_id}` | 调整排班 | 值班负责人 |
| GET | `/manager/transfer-requests` | 获取交接队列 | 值班负责人 |
| POST | `/manager/transfer-requests/{id}/confirm` | 确认普通交接 | 值班负责人 |
| GET | `/manager/professional-requests` | 获取专业接管请求 | 专业督导 |
| POST | `/manager/professional-requests/{id}/confirm` | 确认专业接管 | 专业督导 |
| POST | `/manager/support-reviews` | 记录支持分层复核 | 专业督导 |
| GET | `/manager/audit-logs` | 查询审计日志 | 授权管理员 |
| POST | `/manager/resources` | 新增或更新资源 | 资源管理员 |

## 4.4 AI 辅助接口

| 方法 | 路径 | 说明 | 权限 |
|---|---|---|---|
| POST | `/ai/diary/draft` | 生成日记草稿 | 用户 |
| POST | `/ai/chat/suggestion` | 生成小频回复候选 | 用户或志愿者 |
| POST | `/ai/transfer-summary` | 生成最小交接摘要 | 当前承接者 |
| POST | `/ai/safety-check` | 检查建议是否越界 | 服务端内部 |
| POST | `/ai/resource-recommendation` | 检索资源候选 | 志愿者或专业督导 |
| GET | `/ai/health` | 获取 AI 服务可用性 | 服务端内部 |
| GET | `/manager/ai/config` | 获取当前 AI 配置 | AI 配置管理员 |
| POST | `/manager/ai/config` | 创建 AI 配置草稿 | AI 配置管理员 |
| POST | `/manager/ai/config/{id}/publish` | 发布 AI 配置 | AI 配置管理员 |
| POST | `/manager/ai/config/{id}/test` | 使用合成输入测试配置 | AI 配置管理员 |
| GET | `/manager/ai/prompts` | 获取提示词库 | AI 配置管理员 |
| POST | `/manager/ai/prompts` | 新增提示词版本 | AI 配置管理员 |
| PATCH | `/manager/ai/prompts/{id}` | 启用、停用或修改草稿 | AI 配置管理员 |
| GET | `/manager/ai/skills` | 获取技能库 | AI 配置管理员 |
| POST | `/manager/ai/skills` | 新增技能版本 | AI 配置管理员 |
| PATCH | `/manager/ai/skills/{id}` | 启用、停用或修改草稿 | AI 配置管理员 |
| GET | `/manager/ai/tools` | 获取工具注册表 | AI 配置管理员 |
| POST | `/manager/ai/tools` | 新增工具版本 | AI 配置管理员 |
| PATCH | `/manager/ai/tools/{id}` | 启用、停用或修改工具 | AI 配置管理员 |
| GET | `/manager/ai/audit-logs` | 查询 AI 配置和调用审计 | 授权管理员 |

AI 接口不得直接触发外呼、报警、通知家属或外部转介。所有输出必须保留 `requires_human_confirmation=true` 或 `human_confirmed=false`。

AI 配置接口只返回密钥是否存在，不返回 DeepSeek API Key 明文。

---

# 5. 接口详细定义

## 5.1 `POST /auth/user-session`

### 说明

创建匿名用户会话。黑客松阶段可使用演示身份；真实服务需由机构确认匿名规则、设备标识和未成年人处理流程。

### 请求

```json
{
  "age_band": "14-17",
  "district_id": "district_shanghai_a",
  "client_type": "web",
  "client_version": "1.0.0"
}
```

### 响应

```json
{
  "token": "anonymous_token",
  "user": {
    "id": "user_01H8YQ",
    "age_band": "14-17"
  },
  "expires_at": "2026-09-09T12:00:00+08:00"
}
```

### 安全

- 不采集真实姓名。
- 12–13 岁分支需后续完成模拟监护知情同意。
- 未完成身份或监护验证不影响主动求助。

---

## 5.1.1 `POST /auth/volunteer-login`

### 请求

```json
{
  "account": "demo_volunteer",
  "password": "demo-password",
  "client_type": "web"
}
```

### 响应

```json
{
  "token": "volunteer_access_token",
  "refresh_token": "volunteer_refresh_token",
  "role": "volunteer",
  "expires_at": "2026-09-09T12:00:00+08:00"
}
```

### 规则

1. 生产环境应接入机构统一身份认证或专用账号体系，不使用演示账号。
2. 登录失败不提示具体是账号还是密码错误。
3. Token 与会话保存在 MySQL，支持服务端撤销。
4. 登录成功后必须校验资格、培训状态、辖区授权和账号状态。

---

## 5.1.2 `POST /auth/manager-login`

### 请求

```json
{
  "account": "demo_manager",
  "password": "demo-password",
  "client_type": "web"
}
```

### 响应

```json
{
  "token": "manager_access_token",
  "refresh_token": "manager_refresh_token",
  "role": "duty_manager",
  "expires_at": "2026-09-09T12:00:00+08:00"
}
```

`role` 可为 `duty_manager` 或 `professional_supervisor`。专业接管和分层复核必须由专业督导执行。

---

## 5.1.3 `POST /auth/refresh`

### 请求

```json
{
  "refresh_token": "refresh_token"
}
```

### 响应

```json
{
  "token": "new_access_token",
  "refresh_token": "new_refresh_token",
  "expires_at": "2026-09-09T14:00:00+08:00"
}
```

Refresh Token 必须一次性使用或设置短有效期；被撤销后不能继续刷新。

---

## 5.1.4 `POST /auth/logout`

### 请求

```json
{
  "session_id": "session_01H8YQ"
}
```

### 响应

```json
{
  "revoked": true
}
```

退出后当前访问 Token 与 Refresh Token 必须立即失效。

---

## 5.2 `PATCH /me/consents`

### 请求

```json
{
  "device_metrics": true,
  "activity_reminder": true,
  "trend_summary_share": false,
  "diary_share": false
}
```

### 响应

```json
{
  "user_id": "user_01H8YQ",
  "consents": {
    "device_metrics": true,
    "activity_reminder": true,
    "trend_summary_share": false,
    "diary_share": false
  },
  "updated_at": "2026-09-09T10:30:00+08:00"
}
```

### 规则

1. 授权项必须分开控制，不能使用一个总开关。
2. 撤回后，相关派单、摘要和访问权限应立即失效。
3. 未授权时仍可发起主动求助。

---

## 5.3 `POST /me/chat/needs`

### 说明

提交用户当前需要。后端根据安全状态判断是否进入即时安全流程。

### 请求

```json
{
  "current_safety": "unsafe",
  "main_need": "talk_to_human",
  "concern_started_at": "2026-09-08T20:00:00+08:00",
  "life_impact": "sleep_and_study",
  "preferred_support": "volunteer_text",
  "skip_questionnaire": true
}
```

### 响应

```json
{
  "case_id": "case_01H8YQ",
  "response_path": "R0",
  "service_progress": "waiting_support",
  "support_need_level": "immediate_safety",
  "blocked_actions": [
    "questionnaire",
    "activity_reminder",
    "exercise_suggestion"
  ],
  "real_world_resources": [
    {
      "id": "resource_medical_120",
      "name": "医疗急救",
      "phone": "120"
    },
    {
      "id": "resource_shanghai_12356",
      "name": "上海心理援助热线",
      "phone": "12356"
    }
  ]
}
```

### 规则

- `current_safety=unsafe` 时，必须优先返回 R0 流程。
- 不得等待普通志愿者名额。
- 不得要求用户先完成量表。
- 不得把身体急症解释为惊恐或焦虑。

---

## 5.4 `POST /me/support-cases`

### 说明

发起人工陪伴请求。

### 请求

```json
{
  "request_type": "volunteer_text",
  "consent_scope": ["chat_text"],
  "preferred_contact": "text",
  "share_trend_summary": false,
  "share_diary": false,
  "immediate_safety": false
}
```

### 响应

```json
{
  "case_id": "case_01H8YQ",
  "status": "waiting_assignment",
  "service_progress": "waiting_support",
  "district_id": "district_shanghai_a",
  "estimated_wait": null,
  "notice": "当前为演示服务，不代表真实值守。"
}
```

### 规则

- 未授权设备或日记时，仍可发起请求。
- 后端不得自动生成虚假等待时间。
- 请求创建后必须进入辖区队列。
- 支持用户随时取消未接单请求。

---

## 5.5 `POST /me/safety/escalate`

### 说明

用户明确表示当前不安全或无法确认安全时调用。

### 请求

```json
{
  "safety_status": "unsafe",
  "physical_emergency": false,
  "public_safety_danger": false,
  "current_location": null,
  "trusted_contact_available": false
}
```

### 响应

```json
{
  "case_id": "case_01H8YQ",
  "response_path": "R0",
  "service_progress": "waiting_support",
  "support_need_level": "immediate_safety",
  "professional_request_id": "professional_01H8YQ",
  "status": "waiting_confirmation",
  "resources": [
    {
      "name": "医疗急救",
      "phone": "120"
    },
    {
      "name": "上海心理援助热线",
      "phone": "12356"
    },
    {
      "name": "公共安全求助",
      "phone": "110"
    }
  ],
  "warning": "演示系统没有真实值守，不能替代急救或专业危机服务。"
}
```

### 规则

- 不得显示“已有人守护”除非专业督导已确认接管。
- 不得自动拨打 120、110 或热线。
- 不得默认通知家长、教师或紧急联系人。
- 身体急症优先返回 120，公共安全危险优先返回 110。

---

## 5.6 `POST /me/diary/drafts`

### 请求

```json
{
  "feeling": "low",
  "free_text": "今天有点累，也不想说话。",
  "source": "user_input"
}
```

### 响应

```json
{
  "diary_id": "diary_01H8YQ",
  "status": "draft",
  "content": "今天有点累，也不想说话。",
  "ai_generated": false,
  "confirmed": false,
  "shared_scope": null
}
```

### 规则

- 草稿仅用户可见。
- 生成后必须由用户编辑确认。
- 修改后旧版本失效，分享状态撤销。
- 不得自动写入设备数据或医疗结论。

---

## 5.7 `POST /me/diary/{diary_id}/share`

### 请求

```json
{
  "scope": "current_case_volunteer",
  "case_id": "case_01H8YQ"
}
```

### 响应

```json
{
  "diary_id": "diary_01H8YQ",
  "shared_scope": "current_case_volunteer",
  "shared_at": "2026-09-09T11:00:00+08:00",
  "expires_at": "2026-09-10T11:00:00+08:00"
}
```

### 规则

- 仅当前授权服务人员可见。
- 服务结束、交接失效、撤回授权后立即不可见。
- 不得默认分享给管理者或监护人。

---

## 5.8 `POST /me/exercises/{type}/start`

`type` 可取：

```text
breathing
grounding
shoulder_hand
```

### 请求

```json
{
  "safety_confirmed": true,
  "willing_to_try": true
}
```

### 响应

```json
{
  "exercise_id": "exercise_01H8YQ",
  "type": "breathing",
  "status": "in_progress",
  "started_at": "2026-09-09T11:10:00+08:00",
  "safety_notice": "若有明显急症，请先寻求医疗帮助。"
}
```

### 规则

- 切后台或暂停时前端应调用 `PATCH`。
- 重新开始需再次确认。
- 练后感受不自动写入日记。
- 不舒服时必须保留求助入口。

---

## 5.9 `POST /volunteer/cases/{case_id}/accept`

### 请求

```json
{
  "acknowledge_scope": ["chat_text"]
}
```

### 响应

```json
{
  "case_id": "case_01H8YQ",
  "volunteer_id": "volunteer_01H8YQ",
  "status": "in_progress",
  "capacity_after": {
    "daily_count": 3,
    "concurrent_count": 2
  }
}
```

### 校验规则

后端必须同时检查：

1. 志愿者身份有效。
2. 资格有效。
3. 辖区授权有效。
4. 当前处于可服务状态。
5. 当日人数未达上限。
6. 同时服务数未达上限。
7. 个案仍处于 `waiting_assignment`。
8. 非即时安全请求，除非获得专业授权。
9. 请求未他人承接。

并发条件下，仅一个志愿者可成功接单。

### 错误

```json
{
  "error": {
    "code": "CAPACITY_EXCEEDED",
    "message": "同时陪伴已满，请先完成或交接当前服务。"
  }
}
```

---

## 5.10 `POST /volunteer/cases/{case_id}/messages`

### 请求

```json
{
  "content": "谢谢你愿意说出来。此刻，你最希望我怎样陪你？",
  "message_type": "text",
  "ai_assisted": false
}
```

### 响应

```json
{
  "message_id": "msg_01H8YQ",
  "case_id": "case_01H8YQ",
  "sender_role": "volunteer",
  "created_at": "2026-09-09T11:20:00+08:00",
  "delivered": true
}
```

### 规则

- 仅当前承接者或已接管专业督导可发送。
- AI 建议不能自动发送。
- `ai_assisted=true` 时前端必须明示。
- 敏感信息不得出现在通知预览。

---

## 5.11 `POST /volunteer/transfer-requests`

### 请求

```json
{
  "case_id": "case_01H8YQ",
  "type": "shift_transfer",
  "to_volunteer_id": "volunteer_02H8YQ",
  "summary_scope": [
    "event_id",
    "age_band",
    "main_request",
    "verified_safety_facts",
    "actions_taken",
    "open_questions"
  ]
}
```

### 响应

```json
{
  "transfer_request_id": "transfer_01H8YQ",
  "status": "requested",
  "receiver_confirmation_required": true,
  "created_at": "2026-09-09T11:30:00+08:00"
}
```

### 规则

- 摘要只传必要信息。
- 不默认传完整聊天、日记全文、连续原始设备数据或精确位置。
- 原志愿者在确认前保留责任。
- 专业接管后普通交接自动失效。

---

## 5.12 `POST /manager/transfer-requests/{id}/confirm`

### 请求

```json
{
  "receiver_volunteer_id": "volunteer_02H8YQ",
  "capacity_verified": true,
  "note": "已核对接班人剩余名额和资格。"
}
```

### 响应

```json
{
  "transfer_request_id": "transfer_01H8YQ",
  "status": "receiver_confirmed",
  "case_id": "case_01H8YQ",
  "new_owner_id": "volunteer_02H8YQ",
  "old_owner_released": true,
  "confirmed_at": "2026-09-09T11:35:00+08:00"
}
```

### 规则

- 值班负责人确认不等于实际接管，仍需接收方确认。
- 若接收方满额，返回 `RECEIVER_CAPACITY_EXCEEDED`。
- 确认成功后必须写入审计日志。

---

## 5.13 `POST /manager/professional-requests/{id}/confirm`

### 请求

```json
{
  "reviewed_user_situation": true,
  "takeover_reason": "用户明确表示当前不安全。",
  "next_action": "coordinate_external_referral",
  "external_referral_status": "pending"
}
```

### 响应

```json
{
  "professional_request_id": "professional_01H8YQ",
  "status": "professional_taken_over",
  "case_id": "case_01H8YQ",
  "supervisor_id": "supervisor_01H8YQ",
  "taken_over_at": "2026-09-09T11:40:00+08:00",
  "old_transfer_request_status": "superseded"
}
```

### 规则

- 仅专业督导可确认。
- 确认后原志愿者不能继续普通会话操作。
- 未确认前不得显示“已接手”。
- 外部转介状态不得虚构。

---

## 5.14 `POST /ai/chat/suggestion`

### 请求

```json
{
  "case_id": "case_01H8YQ",
  "context_scope": ["chat_text", "main_request"],
  "goal": "empathetic_listening",
  "max_tokens": 200
}
```

### 响应

```json
{
  "suggestion_id": "ai_01H8YQ",
  "candidate_text": "谢谢你愿意说出来。你希望我先听你说，还是先帮你确认接下来想获得的支持？",
  "source_type": "model",
  "provider": "deepseek",
  "model": "deepseek-v4-flash",
  "ai_config_version": 3,
  "confidence": null,
  "uncertainty": "用户尚未说明当前安全状态",
  "requires_human_confirmation": true,
  "human_confirmed": false,
  "tool_calls": [],
  "token_usage": {
    "prompt_tokens": 386,
    "completion_tokens": 42,
    "total_tokens": 428
  },
  "finish_reason": "stop"
}
```

### 规则

1. 不能自动发送。
2. 不能诊断。
3. 不能给出医疗或用药建议。
4. 不能承诺保密无例外。
5. 不能编造资源。
6. 输出中若含越权建议，必须被安全检查拦截。
7. 后端按当前发布的 AI 配置调用 DeepSeek；不得由前端直接传模型参数或提示词覆盖安全规则。
8. 若模型返回 `tool_calls`，后端只能执行已启用的白名单工具；未授权工具返回 `AI_TOOL_FORBIDDEN`。
9. 模型输出不是事实来源；工单状态和用户当前情况仍以数据库和人工确认结果为准。

---

## 5.15 AI 配置管理接口

AI 配置页面面向 AI 配置管理员或技术管理员，用于维护 DeepSeek 调用参数、提示词、skills、tools 和安全规则。配置必须版本化，发布后才能被业务接口使用。

### 5.15.1 `GET /manager/ai/config`

#### 响应

```json
{
  "active_config": {
    "id": "ai_config_01H8YQ",
    "version": 3,
    "status": "published",
    "provider": "deepseek",
    "base_url": "https://api.deepseek.com",
    "model": "deepseek-v4-flash",
    "api_key_configured": true,
    "enabled": true,
    "default_parameters": {
      "thinking": { "type": "disabled" },
      "reasoning_effort": "low",
      "max_tokens": 800,
      "temperature": 0.2,
      "top_p": 0.9,
      "response_format": { "type": "json_object" },
      "stream": false,
      "stop": []
    },
    "task_overrides": {
      "chat_suggestion": {
        "max_tokens": 500,
        "response_format": { "type": "json_object" }
      },
      "diary_draft": {
        "max_tokens": 600,
        "response_format": { "type": "json_object" }
      },
      "transfer_summary": {
        "max_tokens": 700,
        "response_format": { "type": "json_object" }
      }
    },
    "prompt_ids": {
      "chat_suggestion": "prompt_chat_v3",
      "diary_draft": "prompt_diary_v2",
      "transfer_summary": "prompt_transfer_v2",
      "safety_check": "prompt_safety_v1"
    },
    "skill_ids": [
      "skill_empathetic_listening",
      "skill_safety_gate",
      "skill_resource_grounding"
    ],
    "tool_ids": [
      "tool_search_verified_resources",
      "tool_get_case_summary"
    ],
    "timeout_ms": 2000,
    "max_retries": 1,
    "daily_token_budget": 200000,
    "published_at": "2026-09-09T10:00:00+08:00",
    "published_by": "admin_01H8YQ"
  },
  "drafts": [
    {
      "id": "ai_config_01H9AA",
      "version": 4,
      "status": "draft",
      "created_at": "2026-09-09T11:00:00+08:00"
    }
  ]
}
```

#### 规则

1. `api_key_configured` 只表示密钥是否存在，不能返回密钥明文。
2. `model` 默认为 `deepseek-v4-flash`。
3. `stream` 初期建议固定为 `false`，降低实现复杂度。
4. 思考模式默认关闭；如开启，必须配置更长的超时和 token 预算。
5. 每个任务可以覆盖默认参数，但不得覆盖安全检查和人工确认要求。

---

### 5.15.2 `POST /manager/ai/config`

#### 请求

```json
{
  "provider": "deepseek",
  "base_url": "https://api.deepseek.com",
  "model": "deepseek-v4-flash",
  "api_key_secret_ref": "deepseek_prod_key",
  "enabled": true,
  "default_parameters": {
    "thinking": { "type": "disabled" },
    "reasoning_effort": "low",
    "max_tokens": 800,
    "temperature": 0.2,
    "top_p": 0.9,
    "response_format": { "type": "json_object" },
    "stream": false,
    "stop": []
  },
  "task_overrides": {
    "chat_suggestion": {
      "max_tokens": 500
    }
  },
  "prompt_ids": {
    "chat_suggestion": "prompt_chat_v3"
  },
  "skill_ids": ["skill_empathetic_listening"],
  "tool_ids": ["tool_search_verified_resources"],
  "timeout_ms": 2000,
  "max_retries": 1,
  "daily_token_budget": 200000,
  "change_note": "降低温度并限制输出 token。"
}
```

#### 响应

```json
{
  "id": "ai_config_01H9AA",
  "version": 4,
  "status": "draft",
  "created_at": "2026-09-09T11:00:00+08:00",
  "validation_result": {
    "valid": true,
    "errors": []
  }
}
```

#### 参数约束

| 参数 | 约束 |
|---|---|
| `provider` | 初期仅允许 `deepseek` |
| `model` | 默认 `deepseek-v4-flash`；如切换模型需重新验证 |
| `thinking.type` | `enabled` 或 `disabled` |
| `reasoning_effort` | `low`、`high`、`max` |
| `max_tokens` | 产品建议 128–4096；不得超过 DeepSeek 上限 |
| `temperature` | 0–2 |
| `top_p` | 0–1 |
| `response_format.type` | `text` 或 `json_object` |
| `stream` | 初期建议 `false` |
| `timeout_ms` | 建议 1000–10000 |
| `max_retries` | 建议 0–2；429、500、503 才可重试 |

#### 规则

1. 创建后为 `draft`，不得立即影响线上调用。
2. `api_key_secret_ref` 指向服务端密钥引用，不接受明文密钥。
3. 需校验 prompt、skill、tool 均存在且处于启用状态。
4. 需校验 JSON Schema、参数范围和任务映射完整性。
5. 变更必须写入审计日志。

---

### 5.15.3 `POST /manager/ai/config/{id}/publish`

#### 请求

```json
{
  "confirm_no_sensitive_data_in_test": true,
  "safety_review_passed": true,
  "change_note": "发布 v4 配置。"
}
```

#### 响应

```json
{
  "id": "ai_config_01H9AA",
  "version": 4,
  "status": "published",
  "published_at": "2026-09-09T11:10:00+08:00",
  "previous_version": 3,
  "rollback_config_id": "ai_config_01H8YQ"
}
```

#### 规则

1. 只有 `draft` 可以发布。
2. 发布前必须至少通过一次配置测试。
3. 发布是原子操作，旧版本保留，可回滚。
4. 涉及安全提示词或工具权限的变更，生产环境建议双人确认。

---

### 5.15.4 `POST /manager/ai/config/{id}/test`

#### 请求

```json
{
  "task_type": "chat_suggestion",
  "synthetic_input": {
    "age_band": "14-17",
    "main_request": "希望有人听我说说",
    "current_safety": "safe"
  },
  "execute_tools": false
}
```

#### 响应

```json
{
  "test_id": "ai_test_01H8YQ",
  "config_version": 4,
  "provider": "deepseek",
  "model": "deepseek-v4-flash",
  "latency_ms": 820,
  "request_valid": true,
  "output": {
    "candidate_text": "谢谢你愿意说出来。你希望我先听你说，还是先帮你确认接下来想获得的支持？",
    "uncertainty": "合成输入，不用于真实个案。"
  },
  "safety_check_result": "passed",
  "tool_calls": [],
  "token_usage": {
    "prompt_tokens": 210,
    "completion_tokens": 35,
    "total_tokens": 245
  },
  "finish_reason": "stop"
}
```

#### 规则

1. 测试只能使用合成输入，不得使用真实用户、日记或会话数据。
2. 默认不执行 tools；如 `execute_tools=true`，仍只能执行白名单只读工具。
3. 测试结果必须记录，但不保存敏感原文。
4. 若 DeepSeek 返回错误，响应映射为 `AI_PROVIDER_ERROR` 或 `AI_UNAVAILABLE`。

---

### 5.15.5 提示词接口

#### `GET /manager/ai/prompts`

```json
{
  "items": [
    {
      "id": "prompt_chat_v3",
      "task_type": "chat_suggestion",
      "name": "倾听建议提示词",
      "version": 3,
      "status": "enabled",
      "system_prompt": "你是志愿者倾听辅助助手……",
      "user_template": "当前诉求：{{main_request}}；当前安全：{{current_safety}}。",
      "variables": ["main_request", "current_safety", "age_band"],
      "updated_by": "admin_01H8YQ",
      "updated_at": "2026-09-09T10:30:00+08:00"
    }
  ]
}
```

#### `POST /manager/ai/prompts`

```json
{
  "task_type": "chat_suggestion",
  "name": "倾听建议提示词",
  "system_prompt": "你是志愿者倾听辅助助手。你不能诊断、不能建议用药、不能承诺保密无例外。",
  "user_template": "当前诉求：{{main_request}}；当前安全：{{current_safety}}。请输出 JSON。",
  "variables": ["main_request", "current_safety", "age_band"],
  "safety_constraints": [
    "no_diagnosis",
    "no_medication_advice",
    "no_external_action",
    "requires_human_confirmation"
  ],
  "change_note": "增加年龄适配约束。"
}
```

#### 规则

1. 提示词必须版本化。
2. `system_prompt` 必须包含安全边界和输出格式要求。
3. 使用 JSON Output 时，系统或用户提示中必须明确要求输出 JSON，并给出示例结构。
4. 用户输入只能作为数据插入，不得作为指令覆盖系统规则。
5. 修改已发布提示词会生成新版本，不直接覆盖旧版本。

---

### 5.15.6 Skills 接口

#### `GET /manager/ai/skills`

```json
{
  "items": [
    {
      "id": "skill_empathetic_listening",
      "code": "empathetic_listening",
      "name": "共情倾听",
      "description": "帮助志愿者形成非评判、低压力的倾听回应。",
      "instructions": "优先确认用户当前需要，不强迫解释原因。",
      "allowed_task_types": ["chat_suggestion"],
      "forbidden_actions": ["diagnosis", "medication_advice", "promise_no_risk"],
      "enabled": true,
      "version": 2
    }
  ]
}
```

#### `POST /manager/ai/skills`

```json
{
  "code": "safety_gate",
  "name": "安全闸门",
  "description": "识别需要立即转人工或现实求助的情境。",
  "instructions": "只要出现当前不安全、身体急症或迫近危险，输出 professional_handoff，不给自助建议。",
  "allowed_task_types": ["chat_suggestion", "diary_draft", "transfer_summary"],
  "forbidden_actions": ["diagnosis", "external_call", "notify_contact"],
  "enabled": true
}
```

#### 规则

1. Skill 只是提示词组合和行为约束，不是权限提升。
2. Skill 不能绕过安全检查、人工确认或数据库权限。
3. 每次调用记录实际启用的 skills 和版本。

---

### 5.15.7 Tools 接口

#### `GET /manager/ai/tools`

```json
{
  "items": [
    {
      "id": "tool_search_verified_resources",
      "name": "search_verified_resources",
      "description": "从已核验资源库中检索热线、医院或社区支持资源。",
      "parameters_json_schema": {
        "type": "object",
        "properties": {
          "region": { "type": "string" },
          "category": { "type": "string" }
        },
        "required": ["region", "category"],
        "additionalProperties": false
      },
      "execution_type": "internal_read_only",
      "risk_level": "low",
      "enabled": true,
      "version": 2
    }
  ]
}
```

#### `POST /manager/ai/tools`

```json
{
  "name": "get_case_summary",
  "description": "读取当前授权工单的最小必要摘要。",
  "parameters_json_schema": {
    "type": "object",
    "properties": {
      "case_id": { "type": "string", "format": "uuid" }
    },
    "required": ["case_id"],
    "additionalProperties": false
  },
  "execution_type": "internal_read_only",
  "risk_level": "medium",
  "handler": "internal.case_summary",
  "timeout_ms": 500,
  "enabled": true
}
```

#### 工具风险等级

| 等级 | 允许 |
|---|---|
| `low` | 只读、白名单、无外部副作用 |
| `medium` | 只读但涉及个案摘要，需权限校验 |
| `high` | 必须人工确认，初期建议禁用 |
| `forbidden` | 一律不允许注册或执行 |

#### 禁止工具

1. 自动拨打 120、110、热线。
2. 自动通知家属、监护人或联系人。
3. 自动外部转介。
4. 读取未授权日记或完整聊天历史。
5. 获取精确定位。
6. 任意 HTTP 请求、自由联网搜索、文件系统或 Shell 执行。

#### 规则

1. DeepSeek 只能返回 `tool_calls`，不能自行执行函数。
2. 后端必须校验工具名、参数 JSON Schema、权限、工单关系和风险等级。
3. 高风险工具必须进入人工确认队列。
4. 工具执行结果必须写入 `ai_tool_executions` 并审计。
5. 未知工具或未启用工具返回 `AI_TOOL_FORBIDDEN`。

---

## 5.16 DeepSeek 调用约定

### 5.16.1 基础调用

后端调用 DeepSeek OpenAI 兼容接口：

```http
POST https://api.deepseek.com/chat/completions
Authorization: Bearer <DEEPSEEK_API_KEY>
Content-Type: application/json
```

默认请求体：

```json
{
  "model": "deepseek-v4-flash",
  "messages": [
    {
      "role": "system",
      "content": "你是同频的倾听辅助助手。不能诊断，不能建议用药，不能承诺识别所有危机。输出 JSON。"
    },
    {
      "role": "user",
      "content": "{\"main_request\":\"希望有人听我说说\",\"current_safety\":\"safe\"}"
    }
  ],
  "thinking": { "type": "disabled" },
  "reasoning_effort": "low",
  "max_tokens": 500,
  "temperature": 0.2,
  "top_p": 0.9,
  "response_format": { "type": "json_object" },
  "stream": false,
  "user_id": "anon_user_01H8YQ"
}
```

### 5.16.2 参数映射

| 产品配置 | DeepSeek 参数 |
|---|---|
| `model` | `model` |
| `thinking.type` | `thinking.type` |
| `reasoning_effort` | `reasoning_effort` |
| `max_tokens` | `max_tokens` |
| `temperature` | `temperature` |
| `top_p` | `top_p` |
| `response_format` | `response_format` |
| `stop` | `stop` |
| `stream` | `stream` |
| `tools` | `tools` |
| `tool_choice` | `tool_choice` |
| 匿名用户 ID | `user_id` |

### 5.16.3 强制规则

1. DeepSeek API Key 只保存在服务端环境变量或密钥管理系统中。
2. 前端不得直接调用 DeepSeek，也不得获取 API Key。
3. `user_id` 只能使用匿名标识，不得包含手机号、身份证、姓名或精确位置。
4. DeepSeek Chat Completions 是无状态接口，后端必须在每次调用前从数据库组装最小上下文。
5. 使用 JSON Output 时，提示词必须明确要求 JSON，并设置 `max_tokens`，防止截断。
6. DeepSeek JSON Output 存在偶发空 `content` 的可能；后端必须将其视为无效输出，重试一次或降级为人工流程，不得把空字符串当作安全建议。
7. `finish_reason=insufficient_system_resource` 表示服务端资源不足，应按 `AI_PROVIDER_ERROR` 处理。
6. `frequency_penalty` 和 `presence_penalty` 在 DeepSeek 当前文档中已废弃，产品配置中不提供。
7. 若返回 `finish_reason=length`，输出可能截断，应降低上下文或提高 token 限制后重试；不得直接给用户展示半截建议。
8. 若返回 `finish_reason=content_filter`，不得改写后继续输出，应降级为人工流程。
9. 若返回 `tool_calls`，必须先执行安全校验和白名单工具，再将结果回传模型。
10. 所有调用记录 token 用量、配置版本、模型名和安全检查结果，但不保存敏感原文。

### 5.16.4 错误映射

| DeepSeek 状态 | 同频错误 |
|---|---|
| 400 | `AI_CONFIG_INVALID` |
| 401 | `AI_PROVIDER_ERROR`，并提示密钥配置异常 |
| 402 | `AI_PROVIDER_ERROR`，余额不足 |
| 422 | `AI_CONFIG_INVALID` |
| 429 | `AI_UNAVAILABLE` 或限流提示 |
| 500 | `AI_PROVIDER_ERROR` |
| 503 | `AI_PROVIDER_ERROR` |
| 网络超时 | `AI_UNAVAILABLE` |

所有 AI 错误都必须保留人工求助、资源和会话入口。

---

## 5.17 `GET /events`

### 说明

初期使用 MySQL 事件表加 SSE 推送。后端可将领域事件写入 `events` 表，再由事件服务轮询并推送给对应前端；不引入消息队列。事件订阅适用于用户端、志愿者端和管理端，具体可见事件由服务端按角色、辖区和工单关系过滤。

### 事件示例

```json
{
  "event": "case.assigned",
  "case_id": "case_01H8YQ",
  "volunteer_id": "volunteer_01H8YQ",
  "occurred_at": "2026-09-09T11:25:00+08:00"
}
```

### 事件类型

| 事件 | 说明 |
|---|---|
| `case.created` | 用户发起支援请求 |
| `case.assigned` | 志愿者承接 |
| `case.message.created` | 新消息 |
| `case.transfer.requested` | 发起交接 |
| `case.transfer.confirmed` | 交接确认 |
| `case.professional.requested` | 发起专业求助 |
| `case.professional.taken_over` | 专业督导接管 |
| `case.closed` | 本次服务结束 |
| `safety.escalated` | 用户发起即时安全求助 |
| `volunteer.capacity.updated` | 志愿者容量变化 |
| `resource.updated` | 资源更新 |
| `ai.suggestion.created` | AI 建议生成 |
| `system.degraded` | 服务降级 |

### 规则

- 事件负载只包含必要字段。
- 不把聊天全文、日记全文或精确位置放入事件。
- 断线重连后应按 `Last-Event-ID` 补发必要状态。
- 初期事件事实保存在 MySQL `events` 表，不依赖 MQ。
- 事件服务使用数据库轮询；如后续引入进程内唤醒或 MQ，只作为性能优化，不改变事实来源。
- 事件推送失败不改变业务状态；前端可通过 `GET /events/poll?after_event_id={id}` 兜底轮询。

### 兜底轮询接口

```http
GET /events/poll?after_event_id=evt_01H8YQ&limit=50
```

响应：

```json
{
  "events": [
    {
      "id": "evt_01H9AA",
      "event": "case.message.created",
      "case_id": "case_01H8YQ",
      "occurred_at": "2026-09-09T11:25:00+08:00"
    }
  ],
  "next_after_event_id": "evt_01H9AA",
  "has_more": false
}
```

---

## 5.18 `GET /manager/overview`

### 响应

```json
{
  "district_id": "district_shanghai_a",
  "generated_at": "2026-09-09T11:45:00+08:00",
  "active_cases": 8,
  "waiting_support": 2,
  "immediate_safety_pending": 1,
  "awaiting_transfer": 1,
  "available_volunteers": 3,
  "support_need_distribution": {
    "stable": 3,
    "attention": 2,
    "sustained_support": 1,
    "immediate_safety": 1,
    "unverified": 1
  }
}
```

### 统计规则

1. 同一用户在同一指标中去重。
2. 当日累计接待与当前同时服务分开统计。
3. 每个数字必须能追溯口径。
4. 不默认显示用户精确住址或实时坐标。

---

# 6. 权限矩阵

| 能力 | 用户 | 志愿者 | 值班负责人 | 专业督导 |
|---|---|---|---|---|
| 查看本人日记 | 可以 | 仅单独授权 | 不可以 | 仅专业需要且授权 |
| 查看会话全文 | 本人会话 | 当前承接会话 | 不可以 | 确认接管后 |
| 发起人工请求 | 可以 | 不适用 | 不适用 | 不适用 |
| 承接普通请求 | 不适用 | 可以 | 不可以 | 不占普通席位 |
| 专业接管 | 不适用 | 不可以 | 不可以 | 可以 |
| 分层复核 | 可以反馈 | 可以申请 | 不可以 | 可以 |
| 普通交接确认 | 不适用 | 发起 | 可以 | 专业接管优先 |
| 修改接待参数 | 不可以 | 不可以 | 可以 | 不可以 |
| 恢复接单 | 不适用 | 本人自检 | 不可以 | 不可以 |
| 查看资源库 | 可以 | 可以 | 可以 | 可以 |
| 管理 AI 配置 | 不可以 | 不可以 | 不可以 | 不可以，需独立 AI 配置管理员 |
| 查看审计日志 | 不可以 | 不可以 | 授权范围 | 授权范围 |

---

# 7. 并发、幂等与数据库化实现

初期不使用 Redis 或消息队列，以下规则全部由 MySQL 8 实现：

1. `POST /volunteer/cases/{case_id}/accept` 必须使用事务锁或数据库唯一约束，确保只有一个承接者。
2. `POST /manager/transfer-requests/{id}/confirm` 必须检查工单状态版本，防止过期确认。
3. `POST /me/safety/escalate` 必须幂等，重复提交不生成重复专业接管请求。
4. `POST /volunteer/transfer-requests` 必须幂等，重复提交不产生重复处理。
5. 日记分享和撤回必须版本化，避免旧版本继续被读取。
6. 设备指标写入应按 `user_id + metric_type + sampled_at` 去重。
7. AI 输出必须记录 `human_confirmed` 状态，未确认不能作为正式处置依据。
8. `Idempotency-Key` 保存于 `idempotency_keys` 表，并与业务写入在同一事务中提交。
9. 领域事件先写入 `events` 表，再推送；事件表是事件事实来源。
10. AI 配置发布、提示词、skills、tools 均保存版本记录，不允许原地覆盖历史版本。
11. 对高频列表查询使用索引、分页和必要汇总表；初期不通过缓存规避慢查询。

---

# 8. 数据最小化与回退

## 8.1 数据最小化

- 普通志愿者只获得当前服务所需摘要。
- 管理端只获得协调所需统计和状态。
- AI 请求只发送授权范围内的上下文。
- 事件流不包含聊天全文和日记全文。
- 精确位置默认不采集、不展示、不传播。

## 8.2 失败回退

| 场景 | 后端行为 | 前端表现 |
|---|---|---|
| 设备离线 | 返回 `quality=missing` | 显示“暂无数据”，不阻断求助 |
| AI 不可用 | 返回 `AI_UNAVAILABLE` | 保留人工会话和资源入口 |
| 督导未响应 | 返回 `waiting_confirmation` | 显示未接手、备用路径、现实渠道 |
| 资源过期 | 返回 `RESOURCE_UNAVAILABLE` | 显示备用资源，不显示过期信息 |
| 交接失败 | 返回冲突或容量错误 | 保留原负责人和责任 |
| 数据库失败 | 返回 500 | 保留本地待重试状态，不假装成功 |

---

# 9. 安全要求

1. 所有接口必须使用 HTTPS。
2. 数据库连接信息必须通过环境变量或密钥管理系统注入，禁止写入代码仓库、接口文档或前端配置。
3. Token 使用短有效期，并在服务端可撤销。
4. 敏感接口必须记录审计日志。
5. 志愿者和管理端接口必须绑定角色、辖区和资格有效期。
6. 未成年人接口必须检查年龄分支和授权状态。
7. AI 输出不得执行外部文档或用户输入中的指令。
8. 所有外部转介、外呼、通知联系人必须由人工确认并审计。
9. 用户可撤回授权，撤回后相关访问立即失效。
10. 不把聊天、日记、设备原始数据放入普通日志。
11. 演示数据与真实数据必须隔离。

---

# 10. DeepSeek API 参考

本文 AI 调用约定参考以下 DeepSeek 官方文档：

1. [DeepSeek API 文档首页](https://api-docs.deepseek.com/zh-cn/)
2. [Chat Completions API](https://api-docs.deepseek.com/zh-cn/api/create-chat-completion)
3. [JSON Output](https://api-docs.deepseek.com/zh-cn/guides/json_mode)
4. [Tool Calls](https://api-docs.deepseek.com/zh-cn/guides/tool_calls)
5. [思考模式](https://api-docs.deepseek.com/zh-cn/guides/thinking_mode)
6. [限速与隔离](https://api-docs.deepseek.com/zh-cn/quick_start/rate_limit)
7. [错误码](https://api-docs.deepseek.com/zh-cn/quick_start/error_codes)
8. [模型与价格](https://api-docs.deepseek.com/zh-cn/quick_start/pricing)

以上链接用于接口设计参考，不代表 DeepSeek 对本项目的安全、医疗或心理服务能力背书。
