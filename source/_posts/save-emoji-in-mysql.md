---
title: 怎么将emoji表情存入mysql
date: 2017-04-18 15:52:47
categories: mysql
tags: 
	- mysql
---
 
1. mysql的数据库, 表 ,字段的字符集必须是utf8mb4,怎么设置自酌,
但可以看看自己数据库的字符集 show variables like '%set%
保证 Character set是utf8mb4 -- UTF-8 Unicode 
collation是utf8mb4_unicode_ci就可以了,
utf8和utf8mb4不冲突的,所以不用考虑改变后对其他地方有影响
<!--more-->
2.utf8mb4必须是新版的mysql才支持,网上虽说5.5+就可以了,但是没有具体到几点几,最好还是mysql官网下载最新的版本,
具体的安装卸载过程自酌

3.emoji表情不是图片,而是一种4字节的字符,属于unicode,以前的mysql utf8只能解析到3字节,不要想着怎么做图片替换的问题.

4.确定所有的字符都修改完毕后,在存入数据库之前代码做一下调整就行:我用的springMVC+mybatis ,
所以再需要输入表情的mybatis xml 里面加一个


```
<update id="predo">
set names utfmb4
</update>
```

然后再调用你需要插入表情的语句,
我在service这样调用: 


```
this.dao.predo(); 
this.dao.insert(feedBack);
```


这样就会插入表情到数据库,但是数据库显示的问号,不过没关系,app里面显示出来的的确是表情.
其实就是每次调用之前先执行一下set names utfmb4

我是在之前的数据库改到utfmb4的也许从一开始就设置好也许就不用这样处理了.

5.也许你没成功,也许别的方法或者设置也是可以的,请选择你认为简单的.但是原因都是因为mysql无法存入的问题,从这方面解决就行了