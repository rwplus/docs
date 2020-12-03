# Metric Types

Prometheus client libraries提供了四种核心metric类型。 这些仅在客户端库（以启用针对特定类型的使用量身定制的API）和wire prototcal中有所区别

## 4 Core metric types

### 计数器 - Counter
计数器是一个累积量度，代表一个单调递增的计数器，其值只能增加或在重新启动时重置为零。 例如，您可以使用计数器来表示已服务请求，已完成任务或错误的数量。

不要使用计数器来显示可以减小的值。 例如，请勿对当前正在运行的进程数使用计数器； 而是使用量规。

``` golang
type Counter interface {
    Metric
    Collector

    // Inc increments the counter by 1. Use Add to increment it by arbitrary
    // non-negative values.
    Inc()
    // Add adds the given value to the counter. It panics if the value is <
    // 0.
    Add(float64)
}
```

### 测量 - Gauge
Gauge是一种度量标准，代表可以任意上下波动的单个数值。

Gauges通常用于测量值，例如温度或当前内存使用量，还用于可能上升和下降的“计数”，例如并发请求数。

``` golang
type Gauge interface {
    Metric
    Collector

    // Set sets the Gauge to an arbitrary value.
    Set(float64)
    // Inc increments the Gauge by 1. Use Add to increment it by arbitrary
    // values.
    Inc()
    // Dec decrements the Gauge by 1. Use Sub to decrement it by arbitrary
    // values.
    Dec()
    // Add adds the given value to the Gauge. (The value can be negative,
    // resulting in a decrease of the Gauge.)
    Add(float64)
    // Sub subtracts the given value from the Gauge. (The value can be
    // negative, resulting in an increase of the Gauge.)
    Sub(float64)

    // SetToCurrentTime sets the Gauge to the current Unix time in seconds.
    SetToCurrentTime()
}
```

### 直方图 - Histogram

直方图对观察值（通常是请求持续时间或响应大小之类的东西）进行采样，并将其计数在可配置的存储桶中。 它还提供所有观察值的总和。

基本度量标准名称为<basename>的直方图在抓取期间会显示多个时间序列：

- 观察桶的累积计数器，显示为<basename>_bucket {le="<包含上边界>"}
- 所有观测值的总和，显示为<basename>_sum
- 观察到的事件计数，显示为<basename>_count（与上面的<basename>_bucket {le="+Inf"}相同）

使用histogram_quantile（）函数可以根据直方图甚至是直方图的聚合来计算分位数。 直方图也适用于计算Apdex得分。 在buckets上操作时，请记住直方图是累积的。 有关直方图用法的详细信息以及与摘要的差异，请参见[histograms and summaries](https://prometheus.io/docs/practices/histograms)

``` Golang
type Histogram interface {
    Metric
    Collector

    // Observe adds a single observation to the histogram.
    Observe(float64)
}
```

### 摘要 - Summary
类似于直方图，Summary会采样观察值（通常是请求持续时间和响应大小之类的东西）。 虽然它还提供了观测值的总数和所有观测值的总和，但它可以计算滑动时间窗口内的可配置分位数。

basic metric name为<basename>的summary会在scraped期间显示多个时间序列：


- Streams观察到的事件的φ分位数（0≤φ≤1），显示为`<basename>{quantile ="<φ>"}`
- 所有观测值的总和，显示为`<basename>_sum`
- 观察到的事件计数，显示为`<basename>_count`
- 有关φ分位数的详细说明，摘要用法以及与直方图的差异，请参见[histograms and summaries](https://prometheus.io/docs/practices/histograms) 。

``` Golang
type Summary interface {
    Metric
    Collector

    // Observe adds a single observation to the summary.
    Observe(float64)
}
```

