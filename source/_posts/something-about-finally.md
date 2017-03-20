---
title: finally小结
date: 2017-03-19 16:00:38
categories: "java"
tags: 
	-finally
---


## 记录一下finally的一些简单知识点.

---
1.  正常情况下finally都会执行,且我们最好不要在finally里面执行业务逻辑,一般用来对资源的释放
2. finally虽说是肯定会执行的,但是有些情况下就不会执行
<!--more-->
```
Thread t = new Thread(new Runnable() {
			@Override
			public void run() {
				try {
					System.out.println("abc");
					System.exit(0);//直接结束程序 类似于关闭电源
				} catch (Exception e) {
					e.printStackTrace();
				} finally {
					System.out.println("finally");
				}
			}
		});
		t.start();
```
从学习中我们知道,如果没有一个用户线程(即非守护线程)在运行,jvm就会自动退出, 当t线程start()后,进入main线程(非daemon线程),main线程运行完毕后就没有用户线程了,所有jvm退出,导致finally不能运行

```
public static void main(String args[]) throws Exception {
		Thread t = new Thread(new Runnable() {
			@Override
			public void run() {
				try {
					System.out.println("abc");
					Thread.sleep(1000);
				} catch (Exception e) {
					e.printStackTrace();
				} finally {
					System.out.println("finally");
				}
			}
		});
		t.setDaemon(true);//设置为守护线程
		t.start();
		System.out.println("aaa");
	}
```



3.eclipse用debug模式断点到try的里面,然后鼠标点强制关闭线程 ,finally也不会运行(相当于断电),呵呵

所以,finally理论上是总会运行,但是再怎么理论也需要时间和机会让他运行,就如病毒再强,我把电源拔了,你还有啥用.