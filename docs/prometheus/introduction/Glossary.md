# :fontawesome-brands-wikipedia-w: glossary
---

## 术语表

### Alert
Alerts是Prometheus中积极触发的警报规则的结果。Alerts从Prometheus发送到Alertmanager。

### Alertmanager
Alertmanager接收警报，将警报聚合成组，进行重复数据删除，应用静默，限制，然后将通知发送到电子邮件，Pagerduty，Slack等。

### Bridge
Bridge是从客户端库中获取样本并将其暴露给非Prometheus监视系统的组件。例如，Python，Go和Java客户端可以将指标导出到Graphite。

### Client library
Client library是某种语言（例如Go，Java，Python，Ruby）的库，可以轻松地直接检测代码，编写自定义收集器以从其他系统提取指标并将指标公开给Prometheus。

### Collector
Collector是代表一组metrics的exporter的一部分。如果它是直接检测的一部分，则可能是单个metric，如果是从另一个系统中pull metrics，则可能是多个metrics。

### Direct instrumentation
Direct instrumentation是使用客户端库内联作为程序源代码的一部分内联添加的检测。

### Endpoint
Endpoint是可以被scraped的metric来源，通常对应于单个过程。

### Exporter
Exporter是与要从中获取metrics的应用程序一起运行的二进制文件。Exporter通常通过将以非Prometheus格式公开的metrics转换为Prometheus支持的格式来暴露Prometheus metrics。

### Instance
Instance是唯一标识job中Target的标签（label）。

### Job
Job具有相同目的的target集合（例如，监视一组为可伸缩性或可靠性而复制的相似过程）被称为作业。

### Notification
Notification代表一组一个或多个alerts，并由Alertmanager发送到电子邮件，Pagerduty，Slack等。

### Promdash
Promdash是Prometheus的本地仪表板构建器。它已被弃用，并由Grafana取代。

### Prometheus
Prometheus通常是指Prometheus系统的核心二进制文件。它也可能是整个Prometheus监视系统的参考。

### PromQL
PromQL是Prometheus查询语言。它允许进行多种操作，包括聚合，切片和切块，预测和连接。

### Pushgateway
Pushgateway保留了批处理作业中最新的push of metrics。这样，Prometheus可以在终止指标后抓取其指标。

### Remote Read
Remote Read是Prometheus的一项功能，该功能允许作为查询的一部分从其他系统（例如长期存储）透明读取时间序列。

### Remote Read Adapter
并非所有系统都直接支持远程读取。Remote Read Adapter位于Prometheus和另一个系统之间，可在它们之间转换时间序列请求和响应。

### Remote Read Endpoint
Remote Read Endpoint是Prometheus在进行远程读取时要与之通信的端点。

### Remote Write
Remote Write是Prometheus的一项功能，它允许将摄取的样本（ingested samples）即时发送到其他系统，例如长期存储。

### Remote Write Adapter
并非所有系统都直接支持远程写入。Remote Write Adapter位于Prometheus和另一个系统之间，将远程写操作中的样本转换为另一个系统可以理解的格式。

### Remote Write Endpoint
Remote Write Endpoint是Prometheus在执行远程写时要与之通信的端点。

### Sample
样本是时间序列中某个时间点的单个值
在Prometheus中，每个样本都包含一个float64值和一个毫秒精度的时间戳。

### Silence
Alertmanager中的Silence功能可防止带有与Silence匹配的标签的alerts被包含在通知中。

### Target
Target是要scrape的对象的定义。例如，要应用哪些标签，连接所需的任何身份验证或定义scrape方式的其他信息。