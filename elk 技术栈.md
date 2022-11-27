学习 Elastic Stack：

（1）Beats：捕捉数据，发送到 Elasticsearch；

（2）APM：Application performance monitoring，监视应用程序状态；

（3）Elasticsearch：搜索和分析数据；

（4）Kibana：用户管理平台，可视化 Elasticsearch 数据和管理 Elastic Stack；

（5）Logstash：数据收集引擎；

总结：通过 Beats 收集数据，可以直接或者通过 Logstash 间接发送到 Elasticsearch，在 ES 中进一步处理数据，然后在 Kibana 中可视化数据。

参考文档：https://www.elastic.co/guide/en/elastic-stack/8.5/overview.html