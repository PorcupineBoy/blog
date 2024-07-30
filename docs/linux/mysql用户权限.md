## 1. MySQL 权限介绍

```
mysql中存在4个控制权限的表，`分别为user表，db表，tables_priv表，columns_priv表`，我当前的版本mysql 5.7.22 。

mysql权限表的验证过程为：

1. 先从user表中的Host,User,Password这3个字段中判断连接的ip、用户名、密码是否存在，存在则通过验证。
2. 通过身份认证后，进行权限分配，按照user，db，tables_priv，columns_priv的顺序进行验证。即先检查全局权限表user，如果user中对应的权限为Y，则此用户对所有数据库的权限都为Y，将不再检查db, tables_priv,columns_priv；如果为N，则到db表中检查此用户对应的具体数据库，并得到db中为Y的权限；如果db中为N，则检查tables_priv中此数据库对应的具体表，取得表中的权限Y，以此类推。
```



## 1.1 MySQL 权限级别

分为： 
全局性的管理权限： 作用于整个MySQL实例级别 
数据库级别的权限： 作用于某个指定的数据库上或者所有的数据库上 
数据库对象级别的权限：作用于指定的数据库对象上（表、视图等）或者所有的数据库对象上

 

权限存储在mysql库的user, db, tables_priv, columns_priv, and procs_priv这几个系统表中，待MySQL实例启动后就加载到内存中

查看mysql 有哪些用户：

```
mysql> ``select` `user,host from mysql.user;
```

　　

来看`root 用户在权限系统表中的数据`：

```
mysql> use mysql;``mysql> ``select` `* from user where user=``'root'` `and host=``'localhost'``\G; ``#所有权限都是Y ，就是什么权限都有``mysql> ``select` `* from db where user=``'root'` `and host=``'localhost'``\G; ``# 没有此条记录``mysql> ``select` `* from tables_priv where user=``'root'` `and host=``'localhost'``; ``# 没有此条记录` `mysql> ``select` `* from columns_priv where user=``'root'` `and host=``'localhost'``; ``# 没有此条记录` `mysql> ``select` `* from procs_priv where user=``'root'` `and host=``'localhost'``; ``# 没有此条记录
```

　　

上面说过：权限的验证过程

 

查看root@’localhost’用户的权限

```
mysql> show grants ``for` `root@localhost;``+---------------------------------------------------------------------+``| Grants ``for` `root@localhost                      |``+---------------------------------------------------------------------+``| GRANT ALL PRIVILEGES ON *.* TO ``'root'``@``'localhost'` `WITH GRANT OPTION |``| GRANT PROXY ON ``''``@``''` `TO ``'root'``@``'localhost'` `WITH GRANT OPTION    |``+---------------------------------------------------------------------+``2 rows ``in` `set` `(0.00 sec)
```

　　

## 2. MySQL 权限详解

- `All/All Privileges权限代表全局或者全数据库对象级别的所有权限`
- Alter权限代表允许修改表结构的权限，但必须要求有create和insert权限配合。如果是rename表名，则要求有alter和drop原表， create和insert新表的权限
- Alter routine权限代表允许修改或者删除存储过程、函数的权限
- Create权限代表允许创建新的数据库和表的权限
- Create routine权限代表允许创建存储过程、函数的权限
- Create tablespace权限代表允许创建、修改、删除表空间和日志组的权限
- Create temporary tables权限代表允许创建临时表的权限
- Create user权限代表允许创建、修改、删除、重命名user的权限
- Create view权限代表允许创建视图的权限
- Delete权限代表允许删除行数据的权限
- Drop权限代表允许删除数据库、表、视图的权限，包括truncate table命令
- Event权限代表允许查询，创建，修改，删除MySQL事件
- Execute权限代表允许执行存储过程和函数的权限
- File权限代表允许在MySQL可以访问的目录进行读写磁盘文件操作，可使用的命令包括load data infile,select … into outfile,load file()函数
- Grant option权限代表是否允许此用户授权或者收回给其他用户你给予的权限,重新付给管理员的时候需要加上这个权限
- Index权限代表是否允许创建和删除索引
- Insert权限代表是否允许在表里插入数据，同时在执行analyze table,optimize table,repair table语句的时候也需要insert权限
- Lock权限代表允许对拥有select权限的表进行锁定，以防止其他链接对此表的读或写
- Process权限代表允许查看MySQL中的进程信息，比如执行show processlist, mysqladmin processlist, show engine等命令
- Reference权限是在5.7.6版本之后引入，代表是否允许创建外键
- Reload权限代表允许执行flush命令，指明重新加载权限表到系统内存中，refresh命令代表关闭和重新开启日志文件并刷新所有的表
- Replication client权限代表允许执行show master status,show slave status,show binary logs命令
- Replication slave权限代表允许slave主机通过此用户连接master以便建立主从复制关系
- Select权限代表允许从表中查看数据，某些不查询表数据的select执行则不需要此权限，如Select 1+1， Select PI()+2；而且select权限在执行update/delete语句中含有where条件的情况下也是需要的
- Show databases权限代表通过执行show databases命令查看所有的数据库名
- Show view权限代表通过执行show create view命令查看视图创建的语句
- Shutdown权限代表允许关闭数据库实例，执行语句包括mysqladmin shutdown
- Super权限代表允许执行一系列数据库管理命令，包括kill强制关闭某个连接命令， change master to创建复制关系命令，以及create/alter/drop server等命令
- Trigger权限代表允许创建，删除，执行，显示触发器的权限
- Update权限代表允许修改表中的数据的权限
- Usage权限是创建一个用户之后的默认权限，其本身代表连接登录权限

 

## 2.1 系统权限表

`User表`：存放用户账户信息以及全局级别（所有数据库）权限，决定了来自哪些主机的哪些用户可以访问数据库实例，如果`有全局权限则意味着对所有数据库都有此权限` 
Db表：存放`数据库级别`的权限，决定了来自哪些主机的哪些用户可以访问此数据库 
Tables_priv表：`存放表级别的权限`，决定了来自哪些主机的哪些用户可以访问数据库的这个表 
Columns_priv表：`存放列级别的权限`，决定了来自哪些主机的哪些用户可以访问数据库表的这个字段 
Procs_priv表：`存放存储过程和函数`级别的权限

最重要的还是user表

 

## 2.1.1 User和 db 权限表的结构

| 表名           | `user`                   | `db`                    |
| -------------- | ------------------------ | ----------------------- |
| **范围列**     | `Host`                   | `Host`                  |
|                | `User`                   | `Db`                    |
|                |                          | `User`                  |
| **权限列**     | `Select_priv`            | `Select_priv`           |
|                | `Insert_priv`            | `Insert_priv`           |
|                | `Update_priv`            | `Update_priv`           |
|                | `Delete_priv`            | `Delete_priv`           |
|                | `Index_priv`             | `Index_priv`            |
|                | `Alter_priv`             | `Alter_priv`            |
|                | `Create_priv`            | `Create_priv`           |
|                | `Drop_priv`              | `Drop_priv`             |
|                | `Grant_priv`             | `Grant_priv`            |
|                | `Create_view_priv`       | `Create_view_priv`      |
|                | `Show_view_priv`         | `Show_view_priv`        |
|                | `Create_routine_priv`    | `Create_routine_priv`   |
|                | `Alter_routine_priv`     | `Alter_routine_priv`    |
|                | `Execute_priv`           | `Execute_priv`          |
|                | `Trigger_priv`           | `Trigger_priv`          |
|                | `Event_priv`             | `Event_priv`            |
|                | `Create_tmp_table_priv`  | `Create_tmp_table_priv` |
|                | `Lock_tables_priv`       | `Lock_tables_priv`      |
|                | `References_priv`        | `References_priv`       |
|                | `Reload_priv`            |                         |
|                | `Shutdown_priv`          |                         |
|                | `Process_priv`           |                         |
|                | `File_priv`              |                         |
|                | `Show_db_priv`           |                         |
|                | `Super_priv`             |                         |
|                | `Repl_slave_priv`        |                         |
|                | `Repl_client_priv`       |                         |
|                | `Create_user_priv`       |                         |
|                | `Create_tablespace_priv` |                         |
| **安全专栏**   | `ssl_type`               |                         |
|                | `ssl_cipher`             |                         |
|                | `x509_issuer`            |                         |
|                | `x509_subject`           |                         |
|                | `plugin`                 |                         |
|                | `authentication_string`  |                         |
|                | `password_expired`       |                         |
|                | `password_last_changed`  |                         |
|                | `password_lifetime`      |                         |
|                | `account_locked`         |                         |
| **资源控制列** | `max_questions`          |                         |
|                | `max_updates`            |                         |
|                | `max_connections`        |                         |
|                | `max_user_connections`   |                         |

User权限表结构中的特殊字段：

- Plugin,authentication_string字段存放用户认证信息
- Password_expired设置成’Y’则表明允许DBA将此用户的密码设置成过期而且过期后要求用户的使用者重置密码（alter user/set password重置密码）
- Password_last_changed作为一个时间戳字段代表密码上次修改时间，执行create user/alter user/set password/grant等命令创建用户或修改用户密码时`此数值自动更新`
- Password_lifetime代表从password_last_changed时间开始此密码过期的天数
- `Account_locked代表此用户被锁住，无法使用`

在mysql 5.7 以前在user表有password 这个字段。

## 2.1.2 Tables_priv和columns_priv权限表结构

| 表名       | `tables_priv` | `columns_priv` |
| ---------- | ------------- | -------------- |
| **范围列** | `Host`        | `Host`         |
|            | `Db`          | `Db`           |
|            | `User`        | `User`         |
|            | `Table_name`  | `Table_name`   |
|            |               | `Column_name`  |
| **权限列** | `Table_priv`  | `Column_priv`  |
|            | `Column_priv` |                |
| **其他列** | `Timestamp`   | `Timestamp`    |
|            | `Grantor`     |                |

**Tables_priv和columns_priv权限值**

| Table Name     | Column Name   | Possible Set Elements                                        |
| -------------- | ------------- | ------------------------------------------------------------ |
| `tables_priv`  | `Table_priv`  | `'Select', 'Insert', 'Update', 'Delete', 'Create', 'Drop', 'Grant', 'References', 'Index', 'Alter', 'Create View', 'Show view', 'Trigger'` |
| `tables_priv`  | `Column_priv` | `'Select', 'Insert', 'Update', 'References'`                 |
| `columns_priv` | `Column_priv` | `'Select', 'Insert', 'Update', 'References'`                 |
| `procs_priv`   | `Proc_priv`   | `'Execute', 'Alter Routine', 'Grant'`                        |

## 2.1.3 procs_priv权限表结构

| Table Name            | `procs_priv`   |
| --------------------- | -------------- |
| **Scope columns**     | `Host`         |
|                       | `Db`           |
|                       | `User`         |
|                       | `Routine_name` |
|                       | `Routine_type` |
| **Privilege columns** | `Proc_priv`    |
| **Other columns**     | `Timestamp`    |
|                       | `Grantor`      |

- Routine_type是枚举类型，代表是存储过程还是函数
- Timestamp和grantor两个字段暂时没用

系统权限表字段长度限制表

| Column Name            | Maximum Permitted Characters |
| ---------------------- | ---------------------------- |
| `Host`, `Proxied_host` | 60                           |
| `User`, `Proxied_user` | 32                           |
| `Password`             | 41                           |
| `Db`                   | 64                           |
| `Table_name`           | 64                           |
| `Column_name`          | 64                           |
| `Routine_name`         | 64                           |

**权限认证中的大小写敏感问题**

- 字段user,password,authencation_string,db,table_name大小写敏感
- 字段host,column_name,routine_name大小写不敏感

 

## 2.2 用户权限信息管理

## 2.2.1 查看用户权限信息

查看MYSQL有哪些用户

```
mysql> ``select` `user,host from mysql.user;
```

　　

查看已经授权给用户的权限信息 
例如root

```
mysql> show grants ``for` `root@``'localhost'``;
```

　　

查看用户的其他非授权信息

```
mysql> show create user root@``'localhost'``;
```

　　

![img](https://img2018.cnblogs.com/blog/1033265/201901/1033265-20190118155940698-1700719440.png)

 

 

## 2.2.2 用户组成

MySQL的授权用户由两部分组成： `用户名和登录主机名`

- - 表达用户的语法为’user_name’@’host_name’
  - 单引号不是必须，但如果其中`包含特殊字符则是必须的`
  - ”@‘localhost’代表匿名登录的用户
  - Host_name可以使主机名或者ipv4/ipv6的地址。 Localhost代表本机， 127.0.0.1代表ipv4本机地址， ::1代表ipv6的本机地址
  - Host_name字段允许使用`%和_`两个匹配字符，比如’%’代表所有主机， ’%.mysql.com’代表 
    来自mysql.com这个域名下的所有主机， ‘192.168.1.%’代表所有来自192.168.1网段的主机

 

 

| `User值` | `Host` 值                      | 允许的连接                                                   |
| -------- | ------------------------------ | ------------------------------------------------------------ |
| `'fred'` | `'h1.example.net'`             | `fred`，连接 `h1.example.net`                                |
| `''`     | `'h1.example.net'`             | 任何用户，从中连接 `h1.example.net`                          |
| `'fred'` | `'%'`                          | `fred`，从任何主机连接                                       |
| `''`     | `'%'`                          | 任何用户，从任何主机连接                                     |
| `'fred'` | `'%.example.net'`              | `fred`，从`example.net`域中的任何主机连接                    |
| `'fred'` | `'x.example.%'`                | `fred`，从连接 `x.example.net`，`x.example.com`， `x.example.edu`，等; 这可能没用 |
| `'fred'` | `'198.51.100.177'`             | `fred`，从主机与IP地址连接 `198.51.100.177`                  |
| `'fred'` | `'198.51.100.%'`               | `fred`，从`198.51.100`C类子网中的任何主机连接                |
| `'fred'` | `'198.51.100.0/255.255.255.0'` | 与前面的示例相同                                             |

## 2.2.3 修改用户权限

- 执行Grant,revoke,set password,rename user命令修改权限之后， MySQL会自动将修改后的权限信息同步加载到系统内存中
- 如果执行insert/update/delete操作上述的系统权限表之后，则必须再执行刷新权限命令才能同步到系统内存中，刷新权限命令包括： `flush privileges`/mysqladmin flush-privileges / mysqladmin reload
- 如果是修改tables和columns级别的权限，则客户端的下次操作新权限就会生效
- 如果是修改database级别的权限，则新权限在客户端执行use database命令后生效
- 如果是修改global级别的权限，则需要重新创建连接新权限才能生效
- 如果是修改global级别的权限，则需要重新创建连接新权限才能生效 (例如修改密码)

 

## 2.2.4 创建 mysql 用户

有两种方式创建MySQL授权用户

1. `执行create user/grant命令`（推荐方式）
2. 通过insert语句直接操作MySQL系统权限表

```
# 创建finley 这只是创建用户并没有权限``mysql> CREATE USER ``'finley'``@``'localhost'` `IDENTIFIED BY ``'some_pass'``;``# 把finley 变成管理员用户``mysql> GRANT ALL PRIVILEGES ON *.* TO ``'finley'``@``'localhost'` `WITH``GRANT OPTION;``#创建用户并赋予RELOAD,PROCESS权限 ，在所有的库和表上``mysql> GRANT RELOAD,PROCESS ON *.* TO ``'admin'``@``'localhost'` `identified by ``'123456'``;` `# 创建keme用户，在test库，temp表， 上的id列只有select 权限``mysql> grant ``select``(``id``) on ``test``.temp to keme@``'localhost'` `identified by ``'123456'``;
```

　　

## 2.2.4 回收 mysql 权限

通过revoke命令收回用户权限,回收的时候看一下这个用户有哪些权限然后回收 
我对admin 用户做测试

```
mysql> show grants ``for` `admin@``'localhost'``;``mysql> ``select` `user,host from mysql.user;``mysql> revoke PROCESS ON *.* FROM admin@``'localhost'``;
```

　![img](https://img2018.cnblogs.com/blog/1033265/201901/1033265-20190118160228455-1389217867.png)

 

 2.2.5 删除 mysql 用户

通过执行`drop user`命令删除MySQL用户 
还可以通过系统权限表删除（不建议）

```
mysql> drop user admin@``'localhost'``;
```

　　

## 2.2.6 设置MySQL用户资源限制

- 通过设置全局变量max_user_connections可以限制所有用户在同一时间连接MySQL实例的数量，但此参数无法对每个用户区别对待，所以MySQL提供了对每个用户的资源限制管理
- MAX_QUERIES_PER_HOUR：一个用户在一个小时内可以执行查询的次数（基本包含所有语句）
- MAX_UPDATES_PER_HOUR：一个用户在一个小时内可以执行修改的次数（仅包含修改数据库或表的语句）
- MAX_CONNECTIONS_PER_HOUR：一个用户在一个小时内可以连接MySQL的时间
- MAX_USER_CONNECTIONS：一个用户可以在`同一时间连接MySQL实例的数量`
- 从5.0.3版本开始，对用户‘user’@‘%.example.com’的资源限制是指所有通过example.com域名主机连接user用户的连接，而不是分别指从host1.example.com和host2.example.com主机过来的连接

## 2.2.7 修改 mysql 用户密码

修改用户密码的方式包括：

```
mysql> ALTER USER ``'jeffrey'``@``'localhost'` `IDENTIFIED BY ``'mypass'``;``mysql> SET PASSWORD FOR ``'jeffrey'``@``'localhost'` `= PASSWORD(``'mypass'``);``mysql> GRANT USAGE ON *.* TO ``'jeffrey'``@``'localhost'` `IDENTIFIED BY ``'mypass'``;``shell> mysqladmin -u user_name -h host_name password ``"new_password"
```

　　

创建用户时指定密码

```
mysql> CREATE USER ``'jeffrey'``@``'localhost'` `IDENTIFIED BY ``'mypass'``;
```

　　

 

 修改当前会话本身用户密码的方式包括：

```
mysql> ALTER USER USER() IDENTIFIED BY ``'mypass'``;``mysql> SET PASSWORD = PASSWORD(``'mypass'``);
```

　　

## 2.2.8 设置MySQL用户密码过期策略

设置系统参数default_password_lifetime作用于所有的用户账户

- - default_password_lifetime=180 设置180天过期
  - default_password_lifetime=0 设置密码不过期 
    如果为每个用户设置了密码过期策略，则会覆盖上述系统参数

```
ALTER USER ``'jeffrey'``@``'localhost'` `PASSWORD EXPIRE INTERVAL 90 DAY;``ALTER USER ``'jeffrey'``@``'localhost'` `PASSWORD EXPIRE NEVER; 密码不过期``ALTER USER ``'jeffrey'``@``'localhost'` `PASSWORD EXPIRE DEFAULT; 默认过期策略
```

　　

手动强制某个用户密码过期

```
ALTER USER ``'jeffrey'``@``'localhost'` `PASSWORD EXPIRE;
```

## 2.2.9 mysql 用户 lock

通过执行create user/alter user命令中带account lock/unlock子句设置用户的lock状态

Create user语句默认的用户是unlock状态

```
# 创建的时候给用户锁定``mysql> create user abc2@localhost identified by ``'mysql'` `account lock;
```

　　

Alter user语句默认不会修改用户的lock/unlock状态

```
# 修改用户为unlock``mysql> alter user abc2@``'localhost'` `account unlock;
```

　　

当客户端使用lock状态的用户登录MySQL时，会收到如此报错 
Access denied for user ‘user_name’@’host_name’. 
Account is locked.

　　

官方文档：https://dev.mysql.com/doc/refman/5.7/en/privilege-system.html