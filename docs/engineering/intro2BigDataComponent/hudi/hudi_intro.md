# 快学 Hudi


## 网站链接
* 官网链接：[hudi]
* Roadmap: [hudi roadmap]
* issue 平台：[hudi issue 平台]
* github: [hudi 代码]


## 目标业务场景（是什么）
官方定义：
Hudi 是一个**事务型**的数据湖平台，兼具数据库、数据仓库的功能。
通过 **增量处理框架** 来实现分钟级别的批处理。

官方定义有俩重点：

* 支持**事务**：HDFS 是大数据存储的基石、只支持追加数据；Hudi 支持 **事务** 也就是额外支持了删除（不是全删、删除满足指定条件的数据）、修改操作。但是这里的 **事务** 需要和传统的 OLTP 相区别，
    * 传统的 OLTP 是绝大部分场景是高并发、低延迟地处理多条 Query，每条 Query 只修改很少的数据，比如 12306 订票
    * 此处的**事务**大部分情况下指一次更新一批数据。比如有两个表，一个存了`用户全量的购物记录` user_shopping_list_all, 另外一张表里面存了`用户当日购物记录` user_shopping_list_daily, 每天都需要定时更新`用户全量购物记录表`，user_shopping_list_all 会一直存在数据仓库里面供数据分析人员做分析（OLAP），user_shopping_list_daily 会由线上数据同步到数仓，再和历史的 ，user_shopping_list_all 做合并（update or insert）；最笨的方法是做 fullout join，保留 mismatch 的数据、合并更新历史上出现过的数据。
* **增量处理框架**加快批处理：这个是技术手段，如何做到让**事务**进行地更快（而不是用最笨的办法）。

## 大纲
网站链接

- [x] 官方网站链接
- [x] roadmap
- [x] issue 平台
- [x] 项目代码链接

整体了解

- [x] 目标业务场景？
- [ ] 有哪些 feature？
- [ ] 现有技术有什么问题？
- [ ] hudi 是如何解决这些问题的？
  - [ ] 实际效果如何
- [ ] 有哪些竞品
- [ ] 重要的技术细节

项目代码

- [ ] 各个 module 的作用是什么？
- [ ] 项目的框架图？
- [ ] 有哪些重要的抽象？
  - [ ] 对应的类、函数等
- [ ] demo work 机制（如 一条 SQL 在 Spark 中运行之旅）
- [ ] 系统功能切面
  - [ ] 日志
  - [ ] 监控


[hudi]:https://hudi.apache.org/
[hudi roadmap]: https://hudi.apache.org/roadmap/
[hudi issue 平台]: https://issues.apache.org/jira/projects/HUDI/issues/HUDI-7153?filter=allopenissues
[hudi 代码]: https://github.com/apache/hudi