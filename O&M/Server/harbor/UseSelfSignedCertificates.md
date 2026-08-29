### Why use a self-signed certificate

docker 默认无法使用http进行登陆，正式的ssl证书对于个人来说太过昂贵；自签证书是一个不错的选择。

### 添加到不安全（insecure）注册表

```json
{
  "insecure-registries": [
    "reg.fonzosong.cn:4433"
  ]
}

```

