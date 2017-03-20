---
title: interrupt简单认识
date: 2017-03-20 00:06:43
categories: "java"
tags: 
	-interrupt
---
interrupt意思为中断,为线程运行时提供一个中断机制.
但是中断和阻塞是不一样的,线程的中断并没有结束或暂停线程,而是对线程设置了一个中断的信号,通过此信号来进行具体的其他操作.

<!--more-->

在Thread里面提供了三个方法:
1. interrupt() 对一个线程进行中断操作,就需要调用此方法,设置中断信号
2. interrupted() 判断线程是否设置过中断信号,返回true/false,同时清除1中方法设置的信号,所以连续俩次调用此方法后,第二次会返回false
3. isInterrupted() 判断线程是否设置过中断信号,但并没有清除信号.


所以当一个线程调用interrupt()的时候,其实就是设置了一个中断信号,然后来让线程知道有中断发生,好让其决定怎么去回应这个信号.

1.所以interrupt适合用在join sleep wait这些阻塞型的方法,从而达到提前结束阻塞的目的.原因是 这种阻塞类的操作,在阻塞时调用interrupt()设置中断信号,会导致这些操作抛出interruptException,这样就能提前结束阻塞.

2.在不是这些阻塞情况时,我们可以设置一个votatile变量,来表示结束状态,
然后我们调用interrupt()来发出信号,告诉程序后续处理.

```
class B extends Thread {
	private volatile boolean stop = false;

	@Override
	public void run() {
		int i = 0;
		while (!stop) {
			for (int a = 0; a < 10000000; a++)
				for (int b = 0; b < 5000000; b++) {

				}
			System.out.println("吃饭中.." + i++);
		}

	}

	public void stoped() {
		stop = true;
		this.interrupt();//设置中断信号
	}
}
```


我们在线程运行后可以这样:

```
t2.start();
Thread.sleep(1000);
t.stoped();
if (t.isInterrupted()) {
	throw new InterruptedException();
}
```

通过判断线程是否中断过,主动抛出中断异常,来让更上层的程序进行后续处理.

---

[练习的代码参见github](https://github.com/dressrosa/dailydemo)