# 工具目录扫描清单

本清单记录本轮对各 Go 工具目录的能力缺口、风险点和处理结果。原则是优先做兼容新增和低风险防御性修复，不破坏既有 API。

## 已处理

| 目录 | 发现 | 处理 |
| --- | --- | --- |
| `aes` | 解密 padding 校验不完整，伪造密文可能触发异常 | 补齐 padding 校验和回归测试 |
| `buffer` | `PutBuffer(nil)` 会 panic | 改为 nil 安全返回并补测试 |
| `chinese` | `IsGBK` 截断字节可能越界 | 修复边界判断并补测试 |
| `cmd` | 命令执行失败时可能丢失底层错误 | 保留底层错误并附带 stderr |
| `connection` | `IsWebSocketConn(nil)` 空指针；连接探测 timeout 偏长 | nil 安全返回；缩短默认 dial timeout |
| `curl` | 缺少 `context.Context` 请求构造；缺少 PUT/PATCH/DELETE 便捷构造 | 新增 `New*RequestContext`、`NewPutRequest`、`NewPatchRequest`、`NewDeleteRequest` 并补测试 |
| `download` | 不能注入 HTTP client；默认 buffer 偏大；失败时可能留下半成品目标文件 | 新增可选 `HTTPClient`、`BufferSize`；改为临时文件完成后 rename |
| `email` | embed 模板解析失败时 init `panic` | 改为 fallback 模板，避免非必要 panic |
| `file` | 复制/移动要求目标目录预先存在 | 自动创建目标目录并补测试 |
| `header` | 常用 header 只有部分 Set/Get 辅助 | 新增 Content-Type、Authorization、TraceID Set/Get |
| `id` | `IPV4ToNodeID` 遇到 IPv6 可能 panic | 增加 IPv4 校验并补测试 |
| `ip` | 私网 IPv4 判断对非 IPv4 输入可能越界 | 使用 `To4()` 后判断并补测试 |
| `page` | 部分分页 helper 对 nil 输入不稳 | `ConvertToPageOption`、`HasNextPage`、`CalcPageResponse` 增加 nil 安全 |
| `random` | 空字符集、负长度、负采样等边界输入可能 panic | 增加边界返回并补测试 |
| `reflect` | nil 或不可设置值可能 panic | 增加 nil 和 CanSet 防御并补测试 |
| `thread` | `Go/GoSafe` 不便传 context | 新增 `GoWithContext`，并让 `GoSafeWithContext` 处理 nil context |
| `zip` | 解压路径穿越风险；压缩大文件一次性读入内存；目标 zip 目录需预先存在 | 防 zip slip；改流式复制；自动创建目标目录 |

## 已扫描，暂未改代码

| 目录 | 结论 |
| --- | --- |
| `base64` | API 简单稳定，当前测试覆盖编码/解码；无明确低风险补充项 |
| `filepath` | 已有遍历、读目录、创建/重建目录能力；`RenewDir` 是破坏性操作，已在 README 明确只对受控临时目录使用 |
| `json` | 已复制 buffer 内容后归还复用池，并有并发测试；无明确低风险补充项 |
| `locker` | 测试耗时来自锁过期等待；后续可考虑注入时钟或过期时间以缩短测试，但会影响接口设计，本轮暂不改 |
| `md5` | MD5 安全限制已在 README 标注；保留兼容用途 |
| `operator` | 泛型三元工具足够简单；无明确补充项 |
| `os` | 当前只封装 `IsWindows`；能力较窄但符合包现状 |
| `password` | bcrypt 封装已有测试；无明确低风险补充项 |
| `path` | 基于 `runtime.Caller` 的调试辅助；README 已标注不建议作为业务运行目录配置 |
| `ptr` | 泛型和历史类型函数均已有测试；无明确补充项 |
| `regex` | 常见格式校验已有测试；正则只作格式初筛，README 已标注 |
| `rsa` | 加解密和签名已有测试；密钥安全注意事项已标注 |
| `slice` | 泛型 contains/reverse 已有测试；无明确补充项 |
| `snowflake` | 当前文件带 `//go:build ignore`，不是普通构建入口；保持现状 |
| `sort` | 泛型排序和历史排序函数已有测试；无明确补充项 |
| `string` | snake/camel/string 转换已有测试；无明确补充项 |
| `time` | 时间 helper 测试覆盖较多；无明确补充项 |
| `url` | URL 编码和 query 拼接已有测试；无明确补充项 |
| `uuid` | xid/uuid 生成、解析、排序已有测试；无明确补充项 |
| `writer` | 轮转 writer 有本地文件测试；后续可考虑注入时钟减少 sleep，但本轮不改接口 |

## 后续建议

- `locker` 和 `writer` 的测试仍存在真实 `time.Sleep`，后续如要进一步优化测试速度，可考虑引入可配置过期时间或 clock 注入。
- `random` 当前基于 `math/rand`，只适合非安全随机；如需要安全 token，应新增 `crypto/rand` 专用 API，而不是改变现有函数语义。
- `snowflake` 目录当前 build tag 为 `ignore`，如需要正式暴露该包，应单独评估包名、测试和与 `id` 包的职责边界。
