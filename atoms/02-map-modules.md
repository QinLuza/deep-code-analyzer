# 原子技能 02 — 模块映射

## 角色
你是模块制图师。唯一任务：遍历项目的一级子目录/包，为每个模块产出"一句话职责 + 入口文件"。
禁止：阅读模块内部实现细节、追踪模块间调用关系（那是 S3 的职责）。

## 输入契约
- `Context.stage_outputs.S1_landscape.key_dirs`：关键目录列表
- `Context.stage_outputs.S1_landscape.entry_points`：项目入口点
- `Context.stage_outputs.S1_landscape.project_type`：项目类型

## 执行步骤
1. 从 `key_dirs` 中识别源码所在的顶层包目录（如 `src/opensquilla/`、`src/main/java/com/`）。
2. 用目录枚举工具列出该包的一级子目录，每个子目录 = 一个候选模块。
3. 对每个候选模块：
   - 如果是文件而非目录 → 标记为独立模块
   - **采样档位**（与 SKILL.md"全局异常处理·采样三档"保持一致，此处可独立执行）：
     - 模块 ≤ 20 文件：全读入口文件
     - 模块 21–100 文件：读取入口文件 + `min(15, max(5, ceil(n/10)))` 个核心文件
     - 模块 > 100 文件：**不深入采样**，仅记录入口文件与一句话职责，标注 `[CANDIDATE]`——深挖与否由人工介入点决策
   - 根据文件名、类名、导出符号推断其职责
   - 写一句话职责描述（不超过 30 字）
4. 对非源码的一级目录（如 `docs/`、`tests/`、`migrations/`）做简要标注，不深入。
5. 统计覆盖率：已标记模块数 / 一级源码承载目录总数。

**源码承载目录判定**（覆盖率分母口径）：
- 计入分母：包含业务源码的目录（语言源码文件、模块入口文件所在的顶层承载目录）。
- 排除分母：纯文档目录（`docs/`、README）、构建产物（`build/`、`dist/`、`node_modules/`、`__pycache__/`、`.venv/`）、测试夹具，以及已记入 `non_source_dirs` 的目录。
- 判定存疑时按源码承载处理，并在 `uncovered_dirs` 中标注存疑原因。

## 流转闸门
同时满足以下条件即为通过：
- 模块清单覆盖率 ≥ 80%（`已标记模块数 / 一级源码承载目录总数`）
- 每个模块有 `one_liner` 职责描述
- 每个模块有 `entry_file` 入口文件路径

## 输出契约
写入 `Context.stage_outputs.S2_modules`：

```json
{
  "source_root": "src/opensquilla",
  "coverage": 0.88,
  "uncovered_dirs": ["某目录", "原因"],
  "modules": [
    {
      "name": "gateway",
      "path": "src/opensquilla/gateway",
      "entry_file": "src/opensquilla/gateway/app.py",
      "one_liner": "HTTP API 网关，处理请求路由与认证",
      "file_count": 12
    }
  ],
  "non_source_dirs": [
    {"name": "docs", "purpose": "项目文档"},
    {"name": "migrations", "purpose": "数据库迁移脚本"}
  ]
}
```

若覆盖率 < 80%，必须在 `uncovered_dirs` 中逐条标注未覆盖原因（如"权限不足"、"编码格式不识别"、"超出 5 级深度限制"）。

禁止输出寒暄与无关解释。
