## java线程池的使用

##### [可点击进入参考文档](http://blog.csdn.net/u011531613/article/details/61921473)

``` java

    // ThreadPoolExecutor 继承了  Executor  ExecutorService AbstractExecutorService

    ThreadPoolExecutor executorService = new ThreadPoolExecutor(10, 100, 5, TimeUnit.SECONDS, new LinkedBlockingDeque<Runnable>() {
        });

        executorService.execute(new Runnable() {
            @Override
            public void run() {
                
            }
        });

    //  ExecutorService 继承了Executor  ExecutorService 接口
    ExecutorService executorService=Executors.newCachedThreadPool()
  
```

1. Java通过Executors提供四种线程池，分别为：
<font size="2">
2. **newCachedThreadPool**创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。线程池为无限大，当执行第二个任务时第一个任务已经完成，会复用执行第一个任务的线程，而不用每次新建线程。

3. **newFixedThreadPool** 创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。

4. **newScheduledThreadPool** 创建一个定长线程池，支持定时及周期性任务执行。

5. **newSingleThreadExecutor** 创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。

</font>

#### 1. newCachedThreadPool
> 

#### 2. newFixedThreadPool
> 

#### 解释如何使用**ThreadPoolExecutor**相关属性

**1.线程池状态**
<font size="2">
``` java
volatile int runState;
static final int RUNNING    = 0;
static final int SHUTDOWN   = 1;
static final int STOP       = 2;
static final int TERMINATED = 3;
```
* 如果调用了shutdown()方法，则线程池处于SHUTDOWN状态，此时线程池不能够接受新的任务，它会等待所有任务执行完毕；
* 如果调用了shutdownNow()方法，则线程池处于STOP状态，此时线程池不能接受新的任务，并且会去尝试终止正在执行的任务；
* 当线程池处于SHUTDOWN或STOP状态，并且所有工作线程已经销毁，任务缓存队列已经清空或执行结束后，线程池被设置为TERMINATED状态。

</font>

**2.任务的执行**
<font size="2">
* private final BlockingQueue<Runnable> workQueue;              //任务缓存队列，用来存放等待执行的任务
* private final ReentrantLock mainLock = new ReentrantLock();   //线程池的主要状态锁，对线程池状态（比如线程池大小
                                                              * //、runState等）的改变都要使用这个锁
* private final HashSet<Worker> workers = new HashSet<Worker>();  //用来存放工作集
 
* private volatile long  keepAliveTime;    //线程存货时间   
* private volatile boolean allowCoreThreadTimeOut;   //是否允许为核心线程设置存活时间
* private volatile int   corePoolSize;     //核心池的大小（即线程池中的线程数目大于这个参数时，提交的任务会被放进任务缓存队列）
* private volatile int   maximumPoolSize;   //线程池最大能容忍的线程数
 
* private volatile int   poolSize;       //线程池中当前的线程数
 
* private volatile RejectedExecutionHandler handler; //任务拒绝策略
 
* private volatile ThreadFactory threadFactory;   //线程工厂，用来创建线程
 
* private int largestPoolSize;   //用来记录线程池中曾经出现过的最大线程数
 
* private long completedTaskCount;   //用来记录已经执行完毕的任务个数

</font>


**3.线程池中的线程初始化**

<font size="2">
默认情况下，创建线程池之后，线程池中是没有线程的，需要提交任务之后才会创建线程。

在实际中如果需要线程池创建之后立即创建线程，可以通过以下两个方法办到：

prestartCoreThread()：初始化一个核心线程；

prestartAllCoreThreads()：初始化所有核心线程
</font>


**4.任务缓存队列及排队策略**

<font size="2">
在前面我们多次提到了任务缓存队列，即workQueue，它用来存放等待执行的任务。

workQueue的类型为BlockingQueue<Runnable>，通常可以取下面三种类型：

　　1）ArrayBlockingQueue：基于数组的先进先出队列，此队列创建时必须指定大小；

　　2）LinkedBlockingQueue：基于链表的先进先出队列，如果创建时没有指定此队列大小，则默认为Integer.MAX_VALUE；

　　3）synchronousQueue：这个队列比较特殊，它不会保存提交的任务，而是将直接新建一个线程来执行新来的任务。
</font>

**5.任务拒绝策略**

<font size="2">
当线程池的任务缓存队列已满并且线程池中的线程数目达到maximumPoolSize，如果还有任务到来就会采取任务拒绝策略，通常有以下四种策略

``` java
ThreadPoolExecutor.AbortPolicy:丢弃任务并抛出RejectedExecutionException异常。

ThreadPoolExecutor.DiscardPolicy：也是丢弃任务，但是不抛出异常。

ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）

ThreadPoolExecutor.CallerRunsPolicy：由调用线程处理该任务
```

</font>

**6.线程池的关闭**

<font size="2">
ThreadPoolExecutor提供了两个方法，用于线程池的关闭，分别是shutdown()和shutdownNow()，其中：

shutdown()：不会立即终止线程池，而是要等所有任务缓存队列中的任务都执行完后才终止，但再也不会接受新的任务
shutdownNow()：立即终止线程池，并尝试打断正在执行的任务，并且清空任务缓存队列，返回尚未执行的任务

</font>

**7.线程池容量的动态调整**

<font size="2">
ThreadPoolExecutor提供了动态调整线程池容量大小的方法：setCorePoolSize()和setMaximumPoolSize()，

setCorePoolSize：设置核心池大小
setMaximumPoolSize：设置线程池最大能创建的线程数目大小


</font>


**同步方法**

> wait与notify
>
> wait():使一个线程处于等待状态，并且释放所持有的对象的lock。
>
> sleep():使一个正在运行的线程处于睡眠状态，是一个静态方法，调用此方法要捕捉InterruptedException异常。
>
> notify():唤醒一个处于等待状态的线程，注意的是在调用此方法的时候，并不能确切的唤醒某一个等待状态的线程，而是由JVM确定唤醒哪个线程，而且不是按优先级。
> notifyAll():唤醒所有处入等待状态的线程，注意并不是给所有唤醒线程一个对象的锁，而是让它们竞争。
> 
> ReentrantLock类是可重入、互斥、实现了Lock接口的锁，它与使用synchronized方法和快具有相同的基本行为和语义，并且扩展了其能力。
 ReenreantLock类的常用方法有
 ``` java
    ReentrantLock() : 创建一个ReentrantLock实例 
    lock() : 获得锁 
    unlock() : 释放锁 

    //只给出要修改的代码，其余代码与上同
    class Bank {
        
        private int account = 100;
        //需要声明这个锁
        private Lock lock = new ReentrantLock();
        public int getAccount() {
            return account;
        }
        //这里不再需要synchronized 
        public void save(int money) {
            lock.lock();
            try{
                account += money;
            }finally{
                lock.unlock();
            }
            
        }
    ｝
 ```

 **如果使用ThreadLocal管理变量，则每一个使用该变量的线程都获得该变量的副本，副本之间相互独立，这样每一个线程都可以随意修改自己的变量副本，而不会对其他线程产生影响。
     ThreadLocal 类的常用方法**
``` java
ThreadLocal() : 创建一个线程本地变量 
get() : 返回此线程局部变量的当前线程副本中的值 
initialValue() : 返回此线程局部变量的当前线程的"初始值" 
set(T value) : 将此线程局部变量的当前线程副本中的值设置为value


//只改Bank类，其余代码与上同
public class Bank{
    //使用ThreadLocal类管理共享变量account
    private static ThreadLocal<Integer> account = new ThreadLocal<Integer>(){
        @Override
        protected Integer initialValue(){
            return 100;
        }
    };
    public void save(int money){
        account.set(account.get()+money);
    }
    public int getAccount(){
        return account.get();
    }
}

```