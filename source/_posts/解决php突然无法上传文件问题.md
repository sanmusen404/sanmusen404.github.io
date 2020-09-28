---
title: 解决php突然无法上传文件问题
description: docker的redis服务运行一段时间后无法连接
date: 2020-09-22 09:57:35
author: 三木森
tags:
- PHP
- Nginx
---
<!--more-->
# 问题描述
昨天还好好的，今天突然收到网站所有文件都无法上传，先调试打印上传的文件，发现所有能获取到文件名，但是所有其他参数都为空。

查看Nginx日志，有如下报错：
```log
[error] 4197#0: *109393 FastCGI sent in stderr: "PHP message: PHP Warning:  File 
upload error - unable to create a temporary file in Unknown on line 0
```

# 解决办法
有错误信息一般就解决了一半问题了，谷歌走起：
1. 排除是否为tmp目录问题
php上传文件会先放入tmp目录，所以运行Nginx的用户必须有写入权限，可使用如下代码获取PHP的tmp目录：
```php
$tmp_dir = ini_get('upload_tmp_dir') ? ini_get('upload_tmp_dir') : sys_get_temp_dir();
die($tmp_dir); #一般情况是/tmp
```
- 检查tmp目录是否为777权限
- 检查tmp目录是否有足够的空间。

2. 重启nginx和php-fpm
```shell
sudo systemctl restart php-fpm nginx
```
> 我的tmp目录满足条件，用方法2解决了问题，但是具体原因未知...