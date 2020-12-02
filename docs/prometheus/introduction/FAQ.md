# :fontawesome-solid-question-circle: FAQ（常见问题解答）
---

## 常见的

!!! Question "什么是Prometheus"
    Prometheus是具有活跃生态系统的开源系统监视和警报工具包。

!!! Question "和其他的监控系统比起来怎么样"
    请参考官方的Compare

!!! Question "Prometheus有哪些依赖"
    Prometheus主服务器是standalone模式。没有额外的依赖

!!! Question "Prometheus能否高可用"
    可以高可用模式，在两台或者多台独立的服务器上运行相同的Prometheus。重复的alerts将会被Alertmaneger删除。  
    为了提高Alertmanager的可用性，可以在Mesh集群里运行多个instances，并配置Alertmanager向每个实例发送通知。

!!! Question "有人告诉我Prometheus不是scale的"
    可以Scale，有许多的方式可以scale和federate Prometheus。详细参见[Scaling and Federating Prometheus](https://www.robustperception.io/scaling-and-federating-prometheus/)

!!! Question "Promtheus是用什么写的"
    大部分是Golang，还有Java, Python, and Ruby.

!!! Question "Prometheus功能，存储格式和API的稳定性如何"
    当然稳定

!!! Question "为什么使用了pull模式而不是push模式"

    HTTP的Pulling方式有许多优点： 

    - 进行更改时，可以在笔记本电脑上运行监视。
    - 您可以更轻松地判断target是否down。
    - 您可以手动跳到target并使用Web浏览器检查其运行状况。
    - 总体而言，pulling比pushing略好，但在考虑使用监控系统时，不应将其视为主要重点。

    对于必须推送的情况，使用Pushgateway。

!!! Question "能否将日志传入到Prometheus"
    不可以，Prometheus是一个收集和处理指标的系统，而不是事件记录系统。

!!! Question "Prometheus使用什么License"
    Apache 2.0 license.

!!! Question "能否重新加载Prometheus配置"
    将SIGHUP信号发送到Prometheus进程或使用HTTP POST请求发送到/-/reload端点将重新加载并应用配置文件。 各种组件尝试优雅处理失败的更改。

!!! Question "能否发送告警"
    可以，Alertmanager就是干这事儿，当前支持的告警系统。

    - Email
    - Generic Webhooks
    - OpsGenie
    - PagerDuty
    - Pushover
    - Slack
    - VictorOps
    - WeChat

!!! Question "能否创建Dashboard"
    可以， Grafana或者使用自带的console templates

!!! Question "能否更改时区吗？ 为什么所有内容都采用UTC？"
    避免时区混乱。

## Instrumentation[^1]

!!! Question "哪些语言有Instrumentation库"
    详细参加[client libraries](https://prometheus.io/docs/instrumenting/clientlibs/)

!!! Question "是否可以监控机器"
    可以，Node Exporter在Linux和其他Unix系统上公开了一组广泛的计算机级别指标，例如CPU使用率，内存，磁盘使用率，文件系统完整性和网络带宽。

!!! Question "是否可以监控网络设备"
    可以，SNMP Exporter允许监视支持SNMP的设备。

!!! Question "是否可以监控批处理任务"
    使用Pushgateway。 另请参阅监视批处理作业的[best practices](https://prometheus.io/docs/practices/instrumentation/#batch-jobs)。

!!! Question "Prometheus可以直接使用哪些应用程序进行监视"
    详细列表参见[the list of exporters and integrations](https://prometheus.io/docs/instrumenting/exporters/)

!!! Question "可以通过JMX监视JVM应用程序吗"
    对于无法直接使用Java客户端进行检测的应用程序，可以单独使用JMX Exporter或将其用作Java Agent(代理)。

!!! Question "Instrumentation的性能影响是什么"
    client libraries和语言之间的性能可能会有所不同。 对于Java，基准测试表明，根据争用，使用Java客户端增加计数器/表将花费12-17ns。 除了对延迟最关键的代码之外，所有其他代码都可以忽略不计。

## Troubleshooting

!!! Question "Prometheus 1.x服务器需要很长时间才能启动，并且会向日志中发送有关崩溃恢复的大量信息的垃圾邮件。"
    您正在遭受不干净的关机。 SIGTERM之后，Prometheus必须彻底关闭，对于频繁使用的服务器可能要花一些时间。 如果服务器崩溃或被严重杀死（例如，在等待Prometheus关闭时，内核杀死了OOM或您的运行级别系统不耐烦），则必须执行崩溃恢复，在正常情况下，恢复应该少于一分钟，但是可以 在某些情况下要花很长时间。 有关详细信息，请参见[crash recovery](https://prometheus.io/docs/prometheus/1.8/storage/#crash-recovery)。

!!! Question "Prometheus 1.x服务器OOM(out of memory)"
    有关可用内存量，请参阅有关配置Prometheus的[memory usage](https://prometheus.io/docs/prometheus/1.8/storage/#memory-usage)部分

!!! Question "Prometheus 1.x服务器报告处于“紧急模式(rushed mode)”或“存储需要节流(storage needs throttling)”。"
    存储设备高负载。 阅读有关[configuring the local storage](https://prometheus.io/docs/prometheus/1.8/storage/)获取更好的性能。

## Implementation

!!! Question "为什么所有样本值都是64位浮点数？我想要整数。"
    我们将自己限制在64位浮点数上，以简化设计。 IEEE 754双精度二进制浮点格式支持最大精度为253的整数精度。仅当需要大于253但小于263的整数精度时，支持本机64位整数（仅）才有帮助。原则上，支持不同的样本值类型 （包括某种大整数，甚至支持64位以上）都可以实现，但目前并不是优先考虑的事情。 计数器即使每秒增加一百万次，也只会在超过285年后才会出现精度问题。

!!! Question "为什么Prometheus服务器组件不支持TLS或身份验证？ 我可以添加那些吗？"

    尽管TLS和身份验证是经常需要的功能，但我们故意没有在Prometheus的任何服务器端组件中实现它们。两者都有很多不同的选项和参数（仅TLS有10多个选项），我们决定集中精力构建最佳的监视系统，而不是在每个服务器组件中都支持完全通用的TLS和身份验证解决方案。

    如果您需要TLS或身份验证，我们建议在Prometheus前面放置一个反向代理。例如，参见使用Nginx将基本身份验证添加到Prometheus。这仅适用于入站连接

[^1]: Instrumentation中文为插桩，一般指的是获取计算机软件或者硬件状态的数据的技术。常常用于程序监控和追踪