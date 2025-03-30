#  服务器关停启动与维修

## 关闭顺序

1. Ubuntu-Docker

> 进入 Nextcloud 路径,关停

```shell
cd Documents/nextcloud/
sudo docker-compose stop
```

2. TrueNAS

```shell
poweroff
```

3. Win-WIFI

3. Ubuntu-Lab（如有）

4. Win-GAME（如有）

5. Win-Lab

6. 不必手动关闭的系统
    1. iStoreOS


## 启动顺序

> 开机后自动进入 Win-Lab 桌面，浏览器进入 PVE 控制台：`192.168.x.x:8006`

1. 自启动系统
    1. iStoreOS
    2. Win-Lab
    3. Win-WIFI
    4. TrueNAS
    5. Ubuntu-Docker

2. TrueNAS

> 进入 Nextcloud 路径

```shell
cd /mnt/DataPool/DataPool/nextcloud/
```


> 添加用户和组 `www-data`
```shell
pw groupadd www-data -g 33
pw useradd www-data -g 33 -u 33
```

> 检查方式

```shell
ls -la /mnt/DataPool/DataPool/nextcloud/
```

第一行和第三行将出现相关数据

3. Ubuntu-Docker

> 进入 Nextcloud 路径

```shell
cd Documents/nextcloud/
```

> 挂载 `nfs`（`Ctrl+r` `nfs`）
>
> 命令中间的 IP 地址在 TrueNAS 首页，或者打开shell `IP a` 查找（太多，看不懂hhh...）

```shell
sudo mount -t nfs -o nfsvers=3 192.168.x.x:/mnt/DataPool/DataPool/nextcloud/data/ DataPool
```

> 启动！

```shell
sudo docker-compose restart
```

> 挂载 SSH

> 常见挂载多个 SSH，推荐打开多个终端，方便检查哪个挂了
>
> `ssh -NfR`： `f` 是后台运行，一般不用加，便于查看报错信息 
>
> `xxxx`：云服务器主机端口
>
> `yyyy`：本地监听端口
>
> `zzz`：用户名
>
> `a.b.c.d`：云服务器主机公网 IP

```shell
ssh -NR xxxx:localhost:yyyy zzz@a.b.c.d
```

4. Ubuntu-Lab

> 挂载 SSH，方法同上

## 维修

> 重启后常见 MySQL 的 IP 变更

> 进入 Ubuntu-Docker
>
> 查询新 IP

```shell
cd /Documents/nextcloud
sudo docker inspect mysql | grep "IPAddress"
```

> 更改配置

```shell
sudo docker exec -it nextcloud /bin/bash
#出现前缀 root@xxxx:var/www/html#
#继续输入
vim config/config.php
#此时进去改掉IP，完成后按“Esc”键确保处于普通模式
#正常方式退出vim :wq (保存并退出)
exit
#前缀消失
exit
```