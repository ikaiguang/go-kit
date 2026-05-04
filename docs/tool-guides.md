# 工具使用指导

本文档覆盖当前仓库根目录下的 Go 工具包。所有示例默认模块路径为 `github.com/ikaiguang/go-kit/<tool>`。

## 通用约定

- 使用前按需导入对应工具包，不建议把工具包封装成跨层全局依赖。
- 涉及 IO、网络、命令执行、随机数、加解密时，应在调用方保留超时、权限、密钥和错误处理边界。
- 定向验证命令格式：

```bash
go test ./<tool>
```

全量验证命令：

```bash
go test ./...
```

## aes

用途：AES-CBC 加解密，支持字符串级 `AESEncryptor` 接口和字节级 CBC 函数。

基础用法：

```go
cipher := aespkg.NewCBCCipher()
encrypted, err := cipher.EncryptToString("plain text", "1234567890123456")
plain, err := cipher.DecryptToString(encrypted, "1234567890123456")
```

注意事项：AES key 长度必须符合 Go 标准库要求，即 16、24 或 32 字节；密钥不要硬编码在业务代码中。旧的 `NewAesCipher` 使用固定 IV，只建议兼容历史数据时使用。

验证：`go test ./aes`

## base64

用途：Base64 编码和解码，提供字节函数与 `Encryptor` 接口。

基础用法：

```go
encoded := base64pkg.Encode([]byte("hello"))
decoded, err := base64pkg.Decode(encoded)
```

注意事项：Base64 不是加密，只适合传输编码；解码外部输入时必须处理错误。

验证：`go test ./base64`

## buffer

用途：复用 `bytes.Buffer`，降低频繁临时 buffer 分配。

基础用法：

```go
buf := bufferpkg.GetBuffer()
defer bufferpkg.PutBuffer(buf)
buf.WriteString("hello")
```

注意事项：归还后不要继续读写该 buffer；跨 goroutine 使用时由调用方保证生命周期。

验证：`go test ./buffer`

## chinese

用途：GB18030、GBK、HZGB2312 与 UTF-8 之间转换，并检测字符串是否为 UTF-8 或 GBK。

基础用法：

```go
gbkBytes, err := chinesepkg.Utf8ToGbk([]byte("中文"))
utf8Bytes, err := chinesepkg.GbkToUtf8(gbkBytes)
ok := chinesepkg.IsGBK(string(gbkBytes))
```

注意事项：编码检测只能作为输入判断辅助；外部输入要按转换错误做兜底处理。

验证：`go test ./chinese`

## cmd

用途：执行外部命令，支持 `context.Context` 和工作目录。

基础用法：

```go
out, err := cmdpkg.RunCommandContext(ctx, "go", []string{"version"})
out, err = cmdpkg.RunCommandWithWorkDirContext(ctx, ".", "go", []string{"test", "./cmd"})
```

注意事项：命令和参数要分开传递，不要拼接未校验的用户输入；长耗时命令必须使用带超时的 context。

验证：`go test ./cmd`

## connection

用途：判断 WebSocket 请求头、检查 TCP 地址或 endpoint 是否可连接、识别连接关闭错误。

基础用法：

```go
ok := connectionpkg.IsWebSocketConn(req)
alive, err := connectionpkg.IsValidConnection("127.0.0.1:8080")
alive, err = connectionpkg.CheckEndpointValidity("http://127.0.0.1:8080")
```

注意事项：连接检查会发起 TCP dial，只适合健康检查或调试路径；不要在高频请求链路中无缓存调用。

验证：`go test ./connection`

## curl

用途：构造 HTTP 请求、创建带超时的 HTTP client、执行请求并读取响应体。

基础用法：

```go
req, err := curlpkg.NewGetRequest("https://example.com", nil)
code, body, err := curlpkg.Do(req, curlpkg.WithTimeout(3*time.Second))
if !curlpkg.IsSuccessCode(code) {
    return curlpkg.ErrRequestFailure(code)
}
```

注意事项：默认不跳过 TLS 证书校验；`WithInsecureSkipVerify` 仅限开发或测试环境。

验证：`go test ./curl`

## download

用途：通过 HTTP 流式下载文件，并自动检查或创建目标目录。

基础用法：

```go
res, err := downloadpkg.StreamDownload(ctx, &downloadpkg.DownloadParam{
    URL:        "https://example.com/file.zip",
    OutputPath: "./runtime/file.zip",
})
```

注意事项：下载外部资源时使用带超时的 context；输出路径由调用方控制，避免覆盖重要文件。

验证：`go test ./download`

## email

用途：构造邮件消息并通过 SMTP 发送。

基础用法：

```go
sender := &emailpkg.Sender{Host: "smtp.example.com", Port: 587, Username: "user", Password: "secret"}
msg := &emailpkg.Message{From: "from@example.com", To: []string{"to@example.com"}, Subject: "subject", Body: "body"}
err := emailpkg.Send(sender, msg)
```

注意事项：SMTP 账号、密码和授权码必须来自配置或密钥管理；测试中不要发送真实邮件。

验证：`go test ./email`

## file

用途：复制文件、移动文件到指定目录、计算文件 hash。

基础用法：

```go
err := filepkg.CopyFile("./a.txt", "./b.txt")
target, err := filepkg.MoveFileToDir("./b.txt", "./runtime")
```

注意事项：移动文件依赖 `os.Rename`，跨设备移动可能失败；调用前确认目标目录存在。

验证：`go test ./file`

## filepath

用途：处理文件路径、扩展名、路径存在性等文件路径辅助逻辑。

基础用法：

```go
paths, entries, err := filepathpkg.WalkDir("./docs")
err = filepathpkg.CreateDir("./runtime")
```

注意事项：路径判断受当前工作目录影响；`RenewDir` 会删除并重建目录，只对受控临时目录使用。

验证：`go test ./filepath`

## header

用途：HTTP header 常量和 header value 匹配。

基础用法：

```go
if headerpkg.ContainsValue(req.Header, "Connection", "upgrade") {
    // ...
}
req.Header.Set(headerpkg.ContentType, headerpkg.ContentTypeJSON)
```

注意事项：`ContainsValue` 支持逗号分隔 header，适合 WebSocket 和 HTTP 协议头判断。

验证：`go test ./header`

## id

用途：生成 xid、uuid、snowflake、sonyflake 等 ID。

基础用法：

```go
id, err := idpkg.NextID()
nodeID, err := idpkg.GenNodeID()
```

注意事项：分布式 ID 使用前确认时钟、节点号和业务唯一性需求；不要把随机 ID 当作权限凭证。

验证：`go test ./id`

## ip

用途：IP 字符串、内网地址、HTTP 请求来源 IP 等辅助判断。

基础用法：

```go
localIP := ippkg.LocalIP()
ok := ippkg.IsValidIP(localIP)
```

注意事项：`LocalIP` 会缓存首次探测结果；网络不可用时会回退到 `127.0.0.1`。

验证：`go test ./ip`

## json

用途：JSON 序列化、反序列化和便捷转换。

基础用法：

```go
data, err := jsonpkg.MarshalWithoutEscapeHTML(v)
data, err = jsonpkg.MarshalIndentWithoutEscapeHTML(v, "", "  ")
```

注意事项：函数会保留 HTML 字符原样输出，并复制 buffer 内容后再归还复用池。

验证：`go test ./json`

## locker

用途：本地锁、缓存锁和分布式锁接口辅助。

基础用法：

```go
locker := lockerpkg.NewLocalLocker()
unlocker, err := locker.Mutex(ctx, "resource-key")
if err == nil {
    defer unlocker.Unlock(ctx)
}
```

注意事项：锁 key 要稳定且粒度清晰；业务代码必须保证成功加锁后释放。

验证：`go test ./locker`

## md5

用途：MD5 摘要计算。

基础用法：

```go
sum, err := md5pkg.Md5([]byte("hello"))
fileSum, err := md5pkg.FileMd5("./README.md")
```

注意事项：MD5 不适合密码存储、签名安全或防篡改场景；安全哈希优先选择 SHA-256/HMAC 等方案。

验证：`go test ./md5`

## operator

用途：三元表达式辅助函数。

基础用法：

```go
name := operatorpkg.Ternary(ok, "yes", "no")
```

注意事项：只适合简单值选择；复杂分支使用普通 `if` 更清晰。

验证：`go test ./operator`

## os

用途：操作系统判断辅助函数。

基础用法：

```go
if ospkg.IsWindows() {
    // windows specific logic
}
```

注意事项：只封装了当前运行时系统判断，复杂平台逻辑仍应由调用方显式处理。

验证：`go test ./os`

## page

用途：分页请求、分页选项、分页结果和 GORM 测试辅助。

基础用法：

```go
req := pagepkg.DefaultPageRequest()
opts := pagepkg.ConvertToPageOption(req)
```

注意事项：调用前校验 page 和 page size，避免超大分页拖慢查询。

验证：`go test ./page`

## password

用途：密码 hash、校验或密码相关辅助逻辑。

基础用法：

```go
hashedBytes, err := passwordpkg.Encrypt("plain-password")
ok := passwordpkg.Verify(string(hashedBytes), "plain-password")
err = passwordpkg.Compare(string(hashedBytes), "plain-password")
```

注意事项：不要记录明文密码；hash 参数和算法升级要兼容历史数据。

验证：`go test ./password`

## path

用途：返回当前工具包源码所在目录。

基础用法：

```go
p := pathpkg.Path()
```

注意事项：该函数基于 `runtime.Caller`，适合调试和测试辅助，不建议作为业务运行目录配置。

验证：`go test ./path`

## ptr

用途：基础类型取指针、指针取值和泛型指针辅助。

基础用法：

```go
namePtr := ptrpkg.Ptr("alice")
name := ptrpkg.Value(namePtr)
```

注意事项：区分零值和 nil 的业务含义，尤其是 PATCH/部分更新 DTO。

验证：`go test ./ptr`

## random

用途：生成随机字符串、验证码、密码、订单号、trace ID，或随机选取/打乱切片。

基础用法：

```go
code := randompkg.VerifyCode(6)
token := randompkg.Token(32)
password := randompkg.Password(12)
```

注意事项：当前包基于 `math/rand`，不适合生成高安全等级密钥；安全 token 应使用 `crypto/rand`。

验证：`go test ./random`

## reflect

用途：反射辅助函数，用于类型、字段或结构体相关操作。

基础用法：

```go
empty := reflectpkg.IsEmpty(v)
zero := reflectpkg.IsDefaultValue(v)
```

注意事项：反射逻辑要控制在边界层或通用工具层；业务主流程优先使用显式类型。

验证：`go test ./reflect`

## regex

用途：正则匹配和常见格式校验。

基础用法：

```go
ok := regexpkg.IsEmail("user@example.com")
```

注意事项：正则适合格式初筛，不代表业务真实性验证。

验证：`go test ./regex`

## rsa

用途：RSA 加解密、签名或密钥相关辅助。

基础用法：

```go
priKey, pubKey, err := rsapkg.GenRsaKey()
cipher, err := rsapkg.NewRsaCipher(pubKey, priKey)
ciphertext, err := cipher.EncryptToString("plain text")
plaintext, err := cipher.DecryptToString(ciphertext)
```

注意事项：密钥必须来自安全配置；不要把私钥写入代码或日志。

验证：`go test ./rsa`

## slice

用途：切片包含、去重、过滤、映射等基础操作，包含泛型辅助。

基础用法：

```go
ok := slicepkg.Contains([]string{"a", "b"}, "a")
```

注意事项：大切片高频查找可改用 map，避免线性扫描造成性能问题。

验证：`go test ./slice`

## snowflake

用途：Snowflake ID 生成。

基础用法：

```go
id, err := snowflakepkg.NextID()
```

注意事项：分布式部署时确认节点号唯一和系统时间稳定。

验证：`go test ./snowflake`

## sort

用途：排序辅助函数，包含基础类型和泛型排序。

基础用法：

```go
sortpkg.Sort(values)
sortpkg.SortFunc(users, func(a, b User) int {
    return cmp.Compare(a.ID, b.ID)
})
```

注意事项：确认函数是否原地修改切片，避免调用方误用共享切片。

验证：`go test ./sort`

## string

用途：字符串判断、截断、大小写、脱敏或格式处理等辅助。

基础用法：

```go
snake := stringpkg.ToSnake("UserID")
camel := stringpkg.ToCamel("user_id")
text := stringpkg.ToString(123)
```

注意事项：涉及用户隐私展示时优先使用脱敏函数，不要直接日志输出原文。

验证：`go test ./string`

## thread

用途：goroutine 安全执行、recover 包装等并发辅助。

基础用法：

```go
threadpkg.GoSafe(func() {
    // async work
})
threadpkg.GoSafeWithContext(ctx, func(ctx context.Context) {
    // async work with context
})
```

注意事项：异步任务仍需处理 context、超时、日志和资源释放。

验证：`go test ./thread`

## time

用途：时间格式化、解析、时间范围和常用时间计算。

基础用法：

```go
today := timepkg.Today()
end := timepkg.EndOfDay(time.Now())
text := timepkg.FormatRFC3339(time.Now())
```

注意事项：跨时区业务要明确 location；不要隐式依赖本机时区。

验证：`go test ./time`

## url

用途：URL 编码、query 参数拼接和请求 URL 生成。

基础用法：

```go
values := url.Values{"q": {"hello world"}}
query := urlpkg.EncodeValues(values)
requestURL := urlpkg.GenRequestURL("https://api.example.com", "/v1/users")
```

注意事项：拼接 URL 前确认 endpoint 和 path 是否已包含斜杠或 query。

验证：`go test ./url`

## uuid

用途：UUID 生成和解析辅助。

基础用法：

```go
id := uuidpkg.New()
```

注意事项：UUID 适合全局唯一标识，不应直接作为访问授权凭证。

验证：`go test ./uuid`

## writer

用途：dummy writer 和按时间或大小轮转的文件 writer。

基础用法：

```go
w, err := writerpkg.NewRotateFile(&writerpkg.ConfigRotate{
    Dir:      "./runtime/logs",
    Filename: "app",
})
```

注意事项：日志目录需要可写；轮转保留策略应结合磁盘容量设置。

验证：`go test ./writer`

## zip

用途：zip 压缩或解压相关辅助。

基础用法：

```go
err := zippkg.Zip("./source", "./target.zip")
err = zippkg.Unzip("./target.zip", "./output")
```

注意事项：解压外部 zip 时要防止路径穿越；输出目录应使用受控路径。

验证：`go test ./zip`
