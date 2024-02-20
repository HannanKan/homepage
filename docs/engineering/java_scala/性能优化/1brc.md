# The One Billion Row Challenge(1BRC): Java 处理亿级别数据的性能调优
跟大家分享一个十分有意思的比赛 —— [The One Billion Row Challenge] 简称 1BRC。

## 比赛介绍
它是由 [Gunnar Morling] 发起的一项挑战：给定计算任务，探索其在只使用 modern Java 的要求下能达到的最佳性能。
比赛的要求只能使用纯 Java 技术，不能使用 JNI、不能使用其他 JVM-based 语言。

**通过这项比赛，可以学习到 modern Java 的各种优化技术**，比如

* （虚拟）线程
* Vector API 和 SIMD
* GC优化
* 利用 AOT 编译
* 等各种其他"奇技淫巧"

[Gunnar Morling] 是一位十分资深的 Java 程序员，他在 Twitter 上有 48.9 k follower. 

**计算任务**：给定一个 csv 文本文件，每一行包括一个键值对，键是 string 类型，值是 double 类型；要求对这一亿行数据做聚合（group by）操作，算出每个分组的 min、mean、max。

本文后半部分是自己想到的优化思路。

另外，再提前挖两个坑，有时间再填 :)

* 学习一下前几名提交的代码，看能否学到一些不知道的优化技巧。
* 比赛讨论区还有很多人提交了非 java 的实现，可能也蛮有意思，比如 duckdb 等

## 原理层面的思考🤔

一个程序执行时分为哪几个阶段呢？主要是三个阶段：

* I/O: 包括读、写两个过程，可能的形式包括 文件读写、网络读写
* CPU 计算
* 缓存：中间结果存在 memory 或者 cache 中

要提高性能的话，每个阶段宏观的优化思想（包括但不限于）：

* I/O: 减少阻塞
* CPU 计算：提高**利用率**（尽可能打满 CPU），提高**有效计算**的利用率（减少浪费）
* 缓存：高效的缓存策略，减少 I/O 开销，尽可能使用更高级的缓存（L1 cache > L2 cache > ... > Lx cache > RAM memory > 持久化内存PMEM（Persistent Memory） > SSD > HDD)


## 具体优化技术
个人看法，具体的优化技术主要可以从如下几个方面思考

* 由聚合操作，想到可以借鉴**数据库**中的优化技术
* JVM 语言相关的优化
* 系统层面的优化
* 数据结构的高效使用
* 等

### 数据库优化
| 优化简介         | 优化原理                          |
| ----------- | ------------------------------------ |
| `partition/分治法`  | I/O,CPU: 并行读取、计算，降低 IO开销、提升 CPU 利用率 |
| `batch processing/批处理` | I/O,CPU,缓存: 相比于 tuple-at-a-time, 有效利用系统缓存降低 IO 开销的，减少了 CPU 等IO的时间; 需要合理选择一个 batch 的大小，保证最大化 CPU 利用率的同时避免 OOM 或者 spill disk。|
| `spill disk/溢写磁盘` | 缓存：考虑内存大小，针对中间结果设置写磁盘的阈值，避免 OOM |

### 系统层面的优化
| 优化简介 | 优化原理 |
| -------- | -------- |
| `多线程` |   提升 CPU 利用率，     |
| `向量化` |   Vector API/SIMD 技术，提升 cache 命中率     |
| [mmap]  |   减少用户态和内核态的切换次数，cpu 友好    |
| `写缓存` | 设置缓存区，大小设置为 4kb 整数倍，避免写放大 |

### JVM 语言相关的优化
| 优化简介       | 优化原理 |
| -------------- | -------- |
| `GC 优化`       |   合适的 gc，减少 STW 时间，提升 CPU 利用率      |
| `AOT 编译优化` |   相比 jit， 占用更少内存、更快达到峰值性能      |
| [unsafe和DirectByteBuffer] |  直接操作堆外内存，避免不必要的 gc，提高 CPU 效率        |


### 数据结构的高效使用
| 优化简介 | 优化原理 |
| -------- | -------- |
| `高效的 HashTable ，使用java自带的、guava的或其他的` |  更少的哈希碰撞，减少 CPU 浪费；同时，避免占用过多内存  |

### 其他优化方法
| 优化简介 | 优化原理 |
| -------- | -------- |
| [火焰图🔥] |  CPU 火焰图，找到热点代码路径，针对性优化；off-cpu 火焰图，识别 I/O、网络等阻塞场景导致的性能下降；锁竞争、死锁导致的性能下降问题；内存火焰图，分析内存占用、泄露问题 |




[The One Billion Row Challenge]: https://github.com/gunnarmorling/1brc?tab=readme-ov-file
[Gunnar Morling]: https://twitter.com/gunnarmorling
[火焰图🔥]: https://www.infoq.cn/article/a8kmnxdhbwmzxzsytlga
[unsafe和DirectByteBuffer]: https://tech.meituan.com/2019/02/14/talk-about-java-magic-class-unsafe.html
[mmap]: https://www.cnkirito.moe/learn-mmap/#mmap-vs-FileChannel