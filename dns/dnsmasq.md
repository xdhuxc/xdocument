### 简介
dnsmasq 是一个小巧且方便地用于配置 DNS 和 DHCP 的工具，适用于小型网络，相比于 bind 和 dhcpd，配置起来更简单。dnsmasq 能够提供本地解析和外部 DNS 服务器代理，通常将其作为一个 DNS 中继代理。

### 安装
可以通过 yum 命令直接安装 dnsmasq，命令如下：
```angularjs
yum install -y dnsmasq
```
安装完成后，可以通过 `dnsmasq --help` 或者 `man dnsmasq` 查看支持的配置。

### 配置
配置文件位置为：`/etc/dnsmasq.conf`，默认情况下，dnsmasq.conf 中只开启了最后的 include 项，可以在 /etc/dnsmasq.d 中编写任意名称的配置文件。

dnsmasq.conf 文件内容如下：
```angularjs
# 监听的端口，DNS 默认为 53 端口，如果设置为 0，则完全禁止 DNS 功能。
port=53
# 监听地址
listen-address=10.10.24.32

# 正确的域名格式才转发
domain-needed

# 设置本地域扩展，相当于域简写，例如 hosts 配置 www 会自动加上 www.xdhuxc.com 
expand-hosts 
local=/xdhuxc.com/

# 配置上游的 nameserver 解析文件 
#resolv-file=/etc/dnsmasq.resolv.conf
# 当 /etc/resolv.conf 或 resolv-file 文件变化时，不重新加载。
no-poll 
# 不使用上游 nameserver 配置文件（/etc/resolv.conf 和 resolv-file）
no-resolv 

# 配置本地解析的 hosts
addn-hosts=/etc/dnsmasq.hosts
# 不使用 /etc/hosts，开启后 expand-hosts 不生效 
# no-hosts 

# 按配置顺序查询上级 nameserver 服务器
strict-order 

# 记录日志，如果打开日志，要及时清理 
log-queries
log-facility=/var/log/dnsmasq.log 
# 启用异步日志记录，缓解阻塞，提高性能。默认队列长度为：5，合理值为：5~25，最大限制为：100
log-async=20

# 缓存地址数量，提高查询速度
cache-size=10000
# 自动加载配置文件目录 
conf-dir=/etc/dnsmasq.d 
```
在 /etc/dnsmasq.d 目录下配置自定义项 server.conf
```angularjs
# 指定 dnsmasq 默认查询的上游域名服务器  
server=8.8.8.8 
server=114.114.114.114
# 可以将特定的域名指定解析它的域名服务器，一般是其他的内部域名服务器。
server=/baidu.com/61.135.165.235
# 指定反向 DNS 192.168.1/24 网段到 192.168.2.1 DNS 查询 
# server=/192.168.1.in-addr.arpa/192.168.2.1

# 给 *.apple.com 和 taobao.com 使用专用的 DNS
server=/taobo.com/223.5.5.5
server=/.apple.com/223.5.5.5

# 把 www.hi-linux.com 解析到特定的 IP
address=/www.hi-linux.com/192.168.101.107 

# 把所有 .cn 的域名全部通过 114.114.114.114 这台国内 DNS 服务器来解析 
server=/cn/114.114.114.114
```
address.conf 配置自定义域名服务器
```angularjs
# 指定 domain 解析地址 
address=/www.test.com/127.0.0.1
# *.xdhuxc.com 匹配 
address=/xdhuxc.com/127.0.0.1
address=/.abc.com/1.1.1.1 
address=/ipv6.xdhuxc.com/fe80::4d7:a0ff:fe00:f9b
```
### 启动服务
启动 dnsmasq 服务
```angularjs
systemctl restart dnsmasq
```
使用 dig 命令查询解析是否正确，例如：
```angularjs
[root@xdhuxc etc]# dig @10.10.24.32 www.xdhuxc.com

; <<>> DiG 9.9.4-RedHat-9.9.4-61.el7_5.1 <<>> @10.10.24.32 www.xdhuxc.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 4987
;; flags: qr aa rd ra ad; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;www.xdhuxc.com.			IN	A

;; ANSWER SECTION:
www.xdhuxc.com.		0	IN	A	127.0.0.1

;; Query time: 0 msec
;; SERVER: 10.10.24.32#53(10.10.24.32)
;; WHEN: Mon Oct 15 14:53:28 CST 2018
;; MSG SIZE  rcvd: 48

```

### 修改 iptables 配置
1、允许本机的 53 端口可对外访问
```angularjs
iptables -A INPUT -p udp -m udp --dport 53 -j ACCEPT
iptables -A INPUT -p tcp -m tcp --dport 53 -j ACCEPT 
```
2、转发 DNS 请求

开启流量转发功能
```angularjs
echo '1' > /proc/sys/net/ipv4/ip_forward
echo '1' > /proc/sys/net/ipv6/ip_forward
```

添加流量转发规则，将外部到 53 的端口的请求映射到 dnsmasq 服务器的 53 端口
```angularjs
iptables -t nat -A PREROUTING -p udp --dport 53 -j REDIRECT --to-port 53
iptables -t nat -A PREROUTING -p tcp --dport 53 -j REDIRECT --to-port 53
```

如果要限制只允许内网的请求，规则如下：
```angularjs
iptables -t nat -A PREROUTING -i eth0 -p udp --dport 53 -j REDIRECT --to-port 53
```


3、使用如下命令保存 iptables 规则并重启 iptables 服务
```
service iptables save      # 保存 iptables 规则  
systemctl restart iptables
```

### dnsmasq 优化

#### 选择最快的上游 DNS 服务器
经常会有这样的情形，dnsmasq 服务器配置了很多上游域名服务器，转发本地的 DNS 请求，但 dnsmasq 实际上是只挑选了一个上游 DNS 服务器来查询并转发结果，这样如果选错服务器的话就会导致 DNS 响应变慢。
解决方法：
修改 dnsmasq.conf 的配置
```angularjs
all-servers
server=8.8.8.8
server=114.114.114.114
```
all-servers 表示对以下设置的所有 server 发起查询，选择回应最快的一条作为查询结果返回。上面设置了两个域名服务器，会同时查询这两个域名服务器，询问 DNS 地址，谁返回快，就采用谁的结果。

#### dnsmasq-china-list 项目
dnsmasq-china-list 项目维护了一张国内常用但是通过国外 DNS 会解析错误的网站域名的列表，保证列表中的国内域名全部走国内 DNS 服务器解析。

项目地址：https://github.com/felixonmars/dnsmasq-china-list

操作步骤：

1、取消 dnsmasq.conf 中 conf-dir=/etc/dnsmasq.d 这一行的注释

2、获取项目文件
```angularjs
git clone https://github.com/felixonmars/dnsmasq-china-list.git
```

3、将 accelerated-domains.china.conf, bogus-nxdomain.china.conf,google.china.conf （可选）放到 /etc/dnsmasq.d/ 目录下

4、将 dnsmasq-update-china-list 放到 /usr/bin/ 目录下，这是一个批量修改 DNS 服务器的工具。

#### dnsmasq 性能优化
bind 在不配合数据库的情况下，经常需要重新载入并读取配置文件，这是造成性能低下的原因。因此，我们可以考虑不读取 /etc/hosts 文件，而是另外指定一个在共享内存里的文件，比如 /dev/shm/dnsrecord.txt，这样就可以提高 DNS 解析速度了。又由于内存是易失性的存储器，重启后就会消失，可以定期同步硬盘上的某个文件内容到内存中。

操作步骤：
1、配置 /etc/dnsmasq.conf
```angularjs
no-hosts
addn-hosts=/dev/shm/dnsrecord.txt 
```

2、解决同步问题

开机启动
```angularjs
echo "cat /etc/hosts > /dev/shm/dnsrecord.txt" >> /etc/rc.local 
```

定时同步内容
```angularjs
crontab -e 
*/10 * * * * cat /etc/hosts > /dev/shm/dnsrecord.txt 
```


### 参考资料

http://chuansong.me/n/471642951828

https://blog.csdn.net/zhu_tianwei/article/details/72632078