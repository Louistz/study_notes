同步
===
[Core JAVA]()

在大多数实际的多线程应用中，两个或者两个以上的线程需要共享同一数据。会产生竞争，从而导致数据的不同步。  
##锁对象
有两种机制可以防止代码块受并发访问的干扰。  
Java语言提供一个 synchronized 关键字达到这一目的，并且java SE 5.0之后引入了ReentrantLock类。  
synchronized提供一个锁已经相关的“条件”。 
	
	private Lock bankLock = new ReentrantLock();

	public void transfer(int from ,int to, int amount){
		bankLock.lock();

		try{
			...
		}
		finally{
			banklock.unlock();
		}
	}
这一结构确保任何时刻只有一个线程进入临界区。一旦一个线程封锁了锁对象，其他任何线程都无法通过lock语句。当其他线程调用lock时，它们被阻塞，直到第一个线程释放锁对象。  
注意：

* 把解锁操作放在finally 很重要。如果在临界区的代码抛出异常，锁必须被释放。否则其他线程将永远被阻塞。
* 如果使用锁，就不能使用带资源的try语句。  

每个对象有自己的ReetrantLock对象。如果两个线程试图访问同一个对象，那么锁以串行方式提供服务。但是如果两个线程访问不同的对象，每个线程得到不同的锁对象，两个形成都不会发生阻塞。 本该如此，因为线程在操作不同的实例的时候，线程之间不会相互影响。  
锁是可重入的。因为线程可以重复获得已经持有的锁。被一个锁保护的代码可以调用另一个使用相同的锁的方法。
###条件对象
通常，线程在进入临界区，却发现在某一条件满足之后它才能执行。要使用一个条件对象来管理那些已经获得一个锁但却无法做有效工作的线程。
	
	public class Bank{
		private Condition sufficientFunds;
		public Bank(){
			this.sufficientFunds = bankLock.newCondition();
		}
		public void transfer(int from ,int to, int amount){
			bankLock.lock();
			try{
				while(accounts[from] < amount){
					sufficientFunds.await();
				}
				.....//do some things

				sufficientFunds.signallAll();
			}
			finally{
				bankLock.unlock();
			}
		}
	}

	while(!(ok to proceed)){
		condition.await();
	}
当达到了条件对象时， condion.await()执行，当前线程被阻塞，并放弃了锁。
等待获得锁的线程和调用await()方法的线程存在本质上的不同。 一旦一个线程调用await()，它会进入该条件的等待集。当锁可用时，该线程不能马上解除阻塞。相反，它会先处于阻塞状态，然后知道另一个线程调用同一条件上的signalAll()时，它才可以继续。  
当 condition.signalAll();  
会重新激活因为这一条件而等待的所有线程。当一个线程调用await()时，他没有办法激活自身，它会寄希望于其他线程。若果没有其他线程来执行signalALL（）,那么它就永远不会再执行了。 这将导致令人不快的`死锁`现象。 
还有个 signal()方法，会随机解除条件等待集中的某个线程的阻塞状态。但仍然存在危险。如果随机选择的线程发现自己仍然不能执行，它将再度被阻塞。 

###sychronized 关键字
Java1.0版本开始，每个对象都有个内部锁。如果一个方法用synchronized关键字声明，那么对象的锁将保护整个方法。相当于java帮我们定义了一个锁。同时也提供了一些相关的操作。  
`wait,notifyAll,notify` 分别对应 await,signalAll，signal
	
	public synchronized void transfer(int from ,int to ,int amount) throws InterruptedException{
		while(accounts[from] < amount){
			wait();
		}
		.....
		notifyAll();
	}
其中，锁和解锁 由 synchronized 隐含其中。   

`内部锁`和条件的局限：

* 不能中断一个正在试图获得锁的线程
* 试图获得锁时不能设定超时
* 每个锁仅有单一的条件，可能是不够的

**代码中应该使用那一种？**

* 最好既不使用 Lock/Condition 也不是用 synchronized. 许多情况下可以使用 java.until.concurrent包的一种机制，它会处理所有的加锁。如：阻塞队列。
* 优先synchronized
* 如特别需要 Lock/Condition 结构提供的特有的机制时，才使用它

###同步阻塞
线程除了可以调用同步方法获得锁外，还有一种机制可以通过进入`同步阻塞`获得锁。

	synchronized(obj){
		...
	}
于是它获得obj的锁。  

几个`特殊的锁`：
	
	private Object lock = new Object();
	public void transfer(int from, int to, int amount){
		synchroized(lock){
			....
		}
	}
此处，lock对象是被创建仅仅是用来使用每个Java对象持有的锁。  

获得一个对象的锁来实现额外的`原子操作`，或者成为 `客户端锁定`.例如：使用 vector（vector是ArrayList的同步或者线程安全形式）  
	public void transfer(Vector<Double> accounts, int from,int to, int amount){
		synchroized(accounts){
			accounts.set(from,accounts.get(from) - amount);
			accounts.set(to,accounts.get(to)+ammount);
		}
	}
这个实现依赖于 Vector类对自己的所有可修改方法都使用内部锁。

###监视器概念
锁和条件是线程同步的强大工具，但他们不是面向对象的。 面向对象考虑：在无需程序员思考如何枷锁的情况下，就可以保证多线程的安全性。 最成功的方案就是监视器：  

* 监视器是只包含私有域的类。
* 每个监视器的对象有一个相关的锁
* 使用该所对所有的方法进行加锁。
* 该所可以有任意多个相关条件

但是java的实现者实现的方式并不是很精确，从而使得线程的安全性下降，差异在于：

* 域不要求必须是private的
* 方法不要求必须是 synchronized的
* 内部锁对客户是可用的

####volatile 域
如果仅仅为了读写一两个域就使用同步，显得开销过大。如：
	
	private boolean done;
	public synchronized boolean getDone(){...}
	public synchronized void setDone(boolean done){...}
而是用 volatile 域的同步访问提供了一种免锁机制。如果声明一个域为volatile，那么编译器和虚拟机就知道该域可能被另一个线程并发更新。如：

	private volatile boolean done;
	public  boolean getDone(){...}
	public  void setDone(boolean done){...}
注： volatile不能提供原子性。
####原子性
java.util.concurrent.atomic包中有很多类使用了很高效的机器级指令（而不是使用锁）来保证其他操作的原子性。但应用程序不该使用这些类，仅供那些开发并发工具的系统应用使用。

####死锁
锁和条件 并不能解决线程中的所有问题。 因资源竞争或其他特殊问题造成的死锁，在java语言中目前还无法避免和打破。需要仔细设计程序，以确保不会出现死锁。

###线程局部变量
线程间共享变量有风险，有时候就要避免共享变量。使用 **`ThreadLocal`**辅助类来为各个线程提供各自的实例。
如：
	
	public static SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-mm-dd");
如果有两个线程都执行以下操作：
	
	String dataStamp = dateFormat.format(new Date());
结果可能很混乱，因为dateFormat使用的内部数据结构可能会被并发的访问所破坏。当然也可以同步，但开销很大。  
要为每个线程构造一个实例，可以使用以下代码：
	
	public static final ThreadLocal<SimpleDateFormat> dateFormat = 
	  new ThreadLocal<SimpleDateFormat>(){
 	  	protected SimpleDateFormat initialValue(){
 		  	return new SimpleDateFormat("yyyy-mm-dd");
 	  	}
   	};
   	String dateStamp = dateFormat.get().format(new Date());
在一个给定的线程中首次调用get时，会调用initialValue方法。在此之后，get方法会返回属于当前线程的那个实例。

######ThreadLocalRandom
随机数的生成，如果多个线程共享一个随机数生成器，效率会很低。可以为每个线程提供一个单独的生成器。
	
	int random = ThreadLocalRandom.current().nextInt(upperBound);

####读写锁 ReentrantReadWriteLock
如果很多线程读数据而少数线程修改数据，那么允许对读者线程共享访问时合适，写者线程依然必须是互斥的。使用ReentrantReadWriteLock:  
1). 构造一个对象：  
	
	private ReentrantReadWriteLock rwl = new ReetrantReadWriteLock();
2). 抽取读锁和写锁：  
	
	private Lock readLock = rwl.readLock();
	private LOck writeLock = rwl.writeLock();
3). 对所有的获取方法加读锁：  
	
	public double getTotalBalance(){
		readLock.lock();
		try{...}
		finally{readlock.unlock();}
	}
4). 对所有的修改方法加写锁：  
	
	public voidtransfer(...){
		writeLock.lock();
		try{...}
		finally{writeLock.unlock();}
	}