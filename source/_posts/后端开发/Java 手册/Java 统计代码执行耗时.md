---
title: Java 统计代码执行耗时

categories:
- 后端开发
- Java 手册

date: 2021-05-11
---
代码耗时统计在日常开发中算是一个十分常见的需求，特别是在需要找出代码性能瓶颈时。

可能也是受限于 Java 的语言特性，总觉得代码写起来不够优雅，大量的耗时统计代码，干扰了业务逻辑。特别是开发功能的时候，有个感受就是刚刚开发完代码很清爽优雅，结果加了一大堆辅助代码后，整个代码就变得臃肿了，自己看着都挺难受。因此总想着能不能把这块写的更优雅一点，今天本文就尝试探讨下“代码耗时统计”这一块。

在开始正文前，先说下前提，“代码耗时统计”的并不是某个方法的耗时，而是任意代码段之间的耗时。这个代码段，可能是一个方法中的几行代码，也有可能是从这个方法的某一行到另一个被调用方法的某一行，因此通过 AOP 方式是不能实现这个需求的。

## 常规方法
#### 时间差统计
这种方式是最简单的方法，记录下开始时间，再记录下结束时间，计算时间差即可。

```java
public class TimeDiffTest {
    public static void main(String[] args) throws InterruptedException {
        final long startMs = TimeUtils.nowMs();

        TimeUnit.SECONDS.sleep(5); // 模拟业务代码

        System.out.println("timeCost: " + TimeUtils.diffMs(startMs));
    }
}

/* output: 
 * timeCost: 5005
 */
public class TimeUtils {
    /**
     * @return 当前毫秒数
     */
    public static long nowMs() {
        return System.currentTimeMillis();
    }

    /**
     * 当前毫秒与起始毫秒差
     * @param startMillis 开始纳秒数
     * @return 时间差
     */
    public static long diffMs(long startMillis) {
       return diffMs(startMillis, nowMs());
    }
}
```

这种方式的优点是实现简单，利于理解；缺点就是对代码的侵入性较大，看着很傻瓜，不优雅。

#### StopWatch
第二种方式是参考 `StopWatch`，`StopWatch` 通常被用作统计代码耗时，各个框架和 Common 包都有自己的实现。

```java
public class TraceWatchTest {
    public static void main(String[] args) throws InterruptedException {
        TraceWatch traceWatch = new TraceWatch();

        traceWatch.start("function1");
        TimeUnit.SECONDS.sleep(1); // 模拟业务代码
        traceWatch.stop();

        traceWatch.start("function2");
        TimeUnit.SECONDS.sleep(1); // 模拟业务代码
        traceWatch.stop();

        traceWatch.record("function1", 1); // 直接记录耗时

        System.out.println(JSON.toJSONString(traceWatch.getTaskMap()));
    }
}

/* output: 
 * {"function2":[{"data":1000,"taskName":"function2"}],"function1":[{"data":1000,"taskName":"function1"},{"data":1,"taskName":"function1"}]}
 */
public class TraceWatch {
    /** Start time of the current task. */
    private long startMs;

    /** Name of the current task. */
    @Nullable
    private String currentTaskName;

    @Getter
    private final Map<String, List<TaskInfo>> taskMap = new HashMap<>();

    /**
     * 开始时间差类型指标记录，如果需要终止，请调用 {@link #stop()}
     *
     * @param taskName 指标名
     */
    public void start(String taskName) throws IllegalStateException {
        if (this.currentTaskName != null) {
            throw new IllegalStateException("Can't start TraceWatch: it's already running");
        }
        this.currentTaskName = taskName;
        this.startMs = TimeUtils.nowMs();
    }

    /**
     * 终止时间差类型指标记录，调用前请确保已经调用
     */
    public void stop() throws IllegalStateException {
        if (this.currentTaskName == null) {
            throw new IllegalStateException("Can't stop TraceWatch: it's not running");
        }
        long lastTime = TimeUtils.nowMs() - this.startMs;

        TaskInfo info = new TaskInfo(this.currentTaskName, lastTime);

        this.taskMap.computeIfAbsent(this.currentTaskName, e -> new LinkedList<>()).add(info);

        this.currentTaskName = null;
    }

    /**
     * 直接记录指标数据，不局限于时间差类型
     *
     * @param taskName 指标名
     * @param data 指标数据
     */
    public void record(String taskName, Object data) {
        TaskInfo info = new TaskInfo(taskName, data);

        this.taskMap.computeIfAbsent(taskName, e -> new LinkedList<>()).add(info);
    }

    @Getter
    @AllArgsConstructor
    public static final class TaskInfo {
        private final String taskName;

        private final Object data;
    }
}
```

## 高级方法
上面提到的两种方法，用大白话来说都是“直来直去”的感觉，我们还可以尝试把代码写的更简便一点。

#### Function
在 JDK 1.8 中，引入了 `java.util.function` 包，通过该类提供的接口，能够实现在指定代码段的上下文执行额外代码的功能。

```java
public class TraceHolderTest {
    public static void main(String[] args) {
        TraceWatch traceWatch = new TraceWatch();

        TraceHolder.run(traceWatch, "function1", i -> {
            try {
                TimeUnit.SECONDS.sleep(1); // 模拟业务代码
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        String result = TraceHolder.run(traceWatch, "function2", () -> {
            try {
                TimeUnit.SECONDS.sleep(1); // 模拟业务代码
                return "YES";
            } catch (InterruptedException e) {
                e.printStackTrace();
                return "NO";
            }
        });

        TraceHolder.run(traceWatch, "function1", i -> {
            try {
                TimeUnit.SECONDS.sleep(1); // 模拟业务代码
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        System.out.println(JSON.toJSONString(traceWatch.getTaskMap()));
    }
}

/* output: 
 * {"function2":[{"data":1004,"taskName":"function2"}],"function1":[{"data":1001,"taskName":"function1"},{"data":1002,"taskName":"function1"}]}
 */
public class TraceHolder {
    /**
     * 有返回值调用
     */
    public static <T> T run(TraceWatch traceWatch, String taskName, Supplier<T> supplier) {
        try {
            traceWatch.start(taskName);

            return supplier.get();
        } finally {
            traceWatch.stop();
        }
    }

    /**
     * 无返回值调用
     */
    public static void run(TraceWatch traceWatch, String taskName, IntConsumer function) {
        try {
            traceWatch.start(taskName);

            function.accept(0);
        } finally {
            traceWatch.stop();
        }
    }
}
```

#### AutoCloseable
除了利用 `Function` 的特性，我们还可以使用 JDK 1.7 的 `AutoCloseable` 特性。说 `AutoCloseable` 可能有同学没听过，但是给大家展示下以下代码，就会立刻明白是什么东西了。

```java
// 未使用 AutoCloseable
public static String readFirstLingFromFile(String path) throws IOException {
    BufferedReader br = null;
    try {
        br = new BufferedReader(new FileReader(path));
        return br.readLine();
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        if (br != null) {
            br.close();
        }
    }
    return null;
}

// 使用 AutoCloseable
public static String readFirstLineFromFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    }
}
```

在 `try` 后方可以加载一个实现了 `AutoCloseable` 接口的对象，该对象作用于整个 `try` 语句块中，并且在执行完毕后回调 `AutoCloseable#close()` 方法。

让我们对 `TraceWatch` 类进行改造。

实现 `AutoCloseable` 接口，实现 `close()` 接口，并修改 `start()` 方法，使其支持链式调用：

```java
public class AutoCloseableTest {
    public static void main(String[] args) {
        TraceWatch traceWatch = new TraceWatch();

        try(TraceWatch ignored = traceWatch.start("function1")) {
            try {
                TimeUnit.SECONDS.sleep(1); // 模拟业务代码
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        try(TraceWatch ignored = traceWatch.start("function2")) {
            try {
                TimeUnit.SECONDS.sleep(1); // 模拟业务代码
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        try(TraceWatch ignored = traceWatch.start("function1")) {
            try {
                TimeUnit.SECONDS.sleep(1); // 模拟业务代码
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        System.out.println(JSON.toJSONString(traceWatch.getTaskMap()));
    }
}

/* output: 
 * {"function2":[{"data":1001,"taskName":"function2"}],"function1":[{"data":1002,"taskName":"function1"},{"data":1002,"taskName":"function1"}]}
 */
public class TraceWatch implements AutoCloseable{
   @Override
   public TraceWatch start(String taskName) throws IllegalStateException {
      if (this.currentTaskName != null) {
         throw new IllegalStateException("Can't start TraceWatch: it's already running");
      }
      this.currentTaskName = taskName;
      this.startMs = TimeUtils.nowMs();
      
      return this;
   }

   @Override
   public void close() {
      this.stop();
   }

   ...
}
```