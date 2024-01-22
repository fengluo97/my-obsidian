# 进程与线程
- 进程是资源分配的基本单位，线程是CPU调度执行的基本单位
- 进程有独立的地址空间和程序代码等资源，切换开销较大；线程共享同一进程的地址空间和程序代码等资源，切换开销小（进程虚拟地址空间切换后导致页表也要切换，进而导致Cache失效，命中率降低，虚拟地址转换为物理地址就会变慢，而线程无需切换地址空间）
- 进程可以包含多个线程，但是一个线程只能属于一个进程

# Java 多线程
## 创建与执行
### 继承 Thread 类


### 实现 Runnable 接口


### 使用 FutureTask 实现 Callable 接口


## 线程执行原理
### 栈帧
每个方法调用会产生一个栈帧，栈帧包含了局部变量表、返回地址、操作数栈、锁记录等。

### 多线程
多个线程之间的方法调用栈独立。

### 上下文切换
因为一些原因导致CPU不在执行当前线程，转而执行另一个线程的代码。
- 线程的CPU时间片用完了
- 垃圾回收的安全点导致用户线程暂停
- 更高优先级的线程需要运行
- 线程自身调用了 sleep、yield、wait、join、park、synchronized、lock 等方法
上下文切换（Thread Context switch）发生时，需要操作系统保存当前线程的运行状态，并恢复另外一个线程的状态，Java中就是通过程序计数器来记录线程运行的状态，它指向了下一条JVM指令的地址。
- 状态包含了程序计数器、虚拟机栈中的每个栈帧的信息，如局部变量、操作数栈、返回地址等
- 频繁的上下文切换会影响性能。

## 常用方法
1、**run**() 方法，相当于执行一次普通方法。
2、**start**() 方法，启动线程，并执行任务，只能被调用一次。

3、**sleep**() 方法，让当前线程休眠一定的时间，并从 Running 状态进入到 Timed_waiting 状态。
4、**yield**() 方法，即让出，让当前线程从 Running 进入到 Runnable 就绪状态，然后调度执行其他线程。
5、线程优先级，设置线程优先级可以让操作系统来决定获得CPU执行权的线程，但是并不是绝对的。优先级和 yield 都是不可靠的线程调度。

6、**join**() 方法，**同步等待**该线程运行结束，可选等待超时时间。
7、**interrupt**() 方法，可以打断一个线程。如果打断了sleep()、wait()、join() 则会清楚打断标记，并抛出中断异常，通过异常来感知被打断。如果是正在运行中的线程，则会将打断标记设置为 true。可以通过 **isInterrupted**() 来判断是否被打断，来做一些其他的处理，isInterrupted() 不会重置打断标记。
8、**interrupted**() 静态方法，与 isInterrupted() 方法不同的是，该方法在获得打断标记之后会重置清楚打断标记。

## 两阶段终止模式
```java
class TowPhaseTermination {  
  
	private Thread monitor;  
	  
	public void start() {  
		monitor = new Thread(() -> {  
			while (true) {  
				if (Thread.currentThread().isInterrupted()) {  
					// 优雅关闭  
					System.out.println("优雅关闭。。");  
					break;  
				}  
				try {  
					Thread.sleep(1000);  
					System.out.println("执行监控操作");  
				} catch (InterruptedException e) {  
					e.printStackTrace();  
					Thread.currentThread().interrupt();  
				}  
			}  
		});  
		monitor.start();  
	}  
	  
	public void stop() {  
		monitor.interrupt();  
	}  
  
}
```

9、不推荐使用的方法，stop()，暴力停止线程，不会释放资源，容易造成死锁。
10、不推荐使用的方法，suspend()，挂起暂停线程。
11、不推荐使用的方法，resume()，恢复运行线程。

## 线程状态
NEW：新建
RUNNABLE：操作系统层面的阻塞、就绪、运行
TIMED_WAITING：有等待时间限制的等待，Thread.sleep()
WAITING：等待，join() 方法
BLOCKED：阻塞，synchronized
TERMINATED：终止

# 共享模型
## 临界区 Critical Section
一个程序运行多个贤臣本身是没有问题的
问题出现在多个线程访问共享资源
- 多个线程读共享资源其实也没有问题
- 如果对共享资源读写操作时发生指令交错，就会出现问题
一个段代码块内如果存在对共享资源的多线程读写操作，就称这段代码为临界区。

竞态条件：计算的正确性取决于多个线程的交替执行时序时，就会发生竞态条件。

## 线程安全分析
成员变量和静态变量是否线程安全？
- 如果它们没有共享，则线程安全
- 如果它们被共享了，根据它们的状态是否能够改变，又分为
	- 如果只有读操作，则线程安全
	- 如果只有读写操作，则这段代码是临界区，需要考虑线程安全问题

局部变量是否线程安全？
- 局部变量是线程安全的，因为栈帧的存在，每个线程都有一个独立的局部变量表
- 局部变量引用的对象则未必，因为引用了堆内存中的对象，堆内存是线程共享的 
	- 如果该对象没有逃离方法的作用访问，它是线程安全的
	- 如果该对象逃离方法的作用范围，则需要考虑线程安全

### 线程安全类
String 不可变类
StringBuffer
Random
Interger 不可变类
Vector
Hashtable
juc下的类
当有多个线程调用它们的同一个实例的某一个方法时，它是线程安全的。

## Synchronized
使用对象锁保证了临界区内的代码的原子性。
加在方法上锁的是 this 对象（不同类实例级别），加在类上锁的是 class 对象（类/JVM 级别）。

## Monitor
### Java 对象头

### Monitor
每个 Java 对象都可以关联一个 Monitor 对象，如果使用 synchronized 给对象上锁（重量级）之后，该对象头的 Mark Word 就会被设置指向 Monitor 对象的指针。Monitor 对象包含了 Owner、EntryList、WaitSet。


## Wait/Notify
wait() 方法，让进入 object 监视器的线程到 waitSet 等待
notify() 方法，在 object 上正在 waitSet 等待的线程中挑一个唤醒
notifyAll() 方法，让 object 上正在 waitSet 等待的线程全部唤醒
它们都是线程之间进行协作的手段，都属于 Object 对象的方法，**必须获得此对象的锁，才能调用这几个方法。**

使用模板
```java
synchronized (lock) {
	while (条件不成立) {
		lock.wait();
	}
	// 干活
}

synchronized (lock) {
	lock.notifyAll();
}
```

## 保护性暂停
通过 wait/notify 机制实现。



# 非共享模型
