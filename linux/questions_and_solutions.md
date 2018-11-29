### 常见问题及解决
1、从 Mac 的终端中 scp VirtualBox 虚拟机中的文件时，报错如下：
```angular2
wanghuans-MacBook-Pro:clickhouse_exporter wanghuan$ scp root@192.168.33.10:/vagrant/clickhouse_exporter ./
root@192.168.33.10: Permission denied (publickey,gssapi-keyex,gssapi-with-mic).
```
解决：在虚拟机内部，修改 /etc/ssh/sshd_config 文件中如下参数：
```
PermitRootLogin yes
PasswordAuthentication yes
PubkeyAuthentication yes
```
然后重启虚拟机内 sshd 服务。


