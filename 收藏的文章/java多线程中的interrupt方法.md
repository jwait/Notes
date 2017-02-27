# java多线程中的interrupt方法

在java中，开启一个多线程是很简单的，只需要new一个runnable就可以了，但是要停止一个线程，却不能简单的使用Thread.stop()方法。

  首先来说说java中的中断机制，Java中断机制是一种协作机制，也就是说通过中断并不能直接终止另一个线程，而需要被中断的线程自己处理中断。当调用interrupt()方法的时候,只是设置了要中断线程的中断状态，而此时被中断的线程的可以通过isInterrupted()或者是interrupted（）方法判断当前线程的中断状态是否标志为中断。我们可以从interrupt()方法来看：

```
  public void interrupt() {
	if (this != Thread.currentThread())
	    checkAccess();

	synchronized (blockerLock) {
	    Interruptible b = blocker;
	    if (b != null) {
		interrupt0();		// Just to set the interrupt flag
		b.interrupt();
		return;
	    }
	}
	interrupt0();
    }
```

从这个方法中我们可以看到，最直接的调用时interrupt0（）这个方法，而这个方法仅仅是设置了线程中断状态。

  我们再看看isInterrupted（）方法：

```
  public boolean isInterrupted() {
	return isInterrupted(false);
    }

    /**
     * Tests if some Thread has been interrupted.  The interrupted state
     * is reset or not based on the value of ClearInterrupted that is
     * passed.
     */
    private native boolean isInterrupted(boolean ClearInterrupted);
```

从这个方法中，我们可以猜测到，isInterrupted()方法仅仅是检查了当前线程的中断状态，但是不会清除这个状态。

我们再来看看静态方法interrupted（）

```
 public static boolean interrupted() {
	return currentThread().isInterrupted(true);
    }
```

这个方法同样是检测当前线程的中断状态，但是这个方法会产生一个副作用，就是会清除当前线程的中断状态。

Thread.interrupt()  VS  Thread.stop()

 这两个方法最大的区别在于：interrupt（）方法是设置线程的中断状态，让用户自己选择时间地点去结束线程；而stop（）方法会在代码的运行处直接抛出一个ThreadDeath错误，这是一个java.lang.Error的子类。所以直接使用stop（）方法就有可能造成对象的不一致性。

调用Thread.sleep()方法的时候，如果当前线程处于中断那状态，那么sleep（）方法不会执行，同时会清除掉该状态，并且抛出interruptedException异常。

中断的使用demo：

```
package com.app.basic;

public class InterruptTest {

	
	public static void main(String[] args) {
		

		Runnable runnable1 = new Runnable() {

			@Override
			public void run() {

				for (int i = 0; i < 10; i++) {

					System.out.println(i);

					if (Thread.currentThread().isInterrupted()) {
						System.out.println("aa");
					}
					try {
						
						Thread.sleep(1000);

					} catch (InterruptedException e) {

						e.printStackTrace();
                       //sleep方法抛出这个异常之后会清除中断状态，所以需要重新设置中断状态
						Thread.currentThread().interrupt();
					}
				}
			}

		};

		final Thread t1 = new Thread(runnable1);

		Runnable runnable2 = new Runnable() {

			@Override
			public void run() {

				try {

					Thread.sleep(3000);

					t1.interrupt();

				} catch (InterruptedException e) {

					e.printStackTrace();
				}

			}

		};

		Thread t2 = new Thread(runnable2);

		t1.start();
		t2.start();
	}

}
```

![img](http://static.oschina.net/uploads/space/2014/0211/103140_VeVi_932977.png)