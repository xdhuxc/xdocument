### 能看到数据库和表，但是select时报不存在
```
mysql> select * from sys_prompt limit 5;
ERROR 1146 (42S02): Table 'yonyou_cloud.sys_prompt' doesn't exist
```
或
```
mysql> drop database yhtdb;
ERROR 1010 (HY000): Error dropping database (can't rmdir './yhtdb', errno: 39)
```
解决：在`/var/lib/mysql`下，删除该数据库对应的目录，重新导入sql文件。