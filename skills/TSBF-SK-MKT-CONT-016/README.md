# TSBF-SK-MKT-CONT-016｜内容战略设计

- 类型：Skill
- 定位：把品牌定位和客户决策阶段转化为有主题、有节奏、有指标、有 CTA 的内容战略。
- 输入：`TSBF-SK-MKT-STR-004` 的定位输出、目标客户、漏斗目标、渠道约束、内容资产、发布节奏。
- 输出：内容目标、客户阶段地图、内容支柱、四周主题日历、内容格式、指标、CTA、风险、假设和人工门。
- 适用场景：季度内容规划、品牌发布、获客活动、销售协同内容和营销团队周/月度排期。
- 不适用：直接生成最终广告文案、未经审核发布内容，或在没有定位输入时假定客户需求。
- 依赖：`TSBF-SK-MKT-STR-004`、`MARKETING-OS`、`TSBF-SK-AIO-001`。
- Owner：MKT
- 来源：internal
- License：internal
- 安全审查：`designed / 2026-10-08`；只生成 draft，不连接渠道。
- 验证：`passed / CASE-001`；承接 STR-004 输出的结构验收通过，真实内容效果待验证。
- 状态：validated（合成 MVP）
- 版本：0.2.0
- 下一步：用真实渠道历史数据检验主题优先级、互动质量和合格线索贡献。

## Agent 设计

- 模式：Prompt Chaining、Reflection、Guardrails、Human-in-the-Loop、Evaluation。
- 阶段：`map_funnel → define_pillars → draft_calendar → critic_check → final_strategy`。
- 关键判断：每个主题是否对应客户阶段、是否复用定位证据、是否有可观察指标、CTA 是否与成熟度匹配。
- 人工门：对外发布、品牌敏感议题、案例与数据引用必须人工审核。
- 副作用：无；只输出策略、排期草案和待审核内容 Brief。

## 输入契约

```yaml
positioning:
  positioning_statement: string
  target_customer: string
  job_to_be_done: string
  differentiators: [string]
  reasons_to_believe: [string]
audience_stage: awareness | consideration | decision | retention
business_goal: string
channels: [string]
cadence: string
available_assets: [string]
constraints: [string]
```

必填：定位陈述、目标客户、业务目标、至少 1 个渠道。定位状态不是 `accepted` 时，输出 `needs_input`。

## 输出契约

```yaml
status: accepted | needs_input | human_gate
objective: string
funnel_map: [{stage: string, job: string, content_role: string}]
content_pillars: [{name: string, promise: string, proof_source: string}]
calendar: [{week: number, topic: string, format: string, channel: string, stage: string, cta: string}]
metrics: [{name: string, definition: string, review_cycle: string}]
risks: [string]
assumptions: [string]
guardrails: [string]
human_gate:
  required: true
  reason: string
side_effects: none
audit:
  skill_id: TSBF-SK-MKT-CONT-016
  upstream_skill: TSBF-SK-MKT-STR-004
  case_id: string
```

幂等键为 `case_id + upstream_output_hash + version`；合成演示超时 5 秒、失败最多重试 1 次，任何渠道动作都保留为 `draft`。

## 最小验收

- 至少 3 个内容支柱，且每个支柱有承诺和证明来源。
- 至少 4 周排期，覆盖认知、考虑、决策中的至少 2 个阶段。
- 每个排期项都有主题、格式、渠道、客户阶段和 CTA。
- 指标可定义、可复盘，不把曝光直接等同于收入。
- 可把输出交给 `TSBF-SK-MKT-DEM-040` 生成 draft 自动化流程。

