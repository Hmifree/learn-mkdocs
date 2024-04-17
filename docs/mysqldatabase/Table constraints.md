# MySQL表的约束

## 概念

表的约束:表中一定要各种约束,通过约束,让我们未来插入数据库表中的数据是符合预期的.约束本质是通过技术手段,倒逼程序员,插入正确的数据.反过来,站在MySQL的视角,
凡是插入进来的数据,都是符合数据约束的.

约束的最终目的:保证数据的完整性和可预期性.

## 非空约束

+ 两个值：null（默认的）和not null(不为空)
+ 数据库默认字段基本都是字段为空，但是实际开发时，尽可能保证字段不为空，因为数据为空没办法参与运
算。

示例:
```sql
create table myclass(
 class_name varchar(20) not null,
 class_room varchar(10) not null);
```
插入为空就报错,示例:
```sql
mysql> insert into myclass(class_name) values('class1');
ERROR 1364 (HY000): Field 'class_room' doesn't have a default value
```

## default约束

default:如果设置了,用户将来插入.有具体数据,就用用户的,没有就默认的.

如果我们没有明确指定一列要插入,用的是default,如果建表中,对应列默认没有default值,无法直接插入.

default和not null不冲突,而是相互补充的.当用户想插入(NULL, 合法数据).当用户忽略这一列的时候,使用默认值(如果设置了),如果用户没有设置,直接报错.

如果没有写not null、也没有写default,他默认会生成一个default NULL,如果写了not null，那么不再生成default,需要自己添加,或者在写入时不省略改列.

## 列描述

列描述：comment，没有实际含义，专门用来描述字段，会根据表创建语句保存，用来给程序员或DBA来进行了
解。

## zerofill
```sql
Create Table: CREATE TABLE `tt3` (
`a` int(5) unsigned zerofill DEFAULT NULL, --具有了zerofill
`b` int(10) unsigned DEFAULT NULL
);
```

原来的1变成00001，这就是zerofill属性的作用，如果宽度小于设定的宽度（这里设置的是
5），自动填充0。要注意的是，这只是最后显示的结果，在MySQL中实际存储的还是1。

## 主键

主键：primary key用来唯一的约束该字段里面的数据，不能重复，不能为空，一张表中最多只能有一个主键；主键
所在的列通常是整数类型。

+ 创建表的时候直接在字段上指定主键
```sql
create table tt13 (
 id int unsigned primary key comment '学号不能为空',
 name varchar(20) not null
);
```

+ 主键约束：主键对应的字段中不能重复，一旦重复，操作失败。

+ 当表创建好以后但是没有主键的时候，可以再次追加主键

alter table 表名 add primary key(字段列表)

+ 删除主键

alter table 表名 drop primary key;

+ 复合主键

在创建表的时候，在所有字段之后，使用primary key(主键字段列表)来创建主键，如果有多个字段作为主键，
可以使用复合主键。
```sql
mysql> create table tt14(
 id int unsigned,
 course char(10) comment '课程代码',
 score tinyint unsigned default 60 comment '成绩',
 primary key(id, course) -- id和course为复合主键
);
```

## 自增长

auto_increment：当对应的字段，不给值，会自动的被系统触发，系统会从当前字段中已经有的最大值+1操作，得
到一个新的不同的值。通常和主键搭配使用，作为逻辑主键.

自增长的特点:
+ 任何一个字段要做自增长，前提是本身是一个索引（key一栏有值）
+ 自增长字段必须是整数
+ 一张表最多只能有一个自增长
+ 在插入后获取上次插入的 AUTO_INCREMENT 的值（批量插入获取的是第一个值）
```sql
create table tt21(
 id int unsigned primary key auto_increment,
 name varchar(10) not null default ''
 )auto_increment = 500;
 ```

 select last_insert_id():最新表的auto_increment的值.

 ## 唯一键

 一张表中有往往有很多字段需要唯一性，数据不能重复，但是一张表中只能有一个主键：唯一键就可以解决表中有多个字段需要唯一性约束的问题。

唯一键的本质和主键差不多，唯一键允许为空，而且可以多个为空，空字段不做唯一性比较。

关于唯一键和主键的区别：
我们可以简单理解成，主键更多的是标识唯一性的。而唯一键更多的是保证在业务上，不要和别的信息出现重复。

## 外键

外键用于定义主表和从表之间的关系：外键约束主要定义在从表上，主表则必须是有主键约束或unique约束。当定
义外键后，要求外键列数据必须在主表的主键列存在或为null。

**语法**:foreign key (字段名) references 主表(列)

**先创立主键表**
```sql
create table myclass (
    id int primary key,
    name varchar(30) not null comment'班级名'
);
```

**再创立从表**
```sql
create table stu (
id int primary key,
    name varchar(30) not null comment '学生名',
    class_id int,
    foreign key (class_id) references myclass(id)
);
```

**正常插入数据**
```sql
insert into myclass values(10, 'C++大牛班'),(20, 'java大神班');
```

**插入一个班级号为30的学生，因为没有这个班级，所以插入不成功**
```sql
insert into stu values(102, 'wangwu',30);
```

**插入班级id为null，比如来了一个学生，目前还没有分配班级**
```sql
insert into stu values(102, 'wangwu', null);
```

**如何理解外键约束**
首先我们承认，这个世界是数据很多都是相关性的。

理论上，上面的例子，我们不创建外键约束，就正常建立学生表，以及班级表，该有的字段我们都有。
此时，在实际使用的时候，可能会出现什么问题？

有没有可能插入的学生信息中有具体的班级，但是该班级却没有在班级表中？

比如班级只开了100班，101班，但是在上课的学生里面竟然有102班的学生(这个班目前并不存在)，这很
明显是有问题的。

因为此时两张表在业务上是有相关性的，但是在业务上没有建立约束关系，那么就可能出现问题。

解决方案就是通过外键完成的。建立外键的本质其实就是把相关性交给mysql去审核了，提前告诉mysql表之间的约束关
系，那么当用户插入不符合业务逻辑的数据的时候，mysql不允许你插入。