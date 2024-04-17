# MySQL内置函数

## 日期函数

|函数名称|描述|
|----|----|
|current_date()|当前日期|
|current_time()|当前时间|
|current_timestamp()|当前时间戳|
|date(datetime)|返回datetime参数的日期部分|
|date_add(date, interval d_value_type)|在date中添加日期或时间 interval后的数值单位可以是:year minute second day|
|date_sub(date, interval d_value_type)|在date中减去日期或时间 interval后的数值单位可以是:year minute second day|
|datediff(date1, date2)|两个日期的差,单位是天|
|now()|当前日期时间|

案例: 

**创建一个表, 查询两分钟以前增加的内容**:select * from msg where date_add(sendtime, interval 2 minute) > now();

## 字符串函数

|函数名称|描述|
|----|----|
|charset(str)|返回字符串字符集|
|concat(string1, string2, ...) |连接字符串|
|instr(string, substring)|返回substring在string中出现的位置,没有返回0|
|ucase(string1)|转换成大写|
|lcase(string1)|转换成小写|
|left(string1, length)|从string1中左边起取length个字符|
|length(string)|string的长度|
|replace(str, search_str, replace_str)|在str中用replace_str替换search_str|
|strcmp(string1, string2)|逐字符比较两字符的大小|
|substring(str, position [,length])|从str的position开始,取length个字符|
|ltrim(string) rtrim(string) trim(string)|去除前空格或后空格|

案例:

**获取emp表的ename列的字符集**:select charset(ename) from EMP;

**要求显示student表中的信息，显示格式：“XXX的语文是XXX分，数学XXX分，英语XXX分”**:select concat(name, '的语文是',chinese,'分，数学是',math,'分') as '分数' from student;

**求学生表中学生姓名占用的字节数**:select length(name), name from student;

**将EMP表中所有名字中有S的替换成'上海'**:select replace(ename, 'S', '上海') ,ename from EMP;

**截取EMP表中ename字段的第二个到第三个字符**:select substring(ename, 2, 2), ename from EMP;

**以首字母小写的方式显示所有员工的姓名**:select concat(lcase(substring(ename, 1, 1)),substring(ename,2)) from EMP;

## 数学函数

|函数名称|描述|
|----|----|
|abs(number)|绝对值函数|
|bin(decimal_number)|十进制转化为二进制|
|hex(decimalNumber)|转换成十六进制|
|conv(number, from_base, to_base)|进制转换|
|celling(number)|向上取整|
|floor(number)|向下取整|
|format(number, decimal_places)|格式化, 保留小数位数|
|rand()|返回随机浮点数,范围[0.0, 1.0)|
|mod(number, denominator)|取模,求余|

## 其他函数

+ user():查询当前用户

+ md5(str):对一个字符串进行摘要,摘要后得到一个32位字符串

+ database():显示当前正在使用的数据库

+ password():MySQL数据库使用该函数对用户加密

+ ifnull(val1， val2):如果val1为null，返回val2，否则返回val1的值