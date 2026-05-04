# 沉淀可复用审计提示词

## 背景

用户提供了一份用于代码库测试、审计、修复和文档补全的临时源文档。它覆盖测试覆盖、功能正确性、安全/性能风险、能力缺口、README 和 docs 索引等内容。

用户希望把这份文档的核心思想沉淀为 skill 或编码核心，使后续每次编码时都能自动考虑这些要点。

## 目标

- 在 `AGENTS.md` 中补充简短常驻规则，让日常编码默认考虑测试、正确性、安全/性能和文档同步。
- 在 `.agents/skills/my-project/references/coding-rules.md` 中补充更具体的“实现时默认检查项”，作为本仓库编码规则的一部分。
- 新增一个 repo-local skill，用于全库审计、补测试、修复低风险问题和补全文档等大范围任务。
- 保留临时源文档的核心审计流程，但避免把完整长提示词放进常驻上下文。
- 最终沉淀后的文档和规则不引用临时源目录，因为该目录不在 Git 中且后续会被删除。

## 非目标

- 本次不执行全库审计。
- 本次不补测试、不改业务代码、不改生成文件。
- 本次不修改 Proto、Wire 装配或 Makefile 生成流程。
- 本次不安装依赖、不发布、不部署、不改写 Git 历史。

## 影响范围

- `AGENTS.md`
- `.agents/skills/my-project/references/coding-rules.md`
- `.agents/skills/code-audit-repair/SKILL.md`
- `.agents/skills/code-audit-repair/references/reusable-prompt.md`
- 可选：`.agents/skills/code-audit-repair/agents/openai.yaml`

## 方案

采用三层分工：

1. `AGENTS.md` 只放短小高频规则，提醒每次编码都检查测试、正确性、安全/性能、文档同步和最小改动。
2. `my-project` 的 `coding-rules.md` 增加“日常实现检查项”，比 `AGENTS.md` 更具体，但仍保持可扫描。
3. 新增 `code-audit-repair` skill，把完整综合审计流程放进 `references/reusable-prompt.md`，仅在用户要求全库审计、补测试、补文档、风险扫描或综合修复时触发。

新增 skill 的 `SKILL.md` 保持精简，说明触发场景、执行边界、风险处理和何时读取 `references/reusable-prompt.md`。完整清单放在 reference 中，避免日常编码时占用过多上下文。

## 任务列表

- [x] 更新 `AGENTS.md` 的编码核心检查规则。
- [x] 更新 `.agents/skills/my-project/references/coding-rules.md` 的实现检查项。
- [x] 创建 `.agents/skills/code-audit-repair/`。
- [x] 编写 `code-audit-repair/SKILL.md`。
- [x] 将临时源文档的核心流程整理到 `code-audit-repair/references/reusable-prompt.md`。
- [x] 如本地脚本可用，验证 skill 基本格式；否则手动检查 frontmatter 和目录结构。
- [x] 更新本规格文档执行记录。

## 验收标准

- 日常编码规则能在 `AGENTS.md` 中被快速读取，不包含大段审计提示词。
- 本仓库编码规则中有明确的测试、风险、文档同步检查项。
- 新 skill 能被仓库本地技能发现，且 description 明确覆盖全库审计、补测试、补文档和综合修复触发场景。
- 完整审计提示词被放入新 skill 的 reference 中，按需读取。
- 没有改动业务源码或生成文件。

## 风险与回滚

- 风险：skill description 过宽可能导致小任务误触发。规避方式是在 description 和正文中明确它用于“全库/多模块/综合审计修复”场景。
- 风险：常驻规则过长会增加上下文负担。规避方式是 `AGENTS.md` 只放短规则，细节进入 skill reference。
- 回滚：删除新增 `.agents/skills/code-audit-repair/`，并还原 `AGENTS.md` 与 `coding-rules.md` 本次新增段落。

## 执行记录

- 已创建任务规格文档，等待用户确认。
- 用户确认后开始执行。
- 已尝试使用 `skill-creator` 的 `init_skill.py` 初始化 `code-audit-repair`，但本机 `python` 由 pyenv shim 接管且未配置版本，命令未成功，也未产生文件。
- 已手动创建 `.agents/skills/code-audit-repair/SKILL.md`、`references/reusable-prompt.md` 和 `agents/openai.yaml`。
- 已更新 `AGENTS.md`，加入日常编码的测试、正确性、安全/稳定性/性能和文档同步短规则。
- 已更新 `.agents/skills/my-project/references/coding-rules.md`，加入“日常实现检查项”。
- 已检查新增 skill 目录结构和 `SKILL.md` frontmatter；已用 `rg` 检查避免在规则和 skill 中引用临时源目录。
- 已执行 `git status --short` 确认本次只涉及规则、skill 和任务文档；未修改业务源码或生成文件。
- 本次未运行 `go test`，因为没有改动 Go 代码。
