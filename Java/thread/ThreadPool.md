线程池
===
[Core JAVA]()

构建一个新的线程的代价很大，因为涉及与OS的交互。如果需要大量生命周期很短的线程，应该使用线程池（Thread pool）。 一个线程池中包含许多准备运行的空闲线程。将Runnable对象交给线程池，就会有一个线程调用run方法。当run方法退出时，线程不会死亡，而是在池中等待为下一个请求提供服务。  
使用线程池的另外一个原因就是：可以减少并发线程的数目。  
###执行器（Executor）
执行器有许多静态工程方法用来构建线程池：
	
* newCachedThreadPool                 必要时创建新线程，空闲线程会被保留60s
* newFiexedThreadPool                 该吃包含固定数量的线程，空闲线程会被保留
* newSingleThreadExecutor             只有一个线程的pool，该线程顺序执行每个提交的任务（类似Swing时间分配线程）
* newScheduledThreadPool              用于预定执行而构建的固定线程池，代替 java.util.Timer
* newSingleThreadScheduleExecutor     用于云顶执行而构建的单线程pool  

exp:
	
	ExecutorService  pool = Executors.newCachedThreadPool();
可以将Run那边了对象或Callable对象提交给ExecutorService:
	
	Future<?> submit(Runnable task)
	Future<T> submit(Runnable task,T result)
	Future<T> submit(Callable task)