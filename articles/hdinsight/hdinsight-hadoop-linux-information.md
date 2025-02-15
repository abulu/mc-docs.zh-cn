---
title: 有关在基于 Linux 的 HDInsight 上使用 Hadoop 的提示 - Azure | Azure
description: 获取有关在 Azure 云中运行的你所熟悉的 Linux 环境中使用基于 Linux 的 HDInsight (Hadoop) 群集的实施提示。
services: hdinsight
ms.service: hdinsight
author: jasonwhowell
ms.custom: hdinsightactive
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: big-data
origin.date: 03/20/2019
ms.date: 07/22/2019
ms.author: v-yiso
ms.openlocfilehash: 1504efeffe7b6187cfd09f7df9b36ce523d36235
ms.sourcegitcommit: f4351979a313ac7b5700deab684d1153ae51d725
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/12/2019
ms.locfileid: "67845439"
---
# <a name="information-about-using-hdinsight-on-linux"></a>有关在 Linux 上使用 HDInsight 的信息

Azure HDInsight 群集提供了基于熟悉的 Linux 环境并在 Azure 云中运行的 Apache Hadoop。 在大多数情况下，它的工作方式应该与其他任何 Hadoop-on-Linux 安装完全相同。 本文档指出了你应该注意的具体差异。

## <a name="prerequisites"></a>先决条件

本文档中的许多步骤使用以下实用程序，这些程序可能需要在系统上安装。

* [cURL](https://curl.haxx.se/) - 用于与基于 Web 的服务进行通信。
* 命令行 JSON 处理程序 **jq**。  请参阅 [https://stedolan.github.io/jq/](https://stedolan.github.io/jq/)。
* [Azure CLI](/cli/install-azure-cli) - 用于远程管理 Azure 服务。
* **SSH 客户端**。 有关详细信息，请参阅[使用 SSH 连接到 HDInsight (Apache Hadoop)](hdinsight-hadoop-linux-use-ssh-unix.md)。


## <a name="domain-names"></a>域名

从 Internet 连接到群集时要使用的完全限定域名 (FQDN) 是 `CLUSTERNAME.azurehdinsight.cn` 或 `CLUSTERNAME-ssh.azurehdinsight.cn`（仅适用于 SSH）。

就内部来说，群集中的每个节点都有一个在群集配置期间分配的名称。 若要查找群集名称，请参阅 Ambari Web UI 上的 **主机** 页。 还可以使用以下方法从 Ambari REST API 返回主机列表：

    curl -u admin -G "https://CLUSTERNAME.azurehdinsight.cn/api/v1/clusters/CLUSTERNAME/hosts" | jq '.items[].Hosts.host_name'

将 `CLUSTERNAME` 替换为群集的名称。 出现提示时，请输入管理员帐户的密码。 此命令返回包含群集中主机列表的 JSON 文档。 [jq](https://stedolan.github.io/jq/) 用于为每个主机提取 `host_name` 元素值。

若需查找特定服务的节点的名称，可查询 Ambari 以获取该组件。 例如，若需查找 HDFS 名称节点的主机，请使用以下命令：

    curl -u admin -G "https://CLUSTERNAME.azurehdinsight.cn/api/v1/clusters/CLUSTERNAME/services/HDFS/components/NAMENODE" | jq '.host_components[].HostRoles.host_name'

此命令会返回一个描述该服务的 JSON 文档，然后 [jq](https://stedolan.github.io/jq/) 就会只拉取主机的 `host_name` 值。

## <a name="remote-access-to-services"></a>对服务的远程访问

* **Ambari (web)**  - https://CLUSTERNAME.azurehdinsight.cn

    使用群集管理员用户和密码进行身份验证，并登录到 Ambari。

    身份验证是纯文本身份验证 - 始终使用 HTTPS 来帮助确保连接是安全的。

    > [!IMPORTANT]
    > 某些 Web UI 可使用内部域名通过 Ambari 访问节点。 内部域名不可通过 Internet 公开访问。 在尝试通过 Internet 访问某些功能时，可能会收到“找不到服务器”错误。
    >
    > 要使用 Ambari web UI 的全部功能，请使用 SSH 隧道通过代理将 Web 流量传送到群集头节点。 请参阅[使用 SSH 隧道访问 Apache Ambari Web UI、ResourceManager、JobHistory、NameNode、Oozie 和其他 Web UI](hdinsight-linux-ambari-ssh-tunnel.md)

* **Ambari (REST)**  - https://CLUSTERNAME.azurehdinsight.cn/ambari

    > [!NOTE]
    > 通过使用群集管理员用户和密码进行身份验证。
    >
    > 身份验证是纯文本身份验证 - 始终使用 HTTPS 来帮助确保连接是安全的。

* **WebHCat (Templeton)**  - https://CLUSTERNAME.azurehdinsight.cn/templeton

    > [!NOTE]
    > 通过使用群集管理员用户和密码进行身份验证。
    >
    > 身份验证是纯文本身份验证 - 始终使用 HTTPS 来帮助确保连接是安全的。

* **SSH** - CLUSTERNAME-ssh.azurehdinsight.cn，使用端口 22 或 23。 端口 22 用于连接主要头节点，而端口 23 用于连接辅助头节点。 有关头节点的详细信息，请参阅 [HDInsight 中的 Apache Hadoop 群集的可用性和可靠性](hdinsight-high-availability-linux.md)。

    > [!NOTE]
    > 只能通过 SSH 从客户端计算机访问群集头节点。 在连接后，可以通过使用 SSH 从头节点访问从节点。

有关详细信息，请参阅 [HDInsight 上的 Apache Hadoop 服务使用的端口](hdinsight-hadoop-port-settings-for-services.md)文档。

## <a name="file-locations"></a>文件位置

Hadoop 相关文件可在群集节点上的 `/usr/hdp`中找到。 此目录包含以下子目录：

* **2.6.5.3006-29**：目录名称是 HDInsight 使用的 Hortonworks 数据平台的版本。 群集上的数字可能与这里列出的有所不同。
* **current**：此目录包含 **2.6.5.3006-29** 目录下的子目录的链接。 由于该目录存在，因此无需记住版本号。

可以在 Hadoop 分布式文件系统上的 `/example` 和 `/HdiSamples` 处找到示例数据和 JAR 文件。

## <a name="hdfs-azure-storage-and-data-lake-storage"></a>HDFS、Azure 存储和 Data Lake Storage

在大部分 Hadoop 发行版中，数据都存储在 HDFS 中，HDFS 由群集中计算机上的本地存储提供支持。 对基于云的解决方案使用本地存储可能费用高昂，因为计算资源以小时或分钟为单位来计费。

使用 HDInsight 时，数据文件使用 Azure Blob 存储以可缩放和复原的方式存储在云中。 这些服务提供以下优势：

* 成本低廉的长期存储。
* 可从外部服务访问，例如网站、文件上传/下载实用程序、各种语言 SDK 和 Web 浏览器。
* 大型文件容量和大型可缩放存储。

有关详细信息，请参阅[了解 Blob](https://docs.microsoft.com/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs)。

使用 Azure 存储时，不需要从 HDInsight 执行任何特殊操作即可访问数据。 例如，以下命令列出 `/example/data` 文件夹中的文件：

    hdfs dfs -ls /example/data

在 HDInsight 中，从计算资源中分离数据存储资源。 因此，你可以根据需要创建 HDInsight 群集来执行计算，然后在工作完成后删除该群集，同时，在云存储中安全地将数据文件持久保存所需的任意时长。


### <a name="URI-and-scheme"></a>URI 和方案

在访问文件时，一些命令可能需要用户将方案指定为 URI 的一部分。 例如，Storm-HDFS 组件就需要指定方案。 使用非默认存储（作为“附加”存储添加到群集的存储）时，必须始终将方案作为 URI 的一部分来使用。

使用 __Azure 存储__时，可以使用以下 URI 方案之一：

* `wasb:///`：使用未加密的通信访问默认存储。

* `wasbs:///`：使用加密的通信访问默认存储。  仅 HDInsight 3.6 及以上版本支持 wasbs 方案。

* `wasb://<container-name>@<account-name>.blob.core.chinacloudapi.cn/`：与非默认存储帐户通信时使用。 例如，具有其他存储帐户或访问可公开访问的存储帐户中存储的数据时。

使用 __Azure Data Lake Storage Gen2__ 时，可以使用以下 URI 方案之一：

* `abfs:///`：使用未加密的通信访问默认存储。

* `abfss:///`：使用加密的通信访问默认存储。  仅 HDInsight 3.6 及以上版本支持 abfss 方案。

* `abfs://<container-name>@<account-name>.dfs.core.chinacloudapi.cn/`：与非默认存储帐户通信时使用。 例如，具有其他存储帐户或访问可公开访问的存储帐户中存储的数据时。
> [!IMPORTANT]  
> 使用 Data Lake Storage 作为 HDInsight 的默认存储时，必须在存储中指定一个用作 HDInsight 存储根目录的路径。 默认路径为 `/clusters/<cluster-name>/`。
>

### <a name="what-storage-is-the-cluster-using"></a>群集使用的是哪种存储

可以使用 Ambari 来检索群集的默认存储配置。 可以使用以下命令通过 curl 检索 HDFS 配置信息，并使用 [jq](https://stedolan.github.io/jq/)对其进行筛选：

```bash
curl -u admin -G "https://CLUSTERNAME.azurehdinsight.cn/api/v1/clusters/CLUSTERNAME/configurations/service_config_versions?service_name=HDFS&service_config_version=1" | jq '.items[].configurations[].properties["fs.defaultFS"] | select(. != null)'```
```

> [!NOTE]
> 此命令会返回应用到服务器的第一个配置 (`service_config_version=1`)，其中包含此信息。 可能需要列出所有配置版本，才能找到最新版本。

此命令返回类似于以下 URI 的值：

* `wasb://<container-name>@<account-name>.blob.core.chinacloudapi.cn` 。

    帐户名是 Azure 存储帐户的名称。 容器名称是作为群集存储的根的 blob 容器。

也可以在 Azure 门户中使用以下步骤查找存储信息：

1. 在 [Azure 门户](https://portal.azure.cn/)中，选择 HDInsight 群集。

2. 在“属性”  部分中，选择“存储帐户”  。 会显示群集的存储信息。

### <a name="how-do-i-access-files-from-outside-hdinsight"></a>如何从 HDInsight 外部访问文件

从 HDInsight 群集外部访问数据的方法有多种。 以下是一些可用于处理数据的实用工具和 SDK 的链接：

如果使用的是 __Azure 存储__，请参阅以下链接了解可用于访问数据的方式：

* [Azure CLI](/cli/install-az-cli2)：适用于 Azure 的命令行接口命令。 在安装后，使用 `az storage` 命令获取有关使用存储的帮助，或者使用 `az storage blob` 获取特定于 Blob 的命令。
* [blobxfer.py](https://github.com/Azure/blobxfer)：用于处理 Azure 存储中的 blob 的 python 脚本。
* 多种 SDK：

    * [Java](https://github.com/Azure/azure-sdk-for-java)
    * [Node.js](https://github.com/Azure/azure-sdk-for-node)
    * [PHP](https://github.com/Azure/azure-sdk-for-php)
    * [Python](https://github.com/Azure/azure-sdk-for-python)
    * [Ruby](https://github.com/Azure/azure-sdk-for-ruby)
    * [.NET](https://github.com/Azure/azure-sdk-for-net)
    * [存储 REST API](https://msdn.microsoft.com/library/azure/dd135733.aspx)

## <a name="scaling"></a>缩放你的群集

使用群集缩放功能可动态更改群集使用的数据节点数。 可以在其他作业或进程正在群集上运行时执行缩放操作。  另请参阅[缩放 HDInsight 群集](./hdinsight-scaling-best-practices.md)

不同的群集类型会受缩放操作影响，如下所示：

* **Hadoop**：减少群集中的节点数时，群集中的某些服务将重新启动。 缩放操作会导致正在运行或挂起的作业在缩放操作完成时失败。 可以在操作完成后重新提交这些作业。
* **HBase**：在完成缩放操作后的几分钟内，区域服务器会自动进行平衡。 若要手动平衡区域服务器，请使用以下步骤：

    1. 使用 SSH 连接到 HDInsight 群集。 有关详细信息，请参阅 [将 SSH 与 HDInsight 配合使用](hdinsight-hadoop-linux-use-ssh-unix.md)。

    2. 使用以下命令来启动 HBase shell：

            hbase shell

    3. 加载 HBase shell 后，使用以下方法来手动平衡区域服务器：

            balancer

* **Storm**：你应在执行缩放操作后重新平衡任何正在运行的 Storm 拓扑。 重新平衡允许拓扑根据群集中的新节点数重新调整并行度设置。 若要重新平衡正在运行的拓扑，请使用下列选项之一：

    * **SSH**：连接到服务器并使用以下命令来重新平衡拓扑：

            storm rebalance TOPOLOGYNAME

        还可以指定参数来替代拓扑原来提供的并行度提示。 例如，`storm rebalance mytopology -n 5 -e blue-spout=3 -e yellow-bolt=10` 会将拓扑重新配置为 5 个辅助角色进程，蓝色的 BlueSpout 组件有 3 个 executor，黄色 YellowBolt 组件有 10 个 executor。

    * **Storm UI**：使用以下步骤来重新平衡使用 Storm UI 的拓扑。

        1. 在 Web 浏览器中打开 **https://CLUSTERNAME.azurehdinsight.cn/stormui** ，其中 CLUSTERNAME 是 Storm 群集的名称。 如果出现提示，请输入在创建 HDInsight 群集时指定的群集管理员用户名和密码。
        2. 选择要重新平衡的拓扑，并选择“重新平衡”  按钮。 输入执行重新平衡操作前的延迟。

* **Kafka**：执行缩放操作后，应重新均衡分区副本。 有关详细信息，请参阅[通过 Apache Kafka on HDInsight 实现数据的高可用性](./kafka/apache-kafka-high-availability.md)文档。

有关缩放 HDInsight 群集的特定信息，请参阅：

* [使用 Azure 门户管理 HDInsight 中的 Apache Hadoop 群集](hdinsight-administer-use-portal-linux.md#scale-clusters)
* [使用 Azure CLI 管理 HDInsight 中的 Apache Hadoop 群集](hdinsight-administer-use-command-line.md#scale-clusters)

## <a name="how-do-i-install-hue-or-other-hadoop-component"></a>如何安装 Hue（或其他 Hadoop 组件）？

HDInsight 是一个托管服务。 如果 Azure 检测到群集存在问题，则可能会删除故障节点，再创建一个节点来代替。 如果在群集节点上手动安装组件，则发生此操作时，这些组件不会保留。 应该改用 [HDInsight 脚本操作](hdinsight-hadoop-customize-cluster-linux.md)。 脚本操作可用于进行以下更改：

* 安装并配置服务或网站。
* 安装和配置需要在群集的多个节点上进行配置更改的组件。

脚本操作是 Bash 脚本。 该脚本在群集创建期间运行，用于安装并配置其他组件。 提供了用于安装以下组件的示例脚本：

* [Apache Giraph](hdinsight-hadoop-giraph-install-linux.md)

有关开发自己的脚本操作的信息，请参阅[使用 HDInsight 进行脚本操作开发](hdinsight-hadoop-script-actions-linux.md)。

### <a name="jar-files"></a>Jar 文件

某些 Hadoop 技术以自包含 jar 文件形式提供，这些文件包含某些函数，这些函数用作 MapReduce 作业的一部分，或来自 Pig 或 Hive 内部。 它们通常不需要进行任何设置，并可以在创建后上传到群集和直接使用。 如需确保组件在群集重置映像后仍存在，可将 jar 文件存储在群集的默认存储（WASB 或 ADL）中。

例如，如果要使用 [Apache DataFu](https://datafu.incubator.apache.org/) 的最新版本，可以下载包含项目的 jar，并将其上传到 HDInsight 群集。 然后按照 DataFu 文档的说明通过 Pig 或 Hive 使用它。

> [!IMPORTANT]
> 某些属于独立 jar 文件的组件通过 HDInsight 提供，但不在路径中。 若要查找特定组件，可使用以下命令在群集上搜索：
>
> ```find / -name *componentname*.jar 2>/dev/null```
>
> 此命令会返回任何匹配的 jar 文件的路径。

要使用不同版本的组件，请上传所需版本，并在作业中使用它。

> [!IMPORTANT]
> 完全支持通过 HDInsight 群集提供的组件，Azure 支持部门帮助找出并解决与这些组件相关的问题。
>
> 自定义组件可获得合理范围的支持，有助于进一步解决问题。 这可能会促进解决问题，或要求使用可用的开源技术渠道，在渠道中可找到该技术的深厚的专业知识。 有许多可以使用的社区站点，例如：[面向 HDInsight 的 MSDN 论坛](https://social.msdn.microsoft.com/Forums/azure/en-US/home?forum=hdinsight)、[Azure CSDN](http://azure.csdn.net)。 此外，Apache 项目在 [https://apache.org](https://apache.org) 上提供了项目站点，例如：[Hadoop](https://hadoop.apache.org/)、[Spark](https://spark.apache.org/)。

## <a name="next-steps"></a>后续步骤

* [从基于 Windows 的 HDInsight 迁移到基于 Linux 的 HDInsight](hdinsight-migrate-from-windows-to-linux.md)
* [使用 Apache Ambari REST API 管理 HDInsight 群集](./hdinsight-hadoop-manage-ambari-rest-api.md)
* [将 Apache Hive 和 HDInsight 配合使用](hadoop/hdinsight-use-hive.md)
* [将 Apache Pig 和 HDInsight 配合使用](hadoop/hdinsight-use-pig.md)
* [将 MapReduce 作业与 HDInsight 配合使用](hadoop/hdinsight-use-mapreduce.md)
