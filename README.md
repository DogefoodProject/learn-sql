# learn-sql
## 非外键约束
### 完整性约束
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
### 例
创建一张学生信息表，包括：学好、姓名、性别、年龄、入学日期、班级、email信息。  
约束：  
* 学号是主键，不能为空且唯一，需要自增
* 姓名不能为空
* email唯一
* 性别默认是男性
* 性别只能是男或女
* 年龄只能在18-50岁之间
```sql
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
```sql
insert into t_student(null, 'hello', '男', '2030-1-2', '6-01', 'hello@gmail.com');
insert into t_student(default, 'world', '男', '2030-1-2', '6-01', 'world@gmail.com');
```
如果sql报错，主键就浪费例，后续插入主键就不连号了。
### 约束分类
* 表级约束：可以约束表中任意一个或多个字段。与列定义相互独立，不包含在列定义中；与定义用`,`分隔；必须指出要约束的列的名称。
* 列级约束：包含在列定义中，直接跟在列的其他定义之后，用空格分隔；不必指定列名。
```sql
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
