---
layout: post
category: "python"
title: "python培训[5]－ splinter自动登录微博和qq空间"
tags: ["python培训"]
---
* Splinter 可以通过api自动模拟用户行为，可以利用Splinter开发浏览器自动化操作。
* Splinter:  <https://github.com/cobrateam/splinter>
- 功能：
	- simple api 接口简单
	- multi webdrivers (chrome webdriver, firefox webdriver, phantomjs webdriver, zopetestbrowser, - remote webdriver) 多个浏览器驱动
	- css and xpath selectors css和xpath作为选择器
	- support to iframe and alert 支持iframe和alert
	- execute javascript 可执行javascript
	- works with ajax and async javascript 支持javascript的ajax的异步操作


#### 1.扩展安装：
* pip install splinter
* sudo pip install selenium
* sudo pip install mozmill
* 要求python2.7+ 
* 默认使用firefox浏览器，如果需要使用chrom且在mac的情况下可用 brew install chromedriver 安装，不过请使用代理，要不download会失败。


#### 2.登录微博代码：

```
#!/usr/bin/python
# coding=utf-8

"""
利用splinter模块模拟发布微博，仅为QA讲例测试
"""

from splinter.browser import Browser
import random
import time

web_firefox = Browser('firefox')
web_firefox.visit('http://weibo.com/login.php')

if web_firefox.find_by_name('username'):
    print 'uname'
    # web_firefox.find_by_name('username').click()
    # 这个地方用by_name能找到，但是模拟点击时有问题，改成by_css
    web_firefox.find_by_css(
        "input[node-type=\"username\"]").fill('xxxx@sina.com')
time.sleep(random.randint(3, 10))

if web_firefox.find_by_name('password'):
    print 'passwd'
    web_firefox.find_by_css("input[node-type=\"password\"]").fill('******')
    
time.sleep(random.randint(3, 15))
print web_firefox.find_by_css(".loginbox .W_login_form .login_btn div a ")[0].click()

time.sleep(random.randint(3, 10))
print web_firefox.find_by_css(".input .W_input").fill(u'我说不让你用微博测试，你非用，得了吧。。。。 封ip了吧')

time.sleep(random.randint(3, 10))
print web_firefox.find_by_css("a[node-type=\"submit\"]").click()

```
* 效果：<http://pan.baidu.com/s/1pJA3QyR>

#### 3.登录QQ空间代码：

```
#!/usr/bin/env python
# coding=utf-8

import time
from splinter import Browser


def splinter(url, q, p):
    # browser=Browser('chrome')
    #browser = Browser('webdriver.chrome')
    browser = Browser('firefox')
    browser.visit(url)
    time.sleep(5)
    #fill in account and password
    if browser.find_by_id('login_frame'):
        with browser.get_iframe('login_frame') as frame:
            frame.find_by_id('switcher_plogin').click()
            print '输入账号...'
            frame.find_by_id('u').fill(q)
            print '输入密码...'
            frame.find_by_id('p').fill(p)
            print '尝试登录...'
            frame.find_by_id('login_button').click()
            print '完成登录动作...'

    browser.find_by_id('aMyFriends').click()
    time.sleep(3)

if __name__ == '__main__':
    website = 'http://qzone.qq.com'
    qq = 'qq号'
    pwd = '***********'
    splinter(website, qq, pwd)
```
* 效果：<http://pan.baidu.com/s/1hqfAaX2>
>
- 请尊重本人劳动成功，可以随意转载但保留以下信息 
- 作者：岁月经年 
- 时间：2015年4月
