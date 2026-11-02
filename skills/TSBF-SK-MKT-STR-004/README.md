# TSBF-SK-MKT-STR-004｜品牌定位设计

- 类型：Skill
- 定位：把产品事实、目标客户、关键问题和可信证据整理成可用于营销与产品协同的品牌定位。
- 输入：产品/服务资料、目标市场、客户痛点、购买任务、竞品差异、已有证明、业务约束。
- 输出：定位陈述、目标客户、Job-to-be-Done、差异化、Reasons-to-Believe、边界、假设与置信度。
- 适用场景：品牌战略启动、产品上市、营销 Brief、销售话术和内容战略的上游输入。
- 不适用：没有事实材料时直接编造市场结论；替代市场研究、法务审查或最终品牌决策。
- 依赖：`MARKETING-OS`、`TSBF-SK-AIO-001`、`TSBF-WF-AIO-001`；可选接收客户研究和竞品分析结果。
- Owner：MKT
- 来源：internal
- License：internal
- 安全审查：`designed / 2026-10-08`；不连接外部系统，不发布品牌承诺。
- 验证：`passed / CASE-001`；结构验收与合成链路通过，真实消费者资料待验证。
- 状态：validated（合成 MVP）
- 版本：0.2.0
- 下一步：用 3 份脱敏品牌资料进行人工复核，记录定位采纳率和反例。

## Agent 设计

- 模式：Prompt Chaining、Reflection、Guardrails、Human-in-the-Loop、Evaluation。
- 阶段：`normalize_input → draft_positioning → critic_check → final_positioning`。
- 关键判断：客户是否足够具体、问题是否可观察、差异化是否有证据、承诺是否超出输入事实。
- 人工门：品牌主张、市场选择和对外承诺在发布前必须由品牌负责人确认。
- 副作用：无；只生成草稿和审计记录。

## 输入契约

```yaml
brand:
  name: string
  product: string
  category: string
target_market: string
customer_pains: [string]
jobs_to_be_done: [string]
proof_points: [string]
competitor_alternatives: [string]
constraints: [string]
```

必填：`brand.name`、`brand.product`、`target_market`、至少 1 条 `customer_pains`。缺失必填字段时输出 `needs_input`，不得自动补齐为事实。

## 输出契约

```yaml
status: accepted | needs_input | human_gate
positioning_statement: string
target_customer: string
job_to_be_done: string
differentiators: [string]
reasons_to_believe: [string]
boundaries: [string]
confidence: high | medium | low
assumptions: [string]
guardrails: [string]
human_gate:
  required: true
  reason: string
side_effects: none
audit:
  skill_id: TSBF-SK-MKT-STR-004
  case_id: string
  stages: [string]
```

幂等键为 `case_id + input_hash + version`；合成演示超时 5 秒、失败最多重试 1 次，失败记录 `recoverable` 审计事件。

## 最小验收

- 输出明确服务对象、核心问题和选择理由。
- 至少有 2 个差异化，且每个差异化都能指向输入中的证明或明确标记为待验证。
- 不出现输入未提供的客户规模、市场份额、收入或效果承诺。
- 输出含 `confidence`、`assumptions`、`guardrails`、`human_gate`、`side_effects`。
- 可作为 `TSBF-SK-MKT-CONT-016` 的输入继续运行。

