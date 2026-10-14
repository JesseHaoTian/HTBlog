# rclone 网盘故障排查与临时使用

换了新电脑之后，需要把原来电脑的文件悉数上传到NextCloud网盘，这时候，大文件传输和数量繁多的小文件传输需要一个稳定的解决方案，使用睿总的[rClone Windows 使用教程](https://peachlake.haohaha.cn/Use/01-rClone-Windows/)。然而，我的个人域名一直拖着安全证书没弄，使用过程中有一些小问题，用这一教程做临时方案。

## 排查

1. 命令行测试网盘

```bat
rclone lsd "serv.jesseht.fun:" -vv
```

报错：

```text
tls: failed to verify certificate
x509: certificate has expired or is not yet valid
```

说明 rclone 配置和 WebDAV 地址已读取，但服务器证书失效。

2. 临时跳过证书测试

```bat
rclone lsd "serv.jesseht.fun:" -vv --no-check-certificate
```

成功列出网盘目录，证明：

- rclone 配置正确；

- WebDAV 地址正确；

- 账号密码有效；

- Nextcloud 服务正常；

- 唯一问题是服务器证书。

# 解决
1. 临时设置 Rclone Browser
   
打开：Rclone Browser → Preferences → General

Default rclone options:

将原来的：``--fast-list``

改成：``--fast-list --no-check-certificate``

点击 OK，完全退出 Rclone Browser（包括系统托盘），然后重新启动。

!!! danger "安全提醒"
    `--no-check-certificate` 仅适用于临时排查。

    该参数会关闭 TLS 证书验证，使连接无法可靠地验证服务器身份，存在中间人攻击风险。
    
    修复服务器 HTTPS 证书后，应立即移除此参数。