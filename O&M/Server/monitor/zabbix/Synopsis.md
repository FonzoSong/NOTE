# Zabbix介绍

Zabbix是一个企业级的分布式开源监控方案。Zabbix是一款能够监控各种网络参数以及服务器健康性和完整性的软件。Zabbix使用灵活的通知机制，允许用户为几乎任何事件配置基于邮件的告警。这样可以快速反馈服务器的问题。基于已存储的数据，Zabbix提供了出色的报告和数据可视化功能。
Zabbix是一个高度集成的网络监控解决方案，一个简单的安装包中提供多样性的功能。
Zabbix结构
Zabbix由几个主要的软件组件构成，这些组件的功能如下。

1. Server
Zabbix server是监控代理程序报告系统可用性、系统完整性和统计信息的核心组件。Zabbix Server是所有配置信息、统计信息和操作数据的核心存储器。
2. 数据库存储
所有配置信息和Zabbix收集到的数据都被存储在数据库中。
3. Web界面
为了在任何地方和任何平台都能轻松地访问Zabbix，Zabbix提供了基于Web的界面。该界面是Zabbix Server的一部分，通常跟Zabbix Server运行在同一台物理机器上。
如果使用SQLite，Zabbix Web界面必须要跟Zabbix Server运行在同一台物理机器上。
4. Proxy代理服务器
Zabbix Proxy可以替Zabbix Server收集性能和可用性数据。Proxy代理服务器是Zabbix软件可选择部署的一部分。当然，Proxy代理服务器可以帮助单台Zabbix Server分担负载压力。
5. Agent监控代理
Zabbix Agents监控代理部署在监控目标上，能够主动监控本地资源和应用程序，并将收集到的数据报告给Zabbix Server。
Zabbix架构体系

![zabbix_annex](https://raw.githubusercontent.com/FonzoSong/NoteImages/master/img/zabbix_annex.png)
