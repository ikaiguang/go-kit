---
name: code-audit-repair
description: 对代码库执行综合审计、补测试、低风险修复和文档补全。Use when the user asks for broad or multi-module codebase audit, test coverage improvement, functional correctness review, security/performance/stability risk scanning, README/docs completion, audit checklist generation, or phrases like 全库审计, 综合审计, 补测试, 补文档, 扫风险, 审计并修复. Do not use for narrow single-bug fixes or ordinary small code changes unless the user explicitly asks to audit surrounding modules.
---

# Code Audit Repair

## Overview

本 skill 用于大范围代码库维护任务：测试覆盖、功能正确性、安全/性能/稳定性风险、能力缺口和使用文档的综合审计与低风险修复。

它不是日常小改动的必选流程。小范围实现和 bug 修复优先遵循仓库 `AGENTS.md`、项目专用 skill 和相邻代码；只有用户明确要求审计、补测、补文档、扫风险或多模块综合修复时才使用本 skill。

## Workflow

1. 先读取项目协作规则和构建说明，例如 `AGENTS.md`、`README.md`、`CONTRIBUTING.md`、`Makefile`、语言包管理文件和 repo-local skills。
2. 如果仓库要求先文档后执行，先创建或更新任务规格文档，等待用户确认后再修改代码或运行有副作用命令。
3. 确认范围：区分工具库、业务模块、生成代码、示例代码、测试数据和第三方代码。
4. 对目标目录逐项检查测试、功能正确性、安全、稳定性、性能、API 兼容性和文档。
5. 低风险问题做最小兼容修复，并补充回归测试；高风险或需要设计取舍的问题写入审计记录，先向用户确认。
6. 优先运行受影响模块的定向测试，再按仓库约定运行全量测试、静态检查或构建。不能运行时记录具体命令、错误和替代验证。
7. 最终输出必须列出实际修改、运行过的验证命令和结果、未处理风险与后续建议。

## Reference

执行综合审计任务时，读取 `references/reusable-prompt.md`。该文件包含完整审计提示词、逐目录检查清单和输出文件建议。

不要在新文档中引用临时、未纳入版本控制的源目录；需要复用的内容应沉淀到本 skill 或仓库 docs 中。

## Boundaries

- 不手改生成文件。
- 不回退用户已有改动。
- 不做无关重构。
- 不安装系统依赖、不发布、不部署、不改写 Git 历史。
- 遇到破坏性操作、线上操作、敏感信息或高风险设计变更时先停止并说明。
