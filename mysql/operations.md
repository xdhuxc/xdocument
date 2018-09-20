### 赋权操作
1、创建只读账户
```
mysql> grant select on *.* to 'xdhuxc'@'%' identified by 'Xdhuxc163';  # 创建只读账户xdhuxc，密码为：Xdhuxc63
mysql> flush privileges;
```
查看其权限
```
mysql> show grants for 'xdhuxc'@'%';
```

2、授予权限
```angular2html

```

### 修改密码

#### 已知原密码修改密码


#### 忘记密码修改密码
##### Linux 系统
1、停止数据库服务。
```angular2html
systemctl stop mysqld
```

2、向 /etc/my.cnf 中添加如下内容
```angular2html
[mysqld]
skip-grant-tables
```

3、重启 mysql 服务
```angular2html
systemctl restart mysqld
```

4、修改密码
```angular2html
mysql> use mysql;
mysql> update user set authentication_string=password('Xdhuxc163') where user='root';
mysql> flush privileges;
mysql> quit
```
5、注释掉 skip-grant-tables，重启 mysqld 服务。

##### Windows 系统
1、关闭MySQL服务
```
net stop mysql57
```

2、进入 MySQL 的 bin 目录下，执行如下命令：
```
mysqld --default-file="E:\Work\MySQL\MySQL Server 5.7 Data\my.ini" --console --skip-grant-tables
```
说明：
--default-file：启动 MySQL 服务时的配置文件；
--skip-grant-tables：启动 MySQL 服务的时候跳过权限表认证；

3、新打开DOS窗口，进入MySQL的bin目录下，执行如下命令：
```
mysql -uroot -p # 回车后，让输入密码，直接回车
```
4、更改root用户密码
```
# MySQL 5.7以前
update mysql.user
set password=password('123456')
 where user='root';
# MySQL 5.7
update mysql.user
set authentication_string=password('123456')
 where user='root';
```
5、更新权限
```
flush privileges;
```
6、退出当前MySQL回话并启动MySQL服务
```
net start mysql57
```

### 修改默认时区

#### 方法一
1、查看 mysql 系统时间
```
mysql> show variables like '%time_zone%';
+------------------+--------+
| Variable_name    | Value  |
+------------------+--------+
| system_time_zone | UTC    |
| time_zone        | +08:00 |
+------------------+--------+
2 rows in set

mysql> select now();
+---------------------+
| now()               |
+---------------------+
| 2018-07-09 13:59:16 |
+---------------------+
1 row in set
```
mysql 的系统时区已经是东八区，系统时间也是当前时间了，

2、设置时区为东八区
```
set global time_zone = '+8:00';
```
3、刷新权限
```
flush privileges;
```
#### 方法二
或者直接在 my.cnf 文件中添加如下内容：
```
[mysqld]
default-time_zone = '+8:00'
```
然后重启 mysql 服务
