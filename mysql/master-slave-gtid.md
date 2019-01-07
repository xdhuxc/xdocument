### 基于 GTID 的主从复制



### 配置 GTID 主从复制
使用 docker 方式部署 MySQL 主从节点

1、创建如下目录
```angular2html
cd /data
mkdir mysql/master mysql/slave
```

#### master 配置
在 /data/mysql/master 目录下创建文件 mysql.cnf:
```angular2html
[mysqld]
############# basic settings #############
server_id = 1
port = 3307
bind_address = 0.0.0.0
max_binlog_size = 100M

############# log settings #############
log_bin = ON
expire_logs_days = 30
binlog_format = row
sync_binlog = 1
binlog_rows_query_log_events = 1

############# replication #############
gtid_mode = ON
enforce_gtid_consistency = ON
log_slave_updates = ON
master_info_repository = table
relay_log_info_repository = table
slave_parallel_workers = 1
```
在 MySQL 5.6 版本时，基于 GTID 的复制中 log-slave-updates 选项是必须的，但是其增大了从服务器的 IO 负载，而在 MySQL 5.7 中该选项已经不是必须项。

mysql master 容器启动命令为：
```angular2html
docker run -d \
    --restart=always \
    --name=mysql-master \
    -p 3307:3307 \
    -v /data/mysql/master/mysql.cnf:/etc/mysql/conf.d/mysql.cnf \
    -v /data/mysql/master/data:/var/lib/mysql \
    -e MYSQL_ROOT_PASSWORD=1qaz1QAZ \
    mysql:5.7.24
```
启动之后进入容器查看效果：
```angular2html
mysql> show global variables like "%gtid%";
+----------------------------------+-------+
| Variable_name                    | Value |
+----------------------------------+-------+
| binlog_gtid_simple_recovery      | ON    |
| enforce_gtid_consistency         | ON    |
| gtid_executed                    |       |
| gtid_executed_compression_period | 1000  |
| gtid_mode                        | ON    |
| gtid_owned                       |       |
| gtid_purged                      |       |
| session_track_gtids              | OFF   |
+----------------------------------+-------+
8 rows in set (0.00 sec)
```
可以看到，GTID 模式已经开启了。

创建复制用户
```angular2html
create user 'xdhuxc'@'%' identified by '1qaz1QAZ';
grant replication slave on *.* to 'xdhuxc'@'%';
flush privileges;
```

#### slave 配置
在 /data/mysql/slave 目录下创建 mysql.cnf 文件：
```angular2html
[mysqld]
server_id = 3

gtid_mode = ON
enforce_gtid_consistency = ON
log_slave_updates = ON
read_only = ON
expire_logs_days = 30
max_binlog_size = 100M
```

启动 mysql salve 容器
```angular2html
docker run -d \
    --restart=always \
    --name=mysql-slave \
    -p 3308:3306 \
    -v /data/mysql/slave/mysql.cnf:/etc/mysql/conf.d/mysql.cnf \
    -v /data/mysql/slave/data:/var/lib/mysql \
    -e MYSQL_ROOT_PASSWORD=1qaz1QAZ \
    mysql:5.7.24
```

slave 连接至 master
```angular2html
change master to master_host='172.27.126.226', master_port=3307, master_user='xdhuxc', master_password='1qaz1QAZ', master_auto_position=1;
```

启动复制线程
```angular2html
start slave;
```

#### 验证主从复制
1、在 mysql-master 容器内
```angular2html
mysql> show slave hosts;
+-----------+------+------+-----------+--------------------------------------+
| Server_id | Host | Port | Master_id | Slave_UUID                           |
+-----------+------+------+-----------+--------------------------------------+
|         3 |      | 3306 |         1 | ab2050d7-1244-11e9-bfa7-0242ac110005 |
+-----------+------+------+-----------+--------------------------------------+
1 row in set (0.00 sec)
```
2、在 mysql-slave 容器内
```angular2html
mysql> show slave status \G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 172.27.126.226
                  Master_User: xdhuxc
                  Master_Port: 3307
                Connect_Retry: 60
              Master_Log_File: ON.000004
          Read_Master_Log_Pos: 791
               Relay_Log_File: c81fccb3bd9c-relay-bin.000004
                Relay_Log_Pos: 990
        Relay_Master_Log_File: ON.000004
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 791
              Relay_Log_Space: 1443
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 0
               Last_SQL_Error:
  Replicate_Ignore_Server_Ids:
             Master_Server_Id: 1
                  Master_UUID: d1623ded-124a-11e9-b46d-0242ac110004
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind:
      Last_IO_Error_Timestamp:
     Last_SQL_Error_Timestamp:
               Master_SSL_Crl:
           Master_SSL_Crlpath:
           Retrieved_Gtid_Set: d1623ded-124a-11e9-b46d-0242ac110004:1-8
            Executed_Gtid_Set: d1623ded-124a-11e9-b46d-0242ac110004:1-8
                Auto_Position: 1
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
1 row in set (0.00 sec)
```
可以看到 IO 和 SQL 线程的状态都为：YES，Retrieved_Gtid_Set 接收了 8 个事务，Executed_Gtid_Set 执行了 8 个事务。

在 mysql-master 容器中创建数据库
```angular2html
mysql> create database blog;
Query OK, 1 row affected (0.00 sec)
```

```angular2html
create table `test` (
    `id` int(11),
    `name` varchar(32),
);
```


在 mysql-slave 中查看
```angular2html
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| blog               |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)
```


#### 参考资料

https://www.hi-linux.com/posts/47176.html

http://www.ywnds.com/?p=3898

