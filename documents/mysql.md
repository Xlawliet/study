

### **增删查改**

增加

```
INSERT INTO 表名 VALUES()
```

删除

```
DELETE FROM 表名 WHERE 条件
```

修改

```
UPDATE 表名 SET 所要修改的数据项（如NEME=XXX） WHERE 条件
```

查询

```
SELECT 所要查询列 FROM 表名
```



### **约束**

**主键约束**

​		能够唯一确定一张表中的一条记录，也就是我们通过给某个字段添加约束就可以使得该字段不重复且不为空。

```sql
create table user(
	id int primary key,
    name varchar(20)
);
--此时id字段中的数据就不得重复且不得为空。

--若是创建表时忘记添加主键则可以通过修改表结构
alter table user add primary key(id);
--来添加主键
--或者修改字段的方式来添加主键约束
alter table user modify id int primary key;
--也可以删除主键
alter table user drop primary key;
```

**联合主键**

```sql
create table user2(
	id int,
	name varchar(20),
	password varchar(20),
	primary key(id,name)
	);
	--此时当向表内插入数据
	insert into user2 values(1,'张三','123');
	insert into user2 values(2,'张三','123');
	--是可以成功的，联合主键只要求成为主键的字段不完全相同即可成功插入,但为主键的任何字段均不得为NULL。
```

**自增约束**

```sql
create table user3(
	id itn primary key auto_increment,
    name varchar(20)
);
--自增约束会对插入的数据中的约束字段自动增加，管控id值等
```

**唯一约束**

约束修饰的字段的值不可以重复

```sql
create table user4(
	id int,
	name varchar(20) unique,
    --unique(name)
    --unique(id,name)
);
--以上方式都是可以的
--或者修改来增加唯一约束
alter table user4 add unique(name);
```

**非空约束**

修饰的字段不能为空NULL

```sql
create table user5(
	id int,
	name varchar(20) not null
);
```

 **默认约束**

当我们插入字段值的时候，如果没有传入值，就会使用默认值

```sql
create table user6(
	id int,
	name varchar(20),
	age int default 10
);
```

**外键约束**

该约束涉及到两个表：父表，子表

主表，副表

```sql
create table classes(
	id int primary key,
	name varchar(20)
);
create table students(
	id int primary key,
	name varchar(20),
	class_id int
    foreign key(class_id) reference classes(id)
    --表名该表中的class_id字段必须来自于classes中的id字段
);

--如插入以下数据后
insert into classes values(1,'一班');
insert into classes values(2,'二班');
insert into classes values(3,'三班');
insert into classes values(4,'四班');

--继续在students表中插入数据
insert into students values(1001,'张三',1);
insert into students values(1002,'李四',2);
insert into students values(1003,'王五',3);
insert into students values(1004,'罗六',4);
--以上插入都是成功的

insert into students values(1005,'杨七',5);
--则是错误的，因为classes表中没有id为5的班级

--主表中没有的数据值，在副表中是不可以使用的
--主表中的记录被副表引用时，是不可以被删除的
```



### 范式

**第一范式**

数据表中的所有字段都是不可分割的原子值

```sql
create table student(
	id int primary key,
	name varchar(20),
	address varchar(30)
);
```

**第二范式**

必须是满足第一范式的前提下，第二范式要求，除主键外的每一列都必须完全依赖于主键。

如果要出现不完全依赖，只可能发生在联合主键的情况下。

```sql
create table myorder(
	product_id int,
	customer_id int,
	product_name varchar(20),
	customer_name varchar(20),
	primary key(product_id,customer_id)
);
--表中可看到product_name只依赖于 product_id
--表中可看到customer_name只依赖于 customer_id
--故该表不符合第二范式，应将该表分为多个表
```

**第三范式**

必须满足第二范式的前提下，除开主键列的其他列之间不能有传递依赖关系。

```sql
create table myorder(
	order_id int primary key,
	product_id int,
	customer_id int,
	customer_phone int
);
--该表中的customer_phone可能与order_id关联，但也与cunstomer_id相关，故不满足第三范式
```



### 查询

建表

```mysql
学生表
student

create table student(
	sno varchar(20) primary key,  #学号
	sname varchar(20) not null,	  #姓名
	ssex varchar(20) not null,    #性别
	sbirthday datetime,			  #出生年月日
	class varchar(20)			  #所在班级
);

课程表
Course
create table course(
	cno varchar(20) primary key, #课程号
    cname varchar(20) not null,  #课程名称
    tno varchar(20) not null,    #教师编号
    foreign key(tno) reference teacher(tno)
);

成绩表
Score
create table score(
	sno varchar(20) primary key,  #学号
    cno varchar(20) not null ,    #课程号
    degree decimal, 			  #成绩
    foreign key(sno) references student(sno),
    foreign key(cno) references student(cno)
);

教师表
Teacher
create table teacher(
	tno varchar(20) primary key,  #教师编号
    tname varchar(20) not null,   #教师名字
    tsex varchar(10) not null,    #教师性别
    tbirthday datetime,  		  #出生年月日
    prof varchar(20),			  #职称
    depart varchar(20) not null	  #部门
);

-- 查看所有表
SHOW TABLES;

-- 添加学生表数据
INSERT INTO student VALUES('101', '曾华', '男', '1977-09-01', '95033');
INSERT INTO student VALUES('102', '匡明', '男', '1975-10-02', '95031');
INSERT INTO student VALUES('103', '王丽', '女', '1976-01-23', '95033');
INSERT INTO student VALUES('104', '李军', '男', '1976-02-20', '95033');
INSERT INTO student VALUES('105', '王芳', '女', '1975-02-10', '95031');
INSERT INTO student VALUES('106', '陆军', '男', '1974-06-03', '95031');
INSERT INTO student VALUES('107', '王尼玛', '男', '1976-02-20', '95033');
INSERT INTO student VALUES('108', '张全蛋', '男', '1975-02-10', '95031');
INSERT INTO student VALUES('109', '赵铁柱', '男', '1974-06-03', '95031');

-- 添加课程表数据
INSERT INTO course VALUES('3-105', '计算机导论', '825');
INSERT INTO course VALUES('3-245', '操作系统', '804');
INSERT INTO course VALUES('6-166', '数字电路', '856');
INSERT INTO course VALUES('9-888', '高等数学', '831');

-- 添加添加成绩表数据
INSERT INTO score VALUES('103', '3-105', '92');
INSERT INTO score VALUES('103', '3-245', '86');
INSERT INTO score VALUES('103', '6-166', '85');
INSERT INTO score VALUES('105', '3-105', '88');
INSERT INTO score VALUES('105', '3-245', '75');
INSERT INTO score VALUES('105', '6-166', '79');
INSERT INTO score VALUES('109', '3-105', '76');
INSERT INTO score VALUES('109', '3-245', '68');
INSERT INTO score VALUES('109', '6-166', '81');

-- 查看表结构
SELECT * FROM course;
SELECT * FROM score;
SELECT * FROM student;
SELECT * FROM teacher;
```

**查询练习**

```mysql
1、查询student表中的所有记录
select * from student;
2、查询student表中的所有记录的sname、ssex和class
select sname,ssex,class from student;
3、查询教师所有的单位即不重复的depart列
select distinct depart from teacher;
4、查询score表中成绩在60到80之间的记录
select * from score where degree between 60 and 80;
5、查询score表中成绩为85，86或88的记录
select * from score where degree==85 OR degree==86 OR degree==88;
select * from score where degree in(85,86,88);
6、查询student表中"95031"班或性别为女的同学记录
select * from student where class='95031' or ssex='女';
7、以class降序查询student表中的所有记录
-- 默认为升序asc ，降序desc  
select * from student order by class desc;
8、以cno升序、degree降序查询score表中的所有记录
select * from score order by cno asc,degree desc;
9、查询"95031"班的学生人数
-- 统计 count
select count(*) from student where class='95031';
10、查询score表中的最高分的学生学号和课程号（子查询或者排序）
select sno,cno from score where degree=(select max(degree) form score);
11、查询每门课的平均成绩
select * from course
-- avg()
select avg(degree) from score where cno='3-105';
-- group by 分组  按cno来分组
select avg(degree) from score group by cno;
12、查询socre表中至少有两名学生学修的并以3开头的课程的平均分数
-- group by后用having添加条件
-- where 行级筛选  having 组级筛选
select cno avg(degree) from score group by cno 
having count(cno)>=2 and cno like '3%';
13、查询分数大于70，小于90的sno列
select sno,degree from score where degree>70 and degree<90
select sno,degree from score where degree between 70 and 90
14、查询所有学生的 sname、cno和degree列(多表查询)
-- 查询姓名
select sname from student;
-- 查询学号和成绩
select cno,degree frome score;
-- 多表
select sname,cno,degree from student,score
where student.sno=score.sno;
15、查询所有学生的sno，cname和degree列。
select cno,cname from course;
select cno,sno,degree from score;

select sno,cname,degree from course,score
where course.cno=score.cno;
16、查询所有学生的sname，cname和degree列

sname->student
cname->course
degree->score

select sname,cname,degree from student,course,score
where student.sno=score.sno and course.cno=scoure.cno;
17、查询"95031"班学生每门课的平均分。
-- 查询学号
select sno from where class='95031';
-- 查询该学号下的成绩
select * from where sno in
(select sno from student where class='95031');
-- 按课程分组求平均分
select cno,avg(degree) 
from score where sno in
(select sno from student where class='95031') 
group by cno;
18、查询选修"3-105"课程的成绩高于109号同学的所有同学的记录
-- 查询 109 号学生的 3-105 课程的成绩
select degree from score where cno='3-105' and sno='109';
-- 查询比 109 号学生的 3-105 课程的成绩高的所有同学的记录
select * from score where
degree>(select degree from score where cno='3-105' and sno='109') and cno='3-105';
19、
20、查询学号为108、101的同学同年出生的所有学生的sno、sname和sbirthday列
select * from student where sno in (108,101);
-- 查询年份
select year(sbirthday) from student where sno in (108,101);

select sno, sname,sbirthday form where year(sbirthday) in (select year(sbirthday) from student where sno in (108,101))
21、
22、查询选修某课程的同学人数多于5人的教师姓名。
-- 得到课程号
select cno from score group by cno having count(*)>5;
-- 得到课程名
-- 得到老师姓名

33、查询至少有2名男生的班号
select class from student where ssex='男' group by class having count(*)>1;

```

**SQL的四种连接查询**



```mysql
-- 两个表
-- person表

create table person(
 id int,
 name varchar(20),
 cardid int);
 
 -- card表
 create table card(
 	id int,
 	name varchar(20)
 	);
 	
 insert into card values(1,'饭卡');
 insert into card values(2,'建行卡');
 insert into card values(3,'农行卡');
 insert into card values(4,'工商卡');
 insert into card values(5,'邮政卡');
 
 insert into person values(1,'张三',1);
 insert into person values(2,'李四',3);
 insert into person values(3,'王五',6);
```

**内连接**

inner join 或者 join

```mysql
-- 当cardid相等的查询出来
select * from person inner join card on person.cardid=card.id;
-- 内联查询，其实就是两张表中的数据，通过某个字段相等，查询出相关记录数据。

1	张三	1	1	饭卡
2	李四	3	3	农行卡
```

**外连接**

1.左连接 left join 或者 left outer join

2.右连接 right join 或者 right outer join

3.完全外连接 full join 或者 full outer join

```mysql
-- left join
select * from person left join card on person.cardid=card.id;

1	张三	1	1	饭卡
2	李四	3	3	农行卡
3	王五	6	Null Null	
-- 左外连接，会把左边表里的所有数据取出来，而右边表中给定数据，如果有相等的，就会显示出来，如果没有就会补NULL

-- right join
select * from person right join card on person.cardid=card.id;
1	  张三	1	1	饭卡
2	  李四	3	3	农行卡
null null null   2	 建行卡
null null null   4	 工商卡
null null null   5	 邮政卡
-- 右外连接，会把右边表里的所有数据取出来，而左边表中给定数据，如果有相等的，就会显示出来，如果没有就会补NULL

-- full join （全外连接）
select * from person full join card on person.cardid=card.id;

-- mysql 不支持full join
-- 可通过 union 来实现
select * from person left join card on person.cardid=card.id
union
select * from person right join card on person.cardid=card.id;
```

![image-20200817155110175](C:\Users\xiong\AppData\Roaming\Typora\typora-user-images\image-20200817155110175.png)



### mysql事务

mysql中，事务其实是一个最小的不可分割的工作单元。事务能够保证一个业务的完整性。比如我们的银行转账：

```mysql
a -> -100

update user set money=money-100 where name='a';

b -> +100

update user set money=money+100 where name='b';

-- 实际的程序中，如果只有一条语句执行成功了，而另外一条没有执行成功？
-- 出现数据的前后不一致。
-- 多条sql语句，可能会有同时完成的要求，要么就同时失败。
```

mysql中如何控制事务？

mysql是默认开启事务的自动提交的。

autocommit = 1 为打开   为0时为关闭

当我们去执行一个sql语句的时候，效果会立即体现出来，且不能回滚。

```mysql
create database bank;

create table user(
	id int primary key,
	name varchar(20),
	money int);
	
insert into user valuse(1,'a',1000);

set autocommit = 0;
-- 此时可以rollback回滚来 撤销刚才的操作
-- 手动提交 commit
-- begin 或者 start transaction 另一种方式帮我们手动开启一个事务

begin / start transaction；
update user set money=money-100 where name='a';
update user set money=money+100 where name='b';
-- 此时可以通过rollback进行回滚
-- 和通过commit提交
```

事务的四大特征：

A 原子性：事务是最小的单位，不可以在分割

C 一致性：事务要求，同一事务中的sql语句，必须保证同时成功或者同时失败。

I 隔离性：事务 1 和 事务 2 之间是具有隔离性的。

D 持久性：事务一旦结束(**commit, rollback**)，就不可以返回。



事务开启：

1. 修改默认提交 set autocommit=0
2. begin；
3. start transaction；

事务手动提交：

​	commit；

事务手动回滚：

​	rollback；



**事务的隔离性:**

1、read uncommitted;   读未提交的

2、read committed;		读已经提交的

3、repeatable read; 	    可以重复读

4、serializable;				 串行化



**1-read uncommitted;**

如果有事务 a 和事务 b，a 事务对数据进行操作，在操作的过程中，事务没有被提交，但是 b 可以看见 a 操作的结果。

如何查看数据库的隔离级别？

`select @@global.tx_isolation；`

如何修改隔离级别？

`set transaction isolation level read uncommitted`

**脏读：**

一个事务读到了另外一个事务没有提交的数据，就叫做脏读



**2-read committed;**

`set transaction isolation level read committed`

会出现**不可重复读**现象

**不可重复读**：是指在数据库访问中，一个事务范围内两个相同的查询却返回了不同数据。

如果事务A 按一定条件搜索， 期间事务B 删除了符合条件的某一条数据，导致事务A 再次读取时数据少了一条。这种情况归为 不可重复读



**3-repeatable read; **    mysql的默认隔离级别

`set transaction isolation level repeatable read`

会出现**幻读**

事务A 按照一定条件进行数据读取， 期间事务B 插入了相同搜索条件的新数据，事务A再次按照原先条件进行读取时，发现了事务B 新插入的数据 称为幻读

影响：会造成一个事务中先产生的锁无法锁住后加入的满足条件的行。



**4-serializable；**   串行化

`set transaction isolation level serializable`

串行化的问题是 性能差

隔离级别越高，性能越差



主从复制：

建立一个和主数据库完全一样的数据库环境，称为从数据库，主数据库一般是准实时的业务数据库。

作用：

1. 在从服务器可以执行查询工作（即常说的读功能），降低主服务器压力；（主库写，从库读，降压）
2. 在从主服务器进行备份，避免备份期间影响主服务器功能；（确保数据安全）
3. 当主服务器出现问题时，可以切换到从服务器。（提升性能）