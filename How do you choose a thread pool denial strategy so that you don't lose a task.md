# The interviewer asked: How do you choose a thread pool denial strategy so that you don't lose a task?

**Original** **Fox** [Fox爱分享](javascript:void(0);)

 *September 30, 2025, 06:01*

**Last week, he helped a fan review the interview, and he mentioned that the interviewer asked“** **How to deal with the thread pool is full？What's the best way to say no without losing the job？** **" when only answered " CallerRunsPolicy , " did not explain " when to use , what risks , " was directly judged "** **only remember the conclusion , do not understand the landing** **. "**

**In fact, this question is not just about "remembering 4 strategies."It is "** **the ability to select strategies based on business scenarios + avoid the risk of losing tasks** **." Today, we will explain it with "principle deconstruction + scenario selection + follow-up prediction" to help you nod to the interviewer next time**

## First, understand: why triggers the denial strategy?

**The trigger of the thread pool rejection strategy is "** **the speed of task submission exceeds the processing capacity of the thread pool** **" , which satisfies two conditions :**

1. **The core thread is busy.** **The core number of threads (corePoolSize) are executing tasks;**
2. **The queue was full.** **The task queue (such as LinkedBlockingQueue) is full of tasks to execute;**
3. **Non-core threads are also full.** **The maximum number of threads (maximumPoolSize) has been reached and no new threads can be created.**

**A real-life scenario: e-commerce slasher activity, thread pool core threads= 5, maximum thread size = 10, queue capacity = 20, Suddenly 50 ordering tasks are flooding every second - 5 core threads get busy, 20 tasks enter the queue, another 5 non-core threads are created, and the remaining 10 tasks will trigger the denial policy.**

**If no policy is selected at this time, either the task is lost (the user order failed) or the system crashes (the thread pool overload), so "refusing policy selection" is directly related to business stability.**

## II. 4 Native Rejection Strategies Dismantled: From "How to Use" to "Would I Lose the Task?"

**The JDK native provides four rejection strategies (implementing the RejectedExecutionHandler interface), but only some of the strategies can avoid losing the task, one by one.**

### 1. AbortPolicy (default policy): Throw the exception directly, the task will be lost!

* **Core logic****Once the rejection is triggered, the RejectedExecutionException exception is thrown directly and the committed task is not executed.**
* **Code examples**

```
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    5, 10, 60L, TimeUnit.SECONDS,
    new LinkedBlockingQueue<>(20),
    new ThreadPoolExecutor.AbortPolicy() // 默认策略，可省略
);
```

* **Applicable scenarios****Can't tolerate the loss of a mission？No!On the contrary -** **it is suitable for the scenario of "task loss will directly lead to business anomaly and need to be perceived immediately"** **, such as financial transfer （lost transfer task must report error immediately, can not swallow silently）.**
* **Risk points****If an exception is not captured, it will cause the thread that submitted the task to crash; If an exception is captured and not processed, the task will still be lost.**

### 2. DiscardPolicy: Silently discard tasks, more pit than AbortPolicy!

* **Core logic****When the rejection is triggered, the task is discarded directly without throwing any exceptions, which is the equivalent of "the task disappearing from nothing."**
* **Code examples**

```
new ThreadPoolExecutor.DiscardPolicy()
```

* **Applicable scenarios****There are hardly any recommended scenes** **It is only suitable for non-core scenarios where " losing a task does not affect the business "** **, such as log printing （ losing a log is not important ） , but even so , it is not recommended to use - it is better to use DiscardOldestPolicy to at least keep the latest task .**
* **Risk points****There is no notice of a lost mission, and when checking for problems, there is no idea that a mission has been lost, which is a "hidden pit."**

### 3. DiscardOldestPolicy: Discard the oldest task in the queue, keep the new task

* **Core logic****When a rejection is triggered, the "oldest task" in the task queue (the queue top task) is discarded and the newly submitted task is added to the queue.**
* **Code examples**

```
new ThreadPoolExecutor.DiscardOldestPolicy()
```

* **Applicable scenarios****Suitable for " new tasks are more important than old tasks " scenarios** **, such as real-time data statistics （ the latest user behavior data is more valuable than 10 seconds ago , lose old data to retain new data ） .**
* **Risk points****Tasks are lost (lost old tasks), and if the queue is a "core task that must be performed" (such as order creation), losing old tasks can cause business exceptions, so use core scenarios carefully.**

### 4. CallerRunsPolicy: Let the thread that submits the task execute itself, do not lose the task!

* **Core logic****When you trigger a denial, you don't lose the task or throw exceptions, but let the "task-submitting thread" (such as the main thread) perform the task themselves, which is equivalent to "temporary seconding the submitting thread to help with the processing."**
* **Code examples**

```
new ThreadPoolExecutor.CallerRunsPolicy()
```

* **Applicable scenarios****Most recommended** **"don't want to lose the task" scene** **For example, the order is killed, the user registers - even if the thread pool is full, the thread that submits the task （such as the working thread of Tomcat） can perform the task, although it will be slower, but the task will not be lost.**
* **Key advantages**

1. **No task loss: As long as the submit thread does not crash, the task will be executed.**
2. **It has a self-limiting effect: when a thread is submitted to perform a task, no new tasks can be submitted, which is equivalent to "slowing the task submission speed," indirectly reducing the pressure on the thread pool.**

* **Risk points****If the thread that submits a task is a "core thread," such as the main thread, the execution of the task will block the main thread and affect other businesses - for example, when Tomcat's working thread performs the next task, it will temporarily be unable to handle other users' requests.**

## III. Advanced: 2 custom denial strategies to address the pain points of the native strategy

**Native strategies either lose tasks or run the risk of blocking, and custom denial strategies are more commonly used in real projects, especially the following two:**

### 1. Custom strategy 1: task team to MQ, asynchronous retry (absolutely do not lose the task)

* **Core logic****When a denial is triggered, instead of processing the task directly, the task is dumped into a message queue (e.g. RabbitMQ, RocketMQ), and a dedicated consumer thread pulls the task from MQ and retries to submit it to a thread pool until execution is successful.**
* **Code examples**

```
// 自定义拒绝策略：任务入MQ重试
RejectedExecutionHandler mqRejectionHandler = (runnable, executor) -> {
    try {
        // 把任务序列化成JSON，发送到MQ
        String taskJson = JSON.toJSONString(runnable);
        rabbitTemplate.convertAndSend("thread-pool-retry-queue", taskJson);
        log.info("任务入MQ重试：{}", taskJson);
    } catch (Exception e) {
        // MQ发送失败时，降级用CallerRunsPolicy，避免任务丢失
        new ThreadPoolExecutor.CallerRunsPolicy().rejectedExecution(runnable, executor);
        log.error("任务入MQ失败，降级处理", e);
    }
};
// 线程池配置自定义策略
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    5, 10, 60L, TimeUnit.SECONDS,
    new LinkedBlockingQueue<>(20),
    mqRejectionHandler
);
// MQ消费者：从队列拉取任务，重试提交到线程池
@RabbitListener(queues = "thread-pool-retry-queue")
public void handleRetryTask(String taskJson) {
Runnable task = JSON.parseObject(taskJson, Runnable.class);
// 重试3次，每次间隔1秒
for (int i = 0; i < 3; i++) {
    if (executor.getQueue().remainingCapacity() > 0) { // 队列有空闲位置再提交
        executor.submit(task);
        log.info("任务重试成功：{}", taskJson);
        return;
    }
    TimeUnit.SECONDS.sleep(1);
}
// 重试3次失败，记录到数据库，人工排查
log.error("任务重试失败，记录到DB：{}", taskJson);
dbService.saveFailedTask(taskJson);
}
```

* **Applicable scenarios****Never lose the core business of the task, such as order payment, transfer, inventory deduction - even if the line program is full for a long time, the task can be temporarily stored in MQ and executed after retrying.**
* **Key advantages****Without the risk of task loss and without blocking the commit thread , it is** **the preferred solution for the core business of Dachaung** **.**

### 2. Custom Policy 2: Dynamically Expand queue / thread number (avoid triggering denial)

* **Core logic****Before triggering a rejection, check whether the queue and thread numbers have expansion space (e.g. the maximum capacity of the queue is set to 100, whereas 80 is currently used), and if there is, expand dynamically to avoid triggering the rejection policy; If you have reached the maximum expansion limit, then use CallerRunsPolicy to downgrade.**
* **Code examples**

```
// 自定义可扩容队列
class ResizableBlockingQueue<E> extends LinkedBlockingQueue<E> {
    private int maxCapacity; // 队列最大扩容上限
    public ResizableBlockingQueue(int initialCapacity, int maxCapacity) {
        super(initialCapacity);
        this.maxCapacity = maxCapacity;
    }
    // 动态扩容队列
    public boolean expandCapacity(int newCapacity) {
        if (newCapacity > maxCapacity) return false;
        this.maxCapacity = newCapacity;
        return true;
    }
    @Override
    public boolean offer(E e) {
        // 队列满了，尝试扩容（每次扩20%）
        if (super.size() == super.remainingCapacity() + super.size()) {
            int newCapacity = (int) (super.size() * 1.2);
            if (newCapacity <= maxCapacity) {
                this.expandCapacity(newCapacity);
                log.info("队列扩容到：{}", newCapacity);
            }
        }
        return super.offer(e);
    }
}
// 自定义拒绝策略：先扩容，再降级
RejectedExecutionHandler expandRejectionHandler = (runnable, executor) -> {
    ThreadPoolExecutor pool = (ThreadPoolExecutor) executor;
    ResizableBlockingQueue<?> queue = (ResizableBlockingQueue<?>) pool.getQueue();
    // 1. 尝试扩容队列
    if (queue.expandCapacity((int) (queue.size() * 1.2))) {
        pool.submit(runnable);
        log.info("队列扩容后，任务提交成功");
        return;
    }
    // 2. 队列无法扩容，尝试增加最大线程数（不超过20）
    if (pool.getMaximumPoolSize() < 20) {
        pool.setMaximumPoolSize(pool.getMaximumPoolSize() + 2);
        pool.submit(runnable);
        log.info("最大线程数扩容到：{}，任务提交成功", pool.getMaximumPoolSize());
        return;
    }
    // 3. 扩容失败，用CallerRunsPolicy降级
    new ThreadPoolExecutor.CallerRunsPolicy().rejectedExecution(runnable, executor);
    log.info("扩容失败，降级处理任务");
};
```

* **Applicable scenarios****Traffic volatility but non-extreme scenarios, such as the transition from the pre-heated to peak period of e-commerce promotion, use dynamic scaling to reduce the frequency of rejection strategy triggers, and combine performance and stability.**

## IV. Interviewers Must Ask: 3 Difficult Questions

### 1. Follow-up 1: "CallerRunsPolicy blocks the commit thread. How can I avoid it?"

**This is to examine the ability to "risk aversion," and the correct answer is to combine "business scenarios + downgrading strategies":**

 **Answer template ** **:“CallerRunsPolicy can indeed block the submission thread, Therefore, there are two limitations when using it in practice: First, determine whether the submit thread is a core thread (such as the Tomcat work thread). If it is acore thread, use the CallerRunsPolicy instead of the MQ retry policy to avoid blocking the core business; Second, add timeout controls to CallerRunsPolicy, such as Future.get (5, TimeUnit.SECONDS), to avoid long-term blockages if the task is interrupted for more than 5 seconds.”**

### 2. Question2: "With the MQ retry strategy, how can I avoid repeating tasks?"

**This is an examination of "data consistency," and the core of this is "power-order design":**

 **Answer template ** **:“To avoid duplication of tasks, two things must be done: First, tasks must be uniquely identified (such as the order ID), and the unique identifier is used as the message ID when submitted to MQ; Second, check whether the uniquely identified task has already been executed before performing a task (for example, checking a database or Redis). If it has already been executed, return successfully, and if it hasn't been executed again - for example, when ordering a task, use the order ID as a unique identifier, check whether Redis has a key of 'order: 123: executed' before executing, skip it if it does, and when it doesn't, execute inventory, and save the key after execution.”**

### 3. Q3: "With a full thread pool, what other way is there to avoid losing a task than to reject the policy?"

**This is an examination of "global thinking," which cannot focus solely on rejection tactics:**

 **Answer template ** **:“In addition to selecting the right refusal strategy, you need to reduce the occurrence of rejections at the source: First, preheat core threads ahead of time (use prestartAllCoreThreads ()) to avoid first task delays causing queue accumulation; Second, set thread pool parameters reasonably (core number of threads)= business peak QPS / single-thread QPS, queue capacity = peak duration * single-threaded QPS), such as kill a peak 1000QPS, a single thread handles 10 tasks per second, set the core number of threads to 100, and set the queue depth to 500, which basically can handle a peak;Third, check the thread pool status before submitting a task if the queue is nearly full (remainingCapacity () < 100）， Just return 'The system is busy, please try again later' in advance to guide the user to try again and avoid triggering the denial policy.”**

## V. Summary: 3 sentences to choose a candidate + avoid pitfalls

1. **Selection logic****Use the MQ Retry Policy for core business (not losing tasks), the DiscardOldest Policy for non-core real-time scenarios (restoring old) and the CallerRuns Policy for simple scenarios, not blocking core threads. AbortPolicy and DiscardPolicy are definitely not recommended;**
2. **The key to avoiding a hole****Don't block the core thread with CallerRunsPolicy, don't forget idempotent design with MQ retry, don't exceed the resource limit with dynamic expansion;**
3. **The Ultimate Principle****A denial strategy is just a "bottom line scenario" and more importantly, "optimizing thread pool parameters + source control traffic in advance" to radically reduce the occurrence of denial.**

**Brother who finds it useful, like it and bookmark it in case you need it in your next interview!**

![picture](https://mmbiz.qpic.cn/mmbiz_png/aSJ8tDK6zEvszlibuuTyznopvibUMu8qbDGFUVNJnkU1XFggWu9yLRlLQzxA99C12ZtGicU4zun2tx3UP1LicmJ9SA/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=1)

**If you want to know more high-frequency interview questions , please follow the WeChat public number** **Fox love sharing** **, I prepared a** **million-word interview bible** **（ already more than 2 million words ） ,** **there are hundreds of questions about high concurrency and distributed project scenarios , which need to be answered** **For those who want to interview , please pick it up .**
