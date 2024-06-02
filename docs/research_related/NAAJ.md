# NAAJ -- 一个由 NULL 值引发的数据正确性问题
> 关键词: NULL, Not In, Not exists

## Intro
先卖个关子，你知道 `NAAJ`  是什么吗？如果不知道，那么恭喜你，这篇文章值得一看，并且可能帮助你避免一次严重的线上事故。

给定如下 SQL，请问返回值是什么?
```sql
-- 表 t1(a):  1,2,3
-- 表 t2(a):  null, 1

select *
from t1
where t1.a not in (
    select t2.a from t2
)
```

应该会有人觉得返回  `{2, 3}`。实际上，前面的SQL会返回`空结果`。眼尖的同学肯定猜到了，跟表 `t2` 中的 NULL 值有关。

为什么会这样呢？首先让我们来理解一下这段 SQL 的含义：选出 t1 中的数据，要求 t1.a 列的值不在 t2.a 列中。乍一听，肯定会觉得这不就是 `Anti Join(antijoin)` 的语义嘛！其实不完全对，antijoin 有两类：

* regular `antijoin`: 对应 SQL 中的 `not exists` 关键字
* **NULL-Aware Anti Join** (即 **NAAJ**): 对应 SQL 中的 `not in` 关键字

这两类 antijoin 语义上很相似，但是当 外层查询或内层查询存在 NULL 值时，计算结果的差异很大。

论文可参考 Oracle 论文 [oracle vldb09 propose NAAJ] 的介绍。个人觉得 Meta 技术文档 [antijoin from velox] 更容易让人理解，推荐大家细读。

## Spark 中的实现
// todo:


## Take away lesson
当 antijoin 的子查询中不包括 `etra filter` 时（在subquery中**没有**使用来自out query的列构造 non-equality 条件）：

* 要表达 `antijoin` 语义时，注意选择 `NAAJ` 还是 `regular antijoin`: 简言之 `not in` vs `not exists`
* `not in`: 
    * Subquery 包含 NULL: 外层查询返回`空`。
    * Subquery 为`空`: 外层查询返回不在 Subquery 中的数据, NULL 值不会被剔除。
    * Subquery 不包含 NULL: 外层查询返回不在 Subquery 中的数据，同时**剔除掉 NULL 值**。
* `not exists`: 对子查询中是否有 `NULL` 值不敏感，外层查询返回不在 Subquery 中的数据, NULL 值不会被剔除。


## Reference
* [antijoin from velox]
* [oracle ask tom for antijoin]
* [oracle vldb09 propose NAAJ]
* [stackoverflow for antijoin]

[antijoin from velox]: nhttps://facebookincubator.github.io/velox/develop/anti-join.html
[oracle ask tom for antijoin]: https://asktom.oracle.com/ords/asktom.search?tag=null-aware-anti-join
[stackoverflow for antijoin]: https://stackoverflow.com/questions/173041/not-in-vs-not-exists
[oracle vldb09 propose NAAJ]: https://www.vldb.org/pvldb/vol2/vldb09-423.pdf

