### BIND
Berkeley Internet Name Domain

#### 安装部署
1、使用如下命令安装 BIND
```angularjs
yum install -y bind 
```
2、启动 BIND
```angularjs
systemctl restart named
```

#### 配置
bind 的主配置文件位置：/etc/named.conf，内容如下：
```angularjs
//
// named.conf
//
// Provided by Red Hat bind package to configure the ISC BIND named(8) DNS
// server as a caching only nameserver (as a localhost DNS resolver only).
//
// See /usr/share/doc/bind*/sample/ for example named configuration files.
//
// See the BIND Administrator's Reference Manual (ARM) for details about the
// configuration located in /usr/share/doc/bind-{version}/Bv9ARM.html

options {
	listen-on port 53 { 127.0.0.1; };
	listen-on-v6 port 53 { ::1; };
	directory 	"/var/named";
	dump-file 	"/var/named/data/cache_dump.db";
	statistics-file "/var/named/data/named_stats.txt";
	memstatistics-file "/var/named/data/named_mem_stats.txt";
	allow-query     { localhost; };

	/* 
	 - If you are building an AUTHORITATIVE DNS server, do NOT enable recursion.
	 - If you are building a RECURSIVE (caching) DNS server, you need to enable 
	   recursion. 
	 - If your recursive DNS server has a public IP address, you MUST enable access 
	   control to limit queries to your legitimate users. Failing to do so will
	   cause your server to become part of large scale DNS amplification 
	   attacks. Implementing BCP38 within your network would greatly
	   reduce such attack surface 
	*/
	recursion yes;

	dnssec-enable yes;
	dnssec-validation yes;

	/* Path to ISC DLV key */
	bindkeys-file "/etc/named.iscdlv.key";

	managed-keys-directory "/var/named/dynamic";

	pid-file "/run/named/named.pid";
	session-keyfile "/run/named/session.key";
};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

zone "." IN {
	type hint;
	file "named.ca";
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";

```
选项含义：

全局配置段：
```angularjs
options {
    # 在启动 DNS 服务时，named 进程所监听的套接字
    listen-on port 53 {127.0.0.1; 10.10.24.32;};
    
    # 定义解析库（区域数据库文件）的根目录，在主配置文件中添加此配置语句之后，可以在后面定义区域数据库时使用相对路径
    directory "/var/named";
    
    # 访问控制语句，意思是允许本服务器处理那些主机发送来的解析查询请求，默认是localhost，即只允许本机以127.0.0.1发送查询请求
    allow-query {localhost};
    
    # 允许本服务器为所有查询请求做递归查询
    recursion yes;
    
    # 访问控制指令，允许这些客户端做递归查询
    allow-recursion {172.16.1.74/16;};
    
    # 定义存放主进程 pid 文件的路径
    pid-file "/run/named/named.pid";
}
```


区域配置段：
```angularjs
# 声明一个区域名称，此名称要使用 FQDN 来表示，例如 xdhuxc.com
zone "FQDN" IN {
    # 区域的类型
    type master;   # 主区域
    type slave;    # 辅助区域
    type hint;     # 提示区域，仅能在根域上设置
    type forward;  # 转发区域 
    
    # 存放于该域有关的解析信息的数据库文件的路径；如果是相对路径，则相对于在主配置文件的全局配置端中的 “directory” 指令所定义的目录而言。
    # 注意：文件的所有权和权限设置必须能够让 named 用户有读取权限。
    file "named.localhost";
    
    # 访问控制指令，允许这些客户端对数据库内容进行动态更新，主要用于 DDNS
    allow-update {none;};
    
    # 访问控制指令，允许这些主机能够从当前服务器进行区域传送 
    allow-transfer {172.16.0.0/16;};
    
    # 访问控制指令，允许这些客户端做递归查询 
    allow-recursion {172.16.1.74/16;};
    
    # 访问控制指令，允许这些主机进行区域内的解析查询 
    allow-query { address_match_element;...};
    
    # 访问控制指令，允许这些主机向当前服务器发送区域变更通知 
    allow-update { address_match_element;...};
}
```

在修改完 bind 的配置文件之后，需要使用 named-checkconf 来检测文件的书写格式是否正确。

在检测完后，配置文件的内容并不会立即生效，因此我们需要重新读取 named 程序，可以使用 systemctl 命令，也可以使用 rndc reload 来读取。

区域配置文件示例：
```angularjs
zone "FQDN" IN {
    type master;
    file "FQDN.zone";
    allow-update {none;};
    allow-transfer {none;};
}
```
当区域配置文件配置完成之后，我们需要在其数据库 /var/named/named.localhost 中添加记录，
```angularjs
$TTL 1D
@	IN SOA	@ rname.invalid. (
					0	    ; serial
					1D	    ; refresh
					1H	    ; retry
					1W	    ; expire
					3H )	; minimum
	NS	@
	A	127.0.0.1
	AAAA	::1
```
在数据库中，有各种记录书写的格式，简要介绍各种记录的书写格式：

##### SOA
DN|FQDN：当前域的域名，例如 xdhuxc.com，或者使用 @ 代替域名，@ 符号会使用主配置文件中定义的域名来代替
VALUE：由以下几个部分组成：
1、当前域中的主域名服务器的 FQDN
2、当前域的数据库管理员的邮箱地址，需要使用 . 来替代 @ ：root.xdhuxc.com
3、主域名服务器进行区域传送的相关时间参数的定义：
(Serial Refresh Retry Expeir TTL)
（
    Serial;
    Refresh;
    Retry;
    Expeir;
    TTL;
）
    
##### NS 记录
name：当前域的域名，可以写完全合格域名 FQDN，可以写 @ 占位，还可以省略不写，如果省略不写，则意味着该资源记录的名称与其上一条资源记录的名称相同。
value：当前区域内被授权的域名服务器的 FQDN。
   
注意：
1、一个域中有多少台域名服务器，就需要写多少个 NS 资源记录。

2、每个 NS 资源记录都必须要有一个 A 记录与之对应。
   
##### MX 记录
name：当前域的域名，可以写完全合格域名 FQDN，可以写 @ 占位，还可以省略不写，如果省略不写，则意味着该资源记录的名称与其上一条资源记录的名称相同。
RR_TYPE：MS Priority
value：当前域中有效的邮件服务器的 FQDN。

注意：
1、一个域中，可以有多条 MX 资源记录，通过优先级的大小决定被使用的次序。

2、每个 MX 资源记录都必须对应一条 A 记录。
   
##### A 记录
name：域中指定主机的 FQDN。
value：该主机上真实有效的 IPV4 地址。

例如：
www.xdhuxc.com 43200 IN A 192.168.1.1
www            43200 IN A 192.168.1.1

泛域名：
*.xdhuxc.com 43200 IN A 192.168.1.1
*            43200 IN A 192.168.1.1

直接域名解析：
xdhuxc.com 43200 IN A 192.168.1.1

通常，泛域名或直接域名都是为了防止用户写错域名而导致无法给出正确的解析结果。
  
##### CNAME 记录
name：域中指定主机的别名。
value：真正的主机的 FQDN。

例如：
ftp.xdhuxc.com [86400] IN CNAME www.xdhuxc.com
ftp [86400] IN CNAME www

##### PTR 记录：
name：
将 IP 地址的四个八位组反过来，将 IP 地址中的主机部分加上反向域的域名后缀

如果 IP 地址是：172.16.1.100/16，其对应的域名的写法为：
```angularjs
100.1 IN PTR www.xdhuxc.com 
100.1.16.172.in-addr.arpa. IN PTR www.xdhuxc.com 
```
       

#### 性能测试



#### 参考资料
1、queryperf 性能测试工具

https://blog.csdn.net/zhu_tianwei/article/details/45202899

2、BIND 配置文件详解

http://blog.51cto.com/liujingyu/2102858

