---
layout: post
category: "ant"
title: "MySQLy调优过程"
tags: ["云服务器管理"]
---
###介绍：
* 广告项目每天要处理大量数据，就想着把mysql调优一下。
* 首先需要大致了解一下mysql日志操作步骤：
* log_buff ---mysql写 (write)---> log_file ---OS刷新 (flush)---> disk

* innodb_flush_log_at_trx_commit 参数解释：
	* 0（延迟写）： log_buff  --每隔1秒--> log_file  —实时—> disk
	* 1（实时写，实时刷）： log_buff  —实时—>  log_file  —实时—> disk
	* 2（实时写，延迟刷）： log_buff  —实时—> log_file --每隔1秒--> disk


###系统指令硬盘测速：
  * 系统指令硬盘测速
  	* dd if=/dev/zero of=hello.txt bs=100M count=1  
  	* 使用同步I/O，每次写都要物理写入磁盘（磁盘会狂响的^_^），巨慢
 	 * dd if=/dev/zero of=/data/test2 bs=4k count=1000 oflag=nonblock,sync


###sar指令监控服务器：
  * yum -y install sysstat
 	 * sar常用选项
	 * -b：报告I/O使用情况以及传输速率。（只适用于2.5及之前的内核，所以新内核有可能不支持这个选项）
	 * -c：报告进程创建情况
	 * -d：报告每一个块设备的使用情况
     * -q：汇报队列长度和负载信息
     * -R：汇报内存情况
	 * -u：汇报CPU使用情况
	 * 可以直接指定进程ID号；


 * CPU利用率查看
	* sar -p （默认查看为当天）
	* sar -u 1 5 （每隔一秒写入5次）


	* 输出项说明：
	* CPU all 表示统计信息为所有 CPU 的平均值。
	* %user 显示在用户级别(application)运行使用 CPU 总时间的百分比。
	* %nice 显示在用户级别，用于nice操作，所占用 CPU 总时间的百分比。
	* %system 在核心级别(kernel)运行所使用 CPU 总时间的百分比。
	* %iowait 显示用于等待I/O操作占用 CPU 总时间的百分比。
	* %steal 管理程序(hypervisor)为另一个虚拟进程提供服务而等待虚拟 CPU 的百分比。
	* %idle 显示 CPU 空闲时间占用 CPU 总时间的百分比。

	* 1. 若 %iowait 的值过高，表示硬盘存在I/O瓶颈
	* 2. 若 %idle 的值高但系统响应慢时，有可能是 CPU 等待分配内存，此时应加大内存容量
	* 3. 若 %idle 的值持续低于 10，则系统的 CPU 处理能力相对较低，表明系统中最需要解决的资源是 CPU。

* 内存利用率查看
	* sar -p 1 10
	* vmstat 1 10

	* 可用内存=free+buffers+cached 已用内存：userd-buffers-cached
	

* 磁盘I/O使用情况查看
	* iostat -x 1 10
	* 等同于sar -d 1 10

	* await表示平均每次设备I/O操作的等待时间（以毫秒为单位）。 
	* svctm表示平均每次设备I/O操作的服务时间（以毫秒为单位）。
	* %util表示一秒中有百分之几的时间用于I/O操作。 
	* 对以磁盘IO性能，一般有如下评判标准：
	* 正常情况下svctm应该是小于await值的，而svctm的大小和磁盘性能有关，CPU、内存的负荷也会对svctm值造成影响，过多的请求也会间接的导致svctm值的增加。
    * await值的大小一般取决与svctm的值和I/O队列长度以及I/O请求模式，如果svctm的值与await很接近，表示几乎没有I/O等待，磁盘性能很好，如果await的值远高于svctm的值，则表示I/O队列等待太长，系统上运行的应用程序将变慢，此时可以通过更换更快的硬盘来解决问题。
	* %util项的值也是衡量磁盘I/O的一个重要指标，如果%util接近100%，表示磁盘产生的I/O请求太多，I/O系统已经满负荷的在工作，该磁盘可能存在瓶颈。长期下去，势必影响系统的性能，可以通过优化程序或者通过更换更高、更快的磁盘来解决此问题。


* 网络流量情况分析
	* sar -n DEV 1 2

	* sar命令使用-n选项可以汇报网络相关信息，可用的参数包括：DEV、EDEV、SOCK和FULL。
	* IFACE：就是网络设备的名称；
	* rxpck/s：每秒钟接收到的包数目
	* txpck/s：每秒钟发送出去的包数目
	* rxbyt/s：每秒钟接收到的字节数
	* txbyt/s：每秒钟发送出去的字节数
	* rxcmp/s：每秒钟接收到的压缩包数目
	* txcmp/s：每秒钟发送出去的压缩包数目
	* txmcst/s：每秒钟接收到的多播包的包数目
	
	
###MySQL压力测试工具：mysqlslap
* mysqlslap 可以用于模拟服务器的负载，并输出计时信息。其被包含在 MySQL 5.1 的发行包中。测试时，可以指定并发连接数，可以指定 SQL 语句。如果没有指定 SQL 语句，mysqlslap 会自动生成查询 schema 的 SELECT 语句。

* [root@Betty libmysql]# mysqlslap --help
   * mysqlslap  Ver 1.0 Distrib 5.6.10, for Linux (x86_64)
Copyright (c) 2005, 2013, Oracle and/or its affiliates. All rights reserved.
 
   * Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.
 
   * Run a query multiple times against the server.
   * Usage: mysqlslap [OPTIONS]
 
   * Default options are read from the following files in the given order:
/etc/my.cnf /etc/mysql/my.cnf /usr/local/mysql/etc/my.cnf ~/.my.cnf 
The following groups are read: mysqlslap client
The following options may be given as the first argument:
   * --print-defaults        Print the program argument list and exit.
   * --no-defaults           Don't read default options from any option file,
                        except for login file.
   * --defaults-file=#       Only read default options from the given file #.
   * --defaults-extra-file=# Read this file after the global files are read.
   * --defaults-group-suffix=#
                        Also read groups with concat(group, suffix)
   * --login-path=#          Read this path from the login file.
   *   -?, --help          Display this help and exit.
   *   -a, --auto-generate-sql 自动生成测试表和数据  Generate SQL where not supplied by file or command line.
   *  --auto-generate-sql-add-autoincrement 增加auto_increment一列
                      Add an AUTO_INCREMENT column to auto-generated tables.
   * --auto-generate-sql-execute-number=# 自动生成的查询的个数
                      Set this number to generate a set number of queries to
                      run.
   *   --auto-generate-sql-guid-primary 增加基于GUID的主键
                      Add GUID based primary keys to auto-generated tables.
   *   --auto-generate-sql-load-type=name 测试语句的类型。取值包括：read，key，write，update和mixed(默认) read:查询 write:插入 key:读主键 update:更新主键 mixed:一半插入一半查询 
  
   *   --auto-generate-sql-secondary-indexes=# 增加二级索引的个数，默认是0
                      Number of secondary indexes to add to auto-generated
                      tables.
   *  --auto-generate-sql-unique-query-number=# 不同查询的数量，默认值是10
                      Number of unique queries to generate for automatic tests.
   *   --auto-generate-sql-unique-write-number=# 不同插入的数量，默认是100
                      Number of unique queries to generate for
                      auto-generate-sql-write-number.
   *   --auto-generate-sql-write-number=# 
                      Number of row inserts to perform for each thread (default
                      is 100).
   *   --commit=#          多少条DML后提交一次
                      Commit records every X number of statements.
   *   -C, --compress      如果服务器和客户端支持都压缩，则压缩信息传递
                      Use compression in server/client protocol.
   *   -c, --concurrency=name 模拟N个客户端并发执行select。可指定多个值，以逗号或者 --delimiter 参数指定的值做为分隔符
                      Number of clients to simulate for query to run.
   *   --create=name       指定用于创建表的.sql文件或者字串
                     
   *   --create-schema=name 指定待测试的数据库名，MySQL中schema也就是database，默认是mysqlslap
                      Schema to run tests in.
   *   --csv[=name]        Generate CSV output to named file or to stdout if no file
                      is named.
   *   -#, --debug[=#]     This is a non-debug version. Catch this and exit.
   *   --debug-check       Check memory and open file usage at exit.
   *   -T, --debug-info    打印内存和CPU的信息
                      Print some debug info at exit.
   *   --default-auth=name Default authentication client-side plugin to use.
   *   -F, --delimiter=name 文件中的SQL语句使用分割符号
                      Delimiter to use in SQL statements supplied in file or
                      command line.
   *   --detach=#          每执行完N个语句，先断开再重新打开连接
                     
   *   --enable-cleartext-plugin 
                      Enable/disable the clear text authentication plugin.
   *  -e, --engine=name   创建测试表所使用的存储引擎，可指定多个
                      
   *  -h, --host=name     Connect to host.
   *   -i, --iterations=#  迭代执行的次数
                      
   *   --no-drop           Do not drop the schema after the test.
   *   -x, --number-char-cols=name 自动生成的测试表中包含多少个字符类型的列，默认1
                     
   *   -y, --number-int-cols=name 自动生成的测试表中包含多少个数字类型的列，默认1
                   
   *   --number-of-queries=# 总的测试查询次数(并发客户数×每客户查询次数)
                     
   *   --only-print        只输出模拟执行的结果，不实际执行
                     
   *   -p, --password[=name] 
                      Password to use when connecting to server. If password is
                      not given it's asked from the tty.
   *   --plugin-dir=name   Directory for client-side plugins.
   *   -P, --port=#        Port number to use for connection.
   *   --post-query=name   测试完成以后执行的SQL语句的文件或者字符串 这个过程不影响时间计算
                     
   *   --post-system=name  测试完成以后执行的系统语句 这个过程不影响时间计算
                      
   *  --pre-query=name    测试执行之前执行的SQL语句的文件或者字符串 这个过程不影响时间计算
                     
   *   --pre-system=name   测试执行之前执行的系统语句 这个过程不影响时间计算
                      
   *   --protocol=name     The protocol to use for connection (tcp, socket, pipe,
                      memory).
   *   -q, --query=name    指定自定义.sql脚本执行测试。例如可以调用自定义的一个存储过程或者sql语句来执行测试
   *   -s, --silent        不输出
                      Run program in silent mode - no output.
   *   -S, --socket=name   The socket file to use for connection.
   *  --ssl               Enable SSL for connection (automatically enabled with
                      other flags).
   *   --ssl-ca=name       CA file in PEM format (check OpenSSL docs, implies
                      --ssl).
   *   --ssl-capath=name   CA directory (check OpenSSL docs, implies --ssl).
   *  --ssl-cert=name     X509 cert in PEM format (implies --ssl).
   *   --ssl-cipher=name   SSL cipher to use (implies --ssl).
   *  --ssl-key=name      X509 key in PEM format (implies --ssl).
   *  --ssl-crl=name      Certificate revocation list (implies --ssl).
   *   --ssl-crlpath=name  Certificate revocation list path (implies --ssl).
   *   --ssl-verify-server-cert 
                      Verify server's "Common Name" in its cert against
                      hostname used when connecting. This option is disabled by
                      default.
   *   -u, --user=name     User for login if not current user.
   *   -v, --verbose       输出更多的信息
                      More verbose output; you can use this multiple times to
                      get even more verbose output.
   *   -V, --version       Output version information and exit.



*  测试样例：
    * 自动生成测试表和数据的形式，分别模拟 50 和 100 个客户端并发连接处理 1000 个 query 的情况，重复执行 5 次，并输出内存和CPU信息
	*  mysqlslap -a --concurrency=50,100 --number-of-queries=1000 --iterations=5 --debug-info 
	* 连主机上 mysql输入用户名密码 进行测试。
	* mysqlslap -a --concurrency=50,100 --number-of-queries=1000 -h 172.16.81.99 -P 3306 -p
	* 实际测试中的复杂情况。
	* 使用 --defaults-file 选项，指定从配置文件中读取选项配置。
	* 使用 --number-int-cols 选项，指定表中会包含 4 个 int 型的列。
	* 使用 --number-char-cols 选项，指定表中会包含 35 个 char 型的列。
	* 使用 --engine 选项，指定针对何种存储引擎进行测试。
	*  mysqlslap --defaults-file=/etc/my.cnf --concurrency=50,100,200 --iterations=1 --number-int-cols=4 --number-char-cols=35 --auto-generate-sql --auto-generate-sql-add-autoincrement --auto-generate-sql-load-type=mixed --engine=myisam,innodb --number-of-queries=200 --debug-info -S /tmp/mysql.sock
	
	* 提供你自己的创建 SQL 语句和查询 SQL 语句，有 50 个客户端查询，每个查询 200 次（在单行上输入命令）
	* mysqlslap --delimiter=";" --create="CREATE TABLE a (b int);INSERT INTO a VALUES (23)" --query="SELECT * FROM a" --concurrency=50 --iterations=200

	* 让 mysqlslap 创建查询的 SQL 语句，使用的表有 2 个 INT 行和 3 个 VARCHAR 行。使用 5 个客户端，每一个查询 20 次！不要创建表或插入数据。(换言之，用之前测试的模式和数据)
    * mysqlslap --concurrency=5 --iterations=20 --number-int-cols=2 --number-char-cols=3 --auto-generate-sql
    * 告诉程序从指定的 create.sql 文件去加载 create，insert 和查询等 SQL 语句。该文件有多个表的 create 和insert 语句, 它们都是通过“；”分隔开的。query.sql 文件则有 多个查询语句, 分隔符也是“；”。执行所有的加载语句，然后通过查询文件执行所有的查询语句，分别在 5 个客户端上每个执行 5 次：

    * [root@Betty ~]# mysqlslap --concurrency=5 --iterations=5 --query=query.sql --create=create.sql --delimiter=";"

###参考内容：

* [http://my.oschina.net/moooofly/blog/152584](http://my.oschina.net/moooofly/blog/152584)
* [http://www.ha97.com/5182.html](http://www.ha97.com/5182.html)
* [http://www.open-open.com/lib/view/open1441070898346.html](http://www.open-open.com/lib/view/open1441070898346.html)
* [http://dev.mysql.com/doc/refman/5.1/en/mysqlslap.html](http://dev.mysql.com/doc/refman/5.1/en/mysqlslap.html)
* [http://blog.csdn.net/shiningzhang/article/details/21725231](http://blog.csdn.net/shiningzhang/article/details/21725231)
* [http://blog.sina.com.cn/s/blog_71d9aee40101an43.html](http://blog.sina.com.cn/s/blog_71d9aee40101an43.html)






  
>
- 请尊重本人劳动成功，可以随意转载但保留以下信息 
- 作者：岁月经年 
- 时间：2015年6月
- 
