# 复合查询

## 基本查询回顾

**查询工资高于500或岗位为MANAGER的雇员，同时还要满足他们的姓名首字母为大写的J**:
```sql
select * from EMP where (sal>500 or job='MANAGER') and ename like 'J%';
```
**按照部门号升序而雇员的工资降序排序**:
```sql
select * from EMP order by deptno, sal desc;
```
**使用年薪进行降序排序**:
```sql
select ename, sal*12+ifnull(comm,0) as '年薪' from EMP order by 年薪 desc;
```
**显示工资最高的员工的名字和工作岗位**:
```sql
select ename, job from EMP where sal = (select max(sal) from EMP);
```
**显示工资高于平均工资的员工信息**:
```sql
select ename, sal from EMP where sal>(select avg(sal) from EMP);
```
**显示每个部门的平均工资和最高工资**:
```sql
select deptno, format(avg(sal), 2) , max(sal) from EMP group by deptno;
```
**显示平均工资低于2000的部门号和它的平均工资**:
```sql
select deptno, format(avg(sal), 2) , max(sal) from EMP group by deptno;
```
**显示平均工资低于2000的部门号和它的平均工资**:
```sql
select deptno, avg(sal) as avg_sal from EMP group by deptno having avg_sal<2000;
```
**显示每种岗位的雇员总数，平均工资**:
```sql
select job,count(*), format(avg(sal),2) from EMP group by job;
```
## 多表查询

**显示部门号为10的部门名，员工名和工资**:
```sql
select ename, sal,dname from EMP, DEPT where EMP.deptno=DEPT.deptno and DEPT.deptno =
10;
```
**显示各个员工的姓名，工资，及工资级别**:
```sql
select ename, sal, grade from EMP, SALGRADE where EMP.sal between losal and hisal;
```
## 自连接

from相同的数据库是不行的,但是as后面改了个名称就可以了
```sql
select * from lkt, lkt; //err
select * from lkt as t1, lkt as t2; // right
```

**显示员工FORD的上级领导的编号和姓名（mgr是员工领导的编号--empno）**:

**1.使用的子查询**:
```sql
select empno,ename from emp where emp.empno=(select mgr from emp where ename='FORD');
```

**2.使用多表查询（自查询）**:
```sql
select leader.empno,leader.ename from emp leader, emp worker where leader.empno =
worker.mgr and worker.ename='FORD';
```
## 子查询

**单行子查询**

显示SMITH同一部门的员工
```sql
select * from EMP WHERE deptno = (select deptno from EMP where ename='smith');
```
**多行子查询**

in关键字；查询和10号部门的工作岗位相同的雇员的名字，岗位，工资，部门号，但是不包含10自己的
```sql
select ename,job,sal,deptno from emp where job in (select distinct job from emp where
deptno=10) and deptno<>10;
```
all关键字；显示工资比部门30的所有员工的工资高的员工的姓名、工资和部门号
```sql
select ename, sal, deptno from EMP where sal > all(select sal from EMP where
deptno=30);
```
any关键字；显示工资比部门30的任意员工的工资高的员工的姓名、工资和部门号（包含自己部门的员工）
```sql
select ename, sal, deptno from EMP where sal > any(select sal from EMP where
deptno=30);
```
**多列子查询**
```sql
select ename from EMP where (deptno, job)=(select deptno, job from EMP where
ename='SMITH') and ename <> 'SMITH';
```
## 子查询与from

**显示每个高于自己部门平均工资的员工的姓名、部门、工资、平均工资**

```sql
select ename, deptno, sal, format(asal,2) from EMP,
(select avg(sal) asal, deptno dt from EMP group by deptno) tmp
where EMP.sal > tmp.asal and EMP.deptno=tmp.dt;
```
**查找每个部门工资最高的人的姓名、工资、部门、最高工资**
```sql
select EMP.ename, EMP.sal, EMP.deptno, ms from EMP,
(select max(sal) ms, deptno from EMP group by deptno) tmp
where EMP.deptno=tmp.deptno and EMP.sal=tmp.ms;
```

**显示每个部门的信息（部门名，编号，地址）和人员数量**

**方法一:使用多表**
```sql
select DEPT.dname, DEPT.deptno, DEPT.loc,count(*) '部门人数' from EMP, DEPT
where EMP.deptno=DEPT.deptno
group by DEPT.deptno,DEPT.dname,DEPT.loc;
```
**方法2：使用子查询**
```sql
select DEPT.deptno, dname, mycnt, loc from DEPT,
(select count(*) mycnt, deptno from EMP group by deptno) tmp
where DEPT.deptno=tmp.deptno;
```

## 合并查询

在实际应用中，为了合并多个select的执行结果，可以使用集合操作符 union，union all

1 union

该操作符用于取得两个结果集的并集。当使用该操作符时，会自动去掉结果集中的重复行。

**案例：将工资大于2500或职位是MANAGER的人找出来**

```sql
select ename, sal, job from EMP where sal>2500 union
select ename, sal, job from EMP where job='MANAGER';--去掉了重复记录
```

2.union all

该操作符用于取得两个结果集的并集。当使用该操作符时，不会去掉结果集中的重复行。

```sql
mysql> select ename, sal, job from EMP where sal>2500 union all
select ename, sal, job from EMP where job='MANAGER';
```