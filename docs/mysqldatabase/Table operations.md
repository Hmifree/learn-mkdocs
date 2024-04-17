# 表的操作

## 创建表

语法:
```sql
CREATE TABLE table_name (
field1 datatype,
field2 datatype,
field3 datatype
) character set 字符集 collate 校验规则 engine 存储引擎;
```
+ field 表示列名
+ datatype 表示列的类型
+ character set 字符集，如果没有指定字符集，则以所在数据库的字符集为准
+ collate 校验规则，如果没有指定校验规则，则以所在数据库的校验规则为准

**创建表案例**

MyIsam引擎
```sql
create table user1 (
id int,
name varchar(20) comment '用户名',
password char(32) comment '密码',
birthday date comment '生日'
) character set utf8 engine MyIsam;
```
/var/lib/mysql/数据库名会出现user1.frm、user1.MYD、user1.MYI

user1.frm：表结构
user2.MYD：表数据
user3.MYI：表索引


innodb引擎
```sql
create table user2 (
id int,
name varchar(20) comment '用户名',
password char(32) comment '密码',
birthday date comment '生日'
) character set utf8 engine=InnoDB;
```

/var/lib/mysql/数据库名会出现user2.frm、user2.ibd

## 查看表

desc 表名;

## 修改表

**在user1表添加二条记录**

insert into user1 values(1, '张三', '12345', '2010-1-15');

insert into user1 values(2, '李四', '54321', '2015-1-15');

**在user1表添加一个字段，用于保存图片路径**

alter table user1 add asserts varchar(100) comment '图片路径' after birthday;

**修改name，将其长度改成60**

alter table user1 modify name varchar(60);

**删除password列**
> 注意：删除字段一定要小心，删除字段及其对应的列数据都没了

alter table user1 drop password;

**修改表名为lkt**

alter table user1 rename to lkt;

or

alter table user1 rename lkt;

**将name列修改为xingming**

alter table lkt change name xingming varchar(60);

注意:不要随意改和删




