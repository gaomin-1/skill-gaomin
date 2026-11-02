# TSBF-SK-MKT-DEM-040｜营销自动化编排

- 类型：Skill
- 定位：把内容策略和获客规则编排成可检查的触发器、分支、任务、SLA 与人工审批流程。
- 输入：`TSBF-SK-MKT-CONT-016` 的内容战略、渠道约束、线索规则、审批政策、负责人和 SLA。
- 输出：事件触发器、条件分支、draft 动作、人工确认门、恢复路径、SLA、审计记录和副作用声明。
- 适用场景：线索培育设计、活动跟进、营销与销售协同、自动化方案评审和课堂演示。
- 不适用：直接发送邮件/消息、修改 CRM、改变广告预算或替负责人做高风险客户判断。
- 依赖：`TSBF-SK-MKT-CONT-016`、`TSBF-SK-MKT-STR-004`、`MARKETING-OS`。
- Owner：MKT
- 来源：internal
- License：internal
- 安全审查：`designed / 2026-10-08`；所有外部动作默认 draft，人工确认前禁止执行。
- 验证：`passed / CASE-001`；触发器、分支、人工门和无副作用验收通过，真实系统适配待验证。
- 状态：validated（合成 MVP）
- 版本：0.2.0
- 下一步：选择一个真实但脱敏的渠道，验证事件字段、负责人 SLA 和人工审批路径。

## Agent 设计

- 模式：Prompt Chaining、Reflection、Guardrails、Human-in-the-Loop、Evaluation。
- 阶段：`normalize_strategy → map_trigger → design_branches → add_approval_gate → recovery_check`。
- 关键判断：触发事件是否可观测、分支是否互斥且可解释、动作是否具备负责人和时限、是否触发外部副作用。
- 人工门：发送、写 CRM、销售分派、优惠承诺和任何客户状态改变前必须人工确认。
- 副作用：MVP 为 `none`；只生成 `draft` action 和执行计划。

## 输入契约

```yaml
content_strategy:
  objective: string
  calendar: [{topic: string, channel: string, cta: string}]
  metrics: [{name: string, definition: string}]
channel_constraints: [string]
lead_rules:
  icp_signals: [string]
  qualification_threshold: number
approval_policy: string
owner: string
sla_hours: number
```

必填：内容目标、至少 1 个触发来源、审批政策、负责人。任何要求真实发送或写入时，仍只返回 `human_gate`。

## 输出契约

```yaml
status: draft | human_gate | needs_input
workflow_name: string
triggers: [{event: string, source: string, dedupe_key: string}]
branches: [{name: string, condition: string, next: string}]
actions: [{type: string, mode: draft, owner: string, due_hours: number}]
approval_gate:
  required: true
  before: [string]
recovery: [{failure: string, response: string, escalation: string}]
sla: {owner: string, response_hours: number}
guardrails: [string]
side_effects: none
audit:
  skill_id: TSBF-SK-MKT-DEM-040
  upstream_skill: TSBF-SK-MKT-CONT-016
  case_id: string
```

幂等键为 `case_id + event_id + version`；合成演示超时 5 秒、失败最多重试 1 次；重复事件必须被去重，任何失败都保留待人工处理的恢复记录。

## 最小验收

- 至少 1 个明确触发器、2 个可解释分支、3 个 draft 动作。
- 每个动作都有负责人、时限和 `mode: draft`。
- 明确外部动作前的人工确认门和失败恢复路径。
- 明确 `side_effects: none`，不伪造发送、写入或转化结果。
- 能承接 `TSBF-SK-MKT-CONT-016` 的内容排期并形成可演示工作流。

