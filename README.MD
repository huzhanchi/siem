## 整体介绍

* 整体架构

![image](https://github.com/huzhanchi/siem/blob/main/images/siem.png)

* 控制后台

![image](https://github.com/huzhanchi/siem/blob/main/images/backend.png)

* 数据处理原则
无论实时检测还是离线检测都遵循如下的漏斗过滤
![image](https://github.com/huzhanchi/siem/blob/main/images/loudou.png)

* 实时入侵检测

基于flink cep做二次扩展 [实时事件关联引擎](https://github.com/huzhanchi/flink-cep) ， 当前关联处理数据量峰值3w EPS。匹配的结果回传到kafka，供下游告警系统消费，告警系统根据白名单以及出发频率进行抑制。

* 离线入侵检测

主要通过调度任务定时抽取flink sql 前一天离线数据，做特征处理结合异常检测模型去做入侵检测。我们有针对性的对java和php类应用做rce，无文件入侵的检测，主要操作步骤，先抽取主进程下第一代子进程序列做nlp 词向量表征然后经异常检测算法比如孤立森林做检测。

![image](https://github.com/huzhanchi/siem/blob/main/images/abnormal.png)

* 数据富化

静态富化：在终端向机房汇集节点发送后，在汇集节点做些必要的数据补充，比如多线ip；kafka消费时补充再回传，比如业务模块等信息。
动态即时富化：在cep引擎或者离线sql中通过自定义udf临时补充关联数据。

* 安全函数--UDF

将安全能力服务化，通过flink udf 暴露给 实时检测引擎 以及 离线flink sql，从而增强检测和统计分析能力。比如IP检测函数，域名检测函数。这一块的udf模块化可以供实时和离线复用。

* 规则上线

根据过往案例总结出IOC，经过flinksql统计一天或多天统计分析看数据量是否可以运营，以及是否要加入名单过滤。避免盲目上线实时规则给规则引擎造成严重性能压力而达不到运营效果。

比如：在某次红蓝对抗之后，针对性的对 “中国菜刀” 的payload研究，发现其有一定的特征，可以通过flinksql查看全网这个特征是否具备可运维可能。

* 人工统计分析

在利用离线数据做数据探索分析时常用的思考方式：
![image](https://github.com/huzhanchi/siem/blob/main/images/tongjifenxi.png)

做waf sql注入算法模型离线预测的时候，检测到类似sqlmap中常见的body，useragent 是 python-requests/2.24.0，根据flink sql分析得到这个useragent在全网不是很多，捞出来的数据绝大部分ip聚集在两到三个ip上，并且发出的payload 除了sql注入还有 目录扫描，源代码泄漏，java rce扫描等。

* 人工调查分析

主要通过es全文检索，当一次入侵事件发生时，可以根据一些IOC指标查看全网的影响面，以及前后的事件记录。
通过neo4j数据库将网络行为数据记录，比如进程外链，dns记录。这里需要处理一些脏数据特别是网络环境有dns反射的垃圾流量，以及大量cdn离散域名链接排除。

* todo
四层网络流量的数据分析，应用层协议重组分析