# Fundamental Scan Pipeline

`fundamental-scan-pipeline` 是一个可移植的 Codex Skill，专门用于两件事：

- 针对一个或多个 A股 / 港股 / 美股标的物获取多源数据
- 基于标准化后的字段生成结构化中文深度报告

它不再面向以下能力：

- 扫描排名与打分榜单
- 覆盖率仪表盘
- 候选池数据库列表页
- 投资组合评分或仓位管理

这个 skill 的最终目标很明确：给定一个或多个标的代码，输出一份字段可追溯、证据可解释、结构固定的中文深度报告。

## Core Positioning

这个 skill 不是一个“简单抓数器”，也不是一个“扫描打分器”。

它的定位是一个可移植的“标的物深度分析引擎”，把多源数据获取、字段标准化、证据记录和中文深度报告生成串成一条稳定链路，用于回答两个问题：

- 这家公司到底靠什么赚钱，赚钱模式是否稳固
- 现在这个价格和时机，是否值得进一步介入或继续观察

## Repository Structure

```text
.
├── SKILL.md
├── README.md
├── agents/
│   └── openai.yaml
└── references/
    ├── workflow.md
    ├── canonical-schema.md
    ├── scoring-and-reporting.md
    └── api-contracts.md
```

## What This Skill Covers

### 1. Multi-Market Data Acquisition

- 支持 `CN / HK / US` 多市场标的物输入
- 支持单个标的或多个标的批量输入
- 支持分阶段抓取、失败重试、断点续跑、部分完成
- 支持多源回退与字段级来源记录

### 2. Canonical Normalization

- 将不同来源映射为统一 canonical schema
- 统一 `symbol / market / asset_type` 规范
- 对价格、成交额、估值、核心财务指标、行业、竞争对手等字段做标准化
- 区分缺失值、占位值和真实 0 值

### 3. Evidence And Freshness

- 记录 row 级别和 field 级别的 `source / confidence`
- 记录 `updated_at / evidence_updated_at / report_updated_at`
- 保留字段证据，不允许低质量回退静默覆盖高质量数据

### 4. Structured Chinese Deep Report

- 先生成结构化 `report_json`
- 再由同一份 `report_json` 渲染中文文本报告
- 支持一个标的一份报告，也支持多个标的逐个输出报告
- 输出结构固定，便于后续 API、导出或数据库落盘

## Report Evaluation Framework

这个 skill 的报告评估框架由两部分组成，并且是整合关系，不是二选一：

- `22步深度分析框架`：负责把一家公司从生意模式、财务、风险、管理层、估值、宏观、护城河、股价时机等角度做完整拆解
- `LEI 准则`：负责提供一组更偏筛选和定量门槛的“硬标准”，用于校验这个标的是否具备成为优质候选的基本条件

换句话说：

- `22步框架` 负责“看深”
- `LEI 准则` 负责“看硬”

报告输出时，两者会被整合到同一份结构化深度报告中。

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

这些步骤并不要求每次都拿到全部满量数据，但要求报告对每一项都给出以下三种状态之一：

- `已验证`
- `部分验证`
- `数据缺失待补`

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
2. 美股/大盘股等流动性不足的标的不应进入高优先级报告队列
3. 趋势判断不能替代基本面，但会影响“时机”章节的结论表达

## How The 22 Steps And LEI Work Together

两套框架的整合方式如下：

- `22步框架` 决定报告章节与推理顺序
- `LEI 准则` 作为其中的硬校验子模块，嵌入到“生意本质”“财务透视”“综合结论”“估值与时机”等章节
- 如果 `22步框架` 给出偏正面结论，但 `LEI` 多项不达标，报告必须明确标注“结论偏弱，硬标准不足”
- 如果 `LEI` 达标但 22 步中的风险、治理、护城河、估值环节出现明显问题，报告同样不能直接给出乐观结论

也就是说，这个 skill 的结论不是单一分数决定，而是：

- 先验证数据
- 再走 22 步结构化分析
- 再用 LEI 硬标准交叉校验
- 最后输出结构化中文结论

## Output Characteristics

最终报告需要体现以下标注特征：

- 每个核心结论尽量对应具体字段或数据来源
- 结论中明确区分“已证实”“高概率判断”“待补证据”
- 对缺失字段要指出缺口，而不是跳过不说
- 报告必须保留 `source / confidence / freshness` 语义，避免变成不可追溯的纯文本评论
- 多标的模式下，必须一标的一报告，不混写

## Reference Files

### [SKILL.md](./SKILL.md)

技能入口，定义触发场景、约束、校验方式和工作边界。

### [references/workflow.md](./references/workflow.md)

说明数据获取、回退、重试、断点续跑、partial completion 和报告生成顺序。

### [references/canonical-schema.md](./references/canonical-schema.md)

说明统一字段结构、symbol 规范、证据字段、merge 策略和报告存储对象。

### [references/scoring-and-reporting.md](./references/scoring-and-reporting.md)

说明结构化中文深度报告的 JSON 结构、中文章节、渲染规则和占位策略。

### [references/api-contracts.md](./references/api-contracts.md)

说明数据导入、抓取任务、报告查询和导出相关的 API 契约。

## Typical Usage

显式调用：

```text
Use $fundamental-scan-pipeline to implement a multi-market symbol data and Chinese report workflow.
```

中文调用：

```text
用 $fundamental-scan-pipeline 帮我实现针对一个或多个标的物的数据获取和结构化中文深度报告输出。
```

## Example Prompts

```text
Use $fundamental-scan-pipeline to fetch data for AAPL and MSFT, normalize the fields, and generate structured Chinese deep reports.
```

```text
用 $fundamental-scan-pipeline 针对 0700.HK、600519.SH 和 NVDA 生成结构化中文深度报告。
```

```text
Use $fundamental-scan-pipeline to add fallback data sources and evidence metadata for one-symbol and multi-symbol report generation.
```

```text
用 $fundamental-scan-pipeline 把报告输出统一成 report_json 和中文文本两种形式，并把 22 步分析和 LEI 准则整合进去。
```

## Recommended Workflow For Another Codex

1. 先读 `SKILL.md`
2. 明确本次任务是在改数据获取、schema、报告结构，还是 API
3. 只加载相关 reference，不要无关扩展
4. 先用一个标的做 smoke test，再验证多标的
5. 分开验证原始数据新鲜度、证据新鲜度和报告新鲜度

## Portability Notes

- 这是文档型 skill，不绑定某个具体框架
- 可用于 Flask、FastAPI、Django、Node.js 或其他后端
- 核心可移植能力只有两类：
  - 数据获取与标准化
  - 结构化中文深度报告生成

## Current Target Domain

默认面向交易辅助系统中的“标的物深度分析”能力，适用于：

- 单标的深度分析
- 多标的批量报告生成
- 报告前的数据标准化与证据保留
- 后续导出、审阅、归档或数据库持久化

## License / Reuse

当前仓库未附加单独许可证文件。若要公开复用，建议后续补充 `LICENSE` 并明确商用/非商用范围。
