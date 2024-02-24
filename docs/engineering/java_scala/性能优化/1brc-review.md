# [高性能 Java] 1BRC 比赛优秀代码分析
[The One Billion Row Challenge(1BRC): Java 处理亿级别数据的性能调优] 这篇文章介绍了 1BRC 比赛，以及笔者个人思考的一些可能的优化。 

本次获奖的第一名大佬 [@thomaswue] 是 GraalVM 项目的 lead，第三名大佬 [@jerrinot] 是 QuestDB 成员之一。

本文将介绍比赛代码中都用了哪些比较有意思的优化。比赛前三名用到的优化大同小异，后续介绍以第一名代码（[No.1代码]）为例。

## 优化一: AOT 编译
使用了 21.0.2-graal。

使用 [GraalVM] 可以 **静态编译** Java 字节码(bytecode, .class 文件) ，得到二进制机器码 (native binary)，提升 Java 程序性能。

Java的执行过程可以分为两个部分：

1. javac 将 Java 源文件编译成 Java 字节码；
2. jvm 将字节码逐条**解释执行**。

在解释执行的过程中，如果发现**热点代码**，使用**即时编译器（JIT，更多细节参考[Java及时编译器]）**将其直接编译成机器码；下次执行时直接运行机器码，这样的速度比解释执行要快。在JIT编译器将字节码翻译成机器码之前的运行，被称作 warmup，jvm 在这个过程中发现热点代码。

GraalVM 支持通过**静态编译**将Java字节码翻译成机器码，这样的好处包括：可执行文件更小；由于无需 warmup，运行的直接就是机器码，能更快的达到峰值性能。


## 优化二: 系统层面优化
先 show 代码
```java
int numberOfWorkers = Runtime.getRuntime().availableProcessors();
try (var fileChannel = FileChannel.open(java.nio.file.Path.of(FILE), java.nio.file.StandardOpenOption.READ)) {
    long fileSize = fileChannel.size();
    final long fileStart = fileChannel.map(FileChannel.MapMode.READ_ONLY, 0, fileSize, java.lang.foreign.Arena.global()).address();
    final long fileEnd = fileStart + fileSize;
    final AtomicLong cursor = new AtomicLong(fileStart);

    // Parallel processing of segments.
    Thread[] threads = new Thread[numberOfWorkers];
    List<Result>[] allResults = new List[numberOfWorkers];
    for (int i = 0; i < threads.length; ++i) {
        final int index = i;
        threads[i] = new Thread(() -> {
            List<Result> results = new ArrayList<>(MAX_CITIES);
            parseLoop(cursor, fileEnd, fileStart, results);
            allResults[index] = results;
        });
        threads[i].start();
    }
```
从前面代码明显可以看出来，分别采用了下面两大优化

1.  多线程优化：拿到 JVM 能够获取的所有processor，启动相同数目的**线程**处理数据。提高 CPU 利用率。
2.  mmap + unsafe: ` FileChannel open(Path path, OpenOption... options)` 和 ` MappedByteBuffer map(MapMode mode, long position, long size)` 将输入文件 mmap 到内存中，获取内存地址后，利用 unsafe 逐个字节的读取处理（而不是直接使用的 Java 中 FileReader)。减少用户、内核态切换，cpu 友好。


前面代码中可以看到，每个线程调用 `parseLoop` 函数来进行处理。
在`parseLoop`函数中，每次读一个 batch 的数据(SEGMENT_SIZE 大小)，进一步将 batch 均分成 3 份，然后一个线程中同时处理这三份数据。代码如下所示。

```java
 private static void parseLoop(AtomicLong counter, long fileEnd, long fileStart, List<Result> collectedResults) {
        Result[] results = new Result[HASH_TABLE_SIZE];
        while (true) {
            long current = counter.addAndGet(SEGMENT_SIZE) - SEGMENT_SIZE;
            if (current >= fileEnd) {
                return;
            }

            /** 处理边界 **/

            long dist = (segmentEnd - segmentStart) / 3;
            long midPoint1 = nextNewLine(segmentStart + dist);
            long midPoint2 = nextNewLine(segmentStart + dist + dist);

            Scanner scanner1 = new Scanner(segmentStart, midPoint1);
            Scanner scanner2 = new Scanner(midPoint1 + 1, midPoint2);
            Scanner scanner3 = new Scanner(midPoint2 + 1, segmentEnd);
            while (true) {
                if (!scanner1.hasNext()) {
                    break;
                }
                if (!scanner2.hasNext()) {
                    break;
                }
                if (!scanner3.hasNext()) {
                    break;
                }
                
                /** 同时迭代 scanner1、scanner2、scanner3 **/
            }

            while (scanner1.hasNext()) {
                /** 迭代剩余 scanner1 **/
            }
            while (scanner2.hasNext()) {
                /** 迭代剩余 scanner2 **/ 
            }
            while (scanner3.hasNext()) {
                /** 迭代剩余 scanner3 **/
            }
        }
    }
```

batch 读取后，切分成3份并发处理，优化思路如下: 

1. batch 读能利用缓存，降低 IO 开销，避免更小粒度读取
2. 并发处理，提高 cpu 利用率
3. 加快 JVM 识别、优化热点代码

## 优化三: partition 策略，work steal 优于 equal split
并行读取大文件，最直观、简单的方法是：先将文件均匀切分(equal split)，然后每个线程读取属于各自的那部分数据。但这样做也可能存在劣势，即某个线程处理完自己那份数据后就开始“摸鱼”了。

第一名则采用了类似 "work stealing" 的策略。使用一个原子变量表示文件指针，每个线程尝试读取文件最新的那个 batch 数据（2MB）。核心代码是
`long current = counter.addAndGet(SEGMENT_SIZE) - SEGMENT_SIZE;`。这样就避免了有活没干完但还有线程“摸鱼”的情况。
```java
 private static void parseLoop(AtomicLong counter, long fileEnd, long fileStart, List<Result> collectedResults) {
        Result[] results = new Result[HASH_TABLE_SIZE];
        while (true) {
            long current = counter.addAndGet(SEGMENT_SIZE) - SEGMENT_SIZE;
            if (current >= fileEnd) {
                return;
            }

            long segmentEnd = nextNewLine(Math.min(fileEnd - 1, current + SEGMENT_SIZE));
            long segmentStart;
            if (current == fileStart) {
                segmentStart = current;
            }
            else {
                segmentStart = nextNewLine(current) + 1;
            }
        // ...
        }
        // ...
 }
```

## 优化四: lookup table 数据结构

此处利用 mask 方法实现了一个 lookup table 数据结果（没有直接用 hashset）。主要逻辑在 [findResult函数] 中。
暂时没有时间深入看，感兴趣的朋友可以自行看下。


## todo: [The Billion Row Challenge (1BRC) - Step-by-step from 71s to 1.7s]




[The One Billion Row Challenge(1BRC): Java 处理亿级别数据的性能调优]: ./1brc.md
[The Billion Row Challenge (1BRC) - Step-by-step from 71s to 1.7s]: https://questdb.io/blog/billion-row-challenge-step-by-step
[No.1代码]: https://github.com/gunnarmorling/1brc/blob/main/src/main/java/dev/morling/onebrc/CalculateAverage_thomaswue.java
[GraalVM]: https://www.graalvm.org/
[@thomaswue]: https://github.com/thomaswue
[GraalVM编译选项]: https://www.graalvm.org/latest/reference-manual/native-image/overview/Options/
[Java及时编译器]: https://tech.meituan.com/2020/10/22/java-jit-practice-in-meituan.html
[findResult函数]: https://github.com/gunnarmorling/1brc/blob/c92346790e8548f52e81254227efc935356e5e53/src/main/java/dev/morling/onebrc/CalculateAverage_thomaswue.java#L192
[@jerrinot]: https://github.com/jerrinot