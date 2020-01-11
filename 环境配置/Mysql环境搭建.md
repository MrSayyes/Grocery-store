### 1、下载mysql安装包解压
### 2、在解压例如c:\web\mysql下面心中my.ini文件,配置如下：
[client]

#设置mysql客户端默认字符集

default-character-set=utf8

[mysqld]

#设置3306端口

port = 3306

#设置mysql的安装目录

basedir=C:\\web\\mysql-8.0.11

#允许最大连接数

max_connections=20

#服务端使用的字符集默认为8比特编码的latin1字符集

character-set-server=utf8

#创建新表时将使用的默认存储引擎

default-storage-engine=INNODB

### 3、初始化数据库(管理员在cmd的解压bin路径下面)：mysqld --initialize --console(产生root密码)
### 4、安装：mysqld install
### 5、启动：net start mysql
### 6、关闭：net stop mysql
### 7、登陆：mysql -uroot -p
### 8、修改默认密码：alter user 'root'@'localhost' identified by 'root';

# 常规操作
1、查看数据库 show databases;

2、查看表 show tables；

3、当前使用的数据库select database();

4、删除数据库drop database <数据库名>

5、创建数据库create database <数据库名>

6、使用数据库use <数据库名>

7、创建表create table <表名>(表字段属性)

8、删除表drop table <表名>

9、修改表名rename table 原表名 to 新表名

10、表字段查看：

​		desc 表名;

​		show columns from 表名;

​		describe 表名;

​		show create table 表名;