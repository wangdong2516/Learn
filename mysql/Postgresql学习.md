<center><h1> Postgresql学习

1. 安装postgresql(Mac OS系统)

    ```shell
brew install postgresql
    ```
    
2. 查看postgresql的版本

    ```shell
    postgresql --version 或者是在psql命令行中输入 select version();
    ```

    ![image-20200927204307496](/Users/wangdong/Library/Application Support/typora-user-images/image-20200927204307496.png)

3. 创建数据库

    + 语法：

    ```sql
    create database test3 或者 createdb 数据库名称
    ```

    ![image-20200927203623790](/Users/wangdong/Library/Application Support/typora-user-images/image-20200927203623790.png)

4. 添加备注信息

    + 给字段添加备注

    ```sql
    comment on column 数据库表名.字段名 is '需要备注的信息'
    ```

    示例：

    ```sql
    comment on column person.id is 'id';
    comment on column person.name is '姓名';
    comment on column person.address is '家庭住址';
    comment on column person.name is '年龄';
    ```

    + 给表添加备注

    ```sql
    comment on table 数据库表名 is '需要备注的信息'
    ```

    示例：

    ```sql
    comment on table test is '测试'
    ```

    ![image-20200925144917743](/Users/wangdong/Library/Application Support/typora-user-images/image-20200925144917743.png)

5. 切换数据库

    ```sql
    \c 数据库名称  或者是 psql 数据库名称
    ```

    ![image-20200927204036554](/Users/wangdong/Library/Application Support/typora-user-images/image-20200927204036554.png)

6. 查看当前存在的数据库

    ```sql
    \l 在psql中输入 或者是  psql -l
    ```

    ![image-20200927203753265](/Users/wangdong/Library/Application Support/typora-user-images/image-20200927203753265.png)

7. 删除数据库

    ```sql
    drop database if exists 数据库的名称 或者 dropdb 数据库名称
    ```

    if exists表示存在的时候才删除，不存在的话，该命令不会报错。

8. 创建表结构

    ```sql
    create table 表名(
    	字段名 字段数据类型,
      字段名 字段数据类型,
      字段名 字段数据类型,
    )
    ```

    示例:

    ```sql
    create table person(
        id smallint,
        name varchar(30),
        address text,
        age int
    );
    ```

9. 查看数据库中的所有表

    ```sql
    \dt
    ```

    ![image-20200925144446970](/Users/wangdong/Library/Application Support/typora-user-images/image-20200925144446970.png)

10. 查看数据库中的表结构(表的详细信息)

    ```sql
    \d 表名
    ```

    ![image-20200925144417454](/Users/wangdong/Library/Application Support/typora-user-images/image-20200925144417454.png)

11. 查看更加详细的表结构，包括注释信息

     ```sql
     \d+ 表名
     ```

     ![image-20200925145332794](/Users/wangdong/Library/Application Support/typora-user-images/image-20200925145332794.png)

12. 修改数据库的表名

    ```psql
    alter table 表名 rename 新的表名
    ```

    示例:

    ![image-20200927210730829](/Users/wangdong/Library/Application Support/typora-user-images/image-20200927210730829.png)

13. 修改表的字段类型

    ```sql
     ALTER TABLE tablename alter column type newtype;
    ```

14. 增加字段

     ```sql
     ALTER TABLE tablename ADD column type;
     ```

15. 根据sql文件创建表

     \i命令可以从指定的文件中读取命令，命令的格式如下：

     ```psql
     \i sql文件名(如果文件在当前目录下，可以直接使用文件名，如果不在当前的目录下，需要使用绝对路径的形式)
     ```

     ![image-20200927212042534](/Users/wangdong/Library/Application Support/typora-user-images/image-20200927212042534.png)

     db.sql文件的内容如下

     ```sql
     create table wangdong(name varchar(32));
     ```

14. 删除表

    ```psql
    drop table tablename
    ```

> postgresql的数据类型

1. 数字类型
    + `smallint`:2个字节，存储的范围是: -32768~32767(对比Mysql相当于smallint)
    
    + `integer`:4个字节，存储的范围是:-2147483648~2147483647（对比Mysql相当于int）
    
    + `bigint`:8个字节，存储的范围是-9223372036854775808～9223372036854775807（对比Mysql相当于bigint）
    
    + `decimal`:可变长decimal类型，精确，可以存储小数点前131072位，小数点后16383位（对比Mysql相当于Decimal）
    
    + `numeric`：可变长，精确，可以存储小数点前131072位，小数点后16383位，可以存储人意精度的数字，但是`numeric类型的算术运算比整数类型和浮点数类型的计算要慢很多。`
    
        precision指的是整个数字中有效位的总数，也就是小数点两边的位数。
    
        scale指的是小数部分的数字位数，也就是小数点右边的部分。
    
        numeric列的最大精度和最大的刻度都是可以配置的
    
        ```sql
        NUMERIC(precision, scale)
        ```
    
        精度必须是整数，刻度可以是0或者是正数
    
        创建一个NUMERIC列但是不指定精度和刻度的话，该列可以存储任何精度或者是刻度的数值。并且值的范围最多可以到实现的精度的上限。
    
        如果值的比要存储的值长，将会尝试四舍五入到指定的位数。
    
    + `real`：4个字节，可变精度，不精确，6位十进制数字的精度。(对比Mysql相当于float)
    
        浮点类型的几个特殊的值:
    
        Infinity:正无穷大
    
        -Infinity: 负无穷大
    
        NaN：不是一个数字
    
        real类型支持float(1)到float(24)的精度
    
    + `double precision`：8个字节，不精确，15位十进制数字的精度
    
    + `smallserial`：2个字节，自增的小范围证书，存储的范围是1～32767(对比mysql来说相当于smallint unsigned)
    
    + `serial`：4个字节，自增的整数，存储的范围是1～2147483647(对比mysql来说相当于int unsigned),一般是``用做自增的主键``来使用的
    
    + `bigserial`：8个字节，自增的大范围证书，存储的范围是1～9223372036854775807(对比mysql来说相当于biting unsigned)
    
2. 字符串类型:
    + character、varying(), varchar():变长，有最大的长度限制。
    + character(), char()：固定的长度，不够的补空白。
    + text，变长的，没有最大的长度限制。
    
3. 时间类型

    + `timestamp`:8个字节，包括日期和时间，可以设置为有时区和无时区的。
    + `date`:4个字节，只包括日期
    + `time`:8个字节，只包括时间,可以设置为有时区和无时区的。
    + `interval`:16个字节，时间间隔

4. 货币类型

    + `money`类型：8个字节，存储带有固定小数精度(默认是四舍五入保留两位小数)的货币的金额，numeric、int、bigint类型的值可以转换为money类型，不建议使用浮点数来存储货币，因为会存在精度的问题。存储的范围是-92233720368547758.08 到 +92233720368547758.07
    + money类型可以存储的小数的精度由数据库的Ic_monetrary设置决定，并且是和区域相关的。在恢复一个转储到一个新数据库中之前，应确保新数据库的`lc_monetary`设置和被转储数据库的相同或者具有等效值。

    示例

    ![image-20200928104210927](/Users/wangdong/Library/Application Support/typora-user-images/image-20200928104210927.png)

5. 布尔类型

6. 枚举类型

7. 几何类型

8. 网络地址类型

9. 位串类型

10. 文本搜索类型

11. UUID类型

12. XML类型

13. JSON类型

14. 数组类型

15. 复合类型

16. 范围类型

17. 对象标识符类型

18. 伪类型

> 数据的增、删、改、查

`增`:

+ 添加一行数据:

    ```sql
    INSERT INTO `tablename` VALUES('value1','value2', 'value3');
    需要注意的就是，这样的写法要求值的顺序和列的顺序是一致的，否则会出现不匹配的一些错误。
    ```

    ```sql
    INSERT INTO `tablename` (column1, column2, column3) VALUES (value1, value2, value3);
    这种写法可以明确的指定需要插入的字段以及字段的顺序，避免出现错误，是推荐使用的方式。
    ```

`删`：

+ 删除数据，一般需要加条件进行限制

    ```sql
    DELETE FROM tablename WHERE 条件;
    ```

    示例

    ```sql
    DELETE FROM weather WHERE city = 'liujiabao'
    ```

`改`:

+ 修改数据，一般需要加条件进行限制。

    ```sql
    UPDATE tablename SET column1 = value1, column2 = value2 WHERE 条件;
    ```

    示例

    ```sql
    UPDATE weather set temp_hi = 99 WHERE city = 'San Francisco' AND temp_lo = 46;
    ```

`查`:

+ 查询所有的数据

    ```sql
    SELECT * FROM tablename;
    ```

+ 查询某个列(某些列)的数据

    ```sql
    SELECT column1, column2 FROM tablename
    ```

+ 运行表达式

    ```sql
    SELECT column / 2 FROM tablename;
    ```

+ 重命名查询的结果

    ```sql
    SELECT column as `new_name` FROM tablename;
    ```

+ 查询某些满足条件的行，where子句中包含一个布尔表达式，只有那些布尔表达式为真的行才会被返回。

    ```sql
    SELECT * FROM tablename where 条件
    ```

+ 排序

    ```sql
    SELECT * FROM tablename ORDER BY column;
    ```

+ 去重

    ```sql
    SELECT DISTINCT column FROM tablename;
    ```

+ 限制返回的结果集

    ```sql
    SELECT * FROM tablename LIMIT 3 OFFSET 10;
    ```

> 添加表的约束

1.  `not null`：不允许为空
2. `unique`:表的字段唯一
3. `check`:字段设置的条件，可以自定义check函数
4. `default`:字段的默认值
5. `primary key(not null, unique)`:主键，非空且唯一

```sql
create table posts(
	id serial primary key,
  title varchar(255) not null,
  content text check(length(content) > 8),
  is_draft boolean default TRUE,
  is_del boolean dafault FALSE,
  create_date timestamp defalut 'now'
)
```

> 表之间的连接

1. `自连接`：自己和自己进行连接，一般用于省市区表的查询。

    示例

    ```sql
    SELECT * FROM weather AS w1 JOIN weather AS w2 on w1.city = w2.city;
    ```

2. `左外链接`:确保左边的表的行在输出中至少出现一次。右边的表的行只有能在找到匹配的行的时候才会被输出，没有匹配的行将会补null。

    示例

    ```sql
    SELECT * FROM weather LEFT OUTER JOIN cities ON weather.city = cities.name;
    ```

    效果等同于使用LEFT JOIN 的形式

    ```sql
    SELECT * FROM weather LEFT JOIN cities ON weather.city = cities.name;
    ```

3. `内连接`:只有全部匹配的行才会被输出，没有匹配的行将会被忽略

    ```sql
    SELECT * FROM weather INNER JOIN cities ON weather.city = cities.name;
    ```

> 聚合函数

1. `最大值`: max

    ```sql
    SELECT max(temp_lo) FROM weather;
    ```

2. `最小值`:min

    ```sql
    SELECT min(temp_lo) FROM weather;
    ```

3. `平均值`: avg

    ```sql
    SELECT avg(temp_lo) FROM weather;
    ```

4. `求和`:sum

    ```sql
    SELECT sum(temp_lo) FROM weather;
    ```

5. `计数`:count

    ```sql
    SELECT count(*) FROM weather;
    ```

> WHERE 子句和HAVING子句之间的区别

1. `WHERE`在分组和聚合之间选取输入的行，控制那些行进入聚合运算。不能使用聚合函数。
2. `HAVING`在分组和聚合之后选择行，可以使用聚合函数。

>模糊查询

1. `LIKE `:

    匹配单个字符

    ```sql
    SELECT * FROM tablename WHERE column LIKE "qiao_"
    ```

    匹配多个字符

    ```sql
    SELECT * FROM tablename WHERE column LIKE "qiao%"
    ```

> 内置函数一览

1. `length`:显示字符串的长度

    ```sql
    select length(city) FROM weather; 
    ```

2. `concat`:连接字符串

    ```sql
    select temp_lo, concat(city, temp_hi) from weather;
    ```

    ![image-20200928172831895](/Users/wangdong/Library/Application Support/typora-user-images/image-20200928172831895.png)

3. `alias`:别名

    ```sql
    SELECT city as address FROM weather;
    ```

4. `substring`: 切割字符串 substring(column, start，end)

    ```sql
    SELECT substring(city, 1, 3) FROM weather;
    ```

5. `random`:随机数，从0到1的任何一个随机数

    ```sql
    SELECT * FROM weather order by random();
    ```

> 创建索引和删除索引

1.  新增索引

    ```sql
    create index indexname on tablename(column);
    ```

    ![image-20200928175505100](/Users/wangdong/Library/Application Support/typora-user-images/image-20200928175505100.png)

2. 删除索引

    ```sql
    drop index indexname;
    ```

    ![image-20200928175618196](/Users/wangdong/Library/Application Support/typora-user-images/image-20200928175618196.png)

> 使用视图

1. `视图`:是从一个或者多个表中导出的对象，视图和表不同，视图是一个虚表，视图所对应的数据不进行实际的存储，数据库中只存储视图的定义，在对视图进行操作的时候，系统会根据视图中定义的操作，查询对应的数据库表。

2. 创建视图

    ```sql
    create view view_name as select 语句
    ```

3. 使用视图

    ```sql
    select * from viewname;
    ```

4. 删除视图

    ```sql
    drop view viewname;
    ```

> 使用事务

1.  事务的四大特性:

    + `原子性`
    + `隔离性`
    + `一致性`
    + `持久性`

2. 开始事务

    ```sql
    begin;
    ```

3. 提交事务

    ```sql
    commit;
    ```

4. 回滚事务

    ```sql
    rollback;
    
    ```

示例：

```sql
BEGIN;
INSERT INTO cities2 VALUES('Berkeley', '(-192.168, 1.1)');
INSERT INTO weather2 VALUES('Berkeley', 45, 53, 0.0, '1994-11-28');
UPDATE weather2 set temp_hi = 100 where city = 'Berkeley';
SELECT * FROM weather2;
COMMIT;
```

`Postgresql`默认将每一个sql语句都作为一个事务来执行，事务也叫做事务块。

`也可以使用保存点来实现更细粒度的事务的控制，保存点允许我们有选择性的放弃事务的一部分而提交剩下的一部分，保存点的定义使用的是SAVEPOINT关键字，我们可以在必要的时候回滚到该保存点，该事务中位于保存点和回滚点之间的数据都将会被放弃，但是早于保存点执行的修改将会被保存下来。`

`在回滚到保存点之后，保存点的定义依然是存在的，因此我们可以多次进行回滚，如果确定不需要回到特定的保存点的话,可以释放资源，不管是释放保存点还是回滚保存点都会释放定义在该保存点之后的其他的保存点。`

> 外键

1. `需要确保数据的引用完整性的情况下，可以使用外键来进行约束。`

2. 外键的使用

    ![image-20200928185410664](/Users/wangdong/Library/Application Support/typora-user-images/image-20200928185410664.png)

    注意事项：

    1. 外键引用的必须是具有唯一约束的列，不然会报错。

    2. 外键引用的表必须在之前已经被创建好。

    如果在cities2中不存在对应的记录之前，在weather2中插入数据，就会发生下面的情况

    ![image-20200928185616578](/Users/wangdong/Library/Application Support/typora-user-images/image-20200928185616578.png)

    ![image-20200928190023602](/Users/wangdong/Library/Application Support/typora-user-images/image-20200928190023602.png)

    正确的使用外键将会提高数据库应用的质量，建议掌握。

> 窗口函数

1. 一个窗口函数在一系列于当前行有某些关联的表上执行的一种计算，和一个聚合函数完成的作用有相似的地方，但是窗口函数不会使得多行被聚集成一个单独的行。
2. 一个窗口函数的调用总是包含在一个直接跟在窗口函数名以及参数之后的OVER子句，OVER子句决定究竟查询中的那些行被分离出来由窗口函数处理， PARTITION BY子句制定了将具有相同 PARTITION BY表达值只的行分到一个组或者是一个分区中。对于每一行，窗口函数都会在当前行的同一分区进行计算。

> 继承

1. 继承是面向对象数据库的概念，展示了数据库设计的新的可能性。

2. 继承的使用

    ```sql
    CREATE TABLE cities (
      name       text,
      population real,
      altitude   int     -- (in ft)
    );
    
    CREATE TABLE capitals (
      state      char(2)
    ) INHERITS (cities);
    ```

    ![image-20200928192553457](/Users/wangdong/Library/Application Support/typora-user-images/image-20200928192553457.png)

    在这种情况之下，capitals表从cities表中继承了name字段以及location字段，并且继承了它们对应的字段类型。

3. 涉及继承的查询

    ```sql
    1. 在继承的所有层次结构中进行查询
    SELECT name, altitude
      FROM cities
      WHERE altitude > 500;
    2. 只在当前的表中进行查询，但是不涉及继承层级中该表下面的其他表
    SELECT name, altitude
        FROM ONLY cities
        WHERE altitude > 500;    
        其中cities之前的ONLY用于指示查询只在cities表上进行而不会涉及到继承层次中位于cities之下的其他表
    ```

4. 注意事项

    尽管继承是很有用并且将来是会被发展的，但是现在没有和唯一约束或者是外键进行集成，导致了可用性现在很低。

> SQL

1. 关键字和不被引号修饰的标识符是大小写不敏感的。
2. 被双引号修饰的标识符叫做受限标识符，一个受限标识符总是一个标识符而不是一个关键字。

> 约束

1. `检查约束`： 一个检查约束是最普通的约束类型，它允许我们指定一个特定列中的值，必须满足一个布尔表达式。

    示例

    ```sql
    CREATE TABLE products(
     product_no integer,
      name text,
      price numeric CHECK(price>0)
    )
    ```

    默认值和约束之间的顺序是没有关系的，都是在数据类型的后面，一个检查约束由关键字CHECK以及在后面括号中包含的表达式一起组成的，检查约束表达式涉及到被约束的列，否则约束将会失去意义。

    `给约束一个名称`

    ```sql
    CREATE TABLE porductions(
      product_no integer,
      name text,
      price numeric CONSTRAINT positive_price CHECK(price > 0)
    )
    ```

    要指定一个命名约束，请在约束名称标识符的前面使用关键字OCNSTRAINT

2. 非空约束`NOT NULL`

    ```sql
    CREATE TABLE products (
        product_no integer NOT NULL,
        name text NOT NULL,
        price numeric
    );
    ```

3. 唯一约束`UNIQUE`

    ```sql
    CREATE TABLE products (
        product_no integer UNIQUE,
        name text,
        price numeric
    );
    ```

4. 主键约束`NOT NULL UNIQUE`

    ```sql
    CREATE TABLE products (
        product_no integer PRIMARY KEY,
        name text,
        price numeric
    );
    ```

5. 外键约束`REFERENCES`

    ```sql
    CREATE TABLE orders (
        order_id integer PRIMARY KEY,
        product_no integer REFERENCES products (product_no),
        quantity integer
    );
    ```

6. 排他约束`EXCLUDE`,增加一个排他约束将在约束声明所指定的类型上自动的创建索引。

    ```sql
    CREATE TABLE circles (
        c circle,
        EXCLUDE USING gist (c WITH &&)
    );
    ```

    