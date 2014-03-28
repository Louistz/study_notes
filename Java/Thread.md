Thread
===
[Core JAVA]()

##多任务
并发的进程数目并不是由CPU数目决定的，操作系统将ＣＰＵ的时间片分配给每个进程，给人并行处理的感觉 。  
多进程和多线程的区别，本质在于每个进程有自己的一套变量，而进程则是共享数据。
##java的线程
1. 实现 Runnable接口，实现run()方法.或者继承Thread类，定义一个线程（不推荐，多任务考虑用线程池解决）
2. Runnable run = new RunnableImpl(); Thread r = new Thread(run);
3. r.start();  （不可执行 run()方法，直接执行run,只会执行同一个线程中的任务，而不会启动新线程。而start才是新建一个执行run的新线程）

###线程中断
Java中断机制是一种协作机制，也就是说通过中断并不能直接终止另一个线程，而需要被中断的线程自己处理中断。这好比是家里的父母叮嘱在外的子女要注意身体，但子女是否注意身体，怎么注意身体则完全取决于自己。  
Java中断模型也是这么简单，每个线程对象里都有一个boolean类型的标识（不一定就要是Thread类的字段，实际上也的确不是，这几个方法最终都是通过native方法来完成的），代表着是否有中断请求（该请求可以来自所有线程，包括被中断的线程本身）。例如，当线程t1想中断线程t2，只需要在线程t1中将线程t2对象的中断标识置为true，然后线程2可以选择在合适的时候处理该中断请求，甚至可以不理会该请求，就像这个线程没有被中断一样。

java.lang.Thread类提供了几个方法来操作这个中断状态，这些方法包括：

* public static boolean interrupted

测试当前线程是否已经中断。线程的中断状态 由该方法清除。换句话说，如果连续两次调用该方法，则第二次调用将返回 false（在第一次调用已清除了其中断状态之后，且第二次调用检验完中断状态前，当前线程再次中断的情况除外）。

* public boolean isInterrupted()

测试线程是否已经中断。线程的中断状态不受该方法的影响。

* public void interrupt()

中断线程。  

其中，interrupt方法是唯一能将中断状态设置为true的方法。静态方法interrupted会将当前线程的中断状态清除，但这个方法的命名极不直观，很容易造成误解，需要特别注意。  

不要把 InterruptedException 抑制在很低的层次上。如：
	
	void myTask(){
		...
		try{...}
		catch(InterruptedException e){}
		...
	}
合理的选择是：

1. 在catch子句中调用 Thread.currentThread().interrupt() 来设置中断状态。 于是调用者可以对其进行检测。
2. 不用try...catch，将InterruptedException throw出去，让最终的run方法来捕获。

###Thread.interrupt VS Thread.stop
Thread.stop方法已经不推荐使用了。而在某些方面Thread.stop与中断机制有着相似之处。如当线程在等待内置锁或IO时，stop跟interrupt一样，不会中止这些操作；当catch住stop导致的异常时，程序也可以继续执行，虽然stop本意是要停止线程，这么做会让程序行为变得更加混乱。

那么它们的区别在哪里？最重要的就是中断需要程序自己去检测然后做相应的处理，而Thread.stop会直接在代码执行过程中抛出ThreadDeath错误，这是一个java.lang.Error的子类。
###线程状态
线程有6中状态：

* New (新建)
* Runnable (可运行)
* Blocked (被阻塞)
* Waiting (等待)
* Timed waiting (计时等待)
* Terminated (被终止)

###线程属性
* 线程的优先级
* 守护线程 （一个为其他线程提供服务的线程。当只剩下了守护线程，虚拟机就退出了）
* 未捕获异常处理器