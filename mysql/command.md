### 常用命令
1、显示当前 MySQL 的连接数
```
show processlist;
或
show full processlist;
```
2、显示 MySQL 的环境变量
```
show variables;
```
3、查找最大连接数或服务器响应的最大连接数
```
show variables like "%conn%";
或
show global status like 'Max_used_connections'; # 查询服务器响应的最大连接数
```
4、临时设置最大连接数
```
set global max_connections=200;  # 在 mysql 终端中临时设置最大连接数为：200
```
或者在 /etc/my.cnf 中添加如下内容：
```angular2html
[mysqld]
max_connections=200;
```

5、查询缓存的参数说明
```
show global variables like "query_cache%";
```
6、查询 MySQL 运行产生的状态值
```
show global status like 'qcache%';
```
7、采用直接重置 autoIncrement 值的方法重置 auto_increment
```
alter table marathon auto_increment = 1;
```
8、查询数据库权限
```angular2html
show grants for 'root'@'localhost';
```