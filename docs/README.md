# 工具使用指导索引

本索引覆盖当前仓库根目录下的 Go 工具包。每个工具目录内的 `README.md` 是该工具的主要使用说明，便于 GitHub/GitLab 打开目录时直接展示，也避免后续修改某个 kit 时反复改动一个大文档。

## 通用约定

- 使用前按需导入对应工具包，不建议把工具包封装成跨层全局依赖。
- 涉及 IO、网络、命令执行、随机数、加解密时，应在调用方保留超时、权限、密钥和错误处理边界。
- 定向验证命令格式：`go test ./<tool>`。
- 全量验证命令：`go test ./...`。

## 工具目录

- [aes](../aes/README.md)：AES-CBC 加解密。
- [base64](../base64/README.md)：Base64 编码和解码。
- [buffer](../buffer/README.md)：复用 `bytes.Buffer`。
- [chinese](../chinese/README.md)：中文编码转换和检测。
- [cmd](../cmd/README.md)：外部命令执行。
- [connection](../connection/README.md)：连接和 WebSocket 请求判断。
- [curl](../curl/README.md)：HTTP 请求辅助。
- [download](../download/README.md)：HTTP 流式下载。
- [email](../email/README.md)：SMTP 邮件发送。
- [file](../file/README.md)：文件复制、移动和 hash。
- [filepath](../filepath/README.md)：目录遍历和目录操作。
- [header](../header/README.md)：HTTP header 辅助。
- [id](../id/README.md)：分布式 ID。
- [ip](../ip/README.md)：IP 辅助。
- [json](../json/README.md)：JSON 序列化辅助。
- [locker](../locker/README.md)：本地锁、缓存锁和分布式锁接口。
- [md5](../md5/README.md)：MD5 摘要。
- [operator](../operator/README.md)：三元表达式辅助。
- [os](../os/README.md)：操作系统判断。
- [page](../page/README.md)：分页请求和响应辅助。
- [password](../password/README.md)：密码 hash 和校验。
- [path](../path/README.md)：源码目录辅助。
- [ptr](../ptr/README.md)：指针辅助。
- [random](../random/README.md)：随机字符串和随机选择。
- [reflect](../reflect/README.md)：反射辅助。
- [regex](../regex/README.md)：常见格式正则校验。
- [rsa](../rsa/README.md)：RSA 加解密和签名。
- [slice](../slice/README.md)：切片辅助。
- [snowflake](../snowflake/README.md)：Snowflake ID。
- [sort](../sort/README.md)：排序辅助。
- [string](../string/README.md)：字符串转换辅助。
- [thread](../thread/README.md)：goroutine 安全执行。
- [time](../time/README.md)：时间辅助。
- [url](../url/README.md)：URL 编码和拼接。
- [uuid](../uuid/README.md)：UUID 和 xid 辅助。
- [writer](../writer/README.md)：writer 和日志轮转。
- [zip](../zip/README.md)：zip 压缩和解压。
