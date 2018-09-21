#### 配置文件
Linux系统下，配置文件位于：
```
/etc/my.cnf 或 /etc/mysql/my.cnf /usr/etc/my.cnf ~/.my.cnf 
```

配置文件内容为：
```
[client]

# 设置 mysql 客户端连接服务端时默认使用的端口。
port = 3306
socket = /usr/local/mysql/mysql.sock 
# 设置 mysql 客户端默认字符集。
default-character-set = utf8

[mysql]
no-auto-rehash

[mysqld]

# 设置 mysqld 服务端默认监听的 TCP/IP 端口。
port = 3306

# 设置 mysql 服务器默认字符集。
default-character-set = utf8

# mysql 的时区默认是服务器的时区，设置 mysql 时区为东八区
default-time_zone = '+8:00'

# 基准路径，其他路径都相对于这个路径。
basedir = /usr/local/mysql 

# mysql 数据库文件所在目录。
datadir = /data/

log_bin = mysql-bin 
binlog_format = mixed
# 超过 30 天的 binlog 自动删除。 
expire_logs_days = 30 
# 错误日志路径
log_error = /logs/mysql-error.log

slow_query_log = 1
# 慢查询时间，超过 1秒 则为慢查询。
long_query_time = 1 
# 慢查询日志
slow_query_log_file = /logs/mysql-slow.log 

performance_schema = 0
explicit_defaults_for_timestamp

# 不区分大小写
# lower_case_table_names = 1

# 创建新表时使用的默认存储引擎。
default-storage-engine = INNODB

# mysql 服务器支持的最大并发连接数，默认值为：150，但总会预留其中一个连接给管理员使用 super 权限登录，即使连接数目达到最大限制。
# 如果服务器的并发连接请求量比较大，建议提高该值，以增加并发连接数量。
# 当然这是建立在机器资源能够支撑的情况下，mysql 会为每个连接提供连接缓冲区，那么连接数越多，内存开销就会越大，因此要适当调整该值。
max_connections = 500

# 对于同一主机，如果有超出该值个数的中断错误连接，则该主机将被禁止连接。如需对该主机进行解禁，需要执行 flush host 命令。
max_connect_errors = 6000

# 服务器关闭交互式连接前等待活动的秒数。
# 交互式客户端定义为在 mysql_real_connect() 中使用 CLIENT_INTERACTIVE 选项的客户端，默认值为：28800秒，即8小时。
interactive_timeout = 28800

# 服务器关闭非交互式连接之前等待活动的秒数。
# 在线程启动时，根据全局 wait_timeout 的值或全局 interactive_timeout 的值初始化 wait_timeout 的值。 
# 取决于客户端类型，由 mysql_real_connect() 的连接选项 CLIENT_INTERACTIVE 定义，参数的默认值为：28800。
# mysql 服务器所支持的最大连接数是有上限的，因为每个连接的建立都会消耗内存，因此，我们希望客户端在连接到 mysql 服务器处理完相应的操作后，应该断开连接并释放占用的内存。
# 如果 mysql 服务器有大量的闲置连接，它们不仅会白白消耗内存，而且如果连接一直在累加而不断开，最终肯定会达到 mysql 服务器的连接上限数，这会报“too many connections” 错误。
# 对于 wait_timeout 的值的设定，应该根据系统的运行情况来判断。在系统运行一段时间后，可以通过 show processlist 命令查看当前系统的连接状态。 
# 如果发现有大量的 sleep 状态的连接进程，则说明该参数设置得过大，可以适当地调小些，要同时设置 interactive_timeout 和 wait_timeout 才会生效。
wait_timeout = 28800

# mysql 能够打开的文件描述符限制，默认最小为：1024。
# 当 open_files_limit 没有被设置的时候，比较 max_connections*5 和 ulimit -n 的值，哪个大用那个。
# 当 open_files_limit 被配置的时候，比较 open_files_limit 和 max_connections*5 的值，哪个大用那个。 
open_files_limit = 65535

# 指定 mysql 查询缓冲区的大小，可以通过在 mysql 控制台观察。
# 使用查询缓冲， mysql 将 select 语句和查询结果存放在缓冲区中，今后对于同样的 select 语句（区分大小写），将直接从缓冲区中读取结果。根据 MySQL 用户手册，使用查询缓冲最多可以提高 238% 的效率。
# 如果 Qcache_lowmem_prunes 的值非常大，则表明经常出现缓冲不够的情况；
# 如果 Qcache_hits 的值非常大，则表明查询缓冲使用非常频繁；如果该值较小反而会影响效率，那么可以考虑不用查询缓冲。
# 如果 Qcache_free_blocks 的值非常大，则表明缓冲区中碎片很多。
# 此外，在 select 语句中加入 SQL_NO_CACHE 可以明确表示不使用查询缓冲。
query_cache_size = 50M

# 指定单个查询能够使用的缓冲区大小，默认为：1M。
query_cache_limit = 2M

# 用于设置 table 高速缓存的数量。
# 由于每个客户端连接都会至少访问一个表，因此此参数的值与 max_connections 有关。
# 当某一连接访问一个表时，mysql 会检查当前已缓存的表的数量。如果该表已经在缓存中打开，则会直接访问缓存中的表以加快查询速度；如果该表未被缓存，则会将当前的表添加进缓存并进行查询。
# 在执行缓存操作之前，table_cache 用于限制缓存表的最大数目，如果当前已经缓存的表未达到 table_cache 所设定的值，则会将新表添加进来；若已经达到此值，mysql 将根据缓存表的最后查询时间、查询率等规则释放之前的缓存。
table_open_cache = 512

# 内存中的每个临时表允许的最大大小。如果临时表大小超过该值，临时表将自动转为基于磁盘的表。
# 
tmp_table_size = 64M

# 缓存的最大线程数，表示可以重新利用保存在缓存中的线程的数量。
# 当客户端连接断开时，如果客户端总连接数小于该值，即缓存中还有空间，则处理客户端任务的线程放回缓存。
# 如果线程被重新请求，那么请求将从缓存中读取；如果缓存中是空的或者是新的请求，那么这个线程将被重新创建。
# 在高并发情况下，如果该值设置得太小，就会有很多线程频繁创建，线程创建的开销会变大，查询效率也会下降。
# 一般来说，如果在应用程序中有良好的多线程处理，这个参数对性能不会有太大的提高。
# 根据物理内存设置规则如下：
# 1G -> 8
# 2G -> 16
# 3G -> 32
# 大于 3G -> 64
thread_cache_size = 8

# 避免 mysql 的外部锁定，减少出错几率，增强稳定性，该选项默认开启。
skip-external-locking

# 禁止 mysql 对外部连接进行 DNS 解析，使用这一选项可以消除 mysql 进行 DNS 解析的时间。
# 但是需要注意，如果开启该选项，则所有远程主机连接授权都要使用 IP 地址方式，否则 mysql 将无法正常处理连接请求。
skip-name-resolve

# 指定在 mysql 暂时停止响应新请求之前的短时间内多少个请求可以被存放到堆栈中。
# 如果系统在一个短时间内有很多连接，则需要增大该参数的值，该值指定到来的的TCP/IP连接的监听队列的大小。
# 不同的操作系统在这个队列大小上有它自己的限制，试图设置 back_log 的值高于操作系统的限制将是无效的，
# 默认值为：80，对于 Linux 系统，推荐设置为小于 512 的整数。
back_log = 150

# 接收的数据包大小，增加该变量的值十分安全，这是因为仅当需要时才会分配额外的内存。
# 例如，仅当发出长查询或 mysqld 必须返回大的结果集时 mysqld 才会分配更多内存。
# 该变量之所以取较小的默认值是一种预防措施，以捕获客户端和服务器之间的错误信息包，并确保不会因偶然使用大的信息包而导致内存溢出。
max_allowed_packet = 32M

# 一个事务在没有提交的时候产生的日志会记录到 cache 中，等到事务需要提交的时候，则把日志持久化到磁盘。
# binlog_cache_size 的默认大小为：32K。
binlog_cache_size = 1M

# 定义了用户可以创建的内存表的大小。
# 该值用来计算内存表的最大行数值，支持动态改变。
max_heap_table_size = 8M

# mysql 读入缓冲区大小。
# 对表进行顺序扫描的请求将分配一个读入缓冲区，mysql 会为它分配一段内存缓冲区，此变量控制这一缓冲区的大小。
# 如果对表的顺序扫描请求非常频繁，并且频繁扫描进行得太慢，可以通过增加此变量的值及内存缓冲区的大小提高其性能。
read_buffer_size = 2M

# mysql 随机读缓冲区大小。
# 当按任意顺序读取行时（例如，按照排序顺序），mysql 将分配一个随机读缓冲区。
# 在进行排序查询时，mysql 会首先扫描一遍该缓冲区，以避免在磁盘上搜索，提高查询速度。如果需要排序大量数据，可适当调高此值。
# 当 mysql 会为每个客户端连接分配该缓冲空间，所以应尽量适当设置该值，以避免内存开销过大。
read_rnd_buffer_size = 8M

# mysql 执行排序时使用的缓冲区大小。
# 如果想要增加 order by 的速度，首先看是否可以让 mysql 使用索引而不是额外的排序阶段，如果不能，可以尝试增加 sort_buffer_size 值的大小。
sort_buffer_size = 8M

# 联合查询操作所能使用的缓冲区大小。
# 和 sort_buffer_size 一样，此参数分配的内存也是每个连接独享的，即连接级参数。
join_buffer_size = 8M

# 指定用于索引的缓冲区大小，增加它可得到更好处理的索引（对所有读和多重写），直到机器所能负担得起。
# 如果此参数值太大，系统将开始换页，并变慢。
# 对于内存在 4GB 左右的服务器，该参数可以设置为：384M 或 512M。
# 通过检查状态值 key_read_request 和 key_reads，可以知道 key_buffer_size 设置是否合理。
# 比例 key_reads / key_read_requests 应该尽可能的低，至少是 1:100，1:1000 更好，上述状态值可以使用 show status like 'key_read%' 获得。
# 注意，该参数值设置得过大反而会使服务器整体效率降低。
key_buffer_size = 4M

# 分词词汇的最小长度，默认为：4。
ft_min_word_len = 4

# mysql 支持四种事物隔离级别，分别是：READ-UNCOMMITTED，READ-COMMITTED，REPEATABLE-READ，SERIALIZABLE，如果没有指定，mysql 默认采用的是：REPEATABLE-READ。
# oracle 默认的事物隔离级别是：READ-COMMITTED。
transaction_isolation = REPEATABLE-READ 

# innodb 引擎配置

innodb_file_per_table = 1

# 限制 InnoDB 同时能打开的表数据文件的最大值，如果数据库中的表特别多，需要增大此参数的值。
# 此参数的值默认为：300，最小值为：10。
innodb_open_files = 500

# InnoDB 使用一个缓冲池来保存索引和原始数据，此处设置的值越大，在存取表里面数据时所需要的磁盘 I/O 越少。
# 在一个独立使用的数据库服务器上，可以设置这个变量到服务器物理内存大小的 80%。
# 不要设置过大，否则由于物理内存的竞争，可能会导致操作系统的换页颠簸。
# 注意，在 32 位系统上，每个进程可能被限制在 2~3.5G 用户层面内存限制，因此，不要设置得太高。
innodb_buffer_pool_size = 64M

# innodb 使用后台线程处理数据页上的读写 I/O 请求，根据机器的 CPU 核数来修改，默认为：4。
# 注意，这两个参数不支持动态改变，需要将该参数写入 my.cnf 文件中，修改完成后重启 mysql 服务，允许值的范围为：1~64。 
innodb_write_io_threads = 4
innodb_read_io_threads = 4

# 默认设置为：0，表示不限制并发数，也推荐设置为：0，更好地发挥多核 CPU 的处理能力，提高并发量。
innodb_thread_concurrency = 0

# InnoDB 中的清除操作是一类定期回收无用数据的操作。
# 在之前的几个版本中，清除操作是主线程的一部分，这意味着运行时它可能会堵塞其他的数据库操作。 
# 从 mysql 5.5.X 版本开始，该操作运行于独立的线程中，并支持更多的并发数。
# 用户可通过设置 innodb_purge_threads 参数来选择清除操作是否使用单独线程。
# 默认情况下，参数设置为：0，表示不使用单独线程；设置为：1 时，表示使用单独的清除线程，建议设置为：1.
innodb_purge_threads = 1

# 0：如果 innodb_flush_log_at_trx_commit 的值为 0，日志缓冲区就会每秒刷写日志文件到磁盘，提交事务的时候不做任何操作。
# 执行是由 mysql 的 master thread 线程来执行的，主线程中每秒会将重做日志缓冲写入磁盘的重做日志文件中，不论事务是否已经提交，默认的日志文件是：ib_logfile0，ib_logfile1。
# 1：当设置为默认值 1 的时候，每次提交事务的时候，都会将日志缓冲区刷写到日志。
# 2：如果 innodb_flush_log_at_trx_commit 的值为 2，每次提交事务都会写日志，但并不会执行刷的操作，每秒定时会刷到日志文件。要注意的是，并不能保证 100% 每秒一定都会刷到磁盘，这要取决于进程的调度。
# 每次事务提交的时候将数据写入事务日志，而这里的写入仅是调用了文件系统的写入操作，文件系统是有缓存的，所以这个写入并不能保证数据已经写入到物理磁盘。
# 默认值 1 是为了保证完整的 ACID，当然，可以将这个配置项设为 1 以外的值来换取更高的性能，但是在系统崩溃的时候，将会失去 1 秒的数据。
# 设为 0 的话，mysqld 进程崩溃的时候，会丢失最后 1 秒的事务。
# 设为 2 的话，只有在操作系统崩溃或者断电的时候才会丢失最后 1 秒的数据，InnoDB在做恢复的时候会忽略这个值。
# 结论：设为 1 当然是最安全的，但性能页是最差的，相对于其他两个而言，但不是不能接受。如果对数据一致性和完整性要求不高，完全可以设置为：2。如果只追求性能，例如高并发写的日志服务器，设为 0 来获得更高性能。
innodb_flush_log_at_trx_commit = 1

# 设置日志缓冲区的大小，以 M 为单位，缓冲区更大能提高性能，但意外的故障将会丢失数据。MySQL 开发人员建议设置为：1M~8M 之间。
innodb_log_buffer_size = 2M

# 设置数据日志文件的大小，更大的值可以提高性能，但也会增加恢复故障数据库所需的时间。
innodb_log_file_size = 32M

# 为提高性能，mysql 可以以循环方式将日志文件写到多个文件，推荐设置为：3。 
innodb_log_files_in_group = 3

# InnoDB 主线程刷新缓存池中的数据，使脏数据比例小于 90%。
innodb_max_dirty_pages_pct = 90

# InnoDB 事务在被回滚之前可以等待一个锁定的超时秒数，默认值为：50秒。
# InnoDB 在它自己的锁定表中自动检测事务死锁并且回滚事务。InnoDB 用 LOCK TABLES 语句锁定表，
innodb_lock_wait_timeout = 120

# MyISAM 引擎配置 

# 批量插入缓存大小，该参数是针对 MyISAM 存储引擎来说的，适用于在一次性插入100~1000+条记录时，提高效率。
# 默认值为：8M，可以针对数据量的大小，翻倍增加。
bulk_insert_buffer_size = 8M

# MyISAM 设置恢复表之时使用的缓冲区的大小，当在 REPAIR TABLE 或用 CREATE INDEX 创建索引或 ALTER TABLE 过程中排序， MyISAM索引分配的缓冲区。
myisam_sort_buffer_size = 8M

# 此参数以字节的形式给出，如果临时文件会变得超过索引，不要使用快速排序索引方法来创建一个索引。
myisam_max_sort_file_size = 10G

# 如果该值大于 1，在 Repair by sorting 过程中并行创建 MyISAM 表索引（每个索引在自己的线程内）
myisam_repair_threads = 1

# sync_binlog 参数控制 mysql 的二进制日志同步到磁盘的频率，取值为：0~n
# sync_binlog=0：当事务提交之后，MySQL不做fsync之类的磁盘同步指令刷新binlog_cache中的信息到磁盘，而让Filesystem自行决定什么时候来做同步，或者cache满了之后才同步到磁盘。这个是性能最好的。
# sync_binlog=1：当每进行1次事务提交之后，MySQL将进行一次fsync之类的磁盘同步指令来将binlog_cache中的数据强制写入磁盘。
# sync_binlog=n：当每进行n次事务提交之后，MySQL将进行一次fsync之类的磁盘同步指令来将binlog_cache中的数据强制写入磁盘。
# 大多数情况下，对数据的一致性并没有很严格的要求，所以并不会把 sync_binlog 配置成 1. 为了追求高并发，提升性能，可以设置为 100 或直接用 0。
sync_binlog=1

[mysqldump]
quick

# 服务器发送和接收的最大包长度
max_allowed_packet = 32M 

[myisamchk]
key_buffer_size = 8M
sort_buffer_size = 8M
read_buffer = 4M
write_buffer = 4M
```
