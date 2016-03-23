---
layout: post
category: "phpext"
title: "php读取php.ini"
tags: ["phpext"]
---

### 1.需要实现的细节

* 在php.ini加上
* [initest] 
* initest.username=test
* nitest.userage=99
* 
* 实现一个initest类 ，实现一个ini_echo方法，打印相关内容

### 2.initest扩展

#### 2.1创建类的扩展：

[root@bogon ext]# cd /usr/local/src/php-7.0.3/ext

[root@bogon ext]# ./ext_skel --extname=initest

#### 2.2 修改配置
[root@bogon ext]# vim initest/config.m4

	dnl PHP_ARG_WITH(initest, for initest support,
	dnl Make sure that the comment is aligned:
	dnl [  --with-initest             Include initest support])
	更改为：
	PHP_ARG_WITH(initest, for initest support,
	dnl Make sure that the comment is aligned:
	[  --with-initest             Include initest support])


#### 2.3 实现代码
在php_initest.h中打开全局变量设置块的注释，改成自己相要的变量如下

```c

ZEND_BEGIN_MODULE_GLOBALS(initest)
	zend_long  userage;
	char	  *username;
ZEND_END_MODULE_GLOBALS(initest)

```

在initest.c打开相关注释，并添加相关代码

```c

ZEND_DECLARE_MODULE_GLOBALS(initest)

PHP_INI_BEGIN()
    STD_PHP_INI_ENTRY("initest.userage","1", PHP_INI_ALL, OnUpdateLong, userage, zend_initest_globals, initest_globals)
    STD_PHP_INI_ENTRY("initest.username","username", PHP_INI_ALL, OnUpdateString, username, zend_initest_globals, initest_globals)
PHP_INI_END()


PHP_MINIT_FUNCTION(initest)
{

	REGISTER_INI_ENTRIES();
	return SUCCESS;
}


PHP_MSHUTDOWN_FUNCTION(initest)
{
	/* uncomment this line if you have INI entries
	UNREGISTER_INI_ENTRIES();
	*/

	UNREGISTER_INI_ENTRIES();
	return SUCCESS;
}

static void php_initest_init_globals(zend_initest_globals *initest_globals)
{
	//initest_globals->userage = 1;
	//initest_globals->username = "testusername";
}

PHP_FUNCTION(ini_echo)
{
	    php_printf("username:%s\n",INITEST_G(username));
	    php_printf("userage:%d\n",INITEST_G(userage));
}



const zend_function_entry initest_functions[] = {
	PHP_FE(confirm_initest_compiled,	NULL)		/* For testing, remove later. */
	PHP_FE(ini_echo,	NULL)		/* For testing, remove later. */
	PHP_FE_END	/* Must be the last line in initest_functions[] */
};

```


#### 2.4 编译
	* [root@bogon hello]# [root@localhost person]# ./configure && make && make install


#### 2.5 扩展安装
	[initest]
	initest.userage=99
	initest.username=test
	
	extension=initest.so



##### 2.6 扩展使用

```shell
[root@bogon tests]# cat test.php
<?php

ini_echo();

[root@bogon tests]# php test.php
username:test
userage:99
```
```

- 请尊重本人劳动成功，可以随意转载但保留以下信息 
- 作者：岁月经年 
- 时间：2016年03月
