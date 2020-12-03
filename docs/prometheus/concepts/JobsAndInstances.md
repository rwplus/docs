# Jobs and Instances

用Prometheus术语来说，您可以抓取scrape的endpoint称为instance，通常对应于单个进程。具有相同目的的instance的集合collation（例如，出于可伸缩性或可靠性而复制的过程）称为作业。

例如，具有四个复制实例的API服务器job：

- job: api-server
    - instance 1: 1.2.3.4:5670
    - instance 2: 1.2.3.4:5671
    - instance 3: 5.6.7.8:5670
    - instance 4: 5.6.7.8:5671

## 自动生成label和时间序列

当Prometheus抓取target时，它会自动在抓取的时间序列上附加一些labels，以识别被抓取的target：

job：target所属的已配置job name。
instance：已抓取的目标URL的`<host>：<port>`部分。

如果这些labels的任何一个已经存在于抓取数据中，则其行为取决于`honor_labels`配置选项。有关更多信息，请参见[scrape configuration documentation](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#scrape_config)。

对于每个instance的scrape，Prometheus按照以下时间序列存储样本：

- `up{job="<job-name>", instance="<instance-id>"}: 1`如果实例运行状况良好（即可达），则为1，如果抓取失败，则为0。
- `scrape_duration_seconds{job="<job-name>", instance="<instance-id>"}: `scrape的持续时间。
- `scrape_samples_post_metric_relabeling{job="<job-name>", instance="<instance-id>"}: ` 应用metric重新标记后剩余的samples数。
- `scrape_samples_scraped{job="<job-name>", instance="<instance-id>"}:` target暴露的样本数。
- `scrape_series_added{job="<job-name>", instance="<instance-id>"}:`此scrape中新序列的大概数量（v2.10的新功能）

运行时间序列对于实例可用性监视很有用。