# zip

zip 压缩或解压相关辅助。

## 基础用法

```go
err := zippkg.Zip("./source", "./target.zip")
err = zippkg.Unzip("./target.zip", "./output")
```

## 注意事项

解压外部 zip 时要防止路径穿越；输出目录应使用受控路径。

## 验证

```bash
go test ./zip
```
