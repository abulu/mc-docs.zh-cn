---
title: 使用 .NET SDK 管理 HDInsight 中的 Apache Hadoop 群集
description: 了解如何使用 HDInsight .NET SDK 针对 HDInsight 中的 Apache Hadoop 群集执行管理任务。
services: hdinsight
editor: cgronlun
manager: jhubbard
tags: azure-portal
author: mumian
documentationcenter: ''
ms.assetid: fd134765-c2a0-488a-bca6-184d814d78e9
ms.service: hdinsight
ms.custom: hdinsightactive
ms.devlang: na
ms.topic: conceptual
origin.date: 05/14/2018
ms.date: 07/22/2019
ms.author: v-yiso
ms.openlocfilehash: eae7f9987085f128e5a3be59c62ba702643f14a2
ms.sourcegitcommit: f4351979a313ac7b5700deab684d1153ae51d725
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/12/2019
ms.locfileid: "67845168"
---
# <a name="manage-apache-hadoop-clusters-in-hdinsight-by-using-net-sdk"></a>使用 .NET SDK 管理 HDInsight 中的 Apache Hadoop 群集
[!INCLUDE [selector](../../includes/hdinsight-portal-management-selector.md)]

了解如何使用 [HDInsight.NET SDK](https://docs.azure.cn/dotnet/api/overview/hdinsight) 管理 HDInsight 群集。

**先决条件**

在开始阅读本文前，必须具有：

* **一个 Azure 订阅**。 请参阅[获取 Azure 试用版](https://www.azure.cn/pricing/1rmb-trial/)。

## <a name="connect-to-azure-hdinsight"></a>连接到 Azure HDInsight

需要以下 NuGet 包：

```powershell
Install-Package Microsoft.Rest.ClientRuntime.Azure.Authentication -Pre
Install-Package Microsoft.Azure.Management.ResourceManager -Pre
Install-Package Microsoft.Azure.Management.HDInsight
```

以下代码示例演示如何先连接到 Azure，并管理 Azure 订阅下面的 HDInsight 群集。

```csharp
using System;
using Microsoft.Azure;
using Microsoft.Azure.Management.HDInsight;
using Microsoft.Azure.Management.HDInsight.Models;
using Microsoft.Azure.Management.ResourceManager;
using Microsoft.IdentityModel.Clients.ActiveDirectory;
using Microsoft.Rest;
using Microsoft.Rest.Azure.Authentication;

    namespace HDInsightManagement
    {
        class Program
        {
            private static HDInsightManagementClient _hdiManagementClient;
            // Replace with your AAD tenant ID if necessary
            private const string TenantId = UserTokenProvider.CommonTenantId; 
            private const string SubscriptionId = "<Your Azure Subscription ID>";
            // This is the GUID for the PowerShell client. Used for interactive logins in this example.
            private const string ClientId = "1950a258-227b-4e31-a9cf-717495945fc2";
            private static Uri BaseUri = new Uri("https://management.chinacloudapi.cn/");

            static void Main(string[] args)
            {
                // Authenticate and get a token
                var authToken = Authenticate(TenantId, ClientId, SubscriptionId);
                // Flag subscription for HDInsight, if it isn't already.
                EnableHDInsight(authToken);
                // Get an HDInsight management client
                _hdiManagementClient = new HDInsightManagementClient(authToken, BaseUri);

                // insert code here

                System.Console.WriteLine("Press ENTER to continue");
                System.Console.ReadLine();
            }

            /// <summary>
            /// Authenticate to an Azure subscription and retrieve an authentication token
            /// </summary>
            static TokenCloudCredentials Authenticate(string TenantId, string ClientId, string SubscriptionId)
            {
                var authContext = new AuthenticationContext("https://login.chinacloudapi.cn/" + TenantId);
                var tokenAuthResult = authContext.AcquireToken("https://management.core.chinacloudapi.cn/", 
                    ClientId, 
                    new Uri("urn:ietf:wg:oauth:2.0:oob"), 
                    PromptBehavior.Always, 
                    UserIdentifier.AnyUser);
                return new TokenCloudCredentials(SubscriptionId, tokenAuthResult.AccessToken);
            }
            /// <summary>
            /// Marks your subscription as one that can use HDInsight, if it has not already been marked as such.
            /// </summary>
            /// <remarks>This is essentially a one-time action; if you have already done something with HDInsight
            /// on your subscription, then this isn't needed at all and will do nothing.</remarks>
            /// <param name="authToken">An authentication token for your Azure subscription</param>
            static void EnableHDInsight(TokenCloudCredentials authToken)
            {
                // Create a client for the Resource manager and set the subscription ID
                var resourceManagementClient = new ResourceManagementClient(BaseUri, new TokenCredentials(authToken.Token));
                resourceManagementClient.SubscriptionId = SubscriptionId;
                // Register the HDInsight provider
                var rpResult = resourceManagementClient.Providers.Register("Microsoft.HDInsight");
            }
        }
    }

You shall see a prompt when you run this program.  If you don't want to see the prompt, see [Create non-interactive authentication .NET HDInsight applications](hdinsight-create-non-interactive-authentication-dotnet-applications.md).

## Create clusters
See [Create Linux-based clusters in HDInsight using the .NET SDK](hdinsight-hadoop-create-linux-clusters-dotnet-sdk.md)

<a name="list-clusters"></a>
## List clusters
The following code snippet lists clusters and some properties:

```csharp
var results = _hdiManagementClient.Clusters.List();
foreach (var name in results.Clusters) {
    Console.WriteLine("Cluster Name: " + name.Name);
    Console.WriteLine("\t Cluster type: " + name.Properties.ClusterDefinition.ClusterType);
    Console.WriteLine("\t Cluster location: " + name.Location);
    Console.WriteLine("\t Cluster version: " + name.Properties.ClusterVersion);
}
```

## <a name="delete-clusters"></a>删除群集
使用以下代码段以同步或异步方式删除群集： 

```csharp
_hdiManagementClient.Clusters.Delete("<Resource Group Name>", "<Cluster Name>");
_hdiManagementClient.Clusters.DeleteAsync("<Resource Group Name>", "<Cluster Name>");
```

## <a name="scale-clusters"></a>缩放群集
使用群集缩放功能，可更改 Azure HDInsight 中运行的群集使用的辅助节点数，而无需重新创建群集。

> [!NOTE]
> 只支持使用 HDInsight 3.1.3 或更高版本的群集。 如果不确定群集的版本，可以查看“属性”页面。  请参阅[列出和显示群集](hdinsight-administer-use-portal-linux.md#list-and-show-clusters)。
> 
> 

更改 HDInsight 支持的每种类型的群集所用数据节点数的影响：

* Apache Hadoop
  
    可顺利增加正在运行的 Hadoop 群集中的辅助节点数，而不会影响任何挂起或运行中的作业。 也可在操作进行中提交新作业。 系统会正常处理失败的缩放操作，让群集始终保持正常运行状态。
  
    减少数据节点数目以缩减 Hadoop 群集时，系统会重新启动群集中的某些服务。 这会导致所有正在运行和挂起的作业在缩放操作完成时失败。 但是，可在操作完成后重新提交这些作业。
* Apache HBase
  
    可在 HBase 群集运行时顺利添加或删除节点。 完成缩放操作后的几分钟内，区域服务器自动平衡。 但也可手动平衡区域服务器，方法是登录到群集的头节点，并在命令提示符窗口中运行以下命令：
  
    ```bash
    >pushd %HBASE_HOME%\bin
    >hbase shell
    >balancer
    ```
* Apache Storm

    可在 Storm 群集运行时顺利添加或删除数据节点。 但是，缩放操作成功完成后，需要重新平衡拓扑。

    可以使用两种方法来完成重新平衡操作：

  * Storm Web UI
  * 命令行界面 (CLI) 工具
    
    有关详细信息，请参阅 [Apache Storm 文档](https://storm.apache.org/documentation/Understanding-the-parallelism-of-a-Storm-topology.html)。
    
    HDInsight 群集上提供了 Storm Web UI：
    
    ![HDInsight Storm 缩放重新平衡](./media/hdinsight-administer-use-powershell/hdinsight-portal-scale-cluster-storm-rebalance.png)
    
    以下是有关如何使用 CLI 命令重新平衡 Storm 拓扑的示例：
    
    ```cli
    ## Reconfigure the topology "mytopology" to use 5 worker processes,
    ## the spout "blue-spout" to use 3 executors, and
    ## the bolt "yellow-bolt" to use 10 executors
    $ storm rebalance mytopology -n 5 -e blue-spout=3 -e yellow-bolt=10
    ```

以下代码片段显示如何以同步或异步方式调整群集的大小：

```csharp
_hdiManagementClient.Clusters.Resize("<Resource Group Name>", "<Cluster Name>", <New Size>);   
_hdiManagementClient.Clusters.ResizeAsync("<Resource Group Name>", "<Cluster Name>", <New Size>);   
```

## <a name="grantrevoke-access"></a>授予/撤消访问权限
HDInsight 群集提供以下 HTTP Web 服务（所有这些服务都有 REST 样式的终结点）：

* ODBC
* JDBC
* Apache Ambari
* Apache Oozie
* Apache Templeton

默认情况下，这些服务会获得访问授权。 可以撤消/授予访问权限。 若要撤消：

```csharp
var httpParams = new HttpSettingsParameters
{
    HttpUserEnabled = false,
    HttpUsername = "admin",
    HttpPassword = "*******",
};
_hdiManagementClient.Clusters.ConfigureHttpSettings("<Resource Group Name>, <Cluster Name>, httpParams);
```

若要授予：

```csharp
var httpParams = new HttpSettingsParameters
{
    HttpUserEnabled = enable,
    HttpUsername = "admin",
    HttpPassword = "*******",
};
_hdiManagementClient.Clusters.ConfigureHttpSettings("<Resource Group Name>, <Cluster Name>, httpParams);
```

> [!NOTE]
> 授予/撤消访问权限时，会重设群集用户的用户名和密码。
> 
> 

也可以通过门户完成此操作。 请参阅[使用 Azure 门户管理 HDInsight 中的 Apache Hadoop 群集](hdinsight-administer-use-portal-linux.md)。

## <a name="update-http-user-credentials"></a>更新 HTTP 用户凭据
此过程与授予/撤销 HTTP 访问权限相同。  如果已授予群集 HTTP 访问权限，必须先撤销该权限。  然后再使用新的 HTTP 用户凭据授予访问权限。

## <a name="find-the-default-storage-account"></a>查找默认存储帐户
以下代码片段演示如何获取群集的默认存储帐户名称和默认存储帐户密钥。

```csharp
var results = _hdiManagementClient.Clusters.GetClusterConfigurations(<Resource Group Name>, <Cluster Name>, "core-site");
foreach (var key in results.Configuration.Keys)
{
    Console.WriteLine(String.Format("{0} => {1}", key, results.Configuration[key]));
}
```

## <a name="submit-jobs"></a>提交作业
**提交 MapReduce 作业**

请参阅[在 HDInsight 中运行 MapReduce 示例](hadoop/apache-hadoop-run-samples-linux.md)。

**提交 Apache Hive 作业** 

请参阅[使用 .NET SDK 运行 Apache Hive 查询](hadoop/apache-hadoop-use-hive-dotnet-sdk.md)。

**提交 Apache Sqoop 作业**

请参阅[将 Apache Sqoop 与 HDInsight 配合使用](hadoop/apache-hadoop-use-sqoop-dotnet-sdk.md)。

**提交 Apache Oozie 作业**

请参阅[在 HDInsight 中将 Apache Oozie 与 Hadoop 配合使用以定义和运行工作流](hdinsight-use-oozie-linux-mac.md)。

## <a name="upload-data-to-azure-blob-storage"></a>将数据上传到 Azure Blob 存储
请参阅[将数据上传到 HDInsight][hdinsight-upload-data]。

## <a name="see-also"></a>另请参阅
* [HDInsight .NET SDK 参考文档](https://docs.azure.cn/dotnet/api/overview/hdinsight)
* [使用 Azure 门户管理 HDInsight 中的 Apache Hadoop 群集](hdinsight-administer-use-portal-linux.md)
* [使用命令行接口管理 HDInsight][hdinsight-admin-cli]
* [创建 HDInsight 群集][hdinsight-provision]
* [将数据上传到 HDInsight][hdinsight-upload-data]
* [Azure HDInsight 入门][hdinsight-get-started]

[azure-purchase-options]: https://www.azure.cn/pricing/overview/
[azure-member-offers]: https://www.azure.cn/pricing/member-offers/
[azure-trial]: https://www.azure.cn/pricing/1rmb-trial/

[hdinsight-get-started]:hadoop/apache-hadoop-linux-tutorial-get-started.md
[hdinsight-provision]: hdinsight-hadoop-provision-linux-clusters.md
[hdinsight-provision-custom-options]: hdinsight-hadoop-provision-linux-clusters.md#configuration
[hdinsight-submit-jobs]:hadoop/submit-apache-hadoop-jobs-programmatically.md

[hdinsight-admin-cli]: hdinsight-administer-use-command-line.md
[hdinsight-admin-portal]: hdinsight-administer-use-portal-linux.md
[hdinsight-storage]: hdinsight-hadoop-use-blob-storage.md
[hdinsight-use-hive]:hadoop/hdinsight-use-hive.md
[hdinsight-use-mapreduce]:hadoop/hdinsight-use-mapreduce.md
[hdinsight-upload-data]: hdinsight-upload-data.md
[hdinsight-flight]: hdinsight-analyze-flight-delay-data.md
