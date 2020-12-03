# Data Model

Prometheus从根本上将所有数据存储为时间序列：带有时间戳的值流属于同一度量标准和同一组标注维。 除了存储的时间序列外，Prometheus可能会生成临时派生的时间序列作为查询的结果。

## Metric names and labels

每一个时间序列由metric name和成为labels的可选的key-value来唯一标识。

 metric name指定了一个被测量（measured）系统的一般功能。比如`http_requests_total` 代表了http request的请求总数。可以包含ASCII字母和数字，也可以是下划线(underscores)和冒号(colons)。它必须与正则表达式`[a-zA-Z _：] [a-zA-Z0-9 _：]*`相匹配。

 > 注意：冒号是为用户定义的记录规则保留的。 export或direct instrumentation不应使用它们。


 Labels启用Prometheus的维度数据模型：具有相同metric name的labels的任何给定的组合都可以标识该metric的特定维度实例（比如：所有使用POST方法到`/api/tracks` handler的HTTP请求）。PromQL允许基于这些维度去过滤和聚合。更改任何label值（包括添加或者删除一个label都会创建一个新的时间序列）

 标签名称可能包含ASCII字母，数字和下划线。 它们必须匹配正则表达式`[a-zA-Z_] [a-zA-Z0-9_]*.` 以`__`开头的标签名称保留供内部使用。


 标签值为空的标签被认为等同于不存在的标签。

 更多请查看[best practices for naming metrics and labels](https://prometheus.io/docs/practices/naming/)

## samples

 samples构成实际的时间序列数据。每一个sample包括：

 - 一个float64的值
 - 一个毫秒精度的时间戳

## Notation符号

 给定一个metric name和一组labels，通常使用遗下表示法标识时间序列
```
<metric name>{<label name>=<label value>, ...}
```

比如，一个metric name为`api_http_requests_total`且label为`method ="POST"`和`handler="/messages"`的时间序列可以这样写：

```
api_http_requests_total{method="POST", handler="/messages"}
```
