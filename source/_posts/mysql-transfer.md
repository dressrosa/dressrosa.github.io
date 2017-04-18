---
title: mysql和转账的实现
date: 2017-04-18 16:00:32
categories: mysql
tags: 
	- mysql
---

做个平台间余额的互相转账,重点在于如何解决并发访问数据错乱的问题.比如俩个人之间同时都在给对方转账.以及很多人同时给一个人转账.  

必须保证数据库里面的数据是随时正确的.
我本没有想到过这一层面,被提醒的.
比如有个转账的方法transfer(业务逻辑主要写在这里) 
<!--more-->
1.首先想到的用多线程加锁,看了资料.是可以实现,但是问题是对方法加锁或者代码块加锁,那么每次只能执行一个线程,a转给b执行了,可c却不能同时转给d,二者根本不会影响啊.所以多线程白看了. 

2.然后就取决在数据库上面了.用的是mysql innodb,所以找了这方面锁的知识.
这篇博客写的不错,感谢:[http://www.cnblogs.com/Arlen/articles/1756616.html](http://www.cnblogs.com/Arlen/articles/1756616.html) 

结合其他的参考学习,我总结了一下: 

mysql innodb主要有表级锁和行级锁,转账之间不能互相影响,所以选择行级锁. 

行级锁有两种:共享锁和排他锁 

如何加锁:
- 共享锁:select xxxx lock in share mode 
- 排他锁:select xxx for update 
                
顾名思义共享锁的重点在于多个事务可以同时共享同一资源上的共享锁,如果一个事务获取了一个数据的共享锁,那么别的事务也可以再获取这个锁.获取共享锁的事务可以对数据进行读操作.,读完了就释放. 

排他锁就是一旦事务获取了这个锁,他就必须持有这个锁到结束,并不准其他事务获取这个锁.持有排他锁的事务可以对数据进行写操作.
 
 
然后明确一点一个事务在一个时刻里只能拥有一把锁

在转账时如果我先查找用户的账户信息,如果我用共享锁的话,别的事务也可以共享这个锁,这时候就会产生死锁. 

这里的死锁产生的原因在于: 

当A向B转账的时候,B也在向C转账,A开始转账的时候获取A,B账户信息,A开始转账的时候获取C,B账户信息,这里是共享锁,然后我需要对账户进行更新,这时候问题就出现了; 

假如A转账在准备更新B的信息时,发现B转账也准备更新B,A就等B释放共享锁,B也等A在释放共享锁,也就产生死锁了. 



如果没有,那么在查询完后,A事务就释放了共享锁,然后这时候别的B事务就可以获取这个用户账户信息的共享锁, 

假如B获取了.那A了刚刚查出来了用户信息,想改的时候(mysql修改默认加上排他锁)发现资源又被锁了.给别人挪用了.别人对他干了啥我哪知道,以后数据就肯能和我刚读出来的不一样了. 


排它锁事务一旦获取了就一直到事务结束都会拥有这把锁,这样就很好的杜绝了数据被修改的可能性.那么这时候重点就在于这里的事务怎么开始和啥时候结束了.我的项目用的是springMVC,就是spring的事务是怎么确定的. 

这篇博客写的不错,感谢:   [http://www.cnblogs.com/davidwang456/p/3832949.html](http://www.cnblogs.com/davidwang456/p/3832949.html) 

这里不需要知道spring事务的深层原理,只需知道spring事务是在进入sercice层执行方法的时候开始定义一个事务,然后在这个方法结束后事务才结束(事务提交).就是说在这个方法里面不管调用了多少个dao方法,都不会影响锁的丢失.也就是说我进入方法查询用户加了排他锁,在这个方法结束之前该数据就是安全的.

好了理论说完了,看程序了:
这是我的service方法

```
@Transactional(readOnly=false)
    public int transfer(String userId, String money, String remarks,
            String otherId) {
        HcUser me = this.userDao.getUserForUpdate(userId);//获取用户信息
        HcUser other = this.userDao.getUserForUpdate(otherId);//获取用户信息
      //这里写转账的一些操作(修改/插入等)
        return 0;
    }
```

这是mybatis的sql:

```
<select id="getUserForUpdate" parameterType="java.lang.String" resultType="HcUser">
    select * 
    from hc_user 
    where id=#{id}
    for update <!-- 这里加锁-->
</select>
```


写的是web项目,需要在浏览器敲地址测试,但是无法实现多线程,于是可以这样:
用java写访问url的程序(网上找的拉):

```
package examples;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.net.HttpURLConnection;
import java.net.URL;

import org.apache.commons.lang3.RandomUtils;

public class TEST {

    public static void main(String args[]) {
        for(int i = 0;i <400; i++) {//设置400个线程
            String money =RandomUtils.nextInt(1, 5)+"";
            System.out.println(money);
            //模拟几个用户
            String userId = "2aefe0c4c88f4ea1b39a68b6e33208d8";
            String otherId="1bb4fc7d35c342a3a19bc23171aa3d22";
            String userId1="1b638bd7dbb445a3bfe1efbec1a61e4e";
            String userId2="551926783cb64524b3127ec02529d371";
            Account a = null;
        //设置不同的转账
            if("1".equals(money)) {
                 a = new Account(money,userId,otherId);
            }
            else if("2".equals(money)) {
                 a = new Account(money,userId1,userId);
            }else if("3".equals(money)) {
                 a = new Account(money,userId2,userId);
            }
            else {
                 a = new Account(money,userId1,otherId);
            }
            Thread t = new Thread(a);
            t.start();
        }
    }
    static class Account implements Runnable {
        String money = "0";
        String userId = "";
        String otherId="";
        public Account(String money,String userId,String otherId) {
            this.money = money;
            this.otherId = otherId;
            this.userId = userId;
        }

        @Override
        public void run() {
            long begintime = System.currentTimeMillis();
              try {
               URL url = new URL("http:xxx?userId="
                       +userId+"&otherId="+otherId+"&remarks=xxx"
                       +"&money="+money);
               HttpURLConnection urlcon = (HttpURLConnection)url.openConnection();
               urlcon.connect();         //获取连接
               InputStream is = urlcon.getInputStream();
               BufferedReader buffer = new BufferedReader(new InputStreamReader(is));
               StringBuffer bs = new StringBuffer();
               String l = null;
               while((l=buffer.readLine())!=null){
                   bs.append(l).append("/n");
               }
               System.out.println(bs.toString());//打印出信息

               System.out.println("总共执行时间为："+(System.currentTimeMillis()-begintime)+"毫秒");
            }catch(IOException e){
               System.out.println(e);
           }
        }
}
}
```

这样写的程序可能出现有两个线程同时进行A给B转账,请先不要在意这种情况.
这是我数据库账户初始值,每人设置100:

1,在不加锁的情况下,跑完后:

总额变小了扇
2,加了共享锁,跑完后:

总额是对的,但是报错了:
Deadlock found when trying to get lock; try restarting transaction;
产生死锁了,但是mysql会自己强制性重启事务,所以不会一直死锁下去,程序之后还会运行.
3加了排他锁后:

没有错误,正确运行

这样就可以保证转账正确运行了,