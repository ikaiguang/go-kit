# AGENTS.md

本文件只保留本项目的常驻规则。通用工作方法使用 `~/.codex/skills` 中的全局 skills。更细的架构、编码、命令和流程说明放在 `.agents/skills/` 中按需读取。

## 项目事实

- Go 版本：`1.26`
- 默认使用中文交流、解释和补充文档；代码标识符保持英文。

## 架构硬约束

- 业务分层遵循 `Service -> Biz -> Data`。
- `Service` 只负责 HTTP/gRPC handler、入参校验和 DTO 转换，不直接访问 `Data`。
- `Biz` 负责业务逻辑和仓储接口；`Data` 负责仓储实现和外部访问。
- 常见数据流：`Proto -> DTO -> BO -> PO`。

## 生成代码

- 不要手改生成文件，如 `*.pb.go`、`*_grpc.pb.go`、`*_http.pb.go`、`*.validate.go`、`wire_gen.go`。
- Proto 生成命令以 `Makefile` 为准。
- 修改 Wire 装配后，按 `Makefile` 中对应目标重新生成。

## 常用命令

```bash
make init
make generate
go test ./...
go vet ./...
make run-service
```

Windows 下如果 `make` 在 PowerShell 中表现异常，优先使用等价的 `go`、`wire` 或在 `git-bash` 中运行。

## Skill 入口

- 本仓库 Go/Proto/Wire/配置/测试修改：先用 `.agents/skills/my-project/`。
- 需要任务记录、风险分级或中高风险确认：用 `.agents/skills/spec-workflow/`。
- 全库审计、补测试、补文档或综合风险扫描：如可用，使用 `.agents/skills/code-audit-repair/`。通用审计方法也可结合 `~/.codex/skills/code-review-and-quality`、`security-and-hardening`、`test-driven-development`。

优先级：系统/会话权限规则 > 本文件 > `.agents/skills/` 中的 repo-local skills > `~/.codex/skills` 中的全局 skills。

全局 skills 只提供通用工程方法，不能覆盖本仓库硬约束、会话权限和风险确认规则。若全局 skill 建议 commit、push、发布、安装依赖、打开 GUI、删除文件、改写 Git 历史或访问线上资源，必须先按本文件和当前会话权限规则确认；用户未明确要求时不自动提交、不推送、不发布。

默认使用最小必要 skill 集：先按 repo-local skill 路由，只有任务确实需要时再叠加全局工程 skill，避免为小任务加载完整通用流程。

常用参考文件：

- `.agents/skills/my-project/references/project-context.md`
- `.agents/skills/my-project/references/service-workflow.md`
- `.agents/skills/my-project/references/coding-rules.md`
- `.agents/skills/my-project/references/commands-and-generation.md`

## 工作方式

- 除非用户明确指定或任务必须处理 `./mytest`，默认忽略 `./mytest` 下的所有文档和文件；搜索、阅读、修改、审计、测试范围判断和规则沉淀都不要主动纳入该目录。
- 对用户已明确要求的低风险本地改动，先读相关文件后可直接执行，并在 `.agents/specs/<task-name>/spec.md` 或最终回复中记录目标、改动、验证和风险；不需要额外等待确认。
- 低风险本地改动包括：读取和搜索文件；修改用户指定的文档、prompt、README、注释、示例文本；小范围修 typo、格式、链接、命令说明；修改当前仓库内相关普通源码或测试且不改变公共接口；为小修补充对应单元测试；运行 `gofmt`、`go test`、`go vet`、`go list`、`make -n ...`、`git diff`、`git status` 等本地验证命令；新增或调整低风险辅助 Makefile 目标但不自动执行批量破坏性动作。
- 中风险任务先在 `.agents/specs/<task-name>/spec.md` 写清目标、方案、任务、验收和风险；若用户目标已经明确，可写完后继续执行，不必停下等确认。中风险包括多文件文档补全、多包低风险测试补充、小型重构、新增本地工具脚本或调整 skill 流程规则。
- 任务文档只作临时过程记录；长期规则沉淀到 `AGENTS.md`、repo-local skill、模块 README 或稳定 docs。
- 涉及删除大量文件、改写 Git 历史、丢弃未确认修改、系统级依赖、发布部署、线上数据、敏感信息、公共接口或配置结构大改、Proto/Wire 跨模块影响、数据库数据或风险不确定的操作，必须先确认。
- 先读目标模块和相邻实现，再做与现有结构一致的最小改动；新增或修改行为时同步考虑测试、文档、安全、稳定性和性能风险。
- 当用户指出规则应沉淀、回答应复用或行为需纠正时，主动更新合适的规则或文档；边界不清时提出 1-3 个具体问题。
