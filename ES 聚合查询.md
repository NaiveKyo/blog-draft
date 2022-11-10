# ElasticSearch 聚合查询

elasticsearch version：7.6

## 1、聚合操作简介

在 ES 中，聚合查询模块可以基于一次查询来构建聚合数据，这种查询方式叫做 aggregations。利用 aggregations，我们为一些数据生成复杂的摘要信息。

可以将聚合看作为一批文档生成分析数据的一个工作单元，它的执行上下文由文档集合来定义，比如说在一次查询请求中，query/filters 的上下文就是顶级聚合（top aggregation）的执行上下文。

聚合有很多种类，出于不同的目的来使用它们从而得到不同的输出结果。为了更好的理解聚合，可以简单的将其分为 4 种类型：

（1）Bucketing；

其中一种聚合叫做构建桶（桶聚合，build buckets），有点类似于 map 的概念，桶将一个 key 和一个文档标准关联起来。在一次聚合操作执行上下文中，涉及到的所有文档数据都会被所有的桶标准所处理，当某个文档匹配上了某个标准时，这个文档将会 "fall in" 到相关的桶中，当聚合操作结束时，我们就可以得到一个桶集合，集合中的每个桶中都有对应的一组文档数据，我们称这些文档 "belong" 对应的桶。

（2）Metric；

Metric 聚合就是跟踪一组文档并计算度量数据（keep trace and compute metrics over a set of documents）。

（3）Matrix；

在一次聚合操作上下文中，Matrix 聚合可以针对文档的某些字段进行操作，比如说基于某些字段的数据进行计算并将结果以矩阵的形式输出，和 Metric、Bucket 聚合不同，这种聚合暂时不支持脚本操作（Elasticsearch version 7.6）。

（4）Pipeline；

顾名思义，Pipeline 聚合是基于其他聚合输出和相关度量数据的基础之上再次进行聚合操作。

有趣的部分来了，我们知道在桶聚合的过程中，每个桶都有属于自己的一组文档数据，这就意味着在 bucket level 上可以构建属于自己的聚合操作，此种聚合的上下文就是所属的桶。这正是聚合操作的强大之处：聚合是可以嵌套的。

## 2、聚合的基本结构

下面的代码结构片段展示了一个基本的聚合操作的组成部分：

```json
"aggregations": {
    "<aggregation_name>": {
        "<aggregation_type>": {
            <aggregation_body>
        }
        [,"meta": { [<meta_data_body>] }]?
    	[,"aggregations": { [<sub_aggregation>]+ }]?
    }
	[,"<aggregation_name_2>": { ... }]
}
```

上述 json 片段中，`aggregations` （别名 `aggs`）对象持有一系列要执行的聚合操作属性。使用者需要为每个聚合操作定义一个逻辑名称（比如说要计算价格平均值的聚合可以叫做 "avg_price"）。在返回结果中这些独特的逻辑名称都会对应一组计算结果数据。

每个聚合都会拥有一个特别的属性：`aggregation_type`，它代表聚合操作的具体类型，对应的值就是 `aggregation_body`，body 的内容就取决于对应聚合操作的特性（比如说计算平均值的聚合操作，它的 body 中需要有一个特殊的属性指定要为文档的哪个属性求平均值）。

从上述代码中可以看到与 `aggregation_type` 同级别的还有一些可选属性，例如 mate、aggregations，但是它们只有在 bucket 聚合中才会生效。在 bucket 级别上定义的 sub-aggregation 将会在构建的所有桶中生效，比如说在 range aggregation 下定义了一组聚合操作，这些子聚合将会在 range bucket 聚合构建的所有桶中进行计算。

## 3、Values Source

有些聚合处理的数据来源于被聚合的所有文档。一般情况下，在聚合中通过设置 `field` key 来指定文档中要参与聚合计算的属性，但是也可以定义 `script`（脚本） 为每个文档动态构建一些值。

当开发者在聚合操作中同时使用了 `field` 和 `script` 配置，`script` 将被看作 `value script` 。

- 常规脚本（normal scripts）生效的级别是 document level（意味着此类脚本拥有文档所有属性的访问权限）；
- values scripts 生效的级别是 value level。在这种模式下，聚合中配置的 `field` 和 `script` 可以看作是一种 "transformation" 作用于文档的特定属性上。

Elasticsearch 在计算聚合操作和处理聚合响应结果时会使用文档 mapping 中配置的 field 的类型。但是 Elasticsearch 在以下两种情况下无法读取这些信息：

（1）unmapped fields（比如说一次查询跨多个索引，并且其中只有某些索引对聚合使用到的 filed 进行了 mapping）；

（2）pure scripts；

在这些情况下，使用 `value_type` 提示 Elasticsearch 聚合操作中 field 的类型是一个很好的选择，这个属性接收以下值：string、long（适用于所有整型数据）、double（适用于所有十进制数，比如 float 或者 scaled_float）、date、ip 以及 boolean。