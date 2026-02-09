---
title: sftp_sql_1
date: 2022-03-20
categories:
  - problem
---

2022/3/20 发现sftp连接不上，显示connection closed

<!-- more -->

###### 查看 sshd_config 配置文件， 发现没有/usr/libexec/sftp-server可执行程序

`override default of no subsystems`

`Subsystem      sftp    /usr/libexec/opensshj/sftp-server`

可以修改如下：

`override default of no subsystems`

`Subsystem      sftp    internal-sftp`

并且发现里头sftp路径为：usr/libexec/opensshj/sftp-server

修改j为好，但仍然connection closed

###### sftp-server 与 internal-sftp 的区别可看

https://serverfault.com/questions/660160/openssh-difference-between-internal-sftp-and-sftp-server

###### ssh可以使用但sftp不能使用

http://bbs.chinaunix.net/thread-4252902-1-1.html

ssh可以使用sftp不可以使用

Vim /etc/passwd
把 bbscgl:500:500::/kssftp:/bin/false
改为 bbscgl:500:500::/kssftp:/bin/bash

但是Vi /etc/passwd 失败

###### 尝试重启服务sshd.service

systemctl restart sshd.service

防火墙

sudo systemctl status wall fired

查看防火墙状态：`sudo systemctl status firewalld`
关闭防火墙:`sudo systemctl stop firewalld`

##### 导入stif.csv

###### 解决办法1:转换成txt再导入

无法转换成txt

###### 解决办法2:切换encoding

format 'csv', delimiter ', encoding 'ISO-8859-1')"

可以导入，但是导入了以后还是乱码

###### 解决办法3:使用工具切换成txt

使用wps另存为txt，解决了大数据量的问题

##### 导入relation.csv

centos更改文件所有者

```bash
chown jay:fefjay a.txt #修改文件所属用户为jay，所属用户组为fefjay
```

linux解压zip

```bash
cd zip
ls
unzip abc.zip
#出现inflating即为成功
```

vi正则

```bash
%s/"//g
```

vim正则

```vim
:%s/foo/bar/g    会在全局范围(%)查找foo并替换为bar，所有出现都会被替换（g）

:w 保存不退出
:w 新文件名 把文件另存为新文件
:q 不保存退出
:wq 保存退出
:! 强制
:q! 强制不保存退出
:wq! 强制保存退出
```

sed

```bash
sed -i "s/&#@/,/g" stif_2022-03-12.csv
```

gpfdist导入

```bash
ps -ef | grep gpfdist
gpfdist -d /home/admin/data/ -p 8081 -l /home/admin/data.log &
```

linux给文件改名

```bash
sudo mv test.txt new.txt
```
