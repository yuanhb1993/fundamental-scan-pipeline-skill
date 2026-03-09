# Fundamental Scan Pipeline

`fundamental-scan-pipeline` 是一个面向交易辅助系统的可移植 Codex Skill，用于构建和扩展多市场标的物数据获取、标准化、证据追踪、基本面评分、覆盖率诊断和结果交付流程。

它适合这类任务：

- 构建 A股 / 港股 / 美股 的证券基础 universe
- 将不同数据源映射为统一 canonical schema
- 为字段记录 `source / confidence / evidence`
- 运行硬过滤、LEI 风格规则和加权评分
- 生成中文深度报告文本
- 实现 staged fetch job、失败重试、断点续跑、部分完成
- 输出 API、数据库视图、CSV 导出和前端结果列表

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

### 1. Data Refresh Workflow

- 多市场分阶段刷新：`CN_REFRESH`、`US_REFRESH`、`HK_REFRESH`、`SCAN_SCORE`
- 按市场选择执行，未选中的阶段标记为 `skipped`
- 支持阶段内重试
- 支持失败后断点续跑
- 支持部分完成，不要求所有市场都成功才能产出扫描结果

### 2. Canonical Normalization

- 统一 `symbol / market / asset_type` 规范
- 统一股票与 ETF 的基础字段结构
- 将不同源的价格、流动性、财务数据映射到同一 row schema
- 保留源侧元数据，例如 `currency_note`、`updated_at`

### 3. Evidence And Confidence

- 记录 row 级别来源与置信度
- 记录 field 级别来源与置信度
- 区分“真实 0”与“缺失值”
- 支持覆盖率仪表盘和未覆盖原因 TopN

### 4. Fundamental Evaluation

- 股票与 ETF 使用不同硬过滤规则
- 支持 LEI 风格规则命中分
- 支持业务、财务、风险、估值、资本效率等多维加权总分
- 输出 `进入深度分析 / 进入观察池 / 暂不纳入` 等中文决策标签

### 5. Report And Delivery

- 生成结构化中文深度报告
- 支持 API 结果查询
- 支持数据库最终结果页
- 支持 CSV 导出，并把报告内容转为文本列

## Reference Files

### [SKILL.md](./SKILL.md)

技能入口。给另一个 Codex 提供何时触发、如何选择参考文件、有哪些操作规则。

### [references/workflow.md](./references/workflow.md)

说明 staged refresh、fallback、失败处理、partial completion、freshness model。

### [references/canonical-schema.md](./references/canonical-schema.md)

说明 canonical row 结构、symbol 规范、evidence 字段、merge 策略和持久化对象。

### [references/scoring-and-reporting.md](./references/scoring-and-reporting.md)

说明硬过滤、LEI 风格规则、组件评分、总分、风险标记和报告结构。

### [references/api-contracts.md](./references/api-contracts.md)

说明 source import、manual scan、schedule、results、coverage、database view 和 fetch-job API 契约。

## Typical Usage

在 Codex 对话中显式调用：

```text
Use $fundamental-scan-pipeline to implement a multi-market fundamental scan backend.
```

或者中文直接描述：

```text
用 $fundamental-scan-pipeline 帮我实现 A股/港股/美股 的分阶段数据获取、标准化和评分系统。
```

## Example Prompts

```text
Use $fundamental-scan-pipeline to add staged refresh with retry and resume for CN/HK/US markets.
```

```text
用 $fundamental-scan-pipeline 统一现有的 universe schema，补上字段级 source/confidence。
```

```text
Use $fundamental-scan-pipeline to add scan coverage metrics and missing-reason diagnostics.
```

```text
用 $fundamental-scan-pipeline 把扫描结果、深度报告和 CSV 导出统一到同一套字段。
```

## Recommended Workflow For Another Codex

1. 先读 `SKILL.md`
2. 根据任务类型加载对应 reference
3. 明确当前修改的是 source adapter、schema、score、API 还是 UI
4. 先做单市场 smoke test，再扩大范围
5. 分开验证 raw freshness、scan freshness 和 UI freshness

## Portability Notes

- 这个 skill 是文档型 skill，不绑定某个固定数据库实现或框架
- 你可以把它用于 Flask、FastAPI、Django、Node.js 或其他后端
- 核心要求是保留同样的流程语义：
  - source refresh
  - canonical normalization
  - evidence tracking
  - coverage diagnostics
  - scoring
  - report generation
  - staged fetch job orchestration

## Current Target Domain

默认目标场景是交易辅助系统里的“基本面扫描与优质标的数据库”能力，适用于：

- 候选标的预筛
- 基本面评分入池
- 深度分析前的数据预处理
- 数据覆盖率和缺失原因诊断

## License / Reuse

当前仓库未附加单独许可证文件。若要公开复用，建议后续补充 `LICENSE` 并明确商用/非商用范围。
