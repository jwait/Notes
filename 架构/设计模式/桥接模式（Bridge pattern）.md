## 桥接模式（Bridge pattern）

桥接模式是一种结构性模式

### 目的

* **对抽象部分与实现部分进行解耦**，使得两者可以单独的改变
* 向使用者隐藏具体的实现



### 为什么使用桥接模式

###### 示例：

假设需要实现一个线程调度的类（Thread Schedule），要求支持两种线程调度方式：抢占式（PreemptiveThreadSchedule）、时间片（TimeSlicedThreadSchedule），以及这两种线程调度方式需要支持在两个不同的平台（Unix、Windows）实现。

###### 使用传统的类继承层次结构

![img](https://raw.githubusercontent.com/KEN-LJQ/MarkdownPics/master/Resource/2016-11-12/Bridge%20Design%20Pattern1.PNG)

可以看出使用传统的类继承层次结构可以很好地解决上面提出的问题需要，但是使用这种解决方式也带来了一个问题：

当我们需要添加一种新的线程调度实现平台时，例如是：Java‘s Virtual Machine，那么我们就需要为抢占式的线程调度方式以及时间片的线程线程调度方式分别添加一个新的 JVM 实现，最后得到的类继承层次图如下：

![](https://raw.githubusercontent.com/KEN-LJQ/MarkdownPics/master/Resource/2016-11-12/Bridge%20Design%20Pattern2.PNG)

当我们再需要添加一种新的线程调度方式时，类的继承层次结构会是怎样？

当我们需要实现一种线程调度的类，包括5种线程调度方式，10个实现的平台的时候，类的继承层次结构又会是怎样

分析后，我们可以得出的是，这样的类继承层次结构种需要定义的类的数量为：线程调度方式数量×线程实现平台数量，故为满足上面的要求，我们共需定义50个类。这意味者使用类继承的方式的可扩展性差



###### 类继承层次结构的缺点

类的继承层次结构在这种问题中导致扩展性不佳的原因是：类的继承层次结构适用于只有一个维度变化的情况，而上述的问题中有两个维度的变化，一个是线程调度方式（也就是抽象部分），另一个是线程的平台实现（也就是实现部分），如果使用继承的方式，继承体系会将抽象部分与它的实现部分固定在一起，一个维度的发生变化则另一个维度也必须进行相应的变化，使得难以对抽象部分和实现部分独立地进行修改、扩充和重用，而且类的继承层次结构也会变得越来越复杂，难以维护。



而桥接模式就是为了将这两个维度分开，也就是将抽象部分和实现部分分开，使得两者可以独立地变化

### 如何使用桥接模式

###### 使用桥接模式进行重构后的示意图：

![](https://raw.githubusercontent.com/KEN-LJQ/MarkdownPics/master/Resource/2016-11-12/Bridge%20Design%20Pattern3.PNG)

上图中可以看出：

* 桥接模式通过引入一个实现的接口（ThreadSchedule_Implement），把实现部分从系统中分离出去
* 而抽象部分则是采用面向实现的接口编程方式，为了将让抽象部分能够很方便地与实现部分结合起来，我们将顶层定义为一个抽象类（ThreadSchedle），在里面持有一个具体实现部分的实例。由于这个具体实现的引用将抽象部分以及实现部分连接在一起，这个持有的实现对象也就是桥接模式中连接抽象和实现的的桥梁

通过上面这种方式，我们可以实现将抽象部分与实现部分解耦，我们可以体会到使用这种方式比使用继承的方式更加的灵活。桥接模式主要是把继承部分改为使用对象组合，从而将两个维度分开，让每一个维度单独的变化，最后通过对象组合的方式，把两个维度组合在一起，而每一种组合方式相当于原来继承中的一种实现，可以有效减少实际类的个数，易于扩展和维护。



###### 总结桥接模式的优点：

1. 将抽象部分和实现部分分离，使两者可以独立地进行变化，不再相互的影响，大大提高了系统的灵活性和扩展性。

2. 抽象部分和实现部分不再是绑定在一起，可以在程序运行时进行动态的绑定和切换具体的实现

3. 一个具体的实现可以被多个抽象对象使用

4. 向使用者隐藏具体的实现

   ​



### 代码演示

###### 上述问题的桥梁模式结构示意图：

![](https://raw.githubusercontent.com/KEN-LJQ/MarkdownPics/master/Resource/2016-11-12/Bridge%20Design%20pattern4.png)

```
package bridgePatern;

/**
 * 实现的接口
 * @author Ken
 *
 */
public interface ThreadSchedule_Implement {

	public void getThread();
}
```

```java
package bridgePatern;

/**
 * windows 平台实现
 * @author Ken
 *
 */
public class WindowsImpl implements ThreadSchedule_Implement {

	@Override
	public void getThread() {
		System.out.println("get a thread on windows!!!");
	}

}

```

```java
package bridgePatern;

/**
 * Unix 平台实现
 * @author Ken
 *
 */
public class UnixImpl implements ThreadSchedule_Implement {

	@Override
	public void getThread() {
		System.out.println("get a thread on Unix!!!");
	}

}

```

```java
package bridgePatern;

/**
 * 线程调度抽象类
 * @author Ken
 *
 */
public abstract class ThreadSchedule {

	private ThreadSchedule_Implement threadImpl;
	
	public void getThread(){
		threadImpl.getThread();
	}
	
	public void setThreadImpl(ThreadSchedule_Implement threadImpl){
		this.threadImpl = threadImpl;
	}
	
	public ThreadSchedule_Implement getThreadImpl(){
		return this.threadImpl;
	}
}

```

```java
package bridgePatern;

/**
 * PreemptiveThreadSchedule 调度方式
 * @author Ken
 *
 */
public class PreemptiveThreadSchedule extends ThreadSchedule {

	public void preemptiveSchedule(){
		System.out.println("ThreadSchedule:PreemptiveThreadSchedule");
		this.getThread();
	}
}

```

```java
package bridgePatern;

/**
 * TimeSlicedThreadSchedule 调度方式
 * @author Ken
 *
 */
public class TimeSlicedThreadSchedule extends ThreadSchedule {

	public void timeSlicedSchedule(){
		System.out.println("ThreadSchedule:TimeSlicedSchedule");
		this.getThread();
	}
}

```

```java
package bridgePatern;

public class Client {
	
	public static void main(String[] args) {
		// 创建线程的实现
		ThreadSchedule_Implement unixImpl = new UnixImpl();// Unix 实现
		ThreadSchedule_Implement windowsImpl = new WindowsImpl();// windows 实现
		
		//线程调度方式
		PreemptiveThreadSchedule preemptiveSchedule = new PreemptiveThreadSchedule();// 抢占式
		TimeSlicedThreadSchedule timeSliceSchedule = new TimeSlicedThreadSchedule();// 时间片
		
		//测试
		// 1、Unix平台的抢占式线程调度
		preemptiveSchedule.setThreadImpl(unixImpl);
		preemptiveSchedule.preemptiveSchedule();
		
		// 2、Windows平台上的抢占式线程调度
		preemptiveSchedule.setThreadImpl(windowsImpl);
		preemptiveSchedule.preemptiveSchedule();
		
		// 3、Unix平台上的时间片线程调度
		timeSliceSchedule.setThreadImpl(unixImpl);
		timeSliceSchedule.timeSlicedSchedule();
		
		// 4、windows平台上的时间片线程调度
		timeSliceSchedule.setThreadImpl(windowsImpl);
		timeSliceSchedule.timeSlicedSchedule();
	}
}

```

输出：

```
ThreadSchedule:PreemptiveThreadSchedule
get a thread on Unix!!!
ThreadSchedule:PreemptiveThreadSchedule
get a thread on windows!!!
ThreadSchedule:TimeSlicedSchedule
get a thread on Unix!!!
ThreadSchedule:TimeSlicedSchedule
get a thread on windows!!!
```