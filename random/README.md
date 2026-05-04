# random

生成随机字符串、验证码、密码、订单号、trace ID，或随机选取/打乱切片。

## 基础用法

```go
code := randompkg.VerifyCode(6)
token := randompkg.Token(32)
password := randompkg.Password(12)
```

## 注意事项

当前包基于 `math/rand`，不适合生成高安全等级密钥；安全 token 应使用 `crypto/rand`。

## 验证

```bash
go test ./random
```
