阻塞队列
===
[Core JAVA]()

阻塞队列（BlockingQueue）是一个支持两个附加操作的队列。这两个附加的操作是：在队列为空时，获取元素的线程会等待队列变为非空。当队列满时，存储元素的线程会等待队列可用。  
阻塞队列常用于生产者和消费者的场景，生产者是往队列里添加元素的线程，消费者是从队列里拿元素的线程。阻塞队列就是生产者存放元素的容器，而消费者也只从容器里拿元素。  
####阻塞队列的方法：
* add       添加一个元素                   如果队列满，则抛出IllegalStateException
* remove    移除并返回头元素                如果队列空，抛出NoSuchElementException
* element   返回队列的头元素                如果队列空，则抛出NoSuchElementException
* offer     添加一个元素并返回true          如果队列满，则返回false
* peek      返回队列的头元素                如果队列空，返回null
* poll      移除并返回队列的头元素           如果队列空，返回null
* put       添加一个元素                    如果队列满，则阻塞
* take      移除并放回头元素                如果队列空，则阻塞 

队列的方法按当队列满或空时期响应方式分为3类：  

1. 将队列当做线程管理工具来使用： ｐｕｔ　、ｔａｋｅ
2. add remove element 会出错时返回异常
3. 在一个多线程程序中，要使用 offer,poll,peek ，这些方法如果不能完成任务，会给个错误提示而非异常

###java 的阻塞队列
JDK7提供了7个阻塞队列。分别是：

* ArrayBlockingQueue ：一个由数组结构组成的有界阻塞队列。
* LinkedBlockingQueue ：一个由链表结构组成的有界阻塞队列。
* PriorityBlockingQueue ：一个支持优先级排序的无界阻塞队列。
* DelayQueue：一个使用优先级队列实现的无界阻塞队列。
* SynchronousQueue：一个不存储元素的阻塞队列。
* LinkedTransferQueue：一个由链表结构组成的无界阻塞队列。
* LinkedBlockingDeque：一个由链表结构组成的双向阻塞队列。

