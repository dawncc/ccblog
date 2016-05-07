---
layout: page
title: JAVA线程
permalink: /JAVA线程/
---
## 1 JAVA线程: 创建与启动
###1.1 继承java.lang.Thread类
```
java.lang.Object
  |-- java.lang.Thread
```
创建线程：
``` java
 class PrimeThread extends Thread {
         long minPrime;
         PrimeThread(long minPrime) {
             this.minPrime = minPrime;
         }
 
         public void run() {
             // compute primes larger than minPrime
              . . .
         }
     }

```
启用线程
``` java
PrimeThread p = new PrimeThread(143);
     p.start();
```
###1.2 实现java.lang.Runnable接口。
*方法摘要 *
```
 void run() 
          使用实现接口 Runnable 的对象创建一个线程时，启动该线程将导致在独立执行的线程中调用对象的 run 方法。 
```
创建线程：
```java
class PrimeRun implements Runnable {
         long minPrime;
         PrimeRun(long minPrime) {
             this.minPrime = minPrime;
         }
 
         public void run() {
             // compute primes larger than minPrime
              . . .
         }
     }
```
使用线程
``` java
PrimeRun p = new PrimeRun(143);
     new Thread(p).start();
```

## 2. 线程状态
java.lang 
枚举 Thread.State
java.lang.Object
  |-- java.lang.Enum<Thread.State>

### 2.1 枚举常量摘要
BLOCKED  
          受阻塞并且正在等待监视器锁的某一线程的线程状态。 
NEW 
          至今尚未启动的线程的状态。 
RUNNABLE 
          可运行线程的线程状态。 
TERMINATED 
          已终止线程的线程状态。 
TIMED_WAITING 
          具有指定等待时间的某一等待线程的线程状态。 
WAITING 
          某一等待线程的线程状态。 
### 2.2 状态转换
![线程状态如图](https://raw.githubusercontent.com/cc2chblog/keepblog/master/images/%E7%BA%BF%E7%A8%8B%E7%8A%B6%E6%80%81%E5%9B%BE.jpg)

## 3. 终止线程
###3.1 Thread.stop()***（废弃方法）***
Thread.stop()方法在结束线程时，会直接终止线程，并且会立即释放这个线程所有的锁。
>**结果：**导致数据不一致

```java
package com.cc.training.thread;

/**
 * 用于测试使用Thread.stop()方法终止线程导致的数据不一致问题
 * 
 * @author cc
 *
 */
public class StopThreadUnsafeTest {
	/**
	 * 定义共享的user用于全局调用，下面两个测试线程都可以使用当前user
	 */
	private static User user = new User();

	/**
	 * 声明一个user，包括uername和id 两个线程同时操作更新这个user的数据
	 */
	public static class User {
		private int id;
		private String username;

		public User() {
			setId(0);
			setUsername("0");
		}

		@Override
		public String toString() {
			// TODO Auto-generated method stub
			return "User {id=" + getId() + ", username=" + getUsername() + "}";
		}

		public int getId() {
			return id;
		}

		public void setId(int id) {
			this.id = id;
		}

		public String getUsername() {
			return username;
		}

		public void setUsername(String username) {
			this.username = username;
		}
	}

	/**
	 * 使用extends Thread的模式创建一个线程 该线程用于设置user的属性
	 * 
	 * @author cc
	 *
	 */
	public static class ChangeObjectThread extends Thread {

		@Override
		public void run() {
			// TODO Auto-generated method stub
			
			while (true) {
				synchronized (user) {
					//在当前进程中使用同一变量，分别赋值给id和user 以用于检查数据不一致性
					int v = (int) (System.currentTimeMillis() / 1000);
					user.setId(v);
					try {
						Thread.sleep(1000);
					} catch (InterruptedException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
					user.setUsername(String.valueOf(v));
				}
				// 暂停当前正在执行的线程对象，并执行其他线程。
				Thread.yield();
			}
		}
	}
	
	/**
	 * 使用extends Thread的模式创建另一个线程 该线程用于读取user的属性
	 * @author cc
	 */
	public static class ReadObjectThread extends Thread {
		/**
		 * 当user下的id和name不一致的时候，读出user
		 */
		@Override
		public void run() {
			System.out.println("Read Thread Start!");
			// TODO Auto-generated method stub
			while (true) {
				synchronized (user) {
					if(user.getId() != Integer.parseInt(user.getUsername())){
						System.out.println(user.toString());
					}
			}
			// 暂停当前正在执行的线程对象，并执行其他线程。
			Thread.yield();
		}
	  }
	}

	public static void main(String[] args) throws InterruptedException {
		new ReadObjectThread().start();
		//每隔1.5秒，启动一个changeOjectThread的线程
		while(true){
			Thread t  = new ChangeObjectThread();
			t.start();
			Thread.sleep(1500);
			t.stop();
		}
	}
}

```
**执行结果：**

![线程数据不一致结果图](https://raw.githubusercontent.com/cc2chblog/keepblog/master/images/image.jpg)

###3.2 正确终止线程
> 使用标志位，让线程自行结束

```java
package com.cc.training.thread;


/**
 * 使用标志位来正确终止线程
 * 
 * @author cc
 *
 */
public class StopTheadSafeTest {
	/**
	 * 定义共享的user用于全局调用，下面两个测试线程都可以使用当前user
	 */
	private static User user = new User();

	/**
	 * 声明一个user，包括uername和id 两个线程同时操作更新这个user的数据
	 */
	public static class User {
		private int id;
		private String username;

		public User() {
			setId(0);
			setUsername("0");
		}

		@Override
		public String toString() {
			// TODO Auto-generated method stub
			return "User {id=" + getId() + ", username=" + getUsername() + "}";
		}

		public int getId() {
			return id;
		}

		public void setId(int id) {
			this.id = id;
		}

		public String getUsername() {
			return username;
		}

		public void setUsername(String username) {
			this.username = username;
		}
	}

	/**
	 * 使用extends Thread的模式创建一个线程 该线程用于设置user的属性
	 * 
	 * @author cc
	 *
	 */
	public static class ChangeObjectThread extends Thread {
		// 线程终止标志位
		volatile static boolean stopFlag = false;
		/**
		 * 提供给外部调用，用于终止线程
		 */
		public static void stopThread(){
			stopFlag = true;
		}
		@Override
		public void run() {
			// TODO Auto-generated method stub
			
			while (!stopFlag) {
				synchronized (user) {
					//在当前进程中使用同一变量，分别赋值给id和user 以用于检查数据不一致性
					int v = (int) (System.currentTimeMillis() / 1000);
					user.setId(v);
					try {
						Thread.sleep(1000);
					} catch (InterruptedException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
					user.setUsername(String.valueOf(v));
				}
				// 暂停当前正在执行的线程对象，并执行其他线程。
				Thread.yield();
			}
		}
	}
	
	/**
	 * 使用extends Thread的模式创建另一个线程 该线程用于读取user的属性
	 * @author cc
	 */
	public static class ReadObjectThread extends Thread {
		/**
		 * 当user下的id和name不一致的时候，读出user
		 */
		@Override
		public void run() {
			System.out.println("Read Thread Start!");
			// TODO Auto-generated method stub
			while (true) {
				synchronized (user) {
					if(user.getId() != Integer.parseInt(user.getUsername())){
						System.out.println(user.toString());
					}
			}
			// 暂停当前正在执行的线程对象，并执行其他线程。
			Thread.yield();
		}
	  }
	}

	public static void main(String[] args) throws InterruptedException {
		new ReadObjectThread().start();
		//每隔1.5秒，启动一个changeOjectThread的线程
		while(true){
			Thread t  = new ChangeObjectThread();
			t.start();
			Thread.sleep(1500);
			((ChangeObjectThread) t).stopThread();
		}
	}
}

```
## 4. 线程中断
>public void interrupt()   //中断线程
>public boolean isInterrupted() //测试当前线程是否已经中断
>public static boolean interrupted()//测试当前线程是否已经中断，并清除线程的中断状态。

interrupt()方法通知目标线程进行中断，但是并不能直接起到类似stop()的直接中断的作用，它提供了一个中断线程的标志位，然后配合isInterruped的判断才能中断线程。
**以下示例是无法中断线程的**
```java
package com.cc.training.thread;

/**	
 * 线程中断的示例
 * @author cc
 *
 */
public class InterruptThreadTest {

	/**
	 * 直接使用Interrupt方法是无法中断线程的
	 * 它提供了一个中断线程的标志位，然后配合isInterruped的判断才能中断线程。
	 * @param args
	 * @throws InterruptedException 
	 */
	public static void main(String[] args) throws InterruptedException {
		// TODO Auto-generated method stub
		Thread t = new Thread(){
			@Override
			public void run() {
				// TODO Auto-generated method stub
				while(true){
					Thread.yield();
				}
			}
		};
		t.start();
		Thread.sleep(2000);
		t.interrupt();
	}
}
```

**正确中断线程的示例**
```java
package com.cc.training.thread;

/**
 * 线程中断的示例
 * 
 * @author cc
 *
 */
public class InterruptThreadTest {

	/**
	 * 直接使用Interrupt方法是无法中断线程的 它提供了一个中断线程的标志位，然后配合isInterruped的判断才能中断线程。
	 * 
	 * @param args
	 * @throws InterruptedException
	 */
	public static void main(String[] args) throws InterruptedException {
		// TODO Auto-generated method stub
		Thread t = new Thread() {
			@Override
			public void run() {
				while (true) {
					// 使用isInterrupted()方法，判断中断线程的标志位来起到中断线程的作用
					if (Thread.currentThread().isInterrupted()) {
						System.out.println("Interrupted");
						break;
					}
					Thread.yield();
				}
			}
		};
		t.start();
		Thread.sleep(2000);
		t.interrupt();
	}

}

```
## 等待（wait）和通知（notify）
> public final void wait() throws InterruptedException //在其他线程调用此对象的 notify() 方法或 notifyAll() 方法前，导致当前线程等待
> public final void notify() //唤醒在此对象监视器上等待的单个线程。

如果一个线程A调用obj.wait()方法，这个线程A就会被放置到等待队列中。
线程A会一直等待到其他线程调用了obj.notify()为止。
<img src="D:\04-make_it_done\01-blog\keepblog\images\notify唤醒等待的线程.png" width="481" height="564" border="0" alt="">

Object.wait()方法必须包含在对于的synchronzied语句块中，无论是wait()或者notify()都需要首先获取目标对象的一个监视器。
<img src="D:\04-make_it_done\01-blog\keepblog\images\wait()和notify()的工作流程细节.png" width="460" height="560" border="0" alt="">

代码示例如下：
```java
package com.cc.training.thread;

/**
 * 一个简单的使用Wait方法和Notify方法的样例
 * @author cc
 *
 */
public class WaitAndNotifyThreadTest {
	/**
	 * 多线程操作的共享资源
	 */
	final static Object obj = new Object();
	
	/**
	 * 定义一个线程，用于执行wait方法进行等待
	 * 直到其他线程调用notify方法唤醒
	 * @author cc
	 *
	 */
	public static class T1 extends Thread{
		@Override
		public void run() {
			// TODO Auto-generated method stub
			synchronized (obj) {
				System.out.println("T1 start!");
				System.out.println("T1 wait for object");
				try {
					obj.wait();
				} catch (InterruptedException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
				System.out.println("T1 End!");
			}
		}
	}
	
	/**
	 * 定义一个线程，用于执行notify方法去唤醒其他线程
	 * @author cc
	 *
	 */
	public static class T2 extends Thread{
		@Override
		public void run() {
			// TODO Auto-generated method stub
			synchronized (obj) {
				System.out.println("T2 start! notify one thread");
				obj.notify();
				System.out.println("T2 End!");
				try {
					Thread.sleep(2000);
				} catch (InterruptedException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			}
		}
	}
	
	public static void main(String[] args) {
		Thread t1 = new T1();
		Thread t2 = new T2();
		t1.start();
		t2.start();
	}
}
```
执行结果如下：
>T1 start!
T1 wait for object
T2 start! notify one thread
T2 End!
T1 End!

>**Object.wait()和Thread.sleep()方法都可以让线程等待若干时间，除了wait可以被唤醒外，另外一个重要的区别就是wait方法会释放所有的对象的锁，而sleep方法不会释放任何资源**