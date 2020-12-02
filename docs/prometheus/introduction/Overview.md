# :fontawesome-regular-laugh-wink:  overview
---
## 什么是Prometheus
Prometheus是最初在SoundCloud上构建的开源系统监视和警报工具包。Prometheus于2016年加入了Cloud Native Computing Foundation（CNCF）
### 特性 - features

主要特性如下：

- 多维数据模型，
- PromQL（DSL），Prometheus查询语言
- 不依赖分布式存储（单服务器节点是自治的）
- 时间序列收集通过HTTP上的拉模型进行
- 通过服务发现或静态配置发现Target
- 支持多种图表和看板（Grafana）

### 组件 - components

Prometheus整个生态系统由许多组件组成。大部分是可选的：

- Prometheus主服务器，它会抓取（Scrap）并存储（Store）时间序列数据
- client libraries用于检测应用程序代码
- 一个支持短期（short-lived）工作的推送网关（Push Gateway）
- 特定用途的exporters
- 一个Alertmanager去处理告警
- 各种支持工具

### 架构 - architecture
下图[^1]说明了Prometheus的体系结构及其某些生态系统组件：

![prometheus](../../../assets/images/architecture.png)

Prometheus直接或通过中间推送网关从已检测的作业中抓取metrics，以处理短暂的Job。 它在本地存储所有已经被抓取的样本，并对这些数据运行规则，以汇总和记录现有数据中的新时间序列，或生成警报。 Grafana或其他API使用者可以用来可视化收集的数据。

## Prometheus适合场景

Prometheus非常适合记录任何纯数字时间序列。 它既适合以机器为中心的监视，也适合监视高度动态的面向服务的体系结构。 在微服务世界中，其对多维数据收集和查询的支持是一种特别的优势。

Prometheus的设计旨在提高可靠性，使其成为中断期间要使用的系统，以使您能够快速诊断问题。 每个Prometheus服务器都是独立的，而不依赖于网络存储或其他远程服务。 当基础结构的其他部分损坏时，您可以依靠它，并且无需设置广泛的基础结构即可使用它。

## Prometheus不适合场景

如果您需要100％的准确性（例如按请求计费），则Prometheus并不是一个不错的选择，因为所收集的数据可能不会足够详细和完整。 

[^1]: [图片来自Prometheus官方](https://prometheus.io/)