# MySQL操作库

## 增删数据库

**创建数据库**:crete database db_name; 

**删除数据库**:drop database db_name;

本质就是在/var/lib/mysql 创建/删除 一个目录, 创建的时候也可以写crete database if not exists db_name;

创建数据库的时候,有两个编码集:

1. 数据库编码集 -- 数据库未来存储数据
2. 数据库校验集 -- 支持数据库,进行字段比较使用的编码,本质也是一种读取数据库中数据的采用编码格式
3. 数据库无论对数据库任何操作,都必须保证操作和编码是一致的.

## 认识系统编码

**查看系统默认字符集以及校验规则**

show variables like 'character_set_database';(系统默认字符集)

show variables like 'collation_database';(校验规则)

**查看数据库支持的字符集**

show charset; 

## 指定编码创建数据库

create database d1 charset=utf8 collate utf8_general_ci;

自己写了校验码,/etc/my.cnf里面配置的我们就不会用了

## 验证不同校验码编码的影响
1.样例一
```sql
1. create database test1 collate utf8_general_ci;
2.use test1;
3.create table if not exists person(name varchar(20));
4.insert into person (name) values ('a);
5.insert into person (name) values ('A);
6.select * from person where name='a';
```
这段运行结果是a和A都显示出来,说明不区分大小写
2.样例二
```sql
1. create database test2 collate utf8_bin;
2.use test1;
3.create table if not exists person(name varchar(20));
4.insert into person (name) values ('a);
5.insert into person (name) values ('A);
6.select * from person where name='a';
```
这段运行只看到了a,说明区分大小写

order by也是一样,会被校验集影响

## 库的删改查

**删**:drop database db_name;

查看当前在那个数据库:select database();

**改**:alter database db_name;

**查**: show create database db_name;
```sql
+----------+----------------------------------------------------------------+
| Database | Create Database |
+----------+----------------------------------------------------------------+
| mytest | CREATE DATABASE `db_name` /*!40100 DEFAULT CHARACTER SET utf */ |  (不是注释,表示当前mysql版本大于4.01版本，就执行这句话)
+----------+----------------------------------------------------------------+
```

## 库的备份与恢复

备份语法:

mysqldump -P3306 -u root -p 密码 -B 数据库名 > 文件名.sql

还原语法:

source 数据库备份存储的文件路径/文件名.sql

**查看链接情况**:show processlist
 
可以告诉我们当前有哪些用户连接到我们的MySQL，如果查出某个用户不是你正常登陆的，很有可能你的数据库被
人入侵了。以后大家发现自己数据库比较慢时，可以用这个指令来查看数据库连接情况。