# MySQL
## 基本概念

| 名称  | 概念                         | 作用              | 操作                                                       |
|-----|----------------------------|-----------------|----------------------------------------------------------|
| DDL | Data Definition Language   | 定义数据库对象         | CREATE、ALTER、DROP、TRUNCATE、COMMENT、GRANT、REVOKE          |
| DML | Data Manipulation Language | 操作数据库对象         | SELECT、INSERT、UPDATE、DELETE、CALL、EXPLAIN PLAN、LOCK TABLE |
| DCL | Data Control Language      | 设置或更改数据库用户或角色权限 | COMMIT、SAVEPOINT、ROLLBACK、SET TRANSACTION                |

## 非外键约束
为了防止不合规范的数据入库，在用户对数据进行插入、修改、删除等操作时，Mysql提供了一种机制来检查数据库中的数据是否满足规定的条件，以保证数据库中
数据的准确性和一致性，这种机制就是完整性约束。 Mysql主要支持一下几种完整性约束。

| 约束条件           | 约束描述                   |
|----------------|------------------------|
| PRIMARY KEY    | 主键约束，约束字段的值可唯一地标识对应的记录 |
| NOT NULL       | 非空约束，约束字段的值不能为空        |
| UNIQUE         | 唯一约束， 约束字段的值是唯一的       |
| CHECK          | 检查约束，限制某个字段的取值范围       |
| DEFAULT        | 默认值约束，约束字段的默认值         |
| AUTO_INCREMENT | 自动增加约束，约束字段的值自动递增      |
| FOREIGN KEY    | 外键约束，约束表与表之间的关系        |

### 分类
* 表级约束：可以约束表中任意一个或多个字段。与列定义相互独立，不包含在列定义中；与定义用`,`分隔；必须指出要约束的列的名称。
* 列级约束：包含在列定义中，直接跟在列的其他定义之后，用空格分隔；不必指定列名。
### 列级约束
创建一张学生信息表，包括：学好、姓名、性别、年龄、入学日期、班级、email信息。  
约束：
* 学号是主键，不能为空且唯一，需要自增
* 姓名不能为空
* email唯一
* 性别默认是男性
* 性别只能是男或女
* 年龄只能在18-50岁之间
```mysql
create table t_student(
    sno int(6) primary key auto_increment,
    sname varchar(5) not null,
    sex char(1) default '男' check(sex='男' or sex='女'),
    age int(3) check(age>=18 and age<=50),
    enter_date date,
    class_name varchar(10),
    email varchar(50) unique
);
```
使用`null`或`default`都可完成主键自增。
```mysql
insert into t_student(null, 'hello', '男', '2030-1-2', '6-01', 'hello@gmail.com');
insert into t_student(default, 'world', '男', '2030-1-2', '6-01', 'world@gmail.com');
```
如果sql报错，主键就浪费例，后续插入主键就不连号了。
### 表级约束
```mysql
create table t_student(
    sno int(6) auto_increment,
    sname varchar(5) not null,
    sex char(1) default '男',
    age int(3),
    enter_date date,
    class_name varchar(10),
    email varchar(50),
    constraint pk_stu primary key (sno),
    constraint chk_stu_sex check (sex='男'or sex='女'),
    constraint chk_stu_age check (age>=18 and age<=50),
    constraint uq_stu_email unique (email)
);
```
创建表后，再添加约束。
```mysql
alter table t_student add constraint pk_stu primary key (sno);
alter table t_student modify sno int(6) auto_increment;
alter table t_student add constraint chk_stu_sex check (sex='男'or sex='女');
alter table t_student add constraint chk_stu_age check (age>=18 and age<=50);
alter table t_student add constraint uq_stu_email unique (email);
```
## 外键约束
外键是指表中某个字段的值依赖于另一张表中的某个字段的值，而被依赖的字段必须有主键约束或唯一约束。被依赖的表称为`父表`或`主表`，设置外键约束的表
称为`子表`或`从表`。
```mysql
-- 创建父表
create table t_class(
    cno int(4) primary key auto_increment,
    cname varchar(16) not null,
    room varchar(16)
);
insert into t_class values(null, "java001", "803");
insert into t_class values(null, "java002", "416");
insert into t_class values(null, "bigdata001", "103");
-- 一次添加多条
insert into t_class values (null, "java001", "803"),(null, "java002", "416"),(null, "bigdata001", "103");
-- 创建子表
create table t_student(
    sno int(6) primary key auto_increment,
    sname varchar(5) not null,
    class_no int(4),
    constraint fk_stu_class_no foreign key (class_no) references t_class(cno)
);
insert into t_class values (null, "hello", 1),(null, "world", 1),(null, "hehe", 2);
```
外键约束只有表级约束，没有列级约束。
```mysql
alter table t_student add constraint fk_stu_class_no foreign key (class_no) references t_class(cno);
```
### 外键策略
* no action：不允许操作。
* cascade：级联操作，操作主表的时候影响从表的外键信息。先删除之前的外键约束，在添加新的外键约束。
```mysql
alter table t_student drop foreign key fk_stu_class_no;
alter table t_student add constraint fk_stu_class_no foreign key (class_no) references t_class(cno) on update cascade on delete cascade;
```
* set null：置空操作。
```mysql
alter table t_student drop foreign key fk_stu_class_no;
alter table t_student add constraint fk_stu_class_no foreign key (class_no) references t_class(cno) on update set null on delete set null;
```
### 快速创建表
```mysql
-- 结构和数据和t_student都是一样的
create table t_teacher as select * from t_student;

-- 结构一致，数据不一致
create table t_teacher2 as select * from t_student where 1=2;

-- 部分列，部分数据
create table t_teacher3 as select sno,snmae,age from t_student where sno=2;
```
### 删除
```mysql
-- 清空数据
delete from t_student;

-- 清空数据
truncate table t_student;
```
## DQL
```mysql
-- 部门表
create table dept(
    deptno int(2) not null,
    dname varchar(14),
    loc varchar(13)
);
alter table dept add constraint pk_dept primary key (deptno);

-- 员工表
create table emp(
    empno int(4) primary key,
    ename varchar(10),
    job varchar(9),
    mgr int(4),
    hiredate DATE,
    sal double(7,2),
    comm double(7,2),
    deptno int(2)
);
alter table emp add constraint fk_deptno foreign key (deptno) references dept(deptno);

-- 薪资等级表
create table salgrade(
    grade int primary key,
    losal double(7,2),
    hisal double(7,2)
);

-- 奖金表
create table bonus(
    ename varchar(10),
    job varchar(9),
    sal double(7,2),
    comm double(7,2)
);

-- 起别名
select empno 员工编号, ename 姓名, sal 工资 from emp;
-- 使用as起别名
select empno as 员工编号, ename as 姓名, sal as 工资 from emp;
-- 别名可以加单（双）引号
select empno as '员工编号', ename as "姓名", sal as 工资 from emp;
-- 有特殊字符，必须加单（双）引号
select empno as '员工 编号', ename as "姓 名", sal as 工资 from emp;
-- 做运算
select empno,ename,sal,sal+1000 as 涨薪后,deptno from emp;
-- 和Null做运算，结果都是Null
select empno,ename,sal,comm,sal+comm from emp;

-- 去重
select distinct job from emp;
-- 对后面所有列组合去重
select distinct job,deptno from emp;

-- 排序
select * from emp order by sal; -- 默认按升序排序
-- 显式升序
select * from emp order by sal asc; 
-- 降序
select * from emp order by sal desc;
-- 在工资升序的情况下，部门按降序排列
select * from emp order by sal asc, deptno desc; 
```
### where
过滤条件放在where后面，筛选符合条件的数据。
```mysql
select * from emp wehre deptno=10;
select * from emp wehre deptno>10;
select * from emp wehre deptno>=10;
select * from emp wehre deptno<10;
select * from emp wehre deptno<=10;
select * from emp wehre deptno<>10;
select * from emp where job='CLERK';
select * from emp where job='clerk'; -- 默认情况下不区分大小写
select * from emp where binary job='clerk'; -- binary区分大小写
select * from emp where hiredate < '1981-12-25';

select * from emp where sal > 1500 and sal < 3000 order by sal; -- (1500, 3000)
select * from emp where sal > 1500 && sal < 3000;
select * from emp where sal between 1500 and 3000; -- [1500,3000]

select * from emp where deptno = 10 or deptno = 20;
select * from emp where deptno = 10 || deptno = 20;
select * from emp where deptno in (10, 20);
select * from emp where job in('MANAGER','CLERK','ANALYST');

-- 模糊查询
select * from emp where ename like '%A%'; -- %任意多个字符
select * from emp where ename like '_A%'; -- -任意一个字符

select * from emp where comm is null;
select * from emp where comm is not null;

select * from emp where (job = 'SALESMAN' or job = 'CLERK') and sal >= 1500;
```
### 函数
* 单行函数：对每一条记录进行计算，得到结果返回给用户。
* 多行函数：对一组数据进行计算，只返回一个结果，也称分组函数。
```mysql
select empno,ename,lower(ename),upper(ename),sal from emp;
select max(sal),min(sal),count(sal),sum(sal),avg(sal) from emp; -- 多行函数
```
注意：除了多行函数，都是多行函数。