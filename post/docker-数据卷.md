---
title:       "doceker系列-数据卷容器"
subtitle:    ""
description: ""
date:        2019-05-20
author:      "麦子"
image:       "https://c.pxhere.com/images/91/4d/7a1a53a9399ed55ee697daf983d1-1589459.jpg!d"
tags:        ["docker", "数据卷"]
categories:  ["Tech" ]
---

[TOC]



**说明：转载地址 https://www.cnblogs.com/wade-luffy/p/6542539.html**

# 数据卷

如果你现在的容器挂掉了， 那么比如你redis里面的RDB或者AOF的数据， 都会丢失， 如果你挂载到了宿主机，那么在重新启动，数据都在。**同时多个容器共享一个配置，多个容器之间的数据还是同步。也就是，一些数据可以统一进行修改处理 。**

# 数据卷容器

如果用户需要在多个容器之间共享一些持续更新的数据，最简单的方式是使用数据卷容器。数据卷容器也是一个容器，但是它的目的是专门用来提供数据卷供其他容器挂载。

# 数据卷应该场景

**转载地址： https://blog.51cto.com/idweb/2322087?source=dra**

- 在多个容器之间共享数据，多个容器可以同时以只读或者读写的方式挂载统一个数据卷，从而共享数据卷中的数据
- 当宿主机不能保证一定存在某一个目录或一些固定的文件路径时，使用数据卷可以规避这种限制带来的问题
- 当想把容器中的数据存储在宿主机之外的地方时，比如远程主机上和云存储上
- 当你需要把容器数据在不同宿主机之间备份、恢复或迁移时，数据卷是很好的选择

## 数据覆盖问题

- 如果挂载一个空的数据卷到容器中的一个非空目录中，那么这个目录下的文件会被复制到数据卷中
- 如果挂载一个非空的数据卷到容器中的一个目录中，那么容器中的目录会显示数据卷中的数据。如果原来容器中的目录有数据，那么原始数据会被隐藏掉

这两个规则都非常重要，灵活利用第一个规则可以帮助我们初始化数据卷中的内容。掌握第二个规则可以保证挂载数据卷后的数据总是你期望的结果。

# 实例操作

```dockerfile
#创建一个数据卷容器dbdata
$ docker run -it -v /dbdata --name dbdata ubuntu
#然后，可以在其他容器中使用--volumes-from来挂载dbdata容器中的数据卷.

#例如创建db1和db2两个容器，并从dbdata容器挂载数据卷：

$ docker run -it --volumes-from dbdata --name db1 ubuntu
$ docker run -it --volumes-from dbdata --name db2 ubuntu

#此时，容器db1和db2都挂载同一个数据卷到相同的/dbdata目录。三个容器任何一方在该目录下的写入，其他容器都可以看到。

#可以多次使用--volumes-from参数来从多个容器挂载多个数据卷。还可以从其他已经挂载了容器卷的容器来挂载数据卷。
#使用--volumes-from参数所挂载数据卷的容器自身并不需要保持在运行状态。

#如果删除了挂载的容器（包括dbdata、db1和db2），数据卷并不会被自动删除。如果要删除一个数据卷，必须在删除最后一个还挂载着它的容器时显式使用docker rm -v命令来指定同时删除关联的容器。

```

# 数据卷容器来迁移数据

可以利用数据卷容器对其中的数据卷进行备份、恢复，以实现数据的迁移。

## 备份

```dockerfile
docker run --volumes-from dbdata -v $(pwd):/backup --name worker ubuntu tar cvf /backup/backup.tar /dbdata

首先利用ubuntu镜像创建了一个容器worker。使用--volumes-from dbdata参数来让worker容器挂载dbdata容器的数据卷(即dbdata数据卷),使用-v  $(pwd):/backup参数来挂载本地的当前目录到worker容器的/backup目录。worker容器启动后，使用了tar cvf  /backup/backup.tar /dbdata命令来将/dbdata下内容备份为容器内的/backup/backup.tar，即宿主主机当前目录下的backup.tar。
```



## 恢复

```dockerfile
如果要将数据恢复到一个容器，可以按照下面的步骤操作。

首先创建一个带有数据卷的容器dbdata2：

$ docker run -v /dbdata --name dbdata2 ubuntu /bin/bash

然后创建另一个新的容器，挂载dbdata2的容器，并使用untar解压备份文件到所挂载的容器卷中：

$ docker run --volumes-from dbdata2 -v $(pwd):/backup --name worker ubuntu bash

cd /dbdata

tar xvf /backup/backup.tar
```



# 把数据存储到SSH服务器

**注意：目前这个不成功 ，后期在进行修改，有可能这个插件需要填写设置秘钥**

```bash
#下载插件
docker plugin install --grant-all-permissions vieux/sshfs

#创建volume 
docker volume create --driver vieux/sshfs \
    -o sshcmd=maizissh@127.0.0.1:/home/maizissh/sshvolume \
    -o password=maizi_today \
    mysshvolume
#启动容器 
docker run -d --name sshfs \
    --mount type=volume,volume-driver=vieux/sshfs,source=mysshvolume,target=/home/maizissh/sshvolume \
    ubuntu /bin/bash

#提示错误
docker: Error response from daemon: error while mounting volume '': VolumeDriver.Mount: sshfs command execute failed: exit status 1 (read: Connection reset by peer

#感觉要设置秘钥， 后期这个需要的时候在进行处理，
```

 