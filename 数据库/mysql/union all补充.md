## 一、union 和 union all(记录合并)

### 1.1 区别

1. `union`：需要进行重复值扫描，效率低。
2. `union all`：合并时没有刻意删除合并行



### 1.2 注意的地方

&emsp;&emsp;两个联合的`sql`语句字段个数必须一样，而且字段类型要“相容”（一致）；



### 1.3 四个合并方式

- `Union`：对两个结果集进行并集操作，不包括重复行，同时进行默认规则的排序；
- `Union All`：对两个结果集进行并集操作，包括重复行，不进行排序；
- `Intersect`：对两个结果集进行交集操作，不包括重复行，同时进行默认规则的排序；
- `Minus`：对两个结果集进行差操作，不包括重复行，同时进行默认规则的排序。

> 可以在最后一个结果集中指定Order by子句改变排序方式。



### 1.4 注意事项

1. 列名则不一定需要相同，oracle会将第一个结果的列名作为结果集的列名
2. 没有必要在每一个select结果集中使用order by子句来进行排序，我们可以在最后使用一条order by来对整个结果进行排序