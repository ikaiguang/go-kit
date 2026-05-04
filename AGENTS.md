# AGENTS.md

本文件为本项目提供最小必要上下文。

## 项目事实

- Go 版本：`1.25.9`

## 语言规则

- 默认使用中文进行交流、解释、说明和文档补充
- 代码中的变量名、函数名、类型名保持英文，遵循既有编程规范
- 技术术语可以保留英文，但解释优先使用中文
- 错误信息和日志说明优先使用中文

## 架构约束

- 业务分层遵循 `Service -> Biz -> Data`
- `Service` 层负责 HTTP/gRPC handler 和 DTO 转换
- `Biz` 层负责业务逻辑，仓储接口放在 `biz/repo/`
- `Data` 层负责仓储实现，实现在 `data/repo/`
- 常见数据流：`Proto -> DTO -> BO -> PO`
- 不要让 `Service` 直接访问 `Data`

## 生成代码与禁止事项

- 不要手改 Proto 生成文件，如 `*.pb.go`、`*_grpc.pb.go`、`*_http.pb.go`、`*.validate.go`
- 修改 Wire 装配后，重新生成：
  - `make generate`
  - `wire ./testdata/ping-service/cmd/ping-service/export`
- Proto 相关生成命令以 `Makefile` 为准，例如：
  - `make protoc-api-protobuf`
  - `make protoc-ping-v1-protobuf`

## 常用命令

```bash
make init
make generate
go test ./...
go vet ./...
make run-service
```

Windows 下部分命令更适合在 `git-bash` 运行；如果 `make` 目标在 PowerShell 中表现异常，优先退回到等价的 `go` / `wire` 命令。

## 修改代码时的优先参考

- 示例服务入口：`testdata/ping-service/cmd/ping-service/main.go`
- Wire 装配：`testdata/ping-service/cmd/ping-service/export/wire.go`
- 服务器初始化：`server_all_in_one.util.go`

## 深层上下文

- Repo-local Codex skill 位于 `.agents/skills/my-project/`
- `AGENTS.md` 只保留常驻规则；更深的架构、流程、生成命令和编码约定放在该 skill 的 `references/` 中按需读取
- 修改 Go、Proto、Wire、配置、测试或生成流程前，按任务需要读取：
  - `.agents/skills/my-project/references/project-context.md`
  - `.agents/skills/my-project/references/service-workflow.md`
  - `.agents/skills/my-project/references/coding-rules.md`
  - `.agents/skills/my-project/references/commands-and-generation.md`

## 编码习惯

- 具体编码规则以 `.agents/skills/my-project/references/coding-rules.md` 为准
- 优先沿用相邻实现和既有命名，不另起一套风格
- 使用 `gofmt`，必要时使用 `goimports`
- 编码时默认同步考虑测试覆盖、功能正确性、安全/稳定性/性能风险和文档是否需要更新
- 低风险缺口优先用最小兼容改动修复并补回归测试；高风险或需要设计取舍的问题先记录并确认
- 全库审计、补测试、补文档或综合风险扫描任务优先使用 `.agents/skills/code-audit-repair/`

## 错误与日志

- 具体错误与日志规范以 `.agents/skills/my-project/references/coding-rules.md` 为准
- 错误处理、日志字段、脱敏方式优先参考相邻实现和现有工具

## 工作方式

- 先文档后执行：除非用户明确要求跳过，收到任何需要修改文件、执行有副作用命令或推进实现的任务时，先在 `docs/<task-name>/*.md` 创建任务说明文档，写清楚“要做什么、为什么这么做、如何做（任务列表）”，并等待用户确认后再开始实际执行。
- `docs/<task-name>/` 下的任务文档只作为临时过程记录；长期规则、skill、README、docs 索引和最终交付说明不要把它作为引用入口。
- 执行过程中如果发现原方案不合适、约束不成立或需要改变方向，先回到对应文档记录“发现的问题、原方案为什么不合适、准备怎么改、为什么这样改”，必要时再次等待用户确认，然后再继续执行。
- 用户确认任务文档后，执行阶段的低风险操作可以静默执行，不需要逐项询问确认；低风险操作包括读取和搜索文件、修改当前仓库内与任务相关的普通文件、运行本地格式化、测试、构建和约定内代码生成。
- 涉及危险或高影响操作时必须先确认，包括改写数据库、请求第三方或线上 API 修改数据、删除大量文件、改写 Git 历史、丢弃未确认修改、安装或修改系统级依赖、发布部署、发送外部消息、处理未脱敏敏感信息，以及任何风险不确定的操作。
- 当用户指出某个回答应沉淀为文档、规则或使用指导时，应主动更新对应文件；如果不能更新，先说明原因，不要只停留在聊天回答。
- 当用户纠正代理行为、指出应沉淀文档或要求以后不要反复提醒时，应主动判断并更新 `AGENTS.md`、skill 或 docs；适合常驻的规则应进入 `AGENTS.md`，细节放入对应 skill 或 docs。
- 已确认任务文档后，读文件、搜索、`gofmt`、`go test`、`wire`、`go list` 等低风险本地操作默认直接执行；只有越出当前 workspace、破坏性操作、线上或系统级影响、处理敏感信息、风险不确定时才询问确认。
- 如果规则边界、执行风险或文档分层不清楚，不要默不作声或只聊天回答；应主动提出 1-3 个具体问题让用户决策。
- 先读将要修改的模块和相邻实现，再决定改法
- 优先做与当前结构一致的最小改动
- 涉及新服务或新模块时，先读取 `.agents/skills/my-project/references/service-workflow.md` 和相邻实现
