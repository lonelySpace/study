## 1.说明
> ThreadPoolExecutor作为java.util.concurrent包对外提供基础实现，以内部线程池的形式对外提供管理任务执行，线程调度，线程池管理等等服务

## 2.参数
| 参数 | 说明 |
| --- | --- |
| corePoolSize | 线程池中的线程数量，即使这些线程处在空闲状态 |
| maximumPoolSize | 线程池中允许的最大线程数量 |
| keepAliveTime | 当线程数大于corePoolSize时，剩余线程等待新任务的最长时间，超过这个时间线程将关闭 |
| unit | 时间单位 |
| workQueue | 存放任务的队列 |
| threadFactory | 创建线程的factory。默认是DefaultThreadFactory |
| handler | 超出线程池处理能力时的解决方式，默认是AbortPolicy |

## 3.核心方法execute
<font color=#DC143C>注意：如果任务不能被提交执行说明线程池已经饱和，或者线程池已经被关闭了</font>
#### execute方法代码
```
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
    /*
     * Proceed in 3 steps:
     *
     * 1. If fewer than corePoolSize threads are running, try to
     * start a new thread with the given command as its first
     * task.  The call to addWorker atomically checks runState and
     * workerCount, and so prevents false alarms that would add
     * threads when it shouldn't, by returning false.
     *
     * 2. If a task can be successfully queued, then we still need
     * to double-check whether we should have added a thread
     * (because existing ones died since last checking) or that
     * the pool shut down since entry into this method. So we
     * recheck state and if necessary roll back the enqueuing if
     * stopped, or start a new thread if there are none.
     *
     * 3. If we cannot queue task, then we try to add a new
     * thread.  If it fails, we know we are shut down or saturated
     * and so reject the task.
     */
    int c = ctl.get();
    if (workerCountOf(c) < corePoolSize) {
        if (addWorker(command, true))
            return;
        c = ctl.get();
    }
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        if (! isRunning(recheck) && remove(command))
            reject(command);
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    else if (!addWorker(command, false))
        reject(command);
}
```
执行流程：
1. 如果正在执行的线程数<corePoolSize,就开启一个新的线程来执行任务。
2. 如果任务成功排队，就会进行double-check来判断是否能够进行排队，因为上一次检查的线程可能已经死了或者线程池已经关了
如果是这样的话就没必要排队了，直接进行1的操作
3. 如果排队失败，就尝试创建一个新的线程，如果失败就拒绝这个任务执行失败处理

#### addWorker代码
```
private boolean addWorker(Runnable firstTask, boolean core) {
    retry:
    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);

        // Check if queue empty only if necessary.
        if (rs >= SHUTDOWN &&
            ! (rs == SHUTDOWN &&
               firstTask == null &&
               ! workQueue.isEmpty()))
            return false;

        for (;;) {
            int wc = workerCountOf(c);
            if (wc >= CAPACITY ||
                wc >= (core ? corePoolSize : maximumPoolSize))
                return false;
            if (compareAndIncrementWorkerCount(c))
                break retry;
            c = ctl.get();  // Re-read ctl
            if (runStateOf(c) != rs)
                continue retry;
            // else CAS failed due to workerCount change; retry inner loop
        }
    }

    boolean workerStarted = false;
    boolean workerAdded = false;
    Worker w = null;
    try {
        w = new Worker(firstTask);
        final Thread t = w.thread;
        if (t != null) {
            final ReentrantLock mainLock = this.mainLock;
            mainLock.lock();
            try {
                // Recheck while holding lock.
                // Back out on ThreadFactory failure or if
                // shut down before lock acquired.
                int rs = runStateOf(ctl.get());

                if (rs < SHUTDOWN ||
                    (rs == SHUTDOWN && firstTask == null)) {
                    if (t.isAlive()) // precheck that t is startable
                        throw new IllegalThreadStateException();
                    workers.add(w);
                    int s = workers.size();
                    if (s > largestPoolSize)
                        largestPoolSize = s;
                    workerAdded = true;
                }
            } finally {
                mainLock.unlock();
            }
            if (workerAdded) {
                t.start();
                workerStarted = true;
            }
        }
    } finally {
        if (! workerStarted)
            addWorkerFailed(w);
    }
    return workerStarted;
}
```
1. 判断当前的状态是否能够进行addTask操作，不能则返回false
2. 判断当前的worker数量是否大于最大上限或者大于自己设定的值，如果是返回false
3. 满足条件任务数量+1，如果+1失败重复检查，直到不满足条件或者+1成功，+1操作是原子操作，线程安全，retry类似于goto

