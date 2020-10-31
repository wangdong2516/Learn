# <center>mysql</center>

1. 创建数据库

    + sql语句结构：

        ```sql
        create database 数据库名 charset=utf8;
        ```

        

    + 例子: 

        ```sql
        create database Blog charset=utf8;
        ```

2. 创建数据库表结构

    + 语法格式：

        ```sql
        create table 数据库表名 （
        	字段 类型 约束
        	字段 类型 约束
        	）
        ```

        

    + 例子

        ```sql
        
        create table users
        (
            id           int auto_increment
                primary key,
            password     varchar(128) not null,
            last_login   datetime(6)  null,
            is_superuser tinyint(1)   not null,
            username     varchar(150) not null,
            first_name   varchar(30)  not null,
            last_name    varchar(150) not null,
            email        varchar(254) not null,
            is_staff     tinyint(1)   not null,
            is_active    tinyint(1)   not null,
            date_joined  datetime(6)  not null,
            mobile       varchar(11)  not null,
            email_active tinyint(1)   not null,
            constraint mobile
                unique (mobile),
            constraint username
                unique (username)
        );
        ```

3. 修改数据库字段

    + 语法格式：

        

