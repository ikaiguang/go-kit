# snowflake

Snowflake ID 生成工具。

## 基础用法

```go
id, err := snowflakepkg.NextID()
```

## 注意事项

分布式部署时确认节点号唯一和系统时间稳定。

## 验证

```bash
go test ./snowflake
```
