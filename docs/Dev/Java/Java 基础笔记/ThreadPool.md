# 线程池

## 基础

参数最多的一个构造函数：

``` java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler)
```

参数解读：

- `corePoolSize`: 核心线程池大小。线程数未达到此数量时，会持续创建新线程来处理新来的任务；当线程数达到此数量时，新任务被保存至阻塞任务队列
- `maximumPoolSize`: 最大线程池大小。阻塞队列满时、新任务到达时，可继续创建线程到此数量进行任务处理
- `keepAliveTime`: 空闲此时间后销毁线程（指超过 corePoolSize 的空闲线程在此时间后被销毁）
- `unit`: 时间单位
- `workQueue`: 等待队列
- `threadFactory`: 线程创建工厂
- `handler`: 拒绝策略

定义一个线程池：

``` java
private static final ThreadPoolExecutor COMMON_EXEC = new ThreadPoolExecutor(
    4,
    20,
    1000L * 60 * 5, // 五分钟
    TimeUnit.MILLISECONDS,
    new LinkedBlockingQueue<>(1024 * 10),
    new ThreadFactoryBuilder().setNameFormat("eightFullDataSync-common-pool-%d").build(),
    new ThreadPoolExecutor.CallerRunsPolicy()
);
```

使用线程池：

``` java
COMMON_EXEC.execute(() -> {
    // 具体代码
}));
```



## 拓展阅读

- [JUC线程池: ThreadPoolExecutor详解 | Java 全栈知识体系](https://www.pdai.tech/md/java/thread/java-thread-x-juc-executor-ThreadPoolExecutor.html)：reject handler 部分

  > `RejectedExecutionHandlerImpl.java`:
  >
  > ``` java
  > // 著作权归https://pdai.tech所有。
  > // 链接：https://www.pdai.tech/md/java/thread/java-thread-x-juc-executor-ThreadPoolExecutor.html
  > 
  > import java.util.concurrent.RejectedExecutionHandler;
  > import java.util.concurrent.ThreadPoolExecutor;
  >  
  > public class RejectedExecutionHandlerImpl implements RejectedExecutionHandler {
  >  
  >     @Override
  >     public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
  >         System.out.println(r.toString() + " is rejected");
  >     }
  >  
  > }
  > ```
  >
  > `MyMonitorThread.java`;
  >
  > ``` java
  > // 著作权归https://pdai.tech所有。
  > // 链接：https://www.pdai.tech/md/java/thread/java-thread-x-juc-executor-ThreadPoolExecutor.html
  > 
  > import java.util.concurrent.ThreadPoolExecutor;
  >  
  > public class MyMonitorThread implements Runnable
  > {
  >     private ThreadPoolExecutor executor;
  >      
  >     private int seconds;
  >      
  >     private boolean run=true;
  >  
  >     public MyMonitorThread(ThreadPoolExecutor executor, int delay)
  >     {
  >         this.executor = executor;
  >         this.seconds=delay;
  >     }
  >      
  >     public void shutdown(){
  >         this.run=false;
  >     }
  >  
  >     @Override
  >     public void run()
  >     {
  >         while(run){
  >                 System.out.println(
  >                     String.format("[monitor] [%d/%d] Active: %d, Completed: %d, Task: %d, isShutdown: %s, isTerminated: %s",
  >                         this.executor.getPoolSize(),
  >                         this.executor.getCorePoolSize(),
  >                         this.executor.getActiveCount(),
  >                         this.executor.getCompletedTaskCount(),
  >                         this.executor.getTaskCount(),
  >                         this.executor.isShutdown(),
  >                         this.executor.isTerminated()));
  >                 try {
  >                     Thread.sleep(seconds*1000);
  >                 } catch (InterruptedException e) {
  >                     e.printStackTrace();
  >                 }
  >         }
  >              
  >     }
  > }
  > ```
  >
  > `WorkerPool.java`:
  >
  > ``` java
  > // 著作权归https://pdai.tech所有。
  > // 链接：https://www.pdai.tech/md/java/thread/java-thread-x-juc-executor-ThreadPoolExecutor.html
  > 
  > import java.util.concurrent.ArrayBlockingQueue;
  > import java.util.concurrent.Executors;
  > import java.util.concurrent.ThreadFactory;
  > import java.util.concurrent.ThreadPoolExecutor;
  > import java.util.concurrent.TimeUnit;
  >  
  > public class WorkerPool {
  >  
  >     public static void main(String args[]) throws InterruptedException{
  >         //RejectedExecutionHandler implementation
  >         RejectedExecutionHandlerImpl rejectionHandler = new RejectedExecutionHandlerImpl();
  >         //Get the ThreadFactory implementation to use
  >         ThreadFactory threadFactory = Executors.defaultThreadFactory();
  >         //creating the ThreadPoolExecutor
  >         ThreadPoolExecutor executorPool = new ThreadPoolExecutor(2, 4, 10, TimeUnit.SECONDS, new ArrayBlockingQueue<Runnable>(2), threadFactory, rejectionHandler);
  >         //start the monitoring thread
  >         MyMonitorThread monitor = new MyMonitorThread(executorPool, 3);
  >         Thread monitorThread = new Thread(monitor);
  >         monitorThread.start();
  >         //submit work to the thread pool
  >         for(int i=0; i<10; i++){
  >             executorPool.execute(new WorkerThread("cmd"+i));
  >         }
  >          
  >         Thread.sleep(30000);
  >         //shut down the pool
  >         executorPool.shutdown();
  >         //shut down the monitor thread
  >         Thread.sleep(5000);
  >         monitor.shutdown();
  >          
  >     }
  > }
  > ```
  >
  > 

