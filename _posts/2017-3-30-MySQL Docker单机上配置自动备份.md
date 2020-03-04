## MySQL Docker 单机上配置自动备份

本文介绍在单一宿主机上如何配置自动备份。建议使用两个容器，其中一个容器作为 [MySQL](http://lib.csdn.net/base/mysql) 的服务器，用来处理数据；另一个容器用于自动备份。这样保证隔离，避免备份的容器影响到 MySQL Server 的可用性。

**配置 MySQL 服务器容器**

建立容器：

```
docker run --name mysql_shacus \
-p 3306:3306 \
-v /my/own/datadir:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD='123456' -d mysql:5.7.6
```

进入容器的 bash ：

```
docker exec -it mysql_shacus bash
```

设置时区：

```
tzselect
```

输入tzselect命令后，根据屏幕提示输入大洲、城市。因为中国的东八区时间，所以选择 Asia/Hong Kong。做好选择后，输入命令：

```
cp /usr/share/zoneinfo/Asia/Hong_Kong /etc/localtime
```

来持久化你的设置。

**配置 MySQL 备份容器**

建立容器：

```
docker run --name mysql_shacus_backup \
-v /my/mysql_backup:/shacus/mysql/backup \
-e MYSQL_ROOT_PASSWORD='123456' -d mysql:5.7.6
```

查看虚拟网络，其中 bridge 是 Docker 默认使用的虚拟网络：

```
docker inspect mysql_shacus
```

在返回的结果中，找到  部分。内容如下：

```
"Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "2e3f49ffafdc207925f8c3ede8926eb499aecf1e41568afc5f2bb27366cd418e",
                    "EndpointID": "81774a5603efa692cd4ebb78da328736d1f36719c3e988d9208465093e1d704c",
                    "Gateway": "192.168.0.1",
                    "IPAddress": "192.168.0.2",
                    "IPPrefixLen": 20,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:c0:a8:00:06"
                }
            }

```

networks 当前镜像的网络信息。IPAddress表示容器mysql_shacus的虚拟IP是 192.168.0.2 。记录下这个IP 。

进入容器的 bash ：

```
docker exec -it mysql-shacus_backup bash
```

更改时区，使用 tzselect 命令和cp命令，具体步骤与上面的mysql-a一样。

安装VIM和CRON定时任务：

```
apt-get update && apt-get install vim
apt-get update && apt-get install cron
```

生成一个shell脚本文件来进行备份。

```
vim /shacus/mysql/backup.sh
#!/bin/bash
lwDATE=$(date "+%Y%m%d%H")
#echo $lwDATE
mkdir /shacus/mysql/backup/$lwDATE
mysqldump -h '192.168.0.2' -uroot -p'123456' --databases Shacus > /shacus
/mysql/backup/$lwDATE/shacus.sql


```

利用这种方式在硬盘上备份数据。-h表示远程的服务器IP。-u 和 -p 分别是远程服务器的用户名和密码。这里不建议使用 mysqldump 的 –all-database 选项。因为该选项会把远程MySQL里的系统数据库也备份下来，包括 root 用户的密码等内容。如果你要把导出的内容导入到别的机器上，这些系统设置（比如用户名和密码）会覆盖你的机器上原来的设置。

设置定时任务的启动时间：

```
crontab -e
```

在打开的文件中输入一下内容（每6小时执行一次）：

```
* */6 * * * bash /shacus/mysql/backup.sh
```

系统默认使用VIM打开这个文件。修改完后用 wq 命令保存文件。 
启动定时任务的服务：

```
service cron restart
```

然后退出容器即可(使用crtl+p+q退出不关闭容器，使用crtl+c退出关闭容器)。