multi-thread context(MTC)
=====================================

解决多线程传递Context的需求。

需求场景
----------------------------


功能
----------------------------

1. 父线程创建子线程时，Context传递。
1. 使用线程池时，执行任务Context能传递。

使用说明
=====================================

### 1. 简单使用MtContext

代码示例
----------------------------

```java
// 在父线程中设置
MtContext.set("key", "value-set-in-parent");

// 在子线程中可以读取, 值是"value-set-in-parent"
String value = MtContext.get("key"); 
```

### 2. 保证线程池中传递MtContext

```java
MtContext.set("key", "value-set-in-parent");

Runnable task = new Task("1");
Runnable mtContextRunnable = MtContextRunnable.get(task); // 额外的处理，生成修饰了的对象mtContextRunnable
executorService.submit(mtContextRunnable);

// Task中可以读取, 值是"value-set-in-parent"
String value = MtContext.get("key");
```

上面演示了`Runnable`，`Callable`的处理类似

```java
MtContext.set("key", "value-set-in-parent");

Callable call = new Call("1");
Callable mtContextCallable = MtContextCallable.get(call); // 额外的处理，生成修饰了的对象mtContextRunnable
executorService.submit(mtContextCallable);

// Call中可以读取, 值是"value-set-in-parent"
String value = MtContext.get("key");
```

### 3. 修饰线程池，简化`Runnable`和`Callable`的修饰操作

```java

MtContextExecutors.getMtcExecutorService(executorService); // 额外的处理，生成修饰了的对象executorService

MtContext.set("key", "value-set-in-parent");

Runnable task = new Task("1");
Callable call = new Call("2");
executorService.submit(task);
executorService.submit(call);

// Task或是Call中可以读取, 值是"value-set-in-parent"
String value = MtContext.get("key");
```

### 使用JDK Agent完成线程池修饰操作

代码透明完成`MtContext`传递。
\# 目前Agent中，修饰了`java.util.concurrent.ThreadPoolExecutor`和`java.util.concurrent.ScheduledThreadPoolExecutor`，使用这2个线程池的实现时，

在Java的启动参数加上`-javaagent:path/to/multithread-context-x.y.z.jar`，示例如下：

```bash
java -javaagent:multithread-context-0.9.0-SNAPSHOT.jar \
    -cp dependency/javassist-3.18.1-GA.jar:dependency/log4j-1.2.17.jar:dependency/slf4j-api-1.5.6.jar:dependency/slf4j-log4j12-1.5.6.jar:multithread-context-0.9.0-SNAPSHOT.jar \
    com.oldratlee.mtc.threadpool.agent.AgentDemo
```

FAQ
=====================================

* Mac OS X下，使用javaagent，报JavaLaunchHelper的出错信息  
JDK Bug: http://bugs.sun.com/bugdatabase/view_bug.do?bug_id=8021205
