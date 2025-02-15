---
title: 使用 Storm 处理来自事件中心的事件 - Azure HDInsight | Azure
description: 了解如何使用在 Visual Studio 中通过用于 Visual Studio 的 HDInsight 工具创建的 C# Storm 拓扑处理来自 Azure 事件中心的数据。
services: hdinsight,notification hubs
documentationcenter: ''
author: Blackmist
manager: jhubbard
editor: cgronlun
ms.assetid: 67f9d08c-eea0-401b-952b-db765655dad0
ms.service: hdinsight
ms.custom: ''
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: big-data
origin.date: 11/27/2017
ms.date: 07/22/2019
ms.author: v-yiso
ms.openlocfilehash: 58ef2a132997cb71d4934f5624b969987968eb85
ms.sourcegitcommit: f4351979a313ac7b5700deab684d1153ae51d725
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/12/2019
ms.locfileid: "67845386"
---
# <a name="process-events-from-azure-event-hubs-with-apache-storm-on-hdinsight-c"></a>使用 Apache Storm on HDInsight 从 Azure 事件中心处理事件 (C#)

了解如何从 [Apache Storm](http://storm.apache.org/) on HDInsight 使用 Azure 事件中心。 本文档使用 C# Storm 拓扑对事件中心读取和写入数据

> [!NOTE]
> 如需此项目的 Java 版，请参阅[使用 Apache Storm on HDInsight 从 Azure 事件中心处理事件 (Java)](https://azure.microsoft.com/resources/samples/hdinsight-java-storm-eventhub/)。

## <a name="scpnet"></a>SCP.NET

本文档中的步骤使用 SCP.NET，后者是一个 NuGet 包，方便用户创建适用于 Storm on HDInsight 的 C# 拓扑和组件。

> [!IMPORTANT]
> 虽然本文档中的步骤依赖于带 Visual Studio 的 Windows 开发环境，但是也可将编译的项目提交到使用 Linux 的 Storm on HDInsight 群集。 仅在 2016 年 10 月 28 日以后创建的基于 Linux 的群集支持 SCP.NET 拓扑。

HDInsight 3.4 及更高版本使用 Mono 运行 C# 拓扑。 本文档中使用的示例适用于 HDInsight 3.6。 如果你计划为 HDInsight 创建自己的 .NET 解决方案，请查看 [Mono 兼容性](http://www.mono-project.com/docs/about-mono/compatibility/)文档了解可能的不兼容性。

### <a name="cluster-versioning"></a>群集版本控制

项目所使用的 Microsoft.SCP.Net.SDK NuGet 包必须与安装在 HDInsight 上的 Storm 的主要版本匹配。 HDInsight 版本 3.5 和 3.6 使用 Storm 1.x，因此必须对这些群集使用 SCP.NET 版本 1.0.x.x。



> [!IMPORTANT]
> 本文档中的示例需要 HDInsight 3.5 或 3.6 群集。
>
> Linux 是 HDInsight 3.4 或更高版本上使用的唯一操作系统。 

C# 拓扑还必须针对 .NET 4.5 运行。

## <a name="how-to-work-with-event-hubs"></a>如何使用事件中心

Microsoft 提供一组可用来与 Storm 拓扑中的事件中心通信的 Java 组件。 可以在 [https://github.com/hdinsight/mvn-repo/raw/master/org/apache/storm/storm-eventhubs/1.1.0.1/storm-eventhubs-1.1.0.1.jar](https://github.com/hdinsight/mvn-repo/raw/master/org/apache/storm/storm-eventhubs/1.1.0.1/storm-eventhubs-1.1.0.1.jar) 上找到包含这些组件的 HDInsight 3.6 兼容版本的 Java 存档 (JAR) 文件。

> [!IMPORTANT]
> 虽然组件是以 Java 编写的，但可通过 C# 拓扑轻松使用它们。

本示例使用以下组件：

* __EventHubSpout__：从事件中心读取数据。
* __EventHubBolt__：将数据写入事件中心。
* __EventHubSpoutConfig__：用于配置 EventHubSpout。
* __EventHubBoltConfig__：用于配置 EventHubBolt。

### <a name="example-spout-usage"></a>Spout 用法示例

SCP.NET 提供了用于将 EventHubSpout 添加到拓扑的方法。 与使用泛型方法添加 Java 组件相比，这些方法可以更轻松地添加 Spout。 以下示例演示了如何使用 SCP.NET 所提供的 __SetEventHubSpout__ 和 **EventHubSpoutConfig** 方法创建 Spout：

```csharp
topologyBuilder.SetEventHubSpout(
    "EventHubSpout",
    new EventHubSpoutConfig(
        ConfigurationManager.AppSettings["EventHubSharedAccessKeyName"],
        ConfigurationManager.AppSettings["EventHubSharedAccessKey"],
        ConfigurationManager.AppSettings["EventHubNamespace"],
        ConfigurationManager.AppSettings["EventHubEntityPath"],
        eventHubPartitions),
    eventHubPartitions);
```

上面的示例创建了名为 __EventHubSpout__ 的全新 Spout 组件，并将其配置为与事件中心通信。 组件的并行度提示设置为事件中心的分区数。 此设置允许 Storm 为每个分区创建一个组件实例。

### <a name="example-bolt-usage"></a>Bolt 用法示例

使用 **JavaComponmentConstructor** 方法创建 Bolt 的实例。 以下示例演示如何创建和配置 **EventHubBolt** 的新实例：

```csharp
// Java construcvtor for the Event Hub Bolt
JavaComponentConstructor constructor = JavaComponentConstructor.CreateFromClojureExpr(
    String.Format(@"(org.apache.storm.eventhubs.bolt.EventHubBolt. (org.apache.storm.eventhubs.bolt.EventHubBoltConfig. " +
        @"""{0}"" ""{1}"" ""{2}"" ""{3}"" ""{4}"" {5}))",
        ConfigurationManager.AppSettings["EventHubPolicyName"],
        ConfigurationManager.AppSettings["EventHubPolicyKey"],
        ConfigurationManager.AppSettings["EventHubNamespace"],
        "servicebus.chinacloudapi.cn",
        ConfigurationManager.AppSettings["EventHubName"],
        "true"));

// Set the bolt to subscribe to data from the spout
topologyBuilder.SetJavaBolt(
    "eventhubbolt",
    constructor,
    partitionCount)
        .shuffleGrouping("Spout");
```

> [!NOTE]
> 本示例使用以字符串形式传递的 Clojure 表达式，而不是像 Spout 示例那样使用 **JavaComponentConstructor** 创建 **EventHubBoltConfig**。 上述任一方法均有效。 使用最适合方法。

## <a name="download-the-completed-project"></a>下载已完成的项目

可以从 [GitHub](https://github.com/Azure-Samples/hdinsight-dotnet-java-storm-eventhub) 下载本文中创建的项目的完整版本。 但是，仍然需要按照本文中的步骤提供配置设置。

### <a name="prerequisites"></a>先决条件

* HDInsight 上的 Apache Storm 群集。 请参阅[使用 Azure 门户创建 Apache Hadoop 群集](../hdinsight-hadoop-create-linux-clusters-portal.md)，并选择 **Storm** 作为**群集类型**。

    > [!WARNING]
    > 本文档中使用的示例需要 Storm on HDInsight 3.5 或 3.6 版。 由于重大类名更改，此示例不适用于旧版 HDInsight。 如需此示例的适用于旧式群集的版本，请参阅 [GitHub](https://github.com/Azure-Samples/hdinsight-dotnet-java-storm-eventhub/releases)。

* 一个 [Azure 事件中心](../../event-hubs/event-hubs-create.md)。

* [Azure .NET SDK](/downloads/)。

* [用于 Visual Studio 的 HDInsight 工具](../hadoop/apache-hadoop-visual-studio-tools-get-started.md)。

* Java JDK 1.8 或更高版本，适用于开发环境。 [Oracle](https://aka.ms/azure-jdks) 提供了 JDK 下载。

  * **JAVA_HOME** 环境变量必须指向包含 Java 的目录。
  * 路径中必须包含 **%JAVA_HOME%/bin** 目录。

## <a name="download-the-event-hubs-components"></a>下载事件中心组件

从 [https://github.com/hdinsight/mvn-repo/raw/master/org/apache/storm/storm-eventhubs/1.1.0.1/storm-eventhubs-1.1.0.1.jar](https://github.com/hdinsight/mvn-repo/raw/master/org/apache/storm/storm-eventhubs/1.1.0.1/storm-eventhubs-1.1.0.1.jar) 下载事件中心 spout 和 bolt 组件。

创建一个名为 `eventhubspout` 的目录，并将文件保存到该目录中。

## <a name="configure-event-hubs"></a>配置事件中心

事件中心是此示例的数据源。 使用[事件中心入门](../../event-hubs/event-hubs-create.md)的“创建事件中心”部分中的信息。

1. 创建事件中心后，在 Azure 门户中查看“事件中心”设置，选择“共享访问策略”   。 选择“+ 添加”添加以下策略： 

   | Name | 权限 |
   | --- | --- |
   | writer |发送 |
   | reader |侦听 |

    ![共享访问策略窗口的屏幕截图](./media/apache-storm-develop-csharp-event-hub-topology/sas.png)

2. 选择“读取者”  和“写入者”  策略。 复制并保存两个策略的主密钥值，因为稍后将使用这些值。

## <a name="configure-the-eventhubwriter"></a>配置 EventHubWriter

1. 如果尚未安装最新版本的用于 Visual Studio 的 HDInsight 工具，请参阅[开始使用用于 Visual Studio 的 HDInsight 工具](../hadoop/apache-hadoop-visual-studio-tools-get-started.md)。

2. 从 [eventhub-storm-hybrid](https://github.com/Azure-Samples/hdinsight-dotnet-java-storm-eventhub)下载解决方案。

3. 在 **EventHubWriter** 项目中，打开 **App.config** 文件。 使用此前配置的事件中心提供的信息，填充以下项的值：

   | 键 | Value |
   | --- | --- |
   | EventHubPolicyName |写入者（如果对具有“发送”  权限的策略使用不同名称，则改用它。） |
   | EventHubPolicyKey |写入者策略的键。 |
   | EventHubNamespace |包含事件中心的命名空间。 |
   | EventHubName |事件中心名称。 |
   | EventHubPartitionCount |事件中心的分区数。 |

4. 保存并关闭 **App.config** 文件。

## <a name="configure-the-eventhubreader"></a>配置 EventHubReader

1. 打开 **EventHubReader** 项目。

2. 打开 **EventHubReader** 的 **App.config** 文件。 使用前面在事件中心配置的信息填写以下键的值：

   | 键 | Value |
   | --- | --- |
   | EventHubPolicyName |读取者（如果对具有“侦听”  权限的策略使用不同名称，则改用它。） |
   | EventHubPolicyKey |读取者策略的键。 |
   | EventHubNamespace |包含事件中心的命名空间。 |
   | EventHubName |事件中心名称。 |
   | EventHubPartitionCount |事件中心的分区数。 |

3. 保存并关闭 **App.config** 文件。

## <a name="deploy-the-topologies"></a>部署拓扑

1. 在“解决方案资源管理器”  中，右键单击 **EventHubReader** 项目，然后选择“提交到 Storm on HDInsight”  。

    ![解决方案资源管理器的屏幕截图，其中突出显示了“提交到 Storm on HDInsight”](./media/apache-storm-develop-csharp-event-hub-topology/submittostorm.png)

2. 在“提交拓扑”  对话框中，选择“Storm 群集”  。 展开“其他配置”  ，选择“Java 文件路径”  ，选择“...”  ，然后选择前面下载的 JAR 文件所在的目录。 最后，单击“提交”  。

    ![“提交拓扑”对话框的屏幕截图](./media/apache-storm-develop-csharp-event-hub-topology/submit.png)

3. 提交拓扑之后，将会出现“Storm 拓扑查看器”  。 **EventHubReader** 拓扑。

    ![“Storm 拓扑查看器”的屏幕截图](./media/apache-storm-develop-csharp-event-hub-topology/topologyviewer.png)

4. 在“解决方案资源管理器”  中，右键单击 **EventHubWriter** 项目，然后选择“提交到 Storm on HDInsight”  。

5. 在“提交拓扑”  对话框中，选择“Storm 群集”  。 展开“其他配置”  ，选择“Java 文件路径”  ，选择“...”  ，然后选择前面下载的 JAR 文件所在的目录。 最后，单击“提交”  。

6. 提交拓扑之后，在“Storm 拓扑查看器”  中刷新拓扑列表以验证这两个拓扑是否正在群集上运行。

7. 在“Storm 拓扑查看器”  中，选择 **EventHubReader** 拓扑。

8. 若要打开 Bolt 的组件摘要，请双击图表中的“LogBolt”组件。 

9. 在“执行器”  部分，选择“端口”  列中的链接之一。 这将显示由组件记录的信息。 记录的信息类似于以下文本：

        2017-03-02 14:51:29.255 m.s.p.TaskHost [INFO] Received C# STDOUT: 2017-03-02 14:51:29,255 [1] INFO  EventHubReader_LogBolt [(null)] - Received data: {"deviceValue":1830978598,"deviceId":"8566ccbc-034d-45db-883d-d8a31f34068e"}
        2017-03-02 14:51:29.283 m.s.p.TaskHost [INFO] Received C# STDOUT: 2017-03-02 14:51:29,283 [1] INFO  EventHubReader_LogBolt [(null)] - Received data: {"deviceValue":1756413275,"deviceId":"647a5eff-823d-482f-a8b4-b95b35ae570b"}
        2017-03-02 14:51:29.313 m.s.p.TaskHost [INFO] Received C# STDOUT: 2017-03-02 14:51:29,312 [1] INFO  EventHubReader_LogBolt [(null)] - Received data: {"deviceValue":1108478910,"deviceId":"206a68fa-8264-4d61-9100-bfdb68ee8f0a"}

## <a name="stop-the-topologies"></a>停止拓扑

若要停止拓扑，请在“Storm 拓扑查看器”  中选择每个拓扑，然后单击“终止”  。

![“Storm 拓扑查看器”的屏幕截图，其中突出显示了“终止”按钮](./media/apache-storm-develop-csharp-event-hub-topology/killtopology.png)

## <a name="delete-your-cluster"></a>删除群集

[!INCLUDE [delete-cluster-warning](../../../includes/hdinsight-delete-cluster-warning.md)]

## <a name="next-steps"></a>后续步骤

本文档已介绍如何使用 C# 拓扑中的 Java 事件中心 Spout 和 Bolt 处理 Azure 事件中心内的数据。 若要了解有关创建 C# 拓扑的详细信息，请参阅以下主题：

* [使用 Visual Studio 开发 Apache Storm on HDInsight 的 C# 拓扑](apache-storm-develop-csharp-visual-studio-topology.md)
* [SCP 编程指南](apache-storm-scp-programming-guide.md)
* [HDInsight 上的 Apache Storm 的示例拓扑](apache-storm-example-topology.md)
<!--Update_Description: wording update-->