---
title: Apache Hadoop 组件和版本 - Azure HDInsight
description: 了解 HDInsight 中的 Apache Hadoop 组件和版本，以及 Hortonworks 数据平台的此云分发版中可用的服务级别。
keywords: hadoop 版本,hadoop 生态系统组件,hadoop 组件,如何检查 hadoop 版本
services: hdinsight
ms.reviewer: jasonh
manager: asadk
author: kkampf
ms.service: hdinsight
ms.custom: hdinsightactive,hdiseo17may2017
ms.devlang: na
ms.topic: article
origin.date: 06/07/2019
ms.date: 07/22/2019
ms.author: v-yiso
ms.openlocfilehash: 48cb9b4d911f0d693e2bdc1317de2a4d126043d5
ms.sourcegitcommit: f4351979a313ac7b5700deab684d1153ae51d725
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/12/2019
ms.locfileid: "67845238"
---
# <a name="what-are-the-apache-hadoop-components-and-versions-available-with-hdinsight"></a>HDInsight 提供了哪些 Apache Hadoop 组件和版本？

了解 Microsoft Azure HDInsight 中的 [Apache Hadoop](https://hadoop.apache.org/) 生态系统组件和版本，以及企业安全性套餐。 另外，还将了解如何检查 HDInsight 中的 Hadoop 组件版本。 

## <a name="apache-hadoop-components-available-with-different-hdinsight-versions"></a>随不同 HDInsight 版本提供的 Apache Hadoop 组件
Azure HDInsight 支持多个可随时部署的 Hadoop 群集版本。 每个版本选项创建 HDP 分发版的一个特定版本和该分发版内包含的一组组件。 从 2017 年 4 月 4 日起，Azure HDInsight 使用的默认群集版本是 3.6 并基于 HDP 2.6。

下表中列出了与 HDInsight 群集版本关联的组件版本： 

> [!NOTE]
> HDInsight 服务的默认版本可能会更改，不会另行通知。 如果依赖某个版本，建议在使用 .NET SDK 以及 Azure PowerShell 和 Azure 经典 CLI 创建群集时指定 HDInsight 版本。

| 组件 | HDInsight 4.0 | HDInsight 3.6（默认） | HDInsight 3.5 | HDInsight 3.4 | HDInsight 3.3 | HDInsight 3.2 |
|---------------------------|---------------|-----------------------------|---------------|---------------|---------------|----------------------|
| Hortonworks 数据平台 | 3.0 | 2.6 | 2.5 | 2.4 | 2.3 | 2.2 |
| Apache Hadoop 和 YARN | 3.1.1 | 2.7.3 | 2.7.3 | 2.7.1 | 2.7.1 | 2.6.0 |
| Apache Tez | 0.9.1 | 0.7.0 | 0.7.0 | 0.7.0 | 0.7.0 | 0.5.2 |
| Apache Pig | 0.16.0 | 0.16.0 | 0.16.0 | 0.15.0 | 0.15.0 | 0.14.0 |
| Apache Hive 和 HCatalog | - | 1.2.1 | 1.2.1 | 1.2.1 | 1.2.1 | 0.14.0 |
| Apache Hive | 3.1.0 | 2.1.0 | - | - | - | - |
| Apache Tez Hive2 | - | 0.8.4 | - | - | - | - |
| Apache Ranger | 1.1.0 | 0.7.0 | 0.6.0 | - | - | - |
| Apache HBase | 2.0.1 | 1.1.2 | 1.1.2 | 1.1.2 | 1.1.1 | 0.98.4 |
| Apache Sqoop | 1.4.7 | 1.4.6 | 1.4.6 | 1.4.6 | 1.4.6 | 1.4.5 |
| Apache Oozie | 4.3.1 | 4.2.0 | 4.2.0 | 4.2.0 | 4.2.0 | 4.1.0 |
| Apache Zookeeper | 3.4.6 | 3.4.6 | 3.4.6 | 3.4.6 | 3.4.6 | 3.4.6 |
| Apache Storm | - | 1.1.0 | 1.0.1 | 0.10.0 | 0.10.0 | 0.9.3 |
| Apache Mahout | - | 0.9.0+ | 0.9.0+ | 0.9.0+ | 0.9.0+ | 0.9.0 |
| Apache Phoenix | 5 | 4.7.0 | 4.7.0 | 4.4.0 | 4.4.0 | 4.2.0 |
| Apache Spark | 2.3.1、2.4 | 2.3.0、2.2.0、2.1.0 | 1.6.2, 2.0 | 1.6.0 | 1.5.2 | 1.3.1（仅限 Windows） |
| Apache Livy | 0.5 | 0.4、0.4、0.3 | 0.3 | 0.3 | 0.2 | - |
| Apache Kafka | 1.1.1、2.1 | 1.1、1.0 *（请参阅下面的“注意”） | 0.10.0 | 0.9.0 | - | - |
| Apache Ambari | 2.7.0 | 2.6.0 | 2.4.0 | 2.2.1 | 2.1.0 | - |
| Apache Zeppelin | 0.8.0 | 0.7.0 | - | - | - | - |
| Mono | 4.2.1 | 4.2.1 | 4.2.1 | 3.2.8 | - | - |

> [!NOTE]
> 出于系统性能考量，对 Kafka 版本 0.10 的支持已在 2019 年 3 月过期。

## <a name="check-for-current-hadoop-component-version-information"></a>检查当前的 Hadoop 组件版本信息

与 HDInsight 群集版本关联的组件版本可能会随 HDInsight 的更新而更改。 若要检查 Hadoop 组件并验证正在为群集使用哪些版本，请使用 Ambari REST API。 **GetComponentInformation** 命令检索有关服务组件的信息。 有关详细信息，请参阅 [Apache Ambari 文档][ambari-docs]。

### <a name="release-notes"></a>发行说明

请参阅 [HDInsight 发行说明](hdinsight-release-notes.md)，了解 HDInsight 最新版本的更多发行说明。

## <a name="supported-hdinsight-versions"></a>支持的 HDInsight 版本
下表列出了 HDInsight 版本。 其中列出了与每个 HDInsight 版本对应的 HDP 版本以及产品发行日期。 还提供了支持到期日期和停用日期（如果已知这些日期）。

### <a name="available-versions"></a>可用版本

下表列出了 Azure 门户中可用的 HDInsight 版本以及其他部署方法（比如 PowerShell 和 .NET SDK）。

| HDInsight 版本 | HDP 版本 | VM OS | 发行日期 | 支持到期日期 | 停用日期 | 高可用性 |  在 Azure 门户上的可用性 | 
| --- | --- | --- | --- | --- | --- | --- | --- |
| HDInsight 4.0 |HDP 3.0 |Ubuntu 16.0.4 LTS |2018 年 9 月 24 日 | | |是 |是 |
| HDInsight 3.6 |HDP 2.6 |Ubuntu 16.0.4 LTS |2017 年 4 月 4 日 | 2020 年 6 月 30 日 |2020 年 12 月 31 日 |是 |是 |


> [!NOTE]  
> 在对某个版本的支持到期后，不能通过 Microsoft Azure 门户获得该版本。 但是，可继续使用 Windows PowerShell [New-AzHDInsightCluster](https://docs.microsoft.com/powershell/module/az.hdinsight/new-azhdinsightcluster) 命令中的 `Version` 参数和 .NET SDK 获取群集版本，直到版本停用的那天为止。
>

### <a name="retired-versions"></a>已停用的版本

下表列出了 Azure 门户中**不**可用的 HDInsight 版本。

| HDInsight 版本 | HDP 版本 | VM OS | 发行日期 | 支持到期日期 | 停用日期 | 高可用性 |  在 Azure 门户上的可用性 | 
| --- | --- | --- | --- | --- | --- | --- | --- |
| HDInsight 3.5 <br> （非 Spark） |HDP 2.5 |Ubuntu 16.0.4 LTS |2016 年 9 月 30 日 |2017 年 9 月 5 日 |2018 年 6 月 28 日 |是 |否 |
| HDInsight 3.4 |HDP 2.4 |Ubuntu 14.0.4 LTS |2016 年 3 月 29 日 |2016 年 12 月 29 日 |2018 年 1 月 9 日 |是 |否 |
| HDInsight 3.3 |HDP 2.3 |Windows Server 2012 R2 |2015 年 12 月 2 日 |2016 年 6 月 27 日 |2018 年 7 月 31 日 |是 |否 |
| HDInsight 3.3 |HDP 2.3 |Ubuntu 14.0.4 LTS |2015 年 12 月 2 日 |2016 年 6 月 27 日 |2017 年 7 月 31 日 |是 |否 |
| HDInsight 3.2 |HDP 2.2 |Ubuntu 12.04 LTS 或 Windows Server 2012 R2 |2015 年 2 月 18 日 |2016 年 3 月 1 日 |2017 年 4 月 1 日 |是 |否 |
| HDInsight 3.1 |HDP 2.1 |Windows Server 2012 R2 |2014 年 6 月 24 日 |2015 年 5 月 18 日 |2016 年 6 月 30 日 |是 |否 |
| HDInsight 3.0 |HDP 2.0 |Windows Server 2012 R2 |2014 年 2 月 11 日 |2014 年 9 月 17 日 |2015 年 6 月 30 日 |是 |否 |
| HDInsight 2.1 |HDP 1.3 |Windows Server 2012 R2 |2013 年 10 月 28 日 |2014 年 5 月 12 日 |2015 年 5 月 31 日 |是 |否 |
| HDInsight 1.6 |HDP 1.1 | |2013 年 10 月 28 日 |2014 年 4 月 26 日 |2015 年 5 月 31 日 |否 |否 |

> [!NOTE]
> 对于 HDInsight 2.1 和更高版本，默认情况下会集部署具有两个头节点的高度可用群集。 它们不适用于 HDInsight 1.6 版本的群集。

### <a name="supported-cluster-types"></a>支持的群集类型

目前只有以下群集类型支持 Enterprise Security 包：

- Hadoop（仅限 HDInsight 3.6）
- Spark
- 交互式查询

### <a name="support-for-azure-data-lake-storage"></a>支持 Azure Data Lake Storage

Enterprise Security 包支持使用 Azure Data Lake Storage 作为主存储和附加存储。

### <a name="pricing-and-service-level-agreement"></a>定价和服务级别协议

有关 Enterprise Security 包的定价和 SLA 的信息，请参阅 [HDInsight 定价](https://azure.microsoft.com/pricing/details/hdinsight/)。


## <a name="service-level-agreement-for-hdinsight-cluster-versions"></a>HDInsight 群集版本的服务级别协议

服务级别协议 (SLA) 是按“支持窗口”  定义的。 支持窗口是 HDInsight 群集版本受 Microsoft 客户服务和支持部门支持的时间段。 如果版本的“支持到期日期”已过，则 HDInsight 处于支持窗口外。  指定的 HDInsight 版本 X 的支持到期日期（在提供更新的 X+1 版本后）为按以下公式计算所得时间的较晚者：  

* 公式 1：发布 HDInsight 群集版本 X 的日期加 180 天。
* 公式 2：在 Azure 门户中提供 HDInsight 群集版本 X+1 的日期加 90 天。

_停用日期_是指在此后不能在 HDInsight 上创建此群集版本的日期。 从 2017 年 7 月 31 日开始，无法在停用日期后调整 HDInsight 群集的大小。 

> [!NOTE]
> HDInsight Windows 群集（包括版本 2.1、3.0、3.1, 3.2 和 3.3）在 Azure 来宾 OS 系列版本 4 上运行，该版本使用 64 位版本的 Windows Server 2012 R2。 Azure 来宾 OS 系列版本 4 支持 .NET Framework 版本 4.0、4.5、4.5.1 和 4.5.2。
>
>

## <a name="hortonworks-release-notes-associated-with-hdinsight-versions"></a>与 HDInsight 版本相关的 Hortonworks 发行说明

本部分提供了与 HDInsight 一起使用的 Hortonworks 数据平台分发版和 Apache 组件的发行说明的链接。
* HDInsight 群集版本 4.0 使用基于 [Hortonworks 数据平台 3.0](https://docs.hortonworks.com/HDPDocuments/HDP3/HDP-3.0.0/release-notes/content/relnotes.html) 的 Hadoop 分发版
* HDInsight 群集版本 3.6 使用基于 [Hortonworks 数据平台 2.6](https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.6.0/bk_release-notes/content/ch_relnotes.html) 的 Hadoop 分发版。
* HDInsight 群集版本 3.5 使用基于 [Hortonworks 数据平台 2.5](https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.5.0/bk_release-notes/content/ch_relnotes_v250.html) 的 Hadoop 分发版。 HDInsight 群集版本 3.5 是在 Azure 门户中创建的“默认”  Hadoop 群集。
* HDInsight 群集版本 3.4 使用基于 [Hortonworks 数据平台 2.4](https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.4.0/bk_HDP_RelNotes/content/ch_relnotes_v240.html)的 Hadoop 分发版。
* HDInsight 群集版本 3.3 使用基于 [Hortonworks 数据平台 2.3](https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.3.0/bk_HDP_RelNotes/content/ch_relnotes_v230.html)的 Hadoop 分发版。

  * Apache 网站上提供了 [Apache Storm 发行说明](https://storm.apache.org/2015/11/05/storm0100-released.html)。
  * Apache 网站上提供了 [Apache Hive 发行说明](https://issues.apache.org/jira/secure/ReleaseNote.jspa?version=12332384&styleName=Text&projectId=12310843)。
* HDInsight 群集版本 3.2 使用基于 [Hortonworks 数据平台 2.2][hdp-2-2] 的 Hadoop 分发版。  

  * 特定 Apache 组件的发行说明提供如下：[Hive 0.14](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12310843&version=12326450)、[Pig 0.14](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12310730&version=12326954)、[HBase 0.98.4](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12310753&version=12326810)、[Phoenix 4.2.0](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12315120&version=12327581)、[M/R 2.6](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12310941&version=12327180)、[HDFS 2.6](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12310942&version=12327181)、[YARN 2.6](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12313722&version=12327197)、[Common](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12310240&version=12327179)、[Tez 0.5.2](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12314426&version=12328742)、[Ambari 2.0](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12312020&version=12327486)、[Storm 0.9.3](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12314820&version=12327112) 和 [Oozie 4.1.0](https://issues.apache.org/jira/secure/ReleaseNote.jspa?version=12324960&projectId=12311620)。
* HDInsight 群集版本 3.1 使用基于 [Hortonworks 数据平台 2.1.7][hdp-2-1-7]. HDInsight 3.1 clusters created before November, 7, 2014, are based on [Hortonworks Data Platform 2.1.1][hdp-2-1-1] 的 Hadoop 分发版。
* HDInsight 群集版本 3.0 使用基于 [Hortonworks 数据平台 2.0][hdp-2-0-8] 的 Hadoop 分发版。
* HDInsight 群集版本 2.1 使用基于 [Hortonworks 数据平台 1.3][hdp-1-3-0] 的 Hadoop 分发版。
* HDInsight 群集版本 1.6 使用基于 [Hortonworks 数据平台 1.1][hdp-1-1-0] 的 Hadoop 分发版。






## <a name="default-node-configuration-and-virtual-machine-sizes-for-clusters"></a>群集的默认节点配置和虚拟机大小
下表列出了 HDInsight 群集的默认虚拟机 (VM) 大小。  此图表是了解在创建 PowerShell 或 Azure CLI 脚本以部署 HDInsight 群集时要使用的 VM 大小所必需的。

> [!IMPORTANT]
> 如果需要在群集中使用 32 个以上的辅助角色节点，则必须选择至少具有 8 个核心和 14 GB RAM 的头节点大小。
> 
>

|群集类型|Hadoop|HBase|交互式查询|Storm|Spark|ML Server|Kafka|
|---|---|---|---|---|---|---|---|
|头：默认 VM 大小|D12 v2|D12 v2|D13 v2|A3|D12 v2|D12 v2|D3v2|
|头：建议的 VM 大小|D3 v2|D3 v2|D13|A4 v2|D12 v2|D12 v2|A2M v2|
||D4 v2|D4 v2|D14|A8 v2|D13 v2|D13 v2|D3 v2|
||D12 v2|D12 v2|E16 v3|A2m v2|D14 v2|D14 v2|D4 v2|
||E4 v3|E4 v3|E32 v3|E4 v3|E4 v3|E4 v3|D12 v2|
|辅助角色：默认 VM 大小|D4 v2|D4 v2|D14 v2|D3 v2|D13 v2|D4 v2|4 D12v2，每个中转站 2 个 S30 磁盘|
|辅助角色：建议的 VM 大小|D3 v2|D3 v2|D13|D3 v2|D4 v2|D4 v2|D13 v2|
||D4 v2|D4 v2|D14|D4 v2|D12 v2|D12 v2|DS12 v2|
||D12 v2|D12 v2|E16 v3|D12 v2|D13 v2|D13 v2|DS13 v2|
||E4 v3|E4 v3|E20 v3|E4 v3|D14 v2|D14 v2|E4 v3|
||||E32 v3||E16 v3|E16 v3|ES4 v3|
||||E64 v3||E20 v3|E20 v3|E8 v3|
||||||E32 v3|E32 v3|ES8 v3|
||||||E64 v3|E64 v3||
|Zookeeper：默认 VM 大小||A4 v2|A4 v2|A4 v2||A2 v2|D3v2|
|Zookeeper：建议的 VM 大小||A4 v2||A2 v2|||A2M v2|
|||A8 v2||A4 v2|||D3 v2|
|||A2m v2||A8 v2|||E8 v3|
|边缘：默认 VM 大小||||||D4 v2||
|边缘：建议的 VM 大小||||||D4 v2||
|||||||D12 v2||
|||||||D13 v2||
|||||||D14 v2||
|||||||E16 v3||
|||||||E20 v3||
|||||||E32 v3||
|||||||E64 v3||
> [!NOTE]
> - 头称为 Storm 群集类型的 *Nimbus* 。
> - 对于 Storm 群集类型，辅助角色称为“主管”。 
> - 对于 HBase 群集类型，辅助角色称为“区域”。 

## <a name="next-steps"></a>后续步骤
- [为 HDInsight 上的 Apache Hadoop、Spark 和其他组件设置群集](hdinsight-hadoop-provision-linux-clusters.md)
- [使用 Windows 电脑在 HDInsight 上的 Apache Hadoop 中工作](hdinsight-hadoop-windows-tools.md)

[Supported HDInsight versions]:(#supported-hdinsight-versions)

[image-hdi-versioning-versionscreen]: ./media/hdinsight-component-versioning/hdi-versioning-version-screen.png

[wa-forums]: https://www.azure.cn/support/forums/

[connect-excel-with-hive-ODBC]: hdinsight-connect-excel-hive-ODBC-driver.md

[hdp-2-2]: https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.2.9/bk_HDP_RelNotes/content/ch_relnotes_v229.html

[hdp-2-1-7]: https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.1.7-Win/bk_releasenotes_HDP-Win/content/ch_relnotes-HDP-2.1.7.html

[hdp-2-1-1]: https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.1.1/bk_releasenotes_hdp_2.1/content/ch_relnotes-hdp-2.1.1.html

[hdp-2-0-8]: https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.0.8.0/bk_releasenotes_hdp_2.0/content/ch_relnotes-hdp2.0.8.0.html

[hdp-1-3-0]: https://docs.hortonworks.com/HDPDocuments/HDP1/HDP-1.3.0/bk_releasenotes_hdp_1.x/content/ch_relnotes-hdp1.3.0_1.html

[hdp-1-1-0]: https://docs.hortonworks.com/HDPDocuments/HDP1/HDP-1.3.0/bk_releasenotes_hdp_1.x/content/ch_relnotes-hdp1.1.1.16_1.html

[ambari-docs]: https://github.com/apache/ambari/blob/trunk/ambari-server/docs/api/v1/index.md

[zookeeper]: https://zookeeper.apache.org/
