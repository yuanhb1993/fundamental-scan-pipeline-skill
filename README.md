# Fundamental Scan Pipeline

`fundamental-scan-pipeline` 是一个可移植的 Codex Skill，专门用于两件事：

- 针对一个或多个 A股 / 港股 / 美股标的物获取多源数据
- 基于标准化后的字段生成结构化中文深度报告

默认输出不再面向以下能力：

- 扫描排名与打分榜单
- 覆盖率仪表盘
- 候选池数据库列表页
- 投资组合评分或仓位管理
- 默认交易执行建议（仓位、止盈、止损、精确入场）

这个 skill 的目标是：给定一个或多个标的代码或公司名，先完成实体解析与数据获取，再输出一份字段可追溯、证据可解释、结构固定的深度报告。

## Core Positioning

这个 skill 不是“抓一点数据就写评论”，也不是“给股票打个分”。

它的定位是一个可移植的“标的物深度分析协议”，把以下过程固定下来：

1. 实体解析
2. 多源数据获取
3. 字段标准化
4. 证据与置信度记录
5. `22步深度分析框架` 校验
6. `LEI 准则` 交叉校验
7. 输出 `report_json`
8. 从同一份 `report_json` 渲染结构化 Markdown 中文报告

## Repository Structure

```text
.
├── SKILL.md
├── README.md
├── agents/
│   └── openai.yaml
├── assets/
│   ├── report_json.template.json
│   └── report_markdown_zh.template.md
└── references/
    ├── workflow.md
    ├── canonical-schema.md
    ├── scoring-and-reporting.md
    ├── api-contracts.md
    └── output-template.md
```

## Core Output Contract

这个 skill 的默认输出必须同时包含两份内容：

### 1. `report_json`

这是主输出，必须先生成。它负责承载：

- 实体解析结果
- 规范化字段
- 来源与置信度
- 22 步分析状态
- LEI 校验状态
- 事实层结论
- 推断层结论
- 缺失项与待补数据

### 2. `report_markdown_zh`

这是面向阅读的结构化 Markdown 文档，必须由 `report_json` 渲染而来，不能独立自由发挥。

它至少应包含：

- 报告头
- 必须补充信息
- 实体解析
- 22 步状态矩阵
- LEI 校验矩阵
- 生意本质
- 财务透视
- 风险与竞争力
- 管理层洞察
- 综合结论
- 估值与时机
- 逆向思考
- 下一步数据需求

## What This Skill Covers

### 1. Multi-Market Data Acquisition

- 支持 `CN / HK / US` 多市场标的物输入
- 支持单个标的或多个标的批量输入
- 支持分阶段抓取、失败重试、断点续跑、部分完成
- 支持多源回退与字段级来源记录

### 2. Entity Resolution

- 输入可以是公司名、股票代码或混合列表
- 每个输入都必须先解析成明确的上市主体
- 解析结果必须显式记录，而不是在正文中隐式跳过
- 如果实体存在歧义，必须输出歧义状态而不是直接继续下结论

### 3. Canonical Normalization

- 将不同来源映射为统一 canonical schema
- 统一 `symbol / market / asset_type` 规范
- 对价格、成交额、估值、核心财务指标、行业、竞争对手等字段做标准化
- 区分缺失值、占位值和真实 0 值

### 4. Evidence And Freshness

- 记录 row 级别和 field 级别的 `source / confidence`
- 记录 `updated_at / evidence_updated_at / report_updated_at`
- 区分一级来源、二级来源、三级来源
- 保留字段证据，不允许低质量回退静默覆盖高质量数据

### 5. Structured Chinese Deep Report

- 先生成结构化 `report_json`
- 再生成 `report_markdown_zh`
- 支持一个标的一份报告，也支持多个标的逐个输出报告
- 输出结构固定，便于 API、导出或数据库落盘

## Report Evaluation Framework

这个 skill 的报告评估框架由两部分组成，并且是整合关系，不是二选一：

- `22步深度分析框架`：负责把一家公司从生意模式、财务、风险、管理层、估值、宏观、护城河、股价时机等角度做完整拆解
- `LEI 准则`：负责提供一组更偏筛选和定量门槛的“硬标准”，用于校验这个标的是否具备成为优质候选的基本条件

换句话说：

- `22步框架` 负责“看深”
- `LEI 准则` 负责“看硬”

报告输出时，两者必须整合到同一份结构化深度报告中，并带状态标注。

## Integrated 22-Step Analysis Skeleton

报告主体按照以下 22 个问题组织证据与判断：

1. 公司一句话是靠什么赚钱的
2. 利润表质量分析
3. 资产负债表风险分析
4. 现金流质量分析
5. ROE、杜邦分解、周转率等关键比率分析
6. 生意模式的颠覆性风险与核心竞争力分析
7. 管理层讨论与分析（MD&A）兑现度检查
8. 与竞争对手和公司历史的横向、纵向对比
9. 决策层资本配置与长期增长质量分析
10. 新赛道与新增量来源分析
11. 当前股价与介入时机分析
12. 当前市盈率是否合理，是否存在泡沫或估值陷阱
13. 宏观环境与政策渠道如何影响公司未来损益
14. 护城河是否真实存在，价格战下能否守住盈利能力
15. 近30天股价波动的驱动因素拆解
16. 公司及分项维度评分
17. 当前价位下买入性价比评分
18. 介入价位、仓位、加减仓、止盈止损建议
19. 客户与供应商集中度风险分析
20. ROIC 与 WACC 的长期比较
21. 股权激励、股东增减持、审计意见等治理信号检查
22. 股息率、利润构成、可持续性与“好公司、好价格”判断

但默认报告协议要求：

- 每一步都必须标记 `已验证 / 部分验证 / 数据缺失待补`
- 每一步都应能回溯到字段与来源
- 第 16/17/18 步若未显式启用 `trade_overlay`，只能作为分析占位项，不得输出伪精确交易建议

## Integrated LEI Rule Set

LEI 准则作为报告中的“硬标准校验层”，主要包括：

### 商业模式定性标准

1. 产品是否标准化、无差别
2. 潜在用户是否足够广泛
3. 是否具备自由定价权
4. 是否具备市场垄断地位或强护城河
5. 所在行业是否属于“提高社会效率”或“提高生活水准”的方向

### 财务表现定量标准

1. 年收入通常不低于 `1亿美元`
2. 理想收入增速通常在 `15% - 20%`
3. `EBIT Margin` 应为正且越高越好
4. `Operating Cash Flow` 应为正
5. `Free Cash Flow` 最好为正
6. `总债务` 不应明显超过 `总收入`

### 行业地位与交易前提

1. 尽量优先行业龙头，不优先行业第二名或边缘跟随者
2. 流动性不足的标的不应进入高优先级结论
3. 趋势判断不能替代基本面，但会影响“估值与时机”章节的边界表达

## Output Guardrails

默认模式下，报告必须遵守以下边界：

- 必须先输出实体解析结果
- 必须先有 `report_json`，再有 `report_markdown_zh`
- 事实层与推断层必须分开
- 不允许把缺失字段包装成肯定结论
- 不允许把媒体报道等三级来源直接写成已验证事实
- 不允许默认输出仓位、止损、止盈、明确买点
- 不允许在没有公式与时间戳的情况下输出伪精确估值结论

如果任务显式要求交易叠加层，才允许额外输出：

- `trade_overlay_json`
- 仓位建议
- 风险回报
- 止损止盈
- 技术位与时机备忘

## Output Characteristics

最终报告需要体现以下标注特征：

- 每个核心结论尽量对应具体字段或数据来源
- 结论中明确区分“已证实”“高概率判断”“待补证据”
- 对缺失字段要指出缺口，而不是跳过不说
- 报告必须保留 `source / confidence / freshness` 语义
- 多标的模式下，必须一标的一报告，不混写
- Markdown 文档必须结构化，不接受纯散文式长文

## Reference Files

### [SKILL.md](./SKILL.md)

技能入口，定义触发场景、约束、校验方式和工作边界。

### [references/workflow.md](./references/workflow.md)

说明实体解析、数据获取、回退、重试、断点续跑、partial completion 和报告生成顺序。

### [references/canonical-schema.md](./references/canonical-schema.md)

说明统一字段结构、symbol 规范、证据字段、merge 策略和报告存储对象。

### [references/scoring-and-reporting.md](./references/scoring-and-reporting.md)

说明结构化中文深度报告的 JSON 结构、22 步状态、LEI 状态、Markdown 渲染规则和占位策略。

### [references/api-contracts.md](./references/api-contracts.md)

说明数据导入、抓取任务、报告生成、报告查询和导出相关的 API 契约。

### [references/output-template.md](./references/output-template.md)

说明如何复用标准输出模板，并指向实际可复制的 JSON 与 Markdown 样例文件。

### [assets/report_json.template.json](./assets/report_json.template.json)

标准 `report_json` 样例，可直接作为产物模板或测试基线。

### [assets/report_markdown_zh.template.md](./assets/report_markdown_zh.template.md)

标准 `report_markdown_zh` 样例，可直接作为渲染输出模板。

## Typical Usage

显式调用：

```text
Use $fundamental-scan-pipeline to implement a multi-market symbol data and Chinese report workflow with report_json and structured Markdown output.
```

中文调用：

```text
用 $fundamental-scan-pipeline 帮我实现针对一个或多个标的物的数据获取、结构化中文深度报告，以及 JSON + Markdown 双输出。
```

## Example Prompts

```text
Use $fundamental-scan-pipeline to fetch data for AAPL and MSFT, resolve entities explicitly, and generate report_json plus structured Markdown Chinese reports.
```

```text
用 $fundamental-scan-pipeline 针对 0700.HK、600519.SH 和 NVDA 生成逐标的 report_json 和结构化 Markdown 深度报告。
```

```text
Use $fundamental-scan-pipeline to add fallback data sources, entity resolution, and evidence metadata for one-symbol and multi-symbol report generation.
```

```text
用 $fundamental-scan-pipeline 把报告输出统一成 report_json 和 Markdown 两种形式，并把 22 步分析和 LEI 准则整合进去。
```

## Recommended Workflow For Another Codex

1. 先读 `SKILL.md`
2. 明确本次任务是在改实体解析、数据获取、schema、报告结构，还是 API
3. 只加载相关 reference，不要无关扩展
4. 先用一个标的做 smoke test，再验证多标的
5. 分开验证原始数据新鲜度、证据新鲜度和报告新鲜度
6. 检查 JSON 与 Markdown 是否逐段一致

## Portability Notes

- 这是文档型 skill，不绑定某个具体框架
- 可用于 Flask、FastAPI、Django、Node.js 或其他后端
- 核心可移植能力只有三类：
  - 实体解析
  - 数据获取与标准化
  - 结构化中文深度报告生成（JSON + Markdown）

## Current Target Domain

默认面向交易辅助系统中的“标的物深度分析”能力，适用于：

- 单标的深度分析
- 多标的批量报告生成
- 报告前的数据标准化与证据保留
- 后续导出、审阅、归档或数据库持久化

## License / Reuse

当前仓库未附加单独许可证文件。若要公开复用，建议后续补充 `LICENSE` 并明确商用/非商用范围。
