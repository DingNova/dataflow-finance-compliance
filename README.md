# dataflow-finance-compliance

> DataFlow Community Extension：金融文本合规治理。敏感信息脱敏 → 合规规则过滤 → 风险评估 → 免责声明注入，端到端流水线 + 30 条金标评测集 + 数据卡与合规说明。
>
> 简体中文 ｜ [English](#english) ｜ License: Apache-2.0

## ✨ 功能

| 算子 | 类型 | 说明 |
|---|---|---|
| `FinanceSensitiveInfoMasker` | 脱敏 | 身份证/手机号/银行卡/邮箱做确定性类型化掩码，输出脱敏文本与逐类计数摘要（无需 LLM） |
| `ComplianceRuleFilter` | 过滤 | 内置收益承诺、荐股诱导、夸大宣传三类合规规则，LLM 逐条判定并保留违规原因 |
| `ComplianceRiskSampleEvaluator` | 评估 | LLM 逐条评定合规风险等级（low/medium/high）并给出理由，用于审计抽检与风险报告 |
| `DisclaimerInjector` | 注入 | 按文档类型（研报/公告/营销材料）追加标准风险提示或免责声明（无需 LLM） |

流水线顺序为 **mask → filter → evaluate → inject**：脱敏在数据进入 LLM 环节之前执行。

Pipeline：`FinanceCompliancePipeline`，配套 YAML 配置。

## 🚀 快速开始

```bash
# 1. 安装主框架与扩展
pip install open-dataflow
pip install -e .

# 2. 配置 LLM 服务
export DF_API_KEY=sk-xxx

# 3. 运行 Pipeline
python -c "
from dataflow.utils.storage import FileStorage
from dataflow.serving import APILLMServing_request
from dataflow_finance_compliance import FinanceCompliancePipeline

storage = FileStorage(first_entry_file_name='./input.jsonl')  # [{'text': ..., 'doc_type': ...}]
llm = APILLMServing_request(api_url='https://api.openai.com/v1/chat/completions')
FinanceCompliancePipeline(llm_serving=llm, storage=storage).run()
"
```

## 📊 评测集

- `eval/finance_compliance_eval_set.jsonl`：30 条人工金标（模拟撰写数据，不含真实客户信息）
- `eval/expand_evalset.py`：LLM 扩充（**发布前须人工抽检 ≥20%**，见 COMPLIANCE.md）

## 🗂 数据卡与合规

- `data_cards/`：数据卡模板与示例（来源/口径/敏感性/质量五节）
- `COMPLIANCE.md`：法规映射（个保法 / 数据安全法 / JR/T 0197 / 广告法）、C3 级数据先脱敏后外发要求、使用边界

## 🧪 测试

```bash
pip install -e ".[dev]" && pytest -v
```

离线测试：通过 stub 替身模拟 DataFlow 框架，无需安装 open-dataflow / 任何 LLM 服务，CI 可直接运行。

## 📖 相关资源

- 上游框架：[OpenDCAI/DataFlow](https://github.com/OpenDCAI/DataFlow)（SIGMOD'27）
- 生态指南：[DataFlow-Doc Ecosystem](https://opendcai.github.io/DataFlow-Doc/zh/guide/df_ecosystem/)
- 同系列扩展：[dataflow-finance-qa](https://github.com/DingNova/dataflow-finance-qa) · [dataflow-finance-text2sql](https://github.com/DingNova/dataflow-finance-text2sql) · [dataflow-finance-eval](https://github.com/DingNova/dataflow-finance-eval)

## <a id="english"></a>English

`dataflow-finance-compliance` is a [DataFlow](https://github.com/OpenDCAI/DataFlow) community extension for financial text compliance governance. It ships 4 operators (`FinanceSensitiveInfoMasker` → `ComplianceRuleFilter` → `ComplianceRiskSampleEvaluator` → `DisclaimerInjector`) — deterministic PII masking runs **before** any LLM stage — one end-to-end pipeline, a 30-sample golden eval set, data-card templates and a compliance guide aligned with PIPL / Data Security Law / JR/T 0197. All operator descriptions support `get_desc(lang="zh"/"en")`.

---

⚠️ **Disclaimer**: for research and engineering purposes only. Not investment advice.
