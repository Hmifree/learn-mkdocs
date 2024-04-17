# MySQL的基本查询

## 基本Insert
```sql
CREATE TABLE students (
id INT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
sn INT NOT NULL UNIQUE COMMENT '学号',
name VARCHAR(20) NOT NULL,
qq VARCHAR(20)
);
```

INSERT [INTO] table_name[(column [, column] ...)] VALUES (value_list) [, (value_list)] ...

指定了column就只能指定插入,如果不写column那么就是全列插入,values后面可以加入多组数据,要加逗号分隔开.

**插入替换1**

insert into students (id, sn, name) values (100, 10010, '唐大师') on duplicate key update sn = 10010, name = '唐大师';

-- 0 row affected: 表中有冲突数据，但冲突数据的值和 update 的值相等

-- 1 row affected: 表中没有冲突数据，数据被插入

-- 2 row affected: 表中有冲突数据，并且数据已经被更新

-- 通过 MySQL 函数获取受到影响的数据行数
SELECT ROW_COUNT();

**插入替换2**

replace into students (sn, name) values (20001,'曹操');

-- 1 row affected: 表中没有冲突数据，数据被插入
-- 2 row affected: 表中有冲突数据，删除后重新插入

## 基本select

```sql
CREATE TABLE exam_result (
id INT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
name VARCHAR(20) NOT NULL COMMENT '同学姓名',
chinese float DEFAULT 0.0 COMMENT '语文成绩',
math float DEFAULT 0.0 COMMENT '数学成绩',
english float DEFAULT 0.0 COMMENT '英语成绩'
);


INSERT INTO exam_result (name, chinese, math, english) VALUES
('唐三藏', 67, 98, 56),
('孙悟空', 87, 78, 77),
('猪悟能', 88, 98, 90),
('曹孟德', 82, 84, 67),
('刘玄德', 55, 85, 45),
('孙权', 70, 73, 78),
('宋公明', 75, 65, 30);
```
**全列查询**:SELECT * FROM exam_result;

**指定列查询**:SELECT id, name, english FROM exam_result;

**查询字段为表达式**:SELECT id, name, chinese+math+english FROM exam_result;

**为查询结果指定别名**:SELECT id, name, chinese + math + english 总分 FROM exam_result;

**结果去重**:select distinct math from exam_result;

**where条件**

|运算符 |说明|
| ---- | ----|
|>, >=, <, <= |大于，大于等于，小于，小于等于|
|= |等于，NULL 不安全，例如 NULL = NULL 的结果是 NULL|
|<=> |等于，NULL 安全，例如 NULL <=> NULL 的结果是 TRUE(1)|
|!=, <>| 不等于|
|BETWEEN a0 AND a1| 范围匹配，[a0, a1]，如果 a0 <= value <= a1，返回 TRUE(1)|
|IN (option, ...) |如果是 option 中的任意一个，返回 TRUE(1)|
|IS NULL| 是 NULL|
|IS NOT NULL |不是 NULL|
|LIKE |模糊匹配。% 表示任意多个（包括 0 个）任意字符；_ 表示任意一个字符|

逻辑运算符:

|运算符| 说明|
|----|----|
|AND| 多个条件必须都为 TRUE(1)，结果才是 TRUE(1)|
|OR |任意一个条件为 TRUE(1), 结果为 TRUE(1)|
|NOT |条件为 TRUE(1)，结果为 FALSE(0)|

**英语不及格的同学及英语成绩**:select name english from exam_result where english < 60;

**语文成绩在 [80, 90] 分的同学及语文成绩**:SELECT name, chinese FROM exam_result WHERE chinese >= 80 AND chinese <= 90;

**数学成绩是 58 或者 59 或者 98 或者 99 分的同学及数学成绩**:SELECT name, math FROM exam_result WHERE math IN (58, 59, 98, 99);

**姓孙的同学 及 孙某同学**:SELECT name FROM exam_result WHERE name LIKE '孙%';SELECT name FROM exam_result WHERE name LIKE '孙_';

**语文成绩好于英语成绩的同学**:SELECT name, chinese, english FROM exam_result WHERE chinese > english;

**总分在 200 分以下的同学**:SELECT name, chinese + math + english 总分 FROM exam_result WHERE chinese + math + english < 200;

注:Chinese + math + english < 200 不能简写,先from exam_result , 再 Chinese + math + english < 200, 最后才是select搜索

**语文成绩 > 80 并且不姓孙的同学**:SELECT name, chinese FROM exam_result WHERE chinese > 80 AND name NOT LIKE '孙%';

**孙某同学，否则要求总成绩 > 200 并且 语文成绩 < 数学成绩 并且 英语成绩 > 80**:
```sql
SELECT name, chinese, math, english, chinese + math + english 总分
FROM exam_result
WHERE name LIKE '孙_' OR (
chinese + math + english > 200 AND chinese < math AND english > 80
);
```

**NULL 的查询**:SELECT name, qq FROM students WHERE qq IS NOT NULL;

**结果排序**

语法:
```sql
SELECT ... FROM table_name [WHERE ...] ORDER BY column [ASC|DESC], [...];
```

+ 默认是升序
+ NULL 视为比任何值都小，升序出现在最上面
+ ORDER BY 中可以使用表达式
+ ORDER BY 子句中可以使用列别名(因为是现有数据，才来排序)

limit:

从 0 开始，筛选 n 条结果:limit n

从 s 开始，筛选 n 条结果:limit s, n

从 s 开始，筛选 n 条结果，比第二种用法更明确，建议使用: limit n offset s

建议：对未知表进行查询时，最好加一条 LIMIT 1，避免因为表中数据过大，查询全表数据导致数据库卡死

## update

**语法**:UPDATE table_name SET column = expr [, column = expr ...][WHERE ...] [ORDER BY ...] [LIMIT ...]

**将孙悟空同学的数学成绩变更为 80 分**:SELECT name, math FROM exam_result WHERE name = '孙悟空';

**将总成绩倒数前三的 3 位同学的数学成绩加上 30 分**:UPDATE exam_result SET math = math + 30 ORDER BY chinese + math + english LIMIT 3;

**将所有同学的语文成绩更新为原来的 2 倍**: UPDATE exam_result SET chinese = chinese * 2;

注:MySQL里面不能简写*=类似这类

## delete

**语法**:DELETE FROM table_name [WHERE ...] [ORDER BY ...] [LIMIT ...]

**删除孙悟空同学的考试成绩**:DELETE FROM exam_result WHERE name = '孙悟空';

**删除整表数据**:DELETE FROM for_delete;

**截断整表数据，注意影响行数是 0**:TRUNCATE [TABLE] table_name

truncate:

1. 只能对整表操作，不能像 DELETE 一样针对部分数据操作；
2. 实际上 MySQL 不对数据操作(不会记录在日志里)，所以比 DELETE 更快，但是TRUNCATE在删除数据的时候，并不经过真正的事
物，所以无法回滚
3. 会重置 AUTO_INCREMENT 项(delete不会重置)


日志:

bin log(1.历史上所操作的所以语句,MySQL优化后,会保留下来 2.记录数据本身 从而主从同步)/redo log(重做日志,保证MySQL宕机,保存下来)/undo log(事务回滚,事务的隔离性)

去重表操作:

1. CREATE TABLE no_duplicate_table LIKE duplicate_table;(空表,但结构和duplicate_table一样)
2. INSERT INTO no_duplicate_table SELECT DISTINCT * FROM duplicate_table;(distinct去重)
3. RENAME TABLE duplicate_table TO old_duplicate_table, no_duplicate_table TO duplicate_table;(重命名)

为什么最后是rename方式进行的？就是单纯的等一切都就绪了，然后统一放入、更新、生效.

## 聚合函数

|函数 |说明|
|----|----|
|COUNT([DISTINCT] expr) |返回查询到的数据的 数量|
|SUM([DISTINCT] expr)| 返回查询到的数据的 总和，不是数字没有意义|
|AVG([DISTINCT] expr)| 返回查询到的数据的 平均值，不是数字没有意义|
|MAX([DISTINCT] expr) |返回查询到的数据的 最大值，不是数字没有意义|
|MIN([DISTINCT] expr) |返回查询到的数据的 最小值，不是数字没有意义|

**统计班级共有多少同学**:SELECT COUNT(*) FROM students;

**统计数学成绩总分**:SELECT SUM(math) FROM exam_result;

**统计平均总分**:SELECT AVG(chinese + math + english) 平均总分 FROM exam_result;

**英语最高分**:SELECT MAX(english) FROM exam_result;

**返回 > 70 分以上的数学最低分**:SELECT MIN(math) FROM exam_result WHERE math > 70;

## 分组聚合统计

**group by**

语法:select column1, column2, .. from table group by column;


分组的目的是为了进行分组之后，方便进行聚合统计

指定列名，实际分组，是用该列的不同行数据来进行分组，组内的条件一定是相同的--可以被聚合压缩

分组就是把一组按条件拆分成了多个组，进行各自组内的统计，分组(分表)，不就是把一张表安装条件在逻辑上拆成了多个子表，然后分别对各自的子表进行聚合统计

**having**

是对聚合后的统计数据,条件筛选

**where和having的区别**:

where:对具体的任意列进行条件筛选

having:对分组之后的结果进行条件筛选 

阶段:from->where->gourp by->select->having

不要单纯认为,只有磁盘上的表结构导入到MySQL，真实存在的表，才叫做表

中间筛选出来的，包括最终结果，全部都是逻辑上的表！“MySQL一切皆表”

未来只要我们能够处理好单表的CURD，所有的sql场景，我们都能用统一的方式进行



