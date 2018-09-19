### 挂载磁盘
1、使用fdisk –l（查看当前系统所有硬盘及分区情况）， 找到待挂载的磁盘。
![image](images/disk.png)

2、创建分区待挂载的目录，例如/xdhuxc，必须是空目录。
```angular2html
mkdir /xdhuxc
```
3、格式化新建的磁盘分区。
```angular2html
mkfs.ext4 /dev/vdb
```
4、修改/etc/fstab，添加如下内容，将分区信息写进去。
```angular2html
/dev/vdb  /xdhuxc      ext4    defaults        0 0
```
5、执行如下命令，加载所有在fstab中记录的文件系统。
```angular2html
mount –a
```