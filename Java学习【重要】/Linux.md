# Linux学习

## 1、top查看整机性能

1）主要看cpu，内存的内容

- cpu:比例高不好

- mem：比例高不好
- command：程序、命令
- pid：进程id

2）多核cpu按数字【键盘1】切换可以看到多核cpu的情况

3）查看id=idle(空闲率)，越大越好

4）load average负载率：值1（1分钟）,值2（5分钟）,值3（15分钟）

- （值1+值2+值3）/3*100%>60%是负担重

5）使用qq退出

6）低配版cpu指令：uptime：看load average的内容

## 2、free查看内存

- free -g：不准确

- free -m：兆展示，推荐

## 3、df查看硬盘

- df -h

## 4、vmstat查看cpu

 vmstat  -n  2  3：每2秒刷一次，共3次

看头看尾即可：r运行，b阻塞；us用户消耗，sy系统消耗，us+sy>80%负担太重

- 查看额外

> 查看所有cpu核信息：mpstat -P ALL 2
>
> 每个进程使用cpu的用量分解信息：pidstat-u 1 -p 进程编号

## 5、iostat查看磁盘IO

iostat -xdk 2 3:秒2秒刷新一次，共3次

> 数据库sql会导致
>
> 看r/s读， w/s写

**输出内容【重要参数】：**

- **util 一秒钟有百分之几的时间用于I/O操作，接近100%时，表示磁盘带宽跑满，需要优化程序或者增加磁盘**

## 6、ifstat查看网络IO

> 默认本地没有，需要下载

1） 本地没有ifstat支持，需下载

wget http://gael.roualland.free.fr/ifstat-1.1.tar.gz

tar xzvf ifstat-1.1.tar.gz

cd ifstat-1.1

./configure

make

make install

2）ifstat 1

查看网络IO

## 7、chmod权限

## 8、ifconfig看ip