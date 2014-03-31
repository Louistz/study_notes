Callable和Future
===
[Core JAVA]()
Callable和Future两功能是Java在或许版本为了适应多并发才加入的。Callable是类似于Runnable的接口，实现Callable接口的类和实现Runnable的类都是可被其他线程执行的任务。   
Callable的接口定义如下: 
	
	public interface Callable<V>{
		V call() throws Exception;
	}

Callable和Runnable的区别：

1. Callable定义的方法是call，而Runnable定义的方法是run
2. Callable的call方法可以有返回值，而Runnable的run方法不能有返回值
3. Callable的call方法可抛出异常，而Runnable的run方法不能抛出异常

####Future
Future表示异步计算的结果，它提供了检查计算是否完成的方法，以等待计算的完成，并检索计算的结果。  

Future的cancel方法可以取消任务的执行，它有一布尔参数，参数为 true 表示立即中断任务的执行，参数为 false 表示允许正在运行的任务运行完成。  
Future的 get 方法等待计算完成，获取计算结果  

	public interface Future<V>{
		
		V get() throws...;
		V get(long timeout, TimeUnit unit) throws...;
		void cancel(boolean mayInterrupt);
		boolean isCancelled();
		boolean isDone();
	}
