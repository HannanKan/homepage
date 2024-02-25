# A Tour of Learning Thread Leak

## 线程泄露的简介
想必大家都听过**内存泄露**，简单来说就是，用户申请的一块内存，因为无法被系统回收而不能再被重复利用；危害是可能导致程序报错 OutOfMemory(OOM)。

笔者此前还没听说过 **线程泄露** 这一概念。实际上，它和内存泄露类似，就是指，一个用户申请的线程因无法被回收而不能重复利用。线程占用 stack memory、线程中的对象又占用 heap memory，因此，线程泄露也是一种形式的内存泄露。被泄露的线程无法关闭，也会让操作系统产生**调度开销（如Context Switch等消耗cpu等）**；如果有大量的线程泄露，会导致操作系统将cpu资源消耗殆尽。

talk is cheap, show me the code! 上代码
```java

1. ExecutorService executor = Executors.newFixedThreadPool(10);

2. List<String> getCommonFriends() throws ExecutionException, InterruptedException = {
3.    Future<List<String>> johnFriendsFuture = executor.submit(() -> getFriends("John"));
4.    Future<List<String>> bobFriendsFuture =    executor.submit(() -> getFriends("Bob")); 

5.    List<String> johnFriends = johnFriendsFuture.get(); //Join
6.    List<String> bobFriends = bobFriendsFuture.get(); //Join 

7.    return commonFriends(johnFriends, bobFriends);
8. }
```

快问：前面这段代码可能在哪里发生线程泄露呢？

快答：LOC 6。 LOC 3、4 提交两个 task 到线程池中，对应两个线程。当 LOC 5 抛异常后，LOC 4 提交的 task 一直占用着一个线程，无法被释放。


## 问题复现
> 系统方法论: 解决问题的第一步是复现问题。

如果线上发生了异常，如何确定是否是线程泄露呢？🤔 

这里我们先复现一下线程泄露，然后看有什么现象。


# Ref
[Finding Java Thread Leaks With JDK Flight Recorder and a Bit Of SQL]

[Finding Java Thread Leaks With JDK Flight Recorder and a Bit Of SQL]: https://www.morling.dev/blog/finding-java-thread-leaks-with-jdk-flight-recorder-and-bit-of-sql/