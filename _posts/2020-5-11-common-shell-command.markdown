---
layout:     post
title:      "常用shell命令"
date:       2020-05-11 18:38:00
author:     "weak old dog"
header-img-credit: false
tags:
    - 运维
---

工作中经常涉及一些运维操作，一些shell命令用过了就会忘，以后有shell命令都会放在这里。

###### 求和、求平均值
比较常见的情况是由下面文件（或者其他命令的输出），需要求第二列的sum与平均值
```s
j@JM sample % cat testfile
c1 c2
abc 2323
efd 332
ghi 22
```
求和的方式比较简单，`NR!=1`表示过滤掉第一行，NR就是行号,`$2`表示第二列，`END`表示所有的行遍历结束后执行的语句，必须要大写，awk语句要放在单引号里面。
```s
j@JM sample % cat testfile | awk 'NR!=1 {sum+=$2} END{print sum/NR}'
2677
```
求平均数，这个地方要除以`NR-1`是因为第一行我们不算。
```s
j@JM sample % cat testfile | awk 'NR!=1 {sum+=$2} END{print sum/(NR-1)}'
892.333
```
###### grep前后
grep -C 5 foo file 显示file文件里匹配foo字串那行以及上下5行

grep -B 5 foo file 显示foo及前5行

grep -A 5 foo file 显示foo及后5行

另外，grep使用双引号表示不对特殊字符进行转义。

grep`或`操作，使用正则表达式，下面是过滤出含有a或者b的行

`grep -E "a|b"`

###### 循环读取文件的内容
下面是循环读取文件的内容，但是是以空格为分隔符的。
```s
#! /bin/bash

for line in $(cat files)
do
        rm -rf $line
done
```
上面命令可以写成一行：`for line in $(cat files); do echo ${line}; done`，如果要以行为分隔符，可以这么写
```s
#!/bin/bash
cat files | xargs -d "\n" echo
```
xargs 用来将前面命令的输出作为后面命令的输入，`-d "\n"`表示以空格为分隔符将输入分割，并把分割后的每一个字符串作为输入传递给后面的命令。xargs 的命令格式为`xargs [-options] [command]`，详细使用方式可以参考[xargs 命令教程](https://ruanyifeng.com/blog/2019/08/xargs-tutorial.html)

###### 查看磁盘读写io
```s
iostat -d -m -x 1 10000
```
* -d: 只看设备
* -m: 以MB为单位
* -x: 显示扩展数据
* 1：每秒打印一次
* 10000： 一共打印10000次

这部分参考：[Linux IO监控与深入分析](https://jaminzhang.github.io/os/Linux-IO-Monitoring-and-Deep-Analysis/)

查看进程写IO，-d: 只显示IO
```s
pidstat -d 1
```

###### nc
在1234端口起一个tcp服务
```s
nc -l 1234
```
往某个主机的端口发送数据
```s
echo -e '{"method":"HelloService.Hello","params":["hello"],"id":1}' | nc localhost 1234
```

###### 变量的默认值
如果变量没有定义，或者为空串，则使用默认值：
```s
a=${a:-default-value}
echo ${a}
```
输出：default-value

###### 安装包管理

**搜索已经安装的docker**

```s
sudo yum list installed | grep docekr

# 输出
docker.x86_64 2:1.12.6-16.el7.centos @extras 
docker-client.x86_64 2:1.12.6-16.el7.centos @extras 
docker-common.x86_64 2:1.12.6-16.el7.centos @extra
```

**删除已经安装的docker**

```s
sudo yum -y remove docker.x86_64
```

###### sed 替换某一行
```s
sed [options] 'command' file(s)
```

假设有如下文件：
```s
a
sometext sometext sometext TEXT_TO_BE_REPLACED sometext sometext sometext
b
c
d
```
将含有 `TEXT_TO_BE_REPLACED` 的行替换为 `This line is removed by the admin.`命令如下:
```s
sed -i '/TEXT_TO_BE_REPLACED/c\This line is removed by the admin.' /tmp/foo
```
其中 `-i` 表示直接修改文件，`c` 命令表示将指定行中的所有内容，替换成该选项后面的字符串。该命令的基本格式为：
```s
[address]c\用于替换的新文本
```

###### sed 注释某一行
比如有下面文件 /tmp/foo
```s
This is a 000 line.
This is 000 yet ano000ther line.
This is still yet another line.
```
注释所有含有 `000` 的行。其命令如下：
```s
sed -i '/000/s/^/#/' /tmp/foo
```
其中：
* `/000/` 匹配包含 `000` 的所有行
* `s` 对匹配的行进行替换。
* `^` 表示在行的开始，`#` 表示要插入的字符

参考[sed-功能强大的流式文本编辑器](https://wangchujiang.com/linux-command/c/sed.html)


###### ansible 基本用法
比如有一批机器，需要在这些机器上执行一条命令，hosts 文件如下：
```s
[all]
11.37.51.75
```
需要查看每个节点的版本内核，命令如下：
```s
ansible all -i ./hosts -m command -a "uname -a"
```

###### ssh 配置免密登录
一路回车，生成秘钥
```s
ssh-keygen
```
配置免密登录（-i参数是不是可以不加？）
```s
ssh-copy-id -i .ssh/id_rsa.pub  用户名字@192.168.x.xxx
```

研究下 strace

###### 参考
[AWK 简明教程](https://coolshell.cn/articles/9070.html)
