# Context 结构契约

本契约是 `deep-code-analyzer` 的唯一数据通道：
所有原子技能只读写本契约定义的字段，禁止依赖隐式上下文传递。

## 统一证据格式

所有分析性结论必须附带标准化的 `evidence` 对象，禁止使用自由文本或仅标注文件路径：

```json
{
  "evidence": {
    "type": "source_line | lsp_call | search_result | config_fragment",
    "source": "src/opensquilla/engine/pipeline.py:42",
    "tool": "read_file | lsp.goToDefinition | search_content",
    "origin": "repo_verified | repo_inferred | external_read | external_unread",
    "raw_output_snippet": "def process(ctx: PipelineContext) -> PipelineResult:"
  }
}
```

| 字段 | 说明 |
|------|------|
| `type` | 证据类型：源码行、LSP 调用结果、文本搜索结果、配置片段 |
| `source` | 证据出处（文件路径:行号 或 工具调用标识） |
| `tool` | 获取证据所使用的工具 |
| `origin` | **来源可信度**：结论是实测还是推断、源码在仓库内还是外部（详见下方"来源可信度四态"） |
| `raw_output_snippet` | 原始输出的关键片段（截取 ≤ 200 字符） |

### 来源可信度四态（origin）

所有证据必须标注 `origin`，禁止缺省。这是可复核性的第一道闸门——"这个结论是读文件得出的还是猜的"必须当场写清：

| origin 值 | 含义 | 触发场景 |
|-----------|------|----------|
| `repo_verified` | 仓库内源码**直接读取/跳转验证** | 读到文件、LSP 确认符号定义 |
| `repo_inferred` | 仓库内**推断**得出，未直接读取 | 从 import 结构、命名约定、配置推断 |
| `external_read` | **外部依赖源码已读取**（venv/pnpm store 等） | 定位到外部包安装路径并读到源码 |
| `external_unread` | **外部依赖未读**，仅知 import 存在 | 无法定位/读取外部包源码，**必须附定位命令** |

> 关键规则：技术栈版本、覆盖率、文件数等**量化数字**必须标注 `origin`（通常为 `repo_verified`——从 `package.json`/`pyproject.toml` 直接读到）。未标来源的量化数字视为 `repo_inferred`，报告中按推断对待。

### 批量证据降级规则
同类证据 ≥ 5 条时，允许合并为证据表（`source | line | confidence` 三列），避免报告膨胀。单条关键结论（直接决定流转闸门、或证伪假设的结论）仍必须使用完整 JSON 证据对象。

## 六级置信度

所有分析性结论使用以下置信度标注，替代旧的二元 `verified` 判断：

| 级别 | 含义 | 触发条件 |
|------|------|----------|
| `confirmed` | 已通过 LSP 或源码直接验证 | 跨文件跳转经 `goToDefinition` 确认 |
| `high` | 高置信度，单文件内直接可见 | 同文件内的直接引用关系 |
| `medium` | 推断型结论，需进一步验证 | 从 import 结构、命名约定推断 |
| `low` | 推测型，证据不足 | 仅有间接线索 |
| `speculative` | 猜测型，无法验证 | 黑盒/闭源/外部依赖未读部分（`origin=external_unread` 时必须降级至此或 `low`） |
| `disproven` | 初始假设已被证伪 | 假设表中原假设被否定 |

> 置信度与来源的联动规则：`origin=external_unread` 的结论置信度**不得超过 `low`**；`origin=repo_inferred` 的结论置信度**不得超过 `high`**。来源不足时置信度必须同步降级，禁止"来源推断、置信度却标 confirmed"。

## 主结构

```json
{
  "input": {
    "project_path": "",
    "target_subsystem": "",
    "supplementary_materials": [],
    "output_dir": ""
  },
  "stage_outputs": {
    "S1_landscape": {
      "project_name": "",
      "project_type": "",
      "tech_stack": {},
      "entry_points": {},
      "design_principles": [],
      "key_dirs": [],
      "docs_map": {
        "docs_root": "",
        "docs_source_type": "",
        "tree": [],
        "read_budget": {},
        "declared_tech_stack": {},
        "declared_modules": [],
        "declared_architecture": [],
        "glossary": [],
        "drift_signals": []
      },
      "anomaly_signals": [],
      "assumptions": []
    },
    "S2_modules": {
      "source_root": "",
      "coverage": 0,
      "uncovered_dirs": [],
      "modules": [],
      "non_source_dirs": []
    },
    "S3_dataflow": {
      "entry_point": "",
      "segments": [],
      "unverified_hypotheses": [],
      "diagram_mermaid": "",
      "self_check": {},
      "unreachable_paths": []
    },
    "S4_patterns": {
      "design_patterns": [],
      "architecture_principles": [],
      "coding_conventions": [],
      "security_designs": [],
      "total_count": 0,
      "fallback_note": ""
    },
    "S5_deep_dives": [],
    "S5_history": [],
    "S6_feature_studies": [],
    "S6_history": [],
    "S6B_defect_scans": [],
    "S6B_history": [],
    "S7_report": {
      "output_dir": "",
      "files_generated": [],
      "mandatory_sections_coverage": {},
      "fallback_mode": false
    },
    "S8_measurements": []
  },
  "human_decisions": {
    "skip_deep_dive": false,
    "deep_dive_targets": [],
    "skip_feature_reproduce": false,
    "feature_targets": [],
    "defect_scan_enabled": false,
    "S4_checkpoint_confirmed": false,
    "pending_confirmations": []
  },
  "metadata": {
    "token_consumed": null,
    "rounds_used": 0,
    "max_rounds": 24,
    "deep_dive_rounds_used": 0,
    "deep_dive_max_rounds": 3,
    "feature_rounds_used": 0,
    "feature_max_rounds": 3,
    "defect_scan_rounds_used": 0,
    "defect_scan_max_rounds": 2,
    "started_at": "",
    "capabilities": {
      "can_execute_commands": false,
      "can_read_external_deps": false,
      "can_report_tokens": false,
      "can_structured_interaction": false
    },
    "aborted": false,
    "abort_reason": "",
    "checkpoint_path": ""
  },
  "logs": []
}
```

## 字段写入责任表

| 字段 | 写入阶段 | 说明 |
|------|----------|------|
| `input` | 流程启动 | 由编排层从用户输入中解析 |
| `S1_landscape` | Stage 1 | 项目全景卡片，含 `anomaly_signals` 异常信号 |
| `S2_modules` | Stage 2 | 模块清单，覆盖率 ≥ 80% |
| `S3_dataflow` | Stage 3 | 数据流图，含 `unverified_hypotheses` 假设表 |
| `S4_patterns` | Stage 4 | 设计模式/原则/约定/安全设计，每条含置信度 |
| `human_decisions` | S4→Checkpoint | S4 后由人工交互填入 |
| `human_decisions.pending_confirmations` | S4→Checkpoint 超时分支 | 3 轮未回复默认跳过时记录待确认项（option + 建议目标 + 跳过时间）；S7 据此生成"待确认提醒"章节 |
| `S5_deep_dives` | Stage 5 | 每轮深挖追加一条，最多 3 条 |
| `S5_history` | Stage 5 | 深挖循环的历史记录 |
| `S6_feature_studies` | Stage 6 | 每项功能复现研究追加一条 |
| `S6_history` | Stage 6 | 功能复现循环的历史记录 |
| `S6B_defect_scans` | Stage 6B | 每轮缺陷扫描追加一条 |
| `S6B_history` | Stage 6B | 缺陷扫描循环的历史记录 |
| `S7_report` | Stage 7 | 最终报告文件清单与完整性验证 |
| `S8_measurements` | 可选 Stage 8 | 实测验证结果，仅当 `metadata.capabilities.can_execute_commands` 为真时追加；不可用时恒为空数组 |
| `metadata` | 全局 | 运行时统计，每阶段结束更新。**逐字段维护规则（硬性）**：`rounds_used` 每完成 1 个阶段 `+=1`；`deep_dive_rounds_used` 每完成 1 轮 S5 深挖 `+=1`；`defect_scan_rounds_used` 每完成 1 轮 S6B `+=1`；`feature_rounds_used` 每完成 1 轮 S6 `+=1`；`started_at` 用 `<TZ_ISO_NOW>` 占位符（由宿主解析为 ISO 8601 时区时间，解析失败留空字符串）；`token_consumed` 仅当宿主可统计 token 时填数值，否则置 `null`（**禁止填 0 冒充已统计**）；`capabilities` 在 S1 探测后一次性写入 |
| `metadata.checkpoint_path` | S1 结束 | `_context_checkpoint.json` 的绝对路径，随 `output_dir` 一并解析 |
| `logs` | 每阶段 | 审计日志，格式见下方 |

## 审计日志格式

每条日志记录：

```json
{
  "stage": "S1",
  "timestamp": "2026-08-09T14:30:00+08:00",
  "input_summary": "项目路径: D:/project, 无补充材料",
  "output_summary": "项目卡片产出，识别为全栈应用，4 个入口点，2 个异常信号",
  "result": "pass",
  "reason": "所有必填字段全部填充",
  "source_paths_verified": 6,
  "unverified_count": 2,
  "confidence_breakdown": {"confirmed": 3, "high": 5, "medium": 1, "low": 0, "speculative": 0},
  "token_consumed": 3200
}
```

新增字段说明：
- `source_paths_verified`：本阶段已通过工具验证的源码路径数
- `unverified_count`：本阶段尚未验证的结论数
- `confidence_breakdown`：本阶段结论的置信度分布

## 关键边界说明

- `S5_deep_dives` 与 `S5_history` 的区别：`deep_dives` 是最终产物（本轮深挖结果），`history` 是循环记录（之前所有轮次的输出快照）
- `S6_feature_studies` 与 `S6_history` 同理
- `S6B_defect_scans` 与 `S6B_history` 同理；S6B 为可选阶段，`human_decisions.defect_scan_enabled` 为 `false` 时两者恒为空数组
- `S1_landscape.design_principles` 是结构化的 `[{principle, evidence_in_readme, reflected_in_code}]` 数组，不再是自由文本
- `S1_landscape.docs_map` 为 **Docs 优先侦察**产物：`tree` 为文档结构树（2 层）、`declared_tech_stack`/`declared_modules` 为文档声明的技术栈与模块划分（S2 的"预期基线"）、`glossary` 为关键术语表（≤ 20 条）、`drift_signals` 为文档-源码漂移信号（`[必挖]` 级进入 S5 深挖候选）；项目无文档目录时置 `null` 并在 `assumptions` 记录
- `S1_landscape.anomaly_signals` 为高价值深度分析目标列表（超大文件、深嵌套、循环依赖等），每条含 `priority`（`[必挖]`/`[可选]`）；`[必挖]` 信号在人工介入点强制进入 S5 深挖候选（可显式跳过并记录理由）
- `S3_dataflow.unverified_hypotheses` 存储假设验证表，每个假设含 `hypothesis → evidence → explanation → confidence → verify_action`
- `unreachable_paths` 用于 S3 和 S5 中无法验证源码路径的环节，必须逐条说明原因
- `S4.total_count` 为 `design_patterns + architecture_principles + coding_conventions + security_designs` 的四类计数
- `human_decisions.pending_confirmations` 元素格式：`{option: "deep_dive|feature|defect_scan", suggested_targets: [], skipped_at: ""}`；仅人工介入点超时（3 轮未回复）时写入，用户明确选择或回复后清空
- 任何 fallback 降级发生时，必须在对应的 `fallback_note` / `fallback_mode` 字段中说明原因
- `S1_landscape.design_principles.reflected_in_code` 为 `true | false | unverified`，表示该原则是否在源码中得到贯彻

## 持久化契约
- Context 持久化文件为 `output_dir/_context_checkpoint.json`，每阶段结束时写入（覆盖写，UTF-8），写入责任归属于刚结束的阶段。
- `input.output_dir` 在 S1 结束时解析（默认 `<项目根>/<项目名>-analysis/`，已存在分析目录则复用），此后不得为空。
- checkpoint 内容为累积的 `stage_outputs` + `human_decisions` + `metadata`；`logs` 审计流可裁剪至最近 20 条。
- 断点恢复时先读取 checkpoint 重建 Context，再从下一个未完成阶段继续；对话上下文只是缓存，checkpoint 是唯一事实来源。

### checkpoint 分区（受众分离）
checkpoint 内部严格分两区，禁止混用：
- **`metadata` + `logs` = 审计区**：供维护者/复盘者核对"分析了多少、花了多少、有没有诚实记账"。长 snippet 一律进 `logs`，**不得挤占 `stage_outputs`**。
- **`stage_outputs` + `human_decisions` = 结论区**：供报告/下游消费分析结论。只放结构化结论与关键证据，不放流水账。
> 一句话：结论区回答"发现了什么"，审计区回答"怎么分析的、有没有造假"。

### 记账自检 gate（每阶段结束强制）
每阶段写完 checkpoint 后立即校验，**校验失败则阻断下一阶段**（不得静默继续）：
1. `rounds_used` == 已完成阶段数（S1→1、S2→2、…、S7→7，跳过可选阶段不计数）。
2. `deep_dive_rounds_used` == `len(stage_outputs.S5_deep_dives)`（实际完成的 S5 轮数）。
3. `defect_scan_rounds_used` == 已完成的 S6B 轮数；`feature_rounds_used` == 已完成的 S6 轮数。
4. `started_at` 非空。
5. `token_consumed` 为数值或 `null`，**不得为 0**（0 是"已统计但花了 0"的造假信号）。
校验失败时：修正字段后重写 checkpoint，并在 `logs` 追加一条 `accounting_fix` 记录，然后才允许进入下一阶段。
