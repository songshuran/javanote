- Thread
    - 每个线程都有优先级，高优先级的线程可能会优先执行
    - 父线程创建子线程后，优先级、是否是守护线程等属性父子线程是一致的
    - JVM启动时，通常都启动MAIN非守护线程，以下情况发生，线程就会停止：退出方法被调用（Thread.interrupt方法）；所有非守护线程都消亡或者从运行的方法正常返回，或者运行的方法抛出异常
    
- 线程状态
    ![](/assets/iShot2020-09-17下午03.47.05.png)

- 守护线程
    - 默认创建的线程都是非守护线程，创建守护线程时，需要将Thread的daemon属性设置为true，守护线程的优先级很低，当JVM退出时，是不关心有无守护线程的，即使还有很多守护线程，JVM仍然会退出。工作中，我们可能会实现一些监控工作，这时用守护线程去做，这样即使监控抛出异常，但因为是子线程，所以不会影响到主业务线程，因为是守护线程，JVM也无需关心守护线程，该退出就退出。

- ClassLoader：类加载器，把类文件、二进制数组、URL等位置加载成可运行class

- 创建线程的三种方法
    - 继承Thread
    - 新建类实现runnable接口，作为入参
    - 匿名内部类实现
    
- join：当前线程等待另一个线程执行完成之后，才能继续操作
    ```java
Thread main = Thread.currentThread(); log.info("{} is run。",main.getName());
Thread thread = new Thread(new Runnable() {
    @Override
    public void run() {
        log.info("{} begin run",Thread.currentThread().getName()); 
    try {
        Thread.sleep(30000L);
    } catch (InterruptedException e) {
        e.printStackTrace(); }
        log.info("{} end run",Thread.currentThread().getName()); }
    });
// 开一个子线程去执行
thread.start();
// 当前主线程等待子线程执行完成之后再执行，需要在等待线程的内部调优先执行线程join
thread.join();
    ```
    
- yield: 是一个native方法
    - 当前线程做出让步，放弃当前cpu，让cpu重新选择线程，避免线程过度使用cpu，我们在写while死循环的时候，预计短时间内，while可以结束的话，可以在死循环里使用yield方法，防止cpu一直while死循环霸占
    
- sleep：是一个native方法
    - 当前线程沉睡多久，沉睡期间不会释放锁资源，其他线程是无法得到锁的

- interrupt：可以打断终止正在运行的线程
    - Object#wait ()、Thread#join ()、Thread#sleep (long) 这些方法运行后，线程的状态是 WAITING 或 TIMED_WAITING，这时候打断这些线程，就会抛出 InterruptedException 异常，使线程的状态直接到 TERMINATED;
    - 如果 I/O 操作被阻塞了，我们主动打断当前线程，连接会被关闭，并抛出 ClosedByInterruptException 异常;
    
        
- 面试题
    - 创建子线程时，子线程得不到父类的ThreadLocal，有什么办法可以解决这个问题？
     - 可以使用InheritableThreadLocal来代替ThreadLocal，两者都是线程的属性，所以可以做到线程之间的数据隔离，父线程 ThreadLocal 是无法传递给子线程 的，但 InheritableThreadLocal 可以。
    - 线程的start和run之间的区别？
        - start会开启一个新的线程，run方法不会
    - 守护线程和非守护线程区别？
        - 主要区别：JVM退出时，是不会管守护线程，只会管非守护线程，如果非守护线程还在运行，JVM就不会退出；如果有守护线程且没有非守护线程，JVM直接退出。日常工作中日志的收集：根据日志重要程度，不重要就选择守护线程，重要的就选择非守护线程。
    - Thread、Runnable、Callable区别？
        - Thread实现了Runnable，本身就是Runnable，但同时负责线程创建、线程状态变更等操作。Runnable是无返回值任务接口，Callable有返回值，两者的执行必须要有Thread
    - 线程池submit有两个方法，分别是接受Runnable和Callable接口，底层如何适配的？
        - Runnable和Callable是通过FutureTask进行统一的，FT有个属性叫callable，同时实现了Runnable接口，FutureTask 的最终目标是把 Runnable 和 Callable 都转化成 Callable，Runnable 转化成 Callable 是通过 RunnableAdapter 适配器进行实现的。
    - Callable是否可以丢给Thread去执行？
        - 可以，通过FutureTask的构造器入参，封装成FT传给Thread去执行。
    - FutureTask的get、cancel方法的理解？
        - get方法主要作用是得到Callable异步任务执行的结果，无参get会一直等待任务执行完成才返回，有参get可以设定固定的时间，在设定时间内还没执行完成，直接返回异常，实际工作中建议使用有参get。cancel方法主要用来取消任务的。
    - Thread.yield方法有什么用？
        - 表示当前线程放弃CPU，重新参与到CPU的竞争中去，再次竞争自己有可能得到CPU资源，也有可能得不到，好处是防止当前线程一直霸占CPU。工作中可能会写while自旋的代码，如果一直自旋，CPU会一直被while占用，如果能预见while自旋时间很长，会设置判断条件让线程阻塞；如果能预见自旋时间很短，通常使用Thread.yield方法，使当前线程让步
    - sleep和wait方法异同？
        - 相同点：两者都让线程进入TIMED_WATING状态并且可以设置等待的时间
        - 不同点：wait是Object类的方法，sleep是Thread类的方法；sleep不会释放锁，沉睡的时候，其他线程无法获得锁，但是wait会释放锁。
     

