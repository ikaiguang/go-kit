# id

分布式 ID 工具，封装 bwmarrin/snowflake 和 sonyflake，并提供节点 ID 推导。

## 基础用法

```go
id, err := idpkg.NextID()
nodeID, err := idpkg.GenNodeID()
```

## 注意事项

- 分布式 ID 使用前确认系统时间稳定。
- 多节点部署时必须确保节点号唯一。
- ID 不应直接作为访问授权凭证。

## 验证

```bash
go test ./id
```

## 参考

参考文档

- [美团:Leaf](https://github.com/Meituan-Dianping/Leaf)
- [百度:uid-generator](https://github.com/baidu/uid-generator)
- [bwmarrin/snowflake](https://github.com/bwmarrin/snowflake)
- [sony/sonyflake](https://github.com/sony/sonyflake)
- [edwingeng/wuid](https://github.com/edwingeng/wuid)
