# writer

dummy writer 和按时间或大小轮转的文件 writer。

## 基础用法

```go
w, err := writerpkg.NewRotateFile(&writerpkg.ConfigRotate{
	Dir:      "./runtime/logs",
	Filename: "app",
})
```

## 注意事项

日志目录需要可写；轮转保留策略应结合磁盘容量设置。

## 验证

```bash
go test ./writer
```
