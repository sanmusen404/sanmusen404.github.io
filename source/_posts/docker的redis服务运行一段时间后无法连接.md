---
title: docker的redis服务运行一段时间后无法连接
description: docker的redis服务运行一段时间后无法连接
date: 2020-09-17 10:10:36
author: 三木森
tags:
- Docker
- Redis
---

# 定位问题
## 查看redis日志
```shell
[root@cloud ~]# docker logs xxx | less
...
1:M 17 Sep 2020 00:59:36.287 * 100 changes in 300 seconds. Saving...
1:M 17 Sep 2020 00:59:36.690 * Background saving started by pid 12647
12647:C 17 Sep 2020 01:00:26.706 # Write error saving DB on disk: No space left on device
1:M 17 Sep 2020 01:00:27.646 # Background saving error
1:M 17 Sep 2020 01:00:27.746 * 100 changes in 300 seconds. Saving...
1:M 17 Sep 2020 01:00:28.243 * Background saving started by pid 12648
12648:C 17 Sep 2020 01:01:06.715 # Write error saving DB on disk: No space left on device
1:M 17 Sep 2020 01:01:07.749 # Background saving error
1:M 17 Sep 2020 01:01:07.850 * 100 changes in 300 seconds. Saving...
1:M 17 Sep 2020 01:01:08.348 * Background saving started by pid 12649
12649:C 17 Sep 2020 01:01:46.865 # Write error saving DB on disk: No space left on device
1:M 17 Sep 2020 01:01:47.763 # Background saving error
```
大概意思是持久化时存储位置磁盘空间不足导致写入错误。
## 查看存储位置以及剩余空间余量
```shell
[root@cloud ~]# docker info | less
...
Docker Root Dir: /mnt/data2/Container/docker
...
[root@cloud ~]# docker info
Filesystem      Size  Used Avail Use% Mounted on
/dev/nvme0n1    477G  475G  2.3G 100% /mnt/data2
```
可以看到已经无剩余的空间了
# 解决办法
1. 修改redis.conf持久化存储路径  
docker安装的redis默认是没有配置文件的，修改还要加redis.conf配置文件、映射外部路径到容器等操作，比较麻烦。
2. 存储扩容  
3. 修改docker的Docker Root Dir  
```shell
[root@cloud ~]# vi /etc/docker/daemon.json
{
  "data-root": "/cloud/docker"
}
[root@cloud ~]# systemctl daemon-reload
[root@cloud ~]# systemctl restart docker.service
[root@cloud ~]# docker info | less
...
Docker Root Dir: /cloud/docker
...
```
直接修改docker的根目录，但是无法使用以前的镜像、容器等,所以此方法不推荐
4. 使用软连接
```shell
[root@cloud ~]# cp -rf /mnt/data2/Container/docker/* /cloud/docker/
[root@cloud ~]# rm -rf /mnt/data2/Container/docker
[root@cloud ~]# ln -s /cloud/docker/ /mnt/data2/Container/docker
[root@cloud ~]# systemctl restart docker.service
```
数据完美迁移，能保留数据，也不用改配置，而且操作也简单，推荐！