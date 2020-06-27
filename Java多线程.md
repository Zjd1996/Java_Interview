# Java多线程/并发

## 使用线程

三种方法：

- 实现Runnable接口；
- 实现Callable接口；
- 继承Thread类；
- 另外，通过使用Excutor框架来创建线程

实现Runnable和Callable接口的类只能当作一个可以在线程中运行的任务，不是真正意义上的线程，因此最后还需要通过Thread来调用。可以理解为任务是通过线程驱动而执行的。

#### 实现Runnable接口

需要实现接口中的run()方法。

```java
public class MyRunnable implements Runnable {
	@Override
	public void run() {
		//		
	}
}
```

使用Runnable实例再创建一个Thread实例，然后调用Thread实例的start()方法来启动线程。

```java
public static void main(String[] args) {
	MyRunnable instance = new MyRunnable();
	Thread thread = new Thread(intance);
	thread.start();
}
```

#### 实现Callable接口

与Runnable接口相比，Callable可以有返回值，返回值通过FutureTask进行封装。

```java
public class MyCallable implements Callable<Integer> {
	public Integer call() {
		return 123;
	}
}
```

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
	MyCallable myCallable = new MyCallable();
	FutureTask<Integer> ft = new FutureTask<>(myCallable);
	Thread thread = new Thread(ft);
	thread.start();
	System.out.println(ft.get());
}
```

#### 继承Thread类

同样也是需要实现run()方法，因为Thread类也实现了Runable接口。

当调用start()方法启动一个线程时，虚拟机会将该线程放入就绪队列中等待被调度，当一个线程被调度时会执行该线程的run()方法。

```java
public class MyThread extends Thread {
	public void run() {
		// TODO
	}
}
```

```java
public static void main(String[] args) {
    MyThread mt = new MyThread();
    mt.start();
}
```

#### 线程池

#### 实现接口VS继承Thread

实现接口更好一些：

- 避免Java中单继承的限制：Java不支持多重继承，因此继承了Thread类就无法继承其他类，但是可以实现多个接口；
- runnable/callable实现线程可以对线程进行复用，因为runnable/callable是轻量级的对象，重复new不会消耗太多的资源，而Thread不然，它是重量级对象，而且线程执行完了就完了无法再次利用。类可能只要求可执行就行，继承整个Thread类开销过大。
- 增加程序的健壮性，代码可以被多个线程共享，代码和数据独立
- 线程池只能放入实现Runnable或Callable接口的线程，不能直接放入继承Thread的类

```
new Thread(new Runnable(){
   public void run(){
      System.out.println("runnable -> "+Thread.currentThread().getName());
   }
}){
  public void run(){
      System.out.println("thread -> "+Thread.currentThread().getName());
  }
}.start();
```


这个题目的运行结果是：thread -> Thread-0 

这是一个匿名内部类的线程，运行时，会先找自己的run方法，如果自己没有run方法才会去找目标对象Runnable对象中的run方法。

## 基础线程机制

#### Executor

Executor管理多个异步任务的执行，而无序程序员显式地管理线程的生命周期。这里的异步是指多个任务的执行互不干扰，不需要进行同步操作。

主要有三种Executor：

- CachedThreadPool：一个任务创建一个线程；
- FixedThreadPool：所有任务只能使用固定大小的线程；
- SingleThreadExecutor：相当于大小为1的FixedThreadPool

```java
public static void main(String[] args) {
	ExecutorService executorService = Executors.newCachedThreadPool();
	for (int i = 0; i < 5; i++) {
		executorService.execute(new MyRunnable());
	}
	executorService.shutdown();
}
```

#### Daemon

守护线程是程序运行在后台提供服务的线程，不属于程序中不可或缺的部分。

当所有非守护线程结束时，程序也就终止，同时会杀死所有守护线程。反过来说，只要有任何非守护线程还在运行，程序就不会终止。

守护线程与用户线程的唯一不同：虚拟机的离开，如果用户线程已经全部退出运行了，只剩下守护线程在了，虚拟机也就退出了。

main()属于非守护线程。

在线程启动之前使用setDeamon()方法可以将一个线程设置为守护线程。

```java
public static void main(String[] args) {
	Thread thread = new Thread(new MyRunnable());
	thread.setDaemon(true);
}
```

#### sleep()

Thread.sleep(ms)方法会休眠当前正在执行的线程，单位毫秒。

sleep()可能会抛出InterruptedException，因为异常不能跨线程传播回main中，因此必须在本地进行处理。线程中抛出的其它异常也同样需要在本地处理。

```java
public void run() {
	try {
		Thread.sleep(3000);
	} catch (InterruptedException e) {
		e.printStackTrace();
	}
}
```

#### yield()

对静态方法Thread.ield()的调用声明了当前线程已经完成了生命周期中最重要的部分，可以切换给其它线程来执行。该方法只是对线程调度器的一个建议，而且也只是建议具有相同优先级的其它线程可以运行。

```java
public void run() {
	Thread.yield();
}
```

## 中断

一个线程执行完毕之后会自动结束，如果在运行过程中发生异常也会提前结束。

#### InterruptedException

通过调用一个线程的interrupt()来中断该线程，如果该线程处于阻塞、限期等待或者无限期等待状态，那么就会抛出InterruptedException，从而提前结束该进程。但是不能中断I/O阻塞和sychronized锁阻塞。

对于以下代码，在main()中启动一个线程之后再中断它，由于线程中调用了Thread.sleep()方法，因此会抛出一个InterruptedException，从而提前结束进程，不执行之后的语句。

```java
public class InterruptExample {
	private static class MyThread1 extends Thread {
		@Override
		public void run() {
			try {
				Thread.sleep(2000);
				System.out.println("Thread run");
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
}
```

```java
public static void main(String[] args) throws InterruptedException {
	Thread thread1 = new MyThread();
	thread1.start();
	thread1.interrupt();
	System.out.println("Main run");
}
```

```
Main run
java.lang.InterruptedException: sleep interrupted
    at java.lang.Thread.sleep(Native Method)
    at InterruptExample.lambda$main$0(InterruptExample.java:5)
    at InterruptExample$$Lambda$1/713338599.run(Unknown Source)
    at java.lang.Thread.run(Thread.java:745)
```

#### interrupted()

如果一个线程的run()方法执行一个无限循环，并且没有执行sleep()等会抛出InterruptedException的操作，那么调用线程的interrupted()方法就无法诗线程提前结束。

但是调用interrupt()方法会设置线程的中断标记，此时调用interrupted()方法会返回ture。因此可以在循环体中使用interrupted()方法来判断线程是否处于中断状态，从而提前结束线程。

```java
public class InterruptExample {
	private static class MyThread2 extends Thread {
		@Override
		pubic void run() {
			while(!interrupted()) {
				// TODO
			}
			System.out.println("Thread end");
		}
	}
}
```

```java
public static void main(String[] args) throws InterruptedException {
	Thread thread2 = new MyThread2();
	thread2.start();
	thread2.interrupt();
}

// 输出 Thread end
```

#### Executor的中断操作

调用Executor的shutdown()方法会等待线程都执行完毕之后再关闭，但是如果调用的事shutdownNow()方法，则相当于调用每个线程的interrupt()方法。

以下用Lambda创建线程，相当于创建了一个匿名内部线程。

```java
public static void main(String[] args) {
	ExecutorService executorService = Executors.newCachedThreadPool();
	executorService.execute(() - > {
		try {
			Thread.sleep(2000);
			System.out.println("Thread run");
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	});
	executorService.shutdownNow();
	System.out.println("Main run");
}

// 输出
Main run
java.lang.InterruptedException: sleep interrupted
    at java.lang.Thread.sleep(Native Method)
    at ExecutorInterruptExample.lambda$main$0(ExecutorInterruptExample.java:9)
    at ExecutorInterruptExample$$Lambda$1/1160460865.run(Unknown Source)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
    at java.lang.Thread.run(Thread.java:745)

```

如果只想中断Executor中的一个线程，可以通过使用submit()方法来提交一个线程，它会返回一个Future<?>对象，通过调用该对象的cancel(true)方法就可以中断线程。



```java
Future<?> future = executorService.submit(() - > {
	// TODO
})；
future.cancel(true);
```

## 互斥同步

Java提供了两种锁机制来控制多个线程对共享资源的互斥访问，第一个是JVM实现的sychronized，而另一个是JDK实现的ReentrantLock。

#### sychronized

- 步一个代码块

```java
public void func() {
	synchronized(this) {
		
	}
}
```

它只作用于同一个对象，如果调用两个对象上的同步代码块，就不会进行同步。

对于以下代码，使用ExecutorService执行了两个线程，由于调用的是同一个对象的同步代码块，因此这两个线程会进行同步，当一个线程进入同步语句块时，另一个线程就必须等待。

```java
public class SynchronizedExample {
	public void func() {
		synchronized(this) {
			for(int i = 0; i < 10; i++) {
				System.out.println(i + " ");
			}
		}
	}
}

public static void main(String[] args) {
    SynchronizedExample e1 = new SynchronizedExample();
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> e1.func());
    executorService.execute(() -> e1.func());
}

// 输出。0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9
```

对于以下代码，两个线程调用了不同对象的同步代码块，因此这两个线程就不需要同步。从输出结果看出，两个线程交叉执行。

```java
public static void main(String[] args) {
	SynchronizedExample e1 = new SynchronizedExample();
	SynchronizedExample e2 = new SynchronizedExample();
	ExecutorService executorService = Executors.newCachedThreadPool();
	executorService.excute(() -> e1.func());
	executorService.excute(() -> e2.func());
}
// 输出 0 0 1 1 2 2 3 3 4 4 5 5 6 6 7 7 8 8 9 9
```

- 同步一个方法

```java
public synchronized void func() {
	// ..
}
```

作用于同一对象。

- 同步一个类

```java
public void func() {
	synchronized(SychronizedExamlpe.class) {
		// TODO
	}
}
```

作用于整个类，也就是说两个线程调用同一个类的不同对象上的这种同步语句，也会进行同步。

```java
public class SynchronizedExample {
	public void func2(){
		synchronized(SynchronizedExample.class) {
			for(int i = 0; i < 10; i++) {
				System.out.print(i + " ");
			}
		}
	}
}
```

```java
public static void main(String[] args) {
	SynchronizedExample e1 = new SynchronizedExample();
	SynchronizedExample e2 = new SynchronizedExample();
	ExecutorService executorService = Executors.newCachedThreadPool();
	executorService.execute(() -> e1.func2());
	executorService.executr(() -> e2.func2());
}
// 输出 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9
```

- 同步一个静态方法

```java
public synchronized static void fun() {
	// ....
}
```

作用于整个类。

#### ReentrantLock

ReentrantLock是java.util.concurrent(J.U.C)包中的锁。

```java
public class LockExample {
	private Lock lock = new ReentrantLock();
	public void func() {
		lock.lock();
		try {
			for (int i = 0; i < 10; i++) {
				System.out.print(i + " ");
			}
		} finally {
			lock.unlock(); // 确保释放锁，从而避免发生死锁
		}
	}
}
```

```java
public static void main(String[] args) {
	LockEample lockExample = new LockExamlpe();
	ExecutorService executorService = Executors.newCachedThreadPool();
	executorService.execute(() -> lockExample.func()); 
	executorService.execute(() -> lockExample.func());
}

// 输出 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9
```

#### 比较

**锁的实现**

synchronized是JVM实现的，而ReentrantLock是JDK实现的。

**性能**

新版本Java对synchronized进行了很多优化，例如自旋锁等，synchronized与ReentrantLock大致相同。

**等待可中断**

当持有锁的线程长期不释放锁的时候，正在等待的线程可以选择放弃等待，改为处理其它事情。

ReentrantLock可中断，而synchronized不行。

**公平锁**

公平锁是指多个线程在等待同一个锁时，必须按照申请锁的时间顺序来依次获得锁。

synchronized中的锁是非公平的，ReentrantLock默认情况下也是非公平的，但是也可以是公平的。

**锁绑定多个条件**

一个ReentrantLock可以同时绑定多个Condition对象。

#### 使用选择

除非需要使用ReentrantLock的高级功能，否则优先使用synchronized。这是因为synchronized是JVM实现的一种锁机制，JVM原生地支持它，而ReentrantLock不是所有的JDK版本都支持。并且使用synchronized不用担心没有释放锁而导致死锁问题，因为JVM会确保锁的释放。

## 线程之间的协作

当多个线程可以一起工作去解决某个问题时，如果某些部分必须在其它部分之前完成，那么就需要对线程进行协调。

#### join()

在线程中调用另一个线程的join()方法，会将当前线程挂起，而不是忙等待，直到目标线程结束。

对于以下代码，虽然b线程先启动，但是因为在b线程中调用了a线程的join()方法，b线程会等待a线程结束才继续执行，因此最后能够保证a线程的输出先于b线程的输出。

理解：线程a调用线程b的join方法，相当于把线程a挂到线程b的后面，线程b执行完之后才执行线程a

```java
public class JoinExample {
	private class A extends Thread {
		@Override
		public void run() {
			System.out.println("A");
		}
	}
	private class B extends Thread {
		private A a;
		B(A a) {
			this.a = a;
		}
		
		@Override
		public void run() {
			try {
				a.join();
			} catch(InterruptedException e) {
				e.printStackTrace();
			}
			System.out.println("B");
		}
	}
	
	public void test() {
		A a = new A();
		B b = new B(a);
		b.start();
		a.start();
	}
}

public static void main(String[] args) {
	JoinExample example = new JoinExample();
	example.test();
}

// 输出 A B
```

#### wait()/notify()/notifyAll()

调用wait()使得线程等待某个条件满足，线程在等待时会被挂起，当其他线程的运行使得这个条件满足时，其他线程会调用notify()或者notifyAll()来唤醒挂起的线程。

他们都属于Object的一部分，不属于Thread。

只能用在同步方法或者同步控制块中使用，否则会在运行时抛出IllegalMonitorStateException。

使用wait()挂起期间，线程会释放锁。这是因为，如果没有释放锁，那么其他线程就无法进入对象的同步方法或者同步控制块中，那么就无法执行notify()或者notifyAll()来唤醒挂起的线程，造成死锁。

```java
public class WaitNotifyExample {
    public synchronized void before() {
        System.out.println("before");
        notifyAll();
    }
    
    public synchronized void after() {
        try {
            wait();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("after");
    }
}

public static void main(String[] args) {
    ExecutorService executorService = Executors.newCachedThreadPool();
    WaitNotifyExample example = new WaitNotifyExample();
    executorService.execute(() -> example.after());
    executorService.execute(() -> example.before());
}

// 输出 before after
```

#### wait()和sleep()的区别

- wait()是Object的方法，而sleep()是Thread的静态方法；
- wait()会释放锁，sleep()不会。

#### await()/signal()/signalAll()

java.util.concurrent类库中提供了Condition类来实现线程之间的协调，可以在Condition上调用await()方法使线程等待，其他线程调用signal()或signalAll()方法唤醒等待的线程。

相比于wait()这种等待方式，await()可以指定等待的条件，因此更加灵活。

使用Lock来获取一个Condition对象。

```java
public class AwaitSignalExample {
	private Lock lock = new ReentrantLock();
    private Condition condition = lock.newConfition();
    
    public void before() {
        lock.lock();
        try {
            System.out.println("before");
            condition.signalAll();
        } finally {
            lock.unlock();
        }
    }
    
    public void after() {
        lock.lock();
        try {
            condition.await();
            System.out.println("After");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}

public static void main(String[] args) {
    ExecutorService executorService = Executors.newCachedThreadPool();
    AwaitSignalExample example = new AwaitSignalExample();
    executorService.execute(() -> example.after());
    executorService.execute(() -> example.before());
}

// 输出 before after
    
```

## 线程状态

一个线程只能处于一种状态，并且这里的线程状态特指Java虚拟机的线程状态，不能反映线程在特定操作系统下的状态。

#### 新建（NEW）

创建后尚未启动

#### 可运行（RUNNABLE）

正在Java虚拟机中运行。但是在操作系统层面，它可能处于运行状态，也可能等待资源调度（例如处理器资源），资源调度完成就进入运行状态。所以该状态的可运行是指可以被运行，具体有没有运行要看底层操作系统的资源调度。

#### 阻塞（BLOCK）

请求获取monitor lock从而进入synchronized函数或者代码块，但是其他线程已经占用了该monitor lock，所以处于阻塞状态。要结束该状态进入RUNNABLE需要其他线程释放monitor lock。

#### 无限期等待（WAITING）

等待其他线程显式的唤醒。

阻塞和等待的区别在于，阻塞是被动的，它是在等待获取monitor lock。而等待是主动的，通过调用Obect.wait()等方法进入。

| 进入方法                               | 退出方法                           |
| -------------------------------------- | ---------------------------------- |
| 没有设置Timeout参数的Object.wait()方法 | Object.notify()/Object.notifyAll() |
| 没有设置Timeout参数的Thread.join()方法 | 被调用的线程执行完毕               |
| LockSupport.park()方法                 | LockSupport.unpark(Thread)         |

#### 限期等待（TIMED_WAITING）

无需等待其他线程显式地唤醒，在一定时间之后会被系统自动唤醒。

| 进入方法                             | 退出方法                                    |
| ------------------------------------ | ------------------------------------------- |
| Thread.sleep()                       | 时间结束                                    |
| 设置了Timeout参数的Object.wait()方法 | 时间结束/Object.notify()/Object.notifyAll() |
| 设置了Timeout参数的Thread.join()方法 | 时间结束/被调用的线程执行完毕               |
| LockSupport.parkNanos()方法          | LockSupport.unpark(Thread)                  |
| LockSupport.parkUntil()方法          | LockSupport.unpark(Thread)                  |

调用Tread.sleep()方法使进程进入限期等待状态时，常常用“使一个线程睡眠”进行描述。调用Object.wait()方法使线程进入限期等待或者无限期等待时，常常用“挂起一个进程”进行描述。睡眠和挂起是用来描述行为，而阻塞和等待来描述状态。

#### 死亡（TERMINATED）

可以是线程结束任务之后自己结束，或者产生了异常而结束。

## 线程池

线程池的工作主要是控制运行的线程的数量，处理过程中将任务放入队列，然后在线程创建后启动这些任务，如果线程数量超过了最大数量，那么排队等候。

主要特点：线程复用、控制最大并发数、管理线程。

使用线程池的三个好处：

- 降低资源消耗，通过重复利用已创建的线程降低线程创建和销毁造成的消耗
- 提高相应速度，当任务到达时，可以不需要等待线程的创建就可以执行
- 提高线程的可管理性，线程是稀缺资源，如果无限制地创建线程，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以统一进行分配、调优、监控。

### 线程池的实现原理

线程池的处理流程：

1. 线程池判断核心线程里的线程是否都在执行任务，如果不是，则创建一个新的工作线程来执行任务。如果核心线程池里的线程都在执行任务，进入下一个流程；
2. 线程池判断工作队列是否已经满，如果工作队列没有满，则将新提交的任务存储在这个工作队列里，如果工作队列满了，进入下一个流程；
3. 线程池判断线程池的线程是否都处于工作状态，如果没有则创建一个先的工作线程来执行任务。如果已经满了，则交给饱和策略来处理这个任务。



1. 如果当前运行的线程少于corePoolSize，则创建新线程来执行任务（这步需要获取全局锁）
2. 如果运行的线程等于或多于corePoolSize，则将任务加入到BlockingQeue
3. 如果无法加入BlockingQueue（满），则创建新的线程来处理任务（这步需要全局锁）
4. 如果创建新线程将超过maximumPoolSize，任务将被拒绝。并调用RejectedExecutionHandler.rejectedExecution()方法

ThreadPoolExecutor这样的设计，主要是了为乐尽可能避免获取全局锁，预热完成后调用execute()方法都是执行步骤2，而不需要获取全局锁。

![image-20200530172104343](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200530172104343.png)



### 线程池的使用

线程池的创建：通过ThreadPoolExecutor来创建。

![image-20200530161513558](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200530161513558.png)

- corePoolSize：线程池的基本大小，创建到corePoolSize大小。当提交一个任务提交到线程池，线程池会创建一个线程来执行任务，即使其他空闲的基本线程能够执行也要新创建，等到到达了core大小就不创建了。

  - 如果调用p re s ta r t A l l C o re T h re a d s()方法，线程池会提前创建并启动所有基本线程。

- runnableTaskQueue：任务队列，用于保存等待执行的任务的阻塞队列，可选的：

  - ArrayBlockingQueue：基于数组的有界
  - LinkedBlockingQueue：基于链表结构的
  - SynchronousQueue：不存储元素
  - PriorityBlockingQueue：具有优先级的无界阻塞

- maximumPoolSize：线程池最大数量，如果使用了无界的任务队列，这个参数无效果

- ThreadFactory：用于设置创建线程的工厂，可以通过线程工厂给每个创建出来的线程设置更有意义的名字。使用开源框架guava提供的ThreadFactoryBuilder可以快速设置有意义的名字

- RejectedExecutionHandle：饱和策略，拒绝策略，队列满，运行的线程大于等于最大线程数时。均实现了RehectedExecutionHandle接口

  - AbortPolicy：直接抛出异常，默认
  - CallerRunsPolicy：只用调用者所有线程来运行任务，作为一种调节机制，将某些任务会退到调用者
  - DiscardOldestPolicy：丢弃最老的任务，也是最快将被执行的任务
  - DiscardPolicy：不处理，丢弃，如果运行丢弃，这是最好的方案

- keepAliveTime：线程活动保持时间，

- TimeUnit：线程活动保持时间单位

- ```java
  ExecutorService pool = new ThreadPoolExecutor(2,
                  5, 
                  1L, 
                  TimeUnit.SECONDS, 
                  new LinkedBlockingQueue<>(3), 
                  Executors.defaultThreadFactory(),
                  new ThreadPoolExecutor.AbortPolicy());
  
  		try {
              for (int i = 0; i < 20; i++) {
                  pool.execute(() -> {
                      System.out.println(Thread.currentThread().getName() + " do");
                  });
              }
          } catch (Exception e) {
              e.printStackTrace();
          } finally {
              pool.shutdown();
          }
  ```

### 如何合理配置参数

分场景：

- CPU密集型：System.out.println(Runtime.getRuntime().availableProcessors()
  - 任务需要大量运算，没有阻塞，CPU一直全速运行
  - CPU密集型任务配置尽可能少的线程数量
  - 一般公式：CPU核数量+1个线程
- IO密集型
  - IO密集即大量阻塞，单线程会导致浪费大量CPU，可以加大线程加速程序的运行，主要是利用浪费掉的阻塞时间
  - 公式：CPU核心/(1-阻塞系数)，阻塞系数在0.8-0.9之间，如8核心0.9阻塞系数->80线程

### 框架提供线程池

![image-20200530165641345](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200530165641345.png)

用提供的线程池一般：try-finally

```java
ExectorService pool = Executors.newFixedThreadPool(5);
try {
    // do
    // fori
    pool.execute(() -> {
        
    })
} catch () {
    // 
} finally {
    // close
    pool.shutdown();
}
```



### 提交任务

- execute：用于提交不需要返回值的任务，所以无法判断任务是否被线程执行成功。输入一个Runnable类的实例

  ```java
  threadspool.execute(new Runnable() {
      @Override
      public void run() {
          // TODO
      }
  });
  ```

  

- submit：用于提交需要返回值的任务。线程池会返回一个future类型的对象，通过future对象可以判断任务是否执行成功，通过future的get方法获取返回值，该方法阻塞到任务执行完成，get(long timeout, TimeUnit unit)阻塞一定时间即返回。

  ```java
  Future<Object> future = executor.submit(xxxTask);
  ```

### 关闭线程池

- shuntdown：只将线程池的状态设置成SHUTDOWN，然后中断所有没有正在执行任务的线程
- shuntdownNow：首先将线程池的状态设置为STOP，然后尝试停止所有的正在执行或者暂停的线程，并返回等待执行任务的列表

原理：遍历线程池中的工作线程，然后逐个调用线程的interrupt方法来中断线程，所以无法响应中断的任务可能永远 无法终止。

只要调用任意一个，isShutdown方法就会返回true。当所有的任务已关闭后，才表示线程池关闭成功，这时候调用isTerminated方法会返回true。

## J.U.C-Lock

### Synchronized与Lock区别

- 构成：Synchronized为JVM层面的关键字，Lock为API层面的接口/类
- 使用方法：sync不需要手动释放锁，全自动；Lock需要手动加锁解锁，没有正确释放锁可能出现死锁，需要lock和unlock配合try/finally使用
- 等待是否可中断：synchronized不可中断，除非抛异常或正常运行完成；Lock可以中断，设置超时方法tryLock(long timeout, TimeUnit unit);或lockInterruptibly()放代码块中，调用interrupt方法可中断
- 公平非公平：sync非公平；Lock默认非公平，也可以公平；
- 绑定多个条件Condition：sync没有，ReentrantLock可以精确唤醒，结合Condition

例子：多线程之间按顺序调用，A打印A->B打印B->C打印C 重复10轮

```java
class ShareResource {
    private int flag = 1;
    private Lock lock = new ReentrantLock();
    private Condition c1 = lock.newCondition();
    private Condition c2 = lock.newCondition();
    private Condition c3 = lock.newCondition();

    public void printA() {
        lock.lock();
        try {
            while (flag != 1) {
                c1.await();
            }
            System.out.println("A");
            flag = 2;
            c2.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void printB() {
        lock.lock();
        try {
            while (flag != 2) {
                c2.await();
            }
            System.out.println("B");
            flag = 3;
            c3.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void printC() {
        lock.lock();
        try {
            while (flag != 3) {
                c3.await();
            }
            System.out.println("C");
            flag = 1;
            c1.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public static void main(String[] args) {
        ShareResource shareResource = new ShareResource();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                shareResource.printA();
            }
        }, "Thread-A").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                shareResource.printB();
            }
        }, "Thread-B").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                shareResource.printC();
            }
        }, "Thread-C").start();
    }

}
```



## J.U.V-Atomic

valueOffset：内存偏移量，因此unsafe可以像C和指针一样直接操作内存。如果想获取一个对象的属性的值，我们一般通过getter方法获得，而sun.misc.Unsafe却不同，我们可以把一个对象实例想象成一块内存，而这块内存中包含了一些属性，如何获取这块内存中的某个属性呢？那就需要**该属性在该内存的偏移量**了，**每个属性**在该对象**内存中valueOffset偏移量不同**，每个属性的偏移量在该**类加载或者之前**就已经确定了(则**该类的不同实例的同一属性偏移量相等**)，所以**sun.misc.Unsafe**可以通过一个**对象实例**和**该属性的偏移量**用原语获得该对象对应属性的值；

value：值，用volatile修饰，保证可见

Unsafe类中的native方法，直接调用操作系统的底层资源执行相应任务

### CAS

比较并交换。是一条CPU并发原语，判断内存某个位置的值是否符合预期值，如果是则更新为新的值，这个过程是原子的。依赖于硬件。由于cAS是一种系统原语，原语术语操作系统用语范畴，是由若干指令组成，用于完成某个功能的一个过程，并且：

- 原语的执行必须是连续的
- 执行过程不允许中断
- 原子指令，不会造成数据不一致

缺点：

- 循环时间长开销很大

- 只能保证一个共享变量的原子操作

- ABA问题

  - 内存值被其他线程由A改为B又改回A，本线程认为符合预期，进行更新

  - 如何解决：时间戳原子引用

    ```java
    public class Demo {
        static AtomicReference<Integer> atomicReference = new AtomicReference<>(100);
        
        public static void main(String[] args) {
            new Thread(() -> {
                atomicReference.compareAndSet(100, 101);
                atomicReference.compareAndSet(101, 100);
            },"t1").start();
            new Thread(() -> {
                // 保证上面线程完成
                sleep(1000);
                atomicReference.compareAndSet(100,2019);  // 会修改成功，因为没有stamped
            },"t2").start();	
            
            new Thread(() -> {
                int stamp = atomicStampedReference.getStamp();
                sleep(1);
                atomicStampedReference.compareAndSet(100,101,atomicStampedReference.getStamp(), atomicStampedReference.getStamp()+1);
                atomicStampedReference.compareAndSet(101,100,atomicStampedReference.getStamp(), atomicStampedReference.getStamp()+1);
            }, "t3").start();
            
            
            new Thread(() -> {
                int stamp = atomicStampedReference.getStamp();
                sleep(3);
                boolean result = atomicStampedReference.compareAndSet(100,101,stamp, stamp+1);
               
                
            }, "t4").start();
        }
    }
    
    
    ```

    

### 原子更新基本类型类

- AtomicBoolean
- AtomicInteger
- AtomicLong

以AtomicInteger为例，常用方法：

```java
int addAndGet(int delta);
boolean compareAndSet(int expect, int update); //如果输入等于预期，则以原子方式将该值设置为输入的值
int getAndIncrement(); // i++ 返回增加前的值
int getAndSet(int newValue); // 返回旧值
void lazySet(int newValue);
```

以getAndIncrement为例：

```java
public final int getAndIncrement() {
    for(;;) {
        int current = get();
        int next = current + 1;
        if(compareAndSet(current, next))
            return current;
        // 自旋等待
    }
}

public final boolean compareAndSet(int expect, int update) {
    return unsafe.compareAndSet(this, valueOffset, expect, update);
}

// unsafe，提供三种本地方法

public final native boolean compareAndSwapObject(Object o, long offset, Object expected, Object x);
public final native boolean compareAndSwapInt(Object o, long offset, int expected, int x);
public final native boolean compareAndSwapLong(Object o, long offset, long expected, long x);
```

其他基本类型，如Boolean、Char，先转为整形，再使用compareAndSwapInt进行CAS，float、double类似，转为长整型。

### 原子更新数组

- AtomicIntegerArray：原子更新整形数组里的元素，构造函数传入数组即可（复制一份到内部），常用方法：
  - int addAndGet(int i, int delta)：以原子方式将输入值与数组中索引i的元素相加
  - boolean compareAndSet(int i, int expect, int update)：如果当前值等于预期值，则以原子方式将数组位置i的元素设置为update值。
- AtomicLongArray：原子更新长整型数组里的元素
- AtomicReferenceArray：原子更新引用类型数组里的元素

### 原子更新引用类型

- AtomicReference：原子更新引用类型
- AtomicReferenceFieldUpdater：原子更新引用类型里的字段
- AtomicMarkableReference：原子更新带有标记位的引用类型。可以原子更新一个不二类型的标记位和引用类型，构造方法是AtomicMarkableReference（V initialRef, boolean initialMark)

```java
public static AtomicReference<User> atomicUserRef = new AtomicReference<User>();

User user = new User(..);
atomicUserRef.set(user);
User updateUser = new User(..);
atomicUserRef.compareAndSet(user, updateUser);

```

### 原子更新字段类

- AtomicIntegerFieldUpdater：原子更新整形字段的更新器
- AtomicLongFieldUpdater：原子更新长整型字段的更新器
- AtomicStampedReference：原子更新带有版本号的引用类型。该类型将整数值与引用关联起来，可用于原子的更新数据和数据的版本号，可以解决使用CAS进行原子更新时可能出现的ABA问题

使用方法：

1. 因为原子更新字段类都是抽象类，每次使用的时候必须使用静态方法newUpdater()创建一个更新器，并且需要设置想要更新的类和属性
2. 更新类的字段（属性）必须使用public volatile修饰符

例子：

```java
public class AtomicIntegerFieldUpdaterTest {
    private static AtomicIntegerFieldUpdater<User> a = new AtomicIntegerFieldUpdater.newUpdater(User.class, "old"); // old为年龄字段名
    public static void main(String[] args) {
        User conan = new User("conan", 10);
        a.getAndIncrement(conan);
        a.get(conan);
        // ...
    }
}
```



## J.U.C-AQS

java.util.concurrent(J.U.C)大大提高了并发性能，AQS被认为是J.U.C的核心。

### 阻塞队列

有时不得不阻塞。

- 首先是个队列，FIFO。

- 当阻塞队列空时，从队列中获取元素的操作被阻塞
- 当阻塞队列满时，往队列中添加元素的操作被阻塞

为什么要用？

- 多线程领域，所谓阻塞，在某些情况下会挂起线程即阻塞，一旦条件满足，被挂起的线程会被自动唤醒
- 好处是我们不需要关系什么时候需要阻塞线程，什么时候需要唤醒线程，BlockingQueue一手包办

架构？BlockingQueue--->Queue---->Collection

- **ArrayBlockingQueue：由数组结构组成的有界阻塞队列**
- **LinkedBlockingQueue：由链表结构组成的有界（但大小默认为Integer.MAX_VALUE）阻塞队列**
- PriorityBlockingQueue:：支持优先级排序的无界阻塞队列
- DelayQueue：使用优先级队列实现的延迟无界阻塞队列
- **SynchronousQueue：不存储元素的阻塞队列，也即单个元素的队列**，每个put操作必须等待一个take操作，否则不能继续添加元素
- LinkedTransferQueue：由链表结构组成的无界阻塞队列
- LinkedBlocking**Deque**：由链表结构组成的双向阻塞队列

加黑：线程池的底层

核心方法？

![image-20200530005159271](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200530005159271.png)



```java
public class BlockingQueueDemo {
    
  
    public static void main(String[] args) throws Exception {
        BlockingQueue<String> blockingQueue = new ArrayBlockingQueue<>(3); //3
        blockingQueue.add("a"); //返回boolean
        blockingQueue.add("b");
        blockingQueue.add("v");
        
        blockingQueue.element(); // 返回队首
        
        
        blockingQueue.add("d"); // 异常
        blockingQueue.remove(); // 先进献出
        blockingQueue.remove("b"); // 指定出
        blockingQueue.remove(); // 先进献出
        
        blockingQueue.remove(); // 异常
        
        
        blockingQueue.offer("a");  // 返回boolean
                blockingQueue.offer("b");
                blockingQueue.offer("c");
        
                blockingQueue.offer("d"); //返回false，没异常
    }
}
```

#### 阻塞队列的实现原理

使用通知模式实现。即当生产者往满的队列里添加元素时会阻塞生产者，当消费者消费了一个队列中的元素后，会通知生产者当前队列可用。使用了Condition实现。

#### 阻塞队列应用

##### 生产者消费者

```java
class ShareData { // 资源类
    private int number = 0;
    private Lock lock = new ReentrantLock();
    private Condition condition = lock.newCondition();

    
    // 传统版本，但是使用了LOck和Condition
    public void increment() {
        lock.lock();
        try {
            // judge
            while(number != 0) {
                // wait
                condition.await();
            }
            // produce
            number++;
            sout(number);
            // notify
            confition.signalAll();
        } catch () {
            
        } finally {
            lock.unlock();
        }
    }
    
    public void decrement() {
        lock.lock();
        try {
            // judge
            while(number 0= 0) {
                // wait
                condition.await();
            }
            // produce
            number--;
            sout(number);
            // notify
            confition.signalAll();
        } catch () {
            
        } finally {
            lock.unlock();
        }
    }
}

psvm {
    ShareData shareData = new ShareData();
    new Thread(() -> {
        for (int i = 1; i <= 5;i++) {
            try {
                shareData.increment()
            } catch() {
                
            }
        }
    }, "producer").start();
    
    new Thread(() -> {
        for (int i = 1; i <= 5;i++) {
            try {
                shareData.decrement()
            } catch() {
                
            }
        }
    }, "consumer").start();
    
    
}
```

用if判断，有虚假唤醒的可能，需要用while。为什么呢？因为如果使用if，线程被唤醒以后就直接往下执行，但是如果是while，从唤醒处继续执行，还会再次进入while判断，判断是否是虚假唤醒！！

![image-20200530012430448](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200530012430448.png)

##### 生产者消费者阻塞队列版

```java
import java.util.Random;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;


public class ProducerAndConsumer {
    private BlockingQueue<Integer> queue = new LinkedBlockingQueue<Integer>(10);
    class Producer extends Thread {
        @Override
        public void run() {
            producer();
        }
        private void producer() {
            while(true) {
                try {
                    queue.put(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("生产者生产一条任务，当前队列长度为" + queue.size());
                try {
                    Thread.sleep(new Random().nextInt(1000)+500);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
    class Consumer extends Thread {
        @Override
        public void run() {
            consumer();
        }
        private void consumer() {
            while (true) {
                try {
                    queue.take();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("消费者消费一条任务，当前队列长度为" + queue.size());
                try {
                    Thread.sleep(new Random().nextInt(1000)+500);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
    public static void main(String[] args) {
        ProducerAndConsumer pc = new ProducerAndConsumer();
        Producer producer = pc.new Producer();
        Consumer consumer = pc.new Consumer();
        producer.start();
        consumer.start();
    }
}

```

##### 生产消费者Lock版

```java
import java.util.LinkedList;
import java.util.Queue;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * version 1 doesn't use synchronized to improve performance
 */
public class ProducerAndConsumer1 {
    private final int MAX_LEN = 10;
    private Queue<Integer> queue = new LinkedList<Integer>();
    private final Lock lock = new ReentrantLock();
    private final Condition condition = lock.newCondition();
    class Producer extends Thread {
        @Override
        public void run() {
            producer();
        }
        private void producer() {
            while(true) {
                lock.lock();
                try {
                    while (queue.size() == MAX_LEN) {
                        System.out.println("当前队列满");
                        try {
                            condition.await();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                    queue.add(1);
                    condition.signal();
                    System.out.println("生产者生产一条任务，当前队列长度为" + queue.size());
                    try {
                        Thread.sleep(500);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                } finally {
                    lock.unlock();
                }
            }
        }
    }
    class Consumer extends Thread {
        @Override
        public void run() {
            consumer();
        }
        private void consumer() {
            while (true) {
                lock.lock();
                try {
                    while (queue.size() == 0) {
                        System.out.println("当前队列为空");
                        try {
                            condition.await();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                    queue.poll();
                    condition.signal();
                    System.out.println("消费者消费一条任务，当前队列长度为" + queue.size());
                    try {
                        Thread.sleep(500);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                } finally {
                    lock.unlock();
                }
            }
        }
    }
    public static void main(String[] args) {
        ProducerAndConsumer pc = new ProducerAndConsumer();
        Producer producer = pc.new Producer();
        Consumer consumer = pc.new Consumer();
        producer.start();
        consumer.start();
    }
}
```

##### 生产者消费者Sync版

```java
import java.util.LinkedList;
import java.util.Queue;

public class ProducerAndConsumer {
    private final int MAX_LEN = 10;
    private Queue<Integer> queue = new LinkedList<Integer>();
    class Producer extends Thread {
        @Override
        public void run() {
            producer();
        }
        private void producer() {
            while(true) {
                synchronized (queue) {
                    while (queue.size() == MAX_LEN) {
                        queue.notify();
                        System.out.println("当前队列满");
                        try {
                            queue.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                    queue.add(1);
                    queue.notify();
                    System.out.println("生产者生产一条任务，当前队列长度为" + queue.size());
                    try {
                        Thread.sleep(500);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }
    }
    class Consumer extends Thread {
        @Override
        public void run() {
            consumer();
        }
        private void consumer() {
            while (true) {
                synchronized (queue) {
                    while (queue.size() == 0) {
                        queue.notify();
                        System.out.println("当前队列为空");
                        try {
                            queue.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                    queue.poll();
                    queue.notify();
                    System.out.println("消费者消费一条任务，当前队列长度为" + queue.size());
                    try {
                        Thread.sleep(500);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }
    }
    public static void main(String[] args) {
        ProducerAndConsumer pc = new ProducerAndConsumer();
        Producer producer = pc.new Producer();
        Consumer consumer = pc.new Consumer();
        producer.start();
        consumer.start();
    }
}
```



### CountDownLatch

- 让一些线程阻塞知道另一些线程完成一系列操作后才被唤醒，也可以使用jion
- 主要由两个方法：
  - 当一个或多个线程调用await方法是，调用线程会被阻塞。await可以带参数await(long time, TimeUnit unit)
  - 其他线程调用countDonw方法会将计数器减1，并不阻塞
  - 当计数器值变为零时，因调用await方法被阻塞的线程会被唤醒，继续执行。同时不能被重新初始化或修改内部计数器的值

例子：

```java
public class CountDownLatchDemo {
    public static void main(String[] args) {
        CountDownLatch countDownLatch = new CountDownLatch(6);
        
        for (int i = 1; i <= 6; i++) {
            new Thread(() -> {
                sout(同学i离开);
                countDownLatch.countDown();
            }, String.valueOf(i)).start();
        }
        
        coutDownLatch.await();
        sout(最后班长锁门);
    }
}
```



用来控制一个或者多个线程等待多个线程。

维护了一个计数器cnt，每次调用countDown()方法会让计数器的值减1，减到0的时候，那么因为调用await()方法而在等待的线程就会被唤醒。

![image-20200222192750156](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200222192750156.png)

```java
public class CountdownLatchExample {
    public static void main(String[] args) throws InterruptedException {
        final int totalThread = 10;
        CountDownLatch countDownLatch = new CountDownLatch(totalThread);
        ExecutorService executorService = Executors.newCachedThreadPool();
        for(int i = 0; i < totalThread; i++) {
            executorService.execute(() -> {
                System.out.print("run..");
                countDownLatch.countDown();
            });
        }
        countDownLatch.await(); //没有await可能不会输出end？
        System.out.println("end");
        executorService.shutdown();
    }
}

// run..run..run..run..run..run..run..run..run..run..end
```

### CyclicBarrier

用来控制多个线程相互等待，只有当多个线程都到达时，这些线程才会继续执行。

```
Number of parties still waiting. Counts down from parties to 0
* on each generation.  It is reset to parties on each new
* generation or when broken.


底层在做减。 count = parties
每次--count，直到0.
可以理解为做加，等待所有线程执行完完，都到达某一位置，再开始新的循环 
```

例子：

```java
psvm {
	CyclicBarrier cyclicBarrier = new CyclicBarrier(7, () -> {sout("召唤神龙")});
    
    for(int i = 1; i < 7; i++) {
        final int tempInt = i;
        new Thread(() -> {
            sout（收集到第i颗龙珠);
            try {
                cyclicBarrier.await();
            }
        }, String.valueOf(i)).start();
    }
}
```

和CountDownLatch相似，都是通过维护计数器来实现的。线程执行await()方法之后计数器会减1，并进行等待，直到计数器为0，所有调用await()方法而在等待的线程才能继续执行。

CyclicBarrier和CountDownLatch的一个**区别**是，**CyclicBarrier的计数器通过调用reset()方法可以循环使用**，所以它才叫做循环屏障。

CyclicBarrier有两个构造函数，其中parties指示计数器的初始值，barrierAction在所有线程都到达屏障时候会执行一次。

```java
public CyclicBarrier(int parties, Runnable barrierAction) {
    if (parties <= 0) throw new IllegalArgumentException();
    this.parties = parties;
    this.count = parties;
    this.barrierCommand = barrierAction;
}

pubilc CyclicBarrier(int parties) {
    this(parties, null);
}
```

![image-20200222194523389](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200222194523389.png)

```java
public class CyclicBarrierExample {
    public static void main(Stirng[] args) {
        final int totalThread = 10;
        CyclicBarrier cyclicBarrier = new CyclicBarrier(totalThread);
        ExecutorService executorService = Executors.newCachedThreadPool();
        for(int i = 0; i < totalThread; i++) {
            executorService.execute(() -> {
                System.out.print("before");
                try {
                    cyclicBarrier.await();
                } catch (InterruptedException | BrokenBarrierException) {
                    e.printStackTrace();
                }
                System.out.print("after");
            });
        }
        executorService.shutdown();
    }
}

// before..before..before..before..before..before..before..before..before..before..after..after..after..after..after..after..after..after..after..
```

### Semaphore

Semaphore类似于操作系统中的信号量，可以控制对互斥资源的访问线程数。

- 多个共享资源的互斥使用
- 并发线程数的控制（流量控制，比如数据库，控制只有10个线程通知获取数据库连接保存数据）

![image-20200530000956755](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200530000956755.png)

以下代码模拟了对某个服务的并发请求，每次只能有3个客户端同时访问，请求总数为10。

```java
public class SemphoreExample {
    public static void main(String[] args) {
        final int clientCount = 3;
        final int totalRequestCount = 10;
        Semaphore semaphore = new Semphore(clientCount);
        ExecutorService executorService = Executors.newCachedThreadPool();
        for(int i = 0; i < totalRequestCount; i++) {
            executorService.execute(() -> {
                try {
                    semaphore.acquire();
                    System.out.print(semaphore.availablePermits() + " ");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    semaphore.release();
                }
            });
        }
        executorService.shutdown();
    }
}

// 2 1 2 2 2 2 2 1 2 2
```

抢车位的例子：

```java
public class SemaphoreDemo {
    public static void main(String[] args) {
        Semaphore semaphore = new Semaphore(3); // 非公平，默认
        for (int i = 1; i <= 6; i++) {
            new Thread(() -> {
                try{
                    semaphore.acquire();
                    Sout(Thread.currentThread().getName() + "抢到车位");
                    sleep(3000); catch
                    sout(Thread.currentThread().getName() + "停车三秒离开");
                } catch (InteruptedException e) {
                    e.printStackTrace();
                } finally {
                    semphore.release();
                }
                
            }, "" + i).start();
        }
    }
}
```

### Exchanger——线程间交换数据

用于线程协作的工具类。提供一个同步点，在这个同步点，两个线程可以交换彼此的数据。通过exchange方法交换数据，如果第一个线程先执行exhcange方法，它会一直等待第二个线程页执行exchange方法，当两个线程都到达同步点时，这两个线程就可以交换数据，将本线程生产出来的数据传递给对方。

为避免一直等待，可以使用exchange(V x, long timeout, TimeUnit unit)方法。

场景：

- 遗传算法
- 校对工作，比如银行录入流水，两个人录，录完校对

```java
public class ExchangerTest {
    private static final Exchanger<String> exgr = new Exchanger<String>();
    private static ExecutorService threadPool = Executors.newFixedThreadPool(2);
    
    public static void main(String[] args) {
        threadPool.execute(new Runnable() {
            @Override
            public void run() {
                try {
                    String A = "银行流水A";
                    exgr.exchange(A);
                } catch (InterruptedException e) {
                    
                }
            }
        });
        
        threadPool.execute(new Runnable() {
            @Override
            public void run() {
                try {
                    String B = "银行流水B";
                    String A = exgr.exchange(B);
                    sout(A== B);
                } catch () {
                    
                }
            }
        });
        threadPool.shutdown();
    }
    
}
```



## J.U.C-其他组件

#### FutureTask

Callable可以有返回值，返回值通过FutureTask进行封装。FutureTask实现了RunnableFuture接口，该接口继承自Runnable和Future接口，这使得FutureTask既可以当作一个任务执行，也可以有返回值。

```java
public class FutureTask<V> implements RunnableFuture<V>
public interface RunnableFuture<V> extends Runnable, Future<V>
```

FutureTask可用于异步获取执行结果或取消执行任务的场景。当一个任务需要执行很长时间，那么就可以用FutureTask来封装这个任务，主线程在完成自己的任务之后再去获取结果。

```java
public class FutureTaskExample {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        FutureTask<Integer> futureTask = new FutureTask<Integer>(new Callable<Integer>() {
            @Override
            public Integer call() throws Exception {
                int result = 0;
                for (int i = 0; i < 100; i++) {
                    Thread.sleep(10);
                    result += i;
                }
                return result;
            }
        });
        
        Thread computeThread = new Thread(futureTask);
        computeThread.start();
        
        Thread otherTread = new Thread(() -> {
            System.out.println("Other task is running");
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        otherThread.start();
        System.out.println(futureTask.get());
    }
}

// other task is running...
// 4950
```

#### BlockingQueue

Java.util.concurrent.BlockingQueue接口有以下阻塞队列的实现：

- FIFO队列：LInkedBlockingQueue、ArrayBlockingQueue（固定长度）
- 优先级队列：PriorityBlockingQueue

提供了阻塞的take()和put()方法：如果队列为空take()将阻塞，直到队列中有内容，如果队列为满put()将阻塞，直到队列有空闲位置。

**使用BlockingQueue实现生产者消费者问题**

```java
public class ProducerConsumer {
	private static BlockingQueue<String> queue = new ArrayBlockingQueue<>(5);
    private static class Producer extends Thread {
        @Override
        public void run() {
            try {
                queue.put("product");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.print("produce...")
        }
    }
    private static class Consumer extends Thread {
        @Override
        public void run() {
            try {
                String product = queue.take();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.print("consume...");
        }
    }
}
public static void main(String[] args) {
    for (int i = 0; i < 2; i++) {
        Producer producer = new Producer();
        producer.start();
    }
    for (int i = 0; i < 5; i++) {
        Consumer consumer = new Consumer();
        consumer.start();
    }
    for (int i = 0; i < 3; i++) {
        Producer producer = new Producer();
        producer.start();
    }
}

// produce..produce..consume..consume..produce.consume..produce..consume..prodeuce..consume..
```

#### ForkJoin

主要用于并行计算，和MapReduce原理类似，都是把大的计算任务拆分成多个小任务并行计算。

```java
public class ForkJoinExample extends RecursiveTask<Integer> {
    private final int threshold = 5;
    private int first;
    private int last;
    
    public ForkJoinExample(int first, int last) {
        this.first = first;
        this.last = last;
    }
    
    @Override
    protected Integer compute() {
        int result = 0;
        if (last - first <= threshold) {
            // 任务足够小则直接计算
            for (int i = first; i <= last; i++) {
                result += i;
            }
        } else {
            // 拆分成小任务
            int middle = first + (last - first) / 2;
            ForkJoinExample leftTask = new ForkJoinExample(first, middle);
            ForkJoinExample rightTask = new ForkJoinExample(middle + 1, last);
            leftTask.fork();
            rightTask.fork();
            result = leftTask.join() + rightTask.join();
        }
        return result;
    }
}

public static void main(String[] args) throws ExecutionException,InterruptedException {
    ForkJoinExample example = new ForkJoinExample(1,1000);
    ForkJoinPool forkJoinPool = new ForkJoinPool();
    Future result = forkJoinPool.submit(example);
    System.out.println(result.get());
}
```

ForkJoin使用ForkJoinPool来启动，它是一个特殊的线程池，线程数量取决于CPU核数。

```java
public class ForkJoinPool extends AbstractExecutorService
```

ForkJoinPool实现了工作窃取算法来提高CPU的利用率。每个线程都维护了一个双端队列，用来存储需要执行的任务。工作窃取算法允许空闲的线程从其他线程的双端队列中窃取一个任务来执行。窃取的任务必须是最晚的任务，避免和队列所属线程发生竞争。例如下图中，Thread2从Thread1的队列中拿出最晚的Task1任务，Thread1会拿出Task2来执行，这样就避免发生竞争。但是如果队列中只有一个任务时还是会发生竞争。

![image-20200223223532703](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200223223532703.png)

## 线程不安全示例

### 集合类的不安全问题

#### ArrayList

```java
public static void main(String[] args) {
    List<String> list = new ArrayList<>();
    
    for(int i = 0; i < 30; i++) {
        new Thread(() -> {
            list.add(UUID.randomUUID().toString().subString(0,8));
            // concurrentModificationException
            sout(list);
        }).start();
    }
}
```

##### 故障现象

ConncurrentModificationException：并发修改异常

##### 原因导致

并发写

##### 解决方案

1. Vector
2. Collections（集合的工具类)：Collecyions.synchronizedList(new ArrayList<>());
3. juc包下的类：CopuOnWriteArrayList<>(); 写时复制 读写分离 读多写少场景

#### HashSet

```
同上
```

##### 故障现象

ConncurrentModificationException：并发修改异常

##### 原因导致

并发写

##### 解决方案

1. Collections.synchronizedSet(new HashSet<>());
2. juc下的类：CopyOnWriteArraySet<>(); 底层实际上是CopyOnWriteArrayList

#### HashMap

```
同上
```

##### 故障现象

ConncurrentModificationException：并发修改异常

##### 原因导致

并发写

##### 解决方案

1. Collections.synchronizedMap()
2. ConncurrentHashMap



//如果多个线程同时对一个共享数据进行访问而不采取同步操作的话，那么操作的结果是不一致的。

以下代码演示了1000个今年初同时对cnt执行自增操作，操作结束之后它的值有可能小于1000。

```java
public class ThreadUnsafeExample {
    private int cnt = 0;
    public void add() {
        cnt++;
    }
    public int get() {
        return cnt;
    }
}

public static void main(String[] args) thorws InterruptedException {
    final int threadSize = 1000;
    ThreadUnsafeExample example = new ThreadUnsafeExample();
    final CountDownLatch countDownLacth = new CountDownLatch(threadSize);
    ExecutorService executorService = Executors.newCachedThreadPool();
    for(int i = 0; i < threadSize; i++) {
        executorService.execute(() -> {
            example.add();
            countDownLatch.countDown();
        });
    }
    countDownLatch.await();
    executorService.shutdown();
    System.out.println(example.get());
}

// 997
```

## Java内存模型

Java内存模型试图屏蔽各种硬件和操作系统的内存访问差异，以实现让Java程序在各种平台下都能达到一致的内存访问效果。

#### 主内存与工作内存

处理器上的寄存器的读写的速度比内存快几个数量级，为了解决这种速度矛盾，在它们之间加入了高速缓存。

加入高速缓存带来一个新的问题：缓存一致性。如果多个缓存共享同一块主内存区域，那么多个缓存的数据可能会不一致，需要一些协议来解决这个问题。

![image-20200223230950269](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200223230950269.png)

所有的变量都存储在主内存中，每个线程还有自己的工作内存，工作内存在高速缓存或者寄存器中，保存了该线程使用的变量的主内存副本拷贝。

线程只能直接操作工作内存中的变量，不同线程之间的变量值传递需要通过主内存来完成。

![image-20200223231428968](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200223231428968.png)

#### 内存间交互操作

Java内存模型定义了8个操作来完成主内存和工作内存的交互操作。

![image-20200223231555784](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200223231555784.png)

- read：把一个变量的值从主内存传输到工作内存
- load：在read之后执行，把read得到的值放入工作内存的变量副本中
- use：把内存中一个变量的值传递给执行引擎
- assign：把一个从执行引擎接收到的值复制给工作内存的变量
- store：把工作内存的一个变量的值传送到主内存中
- write：在store之后执行，把store得到的值放入主内存的变量中
- lock：作用于主内存的变量
- unlock

#### 内存模型三大特性

- 原子性

  Java内存模型保证了read、load、use、assign、store、write、lock和unlock操作具有原子性，例如对一个int类型的变量执行assign赋值操作，这个操作就是原子性的。但是Java内存模型允许虚拟机没有被volatile修饰的54位数据（long，double）的读写操作划分为两次32位的操作来运行，即load、store、read和write操作可以不具备原子性。

  有一个错误的认识，int等原子性的类型在多线程环境中不会出现线程安全问题。前面的线程不安全示例代码中，cnt属于int型变量，1000个线程对它进行自增操作之后，得到的值是997而不是1000。

  为了方便讨论，将内存间的交互操作简化为3个：load、assign、store

  下图演示了两个线程同时对cnt进行操作，load、assign、store这一系列操作整体上看不具备原子性，那么在T1修改cnt并且还没有将修改后的值写入主内存，T2仍然可以读入旧值。可以看出，这两个线程虽然执行了两次自增运算，但是主内存中cnt的值最后为1而不是2。因此对int类型读写操作满足原子性只是说明load、assign、store这些单个操作具备原子性。

  ![image-20200223233316223](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200223233316223.png)

  AtomicInteger能保证多个线程修改的原子性。

  ![image-20200223233413852](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200223233413852.png)

  使用AtomicInteger重写之前线程不安全的代码之后：

  ```java
  public class AtomicExample {
  	private AtomicInteger cnt = new AtomicInteger();
      
      public void add() {
          cnt.incremenetAndGet();
      }
      
      public int get() {
          return cnt.get();
      }
  }
  
  public static void main(String[] args) throws InterruptedException {
      final int threadSize = 1000;
      AtomicExample example = new AtomicExample(); // 只修改了这句
      final CountDownLatch countDownLatch = new CountDownLatch(threadSize);
      ExecutorService executorService = Executors.newCachedThreadPool();
      for(int i = 0; i < threadSize; i++) {
          executorService.execute(() -> {
              example.add();
              countDownLatch.countDown();
          });
      }
      countDownLatch.await();
      executorService.shutdown();
      System.out.println(example.get());
  }
  
  // 1000
  ```

  除了使用原子类之外，也可以使用synchronized互斥锁来保证操作的原子性。它对应的内存间交互操作为：lock和unlock，在虚拟机实现上对应的字节码指令为monitorenter和monitorexit。

  ```java
  public class AtomicSynchronizedExample {
      private int cnt = 0;
      public synchronized void add() {
          cnt++;
      }
      public synchronized int get() {
          return cnt;
      }
  }
  
  public static void main(String[] args) throws InterruptedException {
      final int threadSize = 1000;
      AtomicSynchronizedExample example = new AtomicSynchronizedExample();
      final CountDownLatch countDownLatch = new CountDownLatch(threadSize);
      ExecutorService executorService = Executors.newCachedThreadPool();
      for(int i = 0; i < threadSize; i++) {
          executorService.execute(() -> {
              example.add();
              countDownLatch.countDown();
          });
      }
      countDownLatch.await();
      executorService.shutdown();
      System.out.println(example.get());
  }
  
  // 1000
  ```

- 可见行

  可见性指当一个线程修改了共享变量的值，其他线程能够立即的值这个修改。Java内存模型是通过在变量修改后将新值同步回主内存，在变量读取前从主内存刷新变量值来实现可见性的。

  主要有三种实现方式：

  - volatile
  - synchronized，对一个变量执行unlock操作之前，必须把变量值同步回主内存。
  - final，被final关键字修饰的字段在构造器中一旦初始化完成，并且没有发生this逃逸（其他线程通过this引用访问到了初始化了一般的对象），那么其他线程就能看见final字段的值。

  对前面的线程不安全示例中的cnt变量使用volatile修饰，不能解决线程不安全问题，因为**volatile并不能保证操作的原子性**。

- 有序性

  有序性是指：在本线程内观察，所有操作都是有序的。在一个线程观察另一个线程，所有操作都是无序的，无序是因为发生了指令重排。在Java内存模型中，允许编译器和处理器对指令进行重排序，重排序过程不会影响到单线程程序的执行，却会影响到多线程并发执行的正确性。

  volatile关键字通过添加内存屏障的方式来禁止指令重排，即重排序时不能把后面的指令放到内存屏障之前。

  也可以通过synchronized来保证有序性，它保证每个时刻只有一个线程执行同步代码，相当于是让线程顺序执行同步代码。

#### 先行发生原则

上面提到了可以用volatile和synchronized来保证有序性。除此之外，JVM还规定了先行发生原则，让一个操作无序控制就能先于另一个操作完成。

##### 单一线程原则

在一个线程内，在程序前面的操作先行发生于后面的操作。

![image-20200224000147515](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200224000147515.png)

##### 管程锁定规则

一个unlock操作先行发生于后面对同一个锁的lock操作。

![image-20200224000257001](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200224000257001.png)

##### volatile变量规则

对一个volatile变量的写操作先行发生于后面对这个变量的读操作。

![image-20200224000347263](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200224000347263.png)

##### 线程启动规则

Thread对象的start()方法调用先行发生于此线程的每一个动作。

![image-20200224000420126](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200224000420126.png)

##### 线程加入规则

Thread对象的结束先行发生于join()方法返回。

![image-20200224000455617](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200224000455617.png)

##### 线程中断规则

对线程interrupt()方法的调用先行发生于被中断线程代码检测到中断事件的发生，可以通过interrupted()方法检测到是否有中断发生。

##### 对象终结规则

一个对象的初始化完成（构造函数执行结束）先行发生于它的finalize()方法的开始。

##### 传递性

如果操作A先行发生于操作B，操作B先行发生于操作C，呢么操作A先行发生于操作C。

## 线程安全

多个线程不管以何种方式访问某个类，并且在主调代码中不需要进行同步，都能表现正确的行为。

线程安全有以下几种实现方式：

#### 不可变

不可变的对象一定是线程安全的，不需要再采取任何的线程安全保障措施。只要一个不可变的对象被正确地构造出来，永远也不会看到它在多个线程之中处于不一致的状态。在多线程环境下，应当尽量使对象成为不可变，来满足线程安全。

不可变的类型：

- final关键字修饰的基本数据类型
- String
- 枚举类型
- Number部分子类，如Long和Double等数值包装类型，BigInteger和BigDecimal等大数据类型。但同为Number的原子类AtomicInteger和AtomicLong则是可变的。

对于集合类型，可以使用Collections.unmodifiableXXX()方法来获取一个不可变集合。

```java
public class ImmutableExample {
    public static void main(String[] args) {
        Map<String, Integer> map = new HashMap<>();
        Map<String, Integer> unmodifiableMap = new Collections.unmodifiableMap(map);
        unmodifiableMap.put("A", 1);
    }
}

Exception in thread "main" java.lang.UnsupportOperationException......
```

Collections.unmodifiableXXX()先对原始的集合进行拷贝，需要对集合进行修改的方法都直接抛出异常。

```java
public V put(K key, V value) {
	throw new UnsupportedOperationException();
}
```

#### 互斥同步

synchronized和ReentrantLock

#### 非阻塞同步

互斥同步最主要的问题就是线程阻塞和唤醒所带来的性能问题，因此这种同步也称为阻塞同步。

互斥同步属于一种悲观的并发策略，总是认为只要不去做正确的同步措施，那就肯定会出现问题。无论共享数据是否真的会出现竞争，它都要进行加锁（这里讨论的是概念模型，实际上虚拟机会优化掉很大一部分不必要的加锁）、用户态和心态转换、维护锁计数器和检查是否有被阻塞的线程需要唤醒等操作。

随着硬件指令集的发展，我们可以使用基于冲突检测的乐观并发策略：先进行操作，如果没有其他线程争用共享数据，那么操作就成功了，否则采取补偿措施（不断地重试，直到成功为止）。这种乐观的并发策略的许多实现都不需要将线程阻塞，因此这种同步操作称为非阻塞同步。

- CAS

  乐观锁需要操作和冲突检测这两个步骤具备原子性，这里就不能再使用互斥同步来保证了，只能靠硬件来完成。硬件支持的原子性操作最典型的是：比较并交换（Compare-and-Swap，CAS）。CAS指令需要有3个操作数，分别是内存地址V、旧的预期值A和新值B。当执行操作时，只有当V的值等于A，才将V的值更新为B。

- AtomicInteger

  J.U.C包里面的整数原子类AtomicInteger的方法调用了Unsafe类的CAS操作。

  以下代码使用了AtomicInteger执行了自增操作。

  ```java
  private AtomicInteger cnt = new AtomicInteger();
  
  public void add() {
      cnt.incrementAndGet();
  }
  ```

  以下代码是incrementAndGet()的源码，它调用了Unsafe的getAndAddInt()

  ```java
  public final int incremenetAndGet() {
      return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
  }
  ```

  以下代码是getAndAddInt()源码，var1指示对象内存地址，var2指示该字段相对对象内存地址的偏移，var4指示操作需要加的数值，这里为1。通过getIntVolatile(var1, var2)得到旧的预期值，通过调用compareAndSwapInt()来进行CAS比较，如果该字段内存地址中的值等于var5，那么就更新内存地址为var1+var2的变量为var5+var4。

  可以看到getAdnAddInt()在一个循环中进行，发生冲突的做法是不断的进行重试。

  ```java
  public final int getAndAddInt(Object var1, long var2, int var4) {
      int var5;
      do {
          var5 = this.getIntVolatile(var1, var2);
      } while (!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
      return var5;
  }
  ```

- ABA

  如果一个变量初次读取的时候是A值，它的值被改成了B，后来又被改回为A，那么CAS操作就会误认为它从来没有被改变过。

  J.U.C包提供了一个带有标记的原子引用类AtomicStampedReference来解决，它可以通过控制变量值的版本来保证CAS的正确性。大部分情况下ABA问题不会影响程序并发的正确性，如果需要解决ABA问题，改用传统的互斥同步可能会比原子类更高效。

#### 无同步方案

要保证线程安全，并不是一定就要进程同步。如果一个方法本来就不涉及共享数据，那它自然就无须任何同步措施去保证正确性。

##### 栈封闭

多个线程访问同一个方法的局部变量时，不会出现线程安全问题，因为局部变量存储在虚拟机栈中，属于线程私有的。

```java
public class StackClosedExample {
    public void add100() {
        int cnt = 0;
        for (int i = 0; i < 100; i++) {
            cnt++;
        }
        System.out.println(cnt);
    }
}

public static void main(String[] args) {
    StackClosedExample example = new StackCloesdExample();
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> example.add100());
    executorService.execute(() -> example.add100());
    executorService.shutdown();
}

// 100 100
```

##### 线程本地存储（Thread Local Storage）

如果一段代码中所需要的数据必须与其他代码共享，那就看看这些共享数据的代码是否能保证在同一个线程中执行。如果能保证，我们就可以把共享数据的可见范围限制在同一个线程之内，这样，无须同步也能保证线程之间不出现数据争用的问题。

符合这种特点的应用并不少见，大部分使用消费队列的架构模式（如”生产者-消费者“模式）都会将产品的消费过程尽量在一个线程中消费完。其中最重要的一个应用实例就是经典Web交互模型中的”一个请求对应一个服务器线程“（Thread-per-Request）的处理方式，这种处理方式的广泛应用使得很多Web服务端应用都可以使用线程本地存储来解决线程安全问题。

可以使用java.lang.ThreadLocal类来实现线程本地存储功能。对于以下代码，thread1中设置threadLocal为1，而thread2设置threadLocal为2。过了一段时间之后，thread1读取threadLocal依然是1，不受thread2影响。

```java
public class ThreadLocalExample {
    public static void main(String[] args) {
        ThreadLocal threadLocal = new ThreadLocal();
        Thread thread1 = new Thread(() -> {
            threadLocal.set(1);
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(threadLocal.get());
            threadLocal.remove();
        });
        Thread thread2 = new Thread(() -> {
            threadLocal.set(2);
            threadLocal.remove();
        });
        thread1.start();
        thread2.start();
    }
}

/// 1
```

理解：

```java
public class ThreadLocalExample1 {
    public static void main(String[] args) {
        ThreadLocal threadLocal1 = new ThreadLocal();
        ThreadLocal threadLocal2 = new ThreadLocal();
        Thread thread1 = new Thread(() -> {
            threadLocal1.set(1);
            threadLocal2.set(1);
        });
        Thread thread2 = new Thread(() -> {
            threadLocal1.set(2);
            threadLocal2.set(2);
        });
        thread1.start();
        thread2.start();
    }
}
```

对应底层结构：

![image-20200224010405902](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200224010405902.png)

每个Thread都有一个ThreadLocal.ThreadLocalMap对象

```java
ThreadLocal.ThreadLocalMap threadLocals = null;
```

当调用一个ThreadLocal的set(T value)方法时，先得到当前线程的ThreadLocalMap对象，然后将ThreadLocal->value键值对插入到该Map中。

```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if(map != null) {
        map.set(this, value);
    } else {
        createMap(t, value);
    }
}

public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if(e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
```

ThreadLocal从理论上讲并不是用来解决多线程并发问题的，因为根本不存在多线程竞争。

在一些场景（尤其是使用线程池）下，由于ThreadLocal.ThreadLocalMap的底层数据结构导致ThreadLocal有内存泄露的情况，应该尽可能在每次使用ThreadLocal后手动remove()，以避免出现ThreadLocal经典的内存泄漏甚至是造成自身业务混乱的风险。

##### 可重入代码（Reentrant Code）

这种代码也叫做纯代码（Pure Code），可以在代码执行的任何时刻中断它，转而去执行另一段代码（包括递归调用它本身），而在控制权返回后，原来的程序不会出现任何错误。

可重入代码有一些共同的特征，例如不依赖存储在堆上的数据和公用的系统资源、用到的状态量都由参数中传入、不调用非可重入的方法等。

## Java锁

### 公平锁与非公平锁

公平锁：先来后到获得锁，按照申请锁的顺序来获得锁，Lock可以是公平锁

非公平锁：随机竞争锁，不按顺序，有可能会造成优先级反转或者饥饿现象。优点：吞吐量高。Synchronized，Lock

ReentrantLock：构造函数传false，非公平，true，公平。默认非公平

### 可重入锁（递归锁）

指的是铜线线程外层函数获得锁之后，内层递归函数仍能获取该锁的代码，在同一个线程在外层方法获取锁的时候，在内层方法会自动获取锁。

也就是说，线程可以进入任何一个它已经拥有的锁所同步着的代码块。

如Synchronized、ReentrantLock，主要是为了避免死锁。

```java
class Phone implement Runnable {
    public synchronized void sendSMS() throws Exception {
        sout(Thread.currentThread().getId() + " invoke sendSMS");
        sendEmail();
    }
             
    public synchronized void sendEmail() throws Exception {
        sout("invoke email");
    }
    
    Lock lock = new ReentrantLock();
    
    public void run() {
        get();
    }
    
    void get() {
        lock.lock();
        // lock.lock();
        try{
            sout();
            set();
        } finally {
            lock.unlock();
            // lock.unlock(); 可以多次加锁解锁，必须匹配，否则死锁
        }
        
    }
    void set() {
        try{
            sout();
        } finally {
            lock.unlock();
        }
    }
}

psvm {
    Phone phone = new Phone();
    new Thread(() -> {
        phone.sendSMS();
    }, "t1").start();
    new Thread(() -> {
        phone.sendSMS();
    }, "t2").start();
}
```

### 自旋锁

指尝试获取锁的线程不得不会立即阻塞，而是采用循环等待的方式再去尝试获取锁，可以减少线程上下文切换的小号，缺点是循环会消耗CPU。

手写自旋锁：

```java
public class SpinLockDemo {
    // 原子引用线程
    AtomicReference<Thread> atomicReference = new AtomicReference();
    
    public void myLock() {
        Thread thread = thread.currentThread();
        sout("come in");
        while (!atomicReference.compareAndSet(null, thread)) {
        	      
        }  
        
        // lock done
    }
    
    public void myUnLock() {
        Thread thread = thread.currentThread();
        atomicReference.compareAnsSet(thread, null);
        sout("unlock");
    }
    
    public static void main(String[] args) {
        SpinLoackDemo spin = new SpinLockDemo();
        new Thread(() -> {
            spin.myLock();
            sleep(5);
            spin.myUnLock();
        }, "aa").start();
        
        sleep(1);
        new Thread(() -> {
            spin.myLock();
            spin.myUnLock();
        }, "bb").start();
        
        
    }
}
```



### 独占锁与共享锁

独占锁：锁一次只能被一个线程池游，Reent、synch

共享锁：该锁可以被多个线程持有，ReentrantReadWriteLock

读写锁，读读共存，读写不共存，写写不共存实例：

```java
// 写操作：原子+独占，整个过程必须是一个完整的

class MyCache { //资源类
    private volatile Map<String, Object> map = new HashMap<>();
    // priavte Lock lock = new ReentrantLock(); 粒度太粗
    private ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();
    
    public void put(String key, Object value) {
        rwLock.writeLock().lock();
        try {
        	sout("正在写入");
        	sleep(300); //模拟网络延迟
        	map.put(key, value);
        	sout("写入完成");
        } finally {
            rwLock.writeLock().unlock();
        }
    }
    
    public void get(String key) {
        rwLock.readLock().lock();
        try {
            sout正在读取;
            sleep(300);
            Object result = map.get(key);
            sout读取完成;
        } finally {
            rwLock.readLock().unlock();
        }
            
    }
    
    
    
}

public class ReadWriteLockDemo {
    public static void main(String[] args) {
        MyCache myCache = new MyCache();
        for(int i = 0; i < 5; i++) {
            final int tempInt = i;
            new Thread(() -> {
                myCache.put(tempInt+"", tempInt +"");
            }).start();
            new Thread(() -> {
                myCache.get(tempInt+"");
            }).start();
        }
    }
}
```



### 锁优化

此处锁优化主要指JVM对synchronized的优化。

#### 自旋锁

互斥同步进入阻塞状态的开销都很大，应该尽量避免。在许多应用中，共享数据的锁定状态只会持续很短一段时间。自旋锁的思想是让一个线程在请求一个共享数据的锁时执行忙循环（自旋）一段时间，如果在这段时间内能获得锁，就可以避免进入阻塞状态。

自旋锁虽然能避免进入阻塞状态从而减少开销，但是它需要进程忙循环操作占用CPU时间，它只适用于共享数据的锁定状态很短的场景。

在JDK1.6中引入了自适应的自旋锁。自适应意味着自旋的次数不再固定了，而是由前一次在同一个锁上的自旋次数及锁的拥有者的状态来决定。

#### 锁消除

锁消除是指对于被检测出不可能存在竞争的共享数据进行锁消除。

锁消除主要是通过逃逸分析来支持，如果堆上的共享数据不可能逃逸出去被其他线程访问到，那么就可以把它们当成私有数据对待，也就可以将它们的锁进行消除。

对于一些看起来没有加锁的代码，其实隐式地加了很多锁。例如，下面的字符串拼接代码就隐式加了锁：

```java
public static String concatString(String s1, String s2, String s3) {
    return s1 + s2 + s3;
}
```

String是一个不可变的类，编译器会对String的拼接自动优化。在JDK1.5之前，会转化为StringBuffer对象的连续append()操作。

```java
public static String concatString(String s1, String s2, String s3) {
    StringBuffer sb = new StringBufer();
    sb.append(s1);
    sb.append(s2);
    sb.append(s3);
    return sb.toString();
}
```

每个append()方法中都有一个同步块。虚拟机观察变量sb，很快会发现它的动态作用于被限制在concatString()方法内部。也就是说，sb的所有引用永远不会逃逸到concatString()方法之外，其他线程无法访问到它，因此可以进行消除。

#### 锁粗化

如果一系列的连续操作都对同一个对象反复加锁和解锁，频繁的加锁操作就会导致性能损耗。

上一节的示例代码中连续的append()方法就属于这类情况。如果虚拟机探测到由这样的一串零碎的操作都对同一个对象加锁，将会把加锁的范围扩展（粗化）到整个操作序列的外部。对于上一节的示例代码就是扩展到第一个append()操作之前直至最后一个append()操作之后，这样只需要加锁一次就可以了。

#### 轻量级锁

JDK1.6引入了偏向锁和轻量级锁，从而让锁拥有了四个状态：无锁状态（unlock）、偏向锁状态（biasble）、轻量级锁状态（lightweight locked）和重量级锁状态（inflated）。

以下是HotSpot虚拟机对象头的内存布局，这些数据被称为Mark Word。其中tag bits对应了五个状态，这些状态在右侧的state表格中给出。除了marked for gc状态，其他四个状态已经在前面介绍过了。

![image-20200224183016766](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200224183016766.png)

下图左侧是一个线程的虚拟机栈，其中有一部分称为Lock Record的区域，这是在轻量级锁运行过程创建的，用于存放锁对象的Mark Word。而右侧就是一个锁对象，包含了Mark Word和其他信息。

![image-20200224183607831](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200224183607831.png)

轻量级锁是相对于传统的重量级锁而言，它使用CAS操作来避免重量级锁使用互斥量的开销。对于绝大部分的锁，在整个同步周期内都是不存在竞争的，因此也就不需要都使用互斥量进行同步，可以先采用CAS操作进行同步，如果是CAS失败了再改用互斥量进行同步。

当尝试获取一个锁对象时，如果锁对象标记为0 01，说明锁对象的锁为未锁定(unlock)状态。此时虚拟机在当前线程的虚拟机栈中创建Lock Record，然后使用CAS操作将对象的Mark Word更新为Lock Record指针。如果CAS操作成功了，那么线程就获取了该对象上的锁，并且对象的Mark Word的锁标记变为00，表示该对象处于轻量级锁状态。

![image-20200224184545171](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200224184545171.png)

如果CAS操作失败了，虚拟机首先会检查对象的Mark Word是否指向当前线程和虚拟机栈，如果是的话说明当前线程已经拥有了这个锁对象，那就可以直接进入同步块继续执行，否则说明这个锁对象已经被其他线程抢占了。如果有两条以上的线程争用同一个锁，那轻量级锁就不再有效，要膨胀为重量级锁。

#### 偏向锁

偏向锁的思想是偏向于第一个获取锁对象的线程，这个线程在之后获取该锁就不再需要进行同步操作，甚至连CAS操作也不再需要。

当锁对象第一次被线程获得的时候，进入偏向状态，标记为1 01。同时使用CAS操作将线程ID记录到Mark Word中，如果CAS操作成功，这个线程以后每次进入这个锁相关的同步块就不需要再进行任何同步操作。？？？？

当有另一个线程去尝试获取这个锁对象时，偏向状态就宣告结束，此时撤销偏向（Revoke Bias）后恢复到韦锁定状态或者轻量级锁状态。

![image-20200224185308430](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200224185308430.png)

## 死锁编码及定位分析

### 死锁?

两个或多个线程，在执行过程中因争抢资源造成的一种互相等待的现象，如无外力干涉都将无法推进。

![image-20200530174547027](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200530174547027.png)

### 写个死锁

```java
/**
 * @program untitled
 * @description:
 * @author: zhaojiangdong
 * @create: 2020/05/30 17:48
 */

class HoldLockThread implements Runnable {

    private Object LockA;
    private Object LockB;

    public HoldLockThread(Object lockA, Object lockB) {
        LockA = lockA;
        LockB = lockB;
    }

    @Override
    public void run() {
        synchronized (LockA) {
            System.out.println("hold a, try to get b");
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            synchronized (LockB) {

            }

        }
    }
}
public class DeadLockDemo {
    public static void main(String[] args) {
        Object lockA = new Object();
        Object lockB = new Object();

        new Thread(new HoldLockThread(lockA, lockB)).start();
        new Thread(new HoldLockThread(lockB, lockA)).start();

        System.out.println("done");

    }
}

```

### 如何定位死锁

两大命令，证明死锁并解决。

- jps：java版的ps，查找编号
- jstack：根据编号看

或jconsole。

## 多线程开发良好的实践

- 给线程起个有意义的名字，方便找bug；
- 缩小同步范围，从而减少锁争用。例如对于synchronized，应该尽量使用同步快而不是同步方法；
- 多用同步工具少用wait()和notify()。首先，CountDownLatch，CyclicBarrier，Semaphore和Exchanger这些同步类简化了编码操作，而用wait()和notify()很难实现复杂控制流；其次，这些同步类是由最好的企业编写和维护，在后续的JDK中还会不断优化和完善；
- 使用BlockingQueue实现生产者消费者队列；
- 多用并发集合少用同步集合，例如应该用ConncurrentHashMap而不是Hashtable；
- 使用本地变量和不可变类来保证线程安全
- 使用线程池而不是直接创建线程，这是因为创建线程代价很高，线程池可以有效地理由有限的线程来启动任务。

## 经典问题

### 哲学家就餐问题

实际上为如何避免死锁，三种方法：

- 限定哲学家就餐的数量，最多有4个哲学家尝试拿筷子
- 拿筷子时加锁，只能同时拿起左右两只筷子
- 就餐策略，奇数哲学家只能先拿左筷子，偶数哲学家先拿右边筷子