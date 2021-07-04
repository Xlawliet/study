# 宏观

### 数据库锁

- 粒度小，方便用于集群环境

### 代码锁

- 粒度大，需要封装



# 微观

### 分类（种类）

#### 行锁&表锁

只有明确指定主键（索引），才会执行行锁，否则执行表锁

- 无锁

  ```sql
  select * from user where id =-1 for update;
  -- 因为主键索引都是从1开始，故该行语句的索引不存在，不会调用任何锁
  ```

- 行锁

  ```sql
  select * from user where id =1 for update;
  select * from user where id =1 and name = 'kkk' for update;
  ```

- 表锁

```sql
-- 主键不明确
select * from user where name ='kkk' for update;
select * from user where id <> 3 for update

-- 主键索引不明确时，会调用表锁锁住整张表
```





### 锁算法

#### 行锁算法

Record Lock （普通行锁）

- 键值在条件范围内

- 记录存在

Gap Lock（间隙锁）

- 对于键值不存在条件范围内，叫做“间隙”（GAP），引擎就会对这个间隙加锁，这种机制就是Gap机制。

Next-key Lock（行-间隙）

- 在键值范围条件内，同时键值又不存在条件范围内。

  ```sql
  -- id 只有1-50
  select * from user where id >49 for update；
  -- 此时的查询的是50以及不存在的50以上的记录，故调用了行锁以及间隙锁
  ```

#### 表锁算法

意向锁（表锁）（升级机制）

- 当一个事务带着表锁去访问一个被加了行锁的资源，那么，此时，这个行锁就会升级成意向锁，将表锁住。

- ```sql
  -- 事务a -升级表锁
  select * from user where id = 10 for update;
  -- 事务b -锁表
  select * from user where name = 'kkk%' for update;
  ```

自增锁（表锁）

- 事务插入自增类型的列时，获取自增所

  >如果一个事务正在往表里插入自增记录，其他事务都必须等待



### 实现

#### 共享锁&排它锁

> 行锁和表锁其实是粒度的概念，共享锁和排他锁是他们的具体实现。

##### 共享锁（S）

- 允许一个事务去读一行，阻止其他事务去获取该行的排它锁。
- 一般理解：能读，不能写

##### 排它锁（X）：写锁

- 允许持有排它锁的事务读写数据，阻止其他事务获取该资源的共享锁和排它锁。
- 不能获取任何锁，不代表不能读

**注意点**

- 某个事务获取数据的排它锁，其他事务不能获取该数据的任何锁，并不代表其他事务不能无锁读取该数据。

  - 无锁

    ```sql
    select ... from ...
    ```

  - 共享锁

    ```sql
    select ... lock in share mode
    ```

    > Mysql 8.0以上，for share 代替了 lock in share mode，仍然支持lock in share mode，但是，nowait，skip locked可以实现跳锁，配合自旋锁，可以高效的实现一个等待队列。

  - 排它锁

    ```sql
    update...
    delete...
    insert...
    select ... for update
    ```

#### 乐观锁&悲观锁

> 不管是什么锁都需要加失败重试

- 乐观锁

  一般通过版本号进行更新操作

  ```sql
  update user set name = 'www' where id = 1 and version = 1;
  ```

- 悲观锁