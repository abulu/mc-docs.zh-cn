---
title: 使用 C# 创建 Azure 数据资源管理器群集和数据库
description: 了解如何使用 C# 创建 Azure 数据资源管理器群集和数据库
author: oflipman
ms.author: v-biyu
ms.reviewer: orspodek
ms.service: data-explorer
ms.topic: conceptual
origin.data: 03/25/2019
ms.date: 07/22/2019
ms.openlocfilehash: 649ad496c66e80ee5a0b17fb5157da0703473ed1
ms.sourcegitcommit: ea5dc30371bc63836b3cfa665cc64206884d2b4b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/10/2019
ms.locfileid: "67717341"
---
# <a name="create-an-azure-data-explorer-cluster-and-database-by-using-c"></a>使用 C# 创建 Azure 数据资源管理器群集和数据库

> [!div class="op_single_selector"]
> * [Portal](create-cluster-database-portal.md)
> * [CLI](create-cluster-database-cli.md)
> * [PowerShell](create-cluster-database-powershell.md)
> * [C#](create-cluster-database-csharp.md)
> * [Python](create-cluster-database-python.md)
>  

Azure 数据资源管理器是一项快速、完全托管的数据分析服务，用于实时分析从应用程序、网站和 IoT 设备等资源流式传输的海量数据。 若要使用 Azure 数据资源管理器，请先创建群集，再在该群集中创建一个或多个数据库。 然后将数据引入（加载）到数据库，以便对其运行查询。 在本文中，将使用 C# 创建群集和数据库。

## <a name="prerequisites"></a>先决条件

* 如果尚未安装 Visual Studio 2019，可以下载并使用**免费的** [Visual Studio 2019 Community Edition](https://www.visualstudio.com/downloads/)。 在安装 Visual Studio 的过程中，请确保启用“Azure 开发”。 

* 如果没有 Azure 订阅，可在开始前创建一个[试用帐户](https://www.azure.cn/pricing/1rmb-trial)。

## <a name="install-c-nuget"></a>安装 C# nuget

1. 安装 [Azure 数据资源管理器 (Kusto) nuget 包](https://www.nuget.org/packages/Microsoft.Azure.Management.Kusto/)。

1. 安装 [Microsoft.IdentityModel.Clients.ActiveDirectory nuget 包](https://www.nuget.org/packages/Microsoft.IdentityModel.Clients.ActiveDirectory/)以进行身份验证。

## <a name="create-the-azure-data-explorer-cluster"></a>创建 Azure 数据资源管理器群集

1. 使用以下代码创建群集：

    ```C#
    string resourceGroupName = "testrg";    
    string clusterName = "mykustocluster";
    string location = "Central US";
    AzureSku sku = new AzureSku("D13_v2", 5);
    Cluster cluster = new Cluster(location, sku);
    
    ar authenticationContext = new AuthenticationContext("https://login.chinacloudapi.cn/{tenantName}");
    var credential = new ClientCredential(clientId: "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx", clientSecret: "xxxxxxxxxxxxxx");
    var result = authenticationContext.AcquireTokenAsync(resource: "https://management.core.chinacloudapi.cn", clientCredential: credential).Result;
    
    var credentials = new TokenCredentials(result.AccessToken, result.AccessTokenType);
     
    KustoManagementClient KustoManagementClient = new KustoManagementClient(credentials)
    {
        SubscriptionId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx"
    };

    KustoManagementClient.Clusters.CreateOrUpdate(resourceGroupName, clusterName, cluster);
    ```

   |**设置** | **建议的值** | **字段说明**|
   |---|---|---|
   | clusterName | *mykustocluster* | 所需的群集名称。|
   | sku | *D13_v2* | 将用于群集的 SKU。 |
   | resourceGroupName | *testrg* | 将在其中创建群集的资源组名称。 |

    可以使用其他可选参数，例如群集的容量。

1. 设置[凭据](https://docs.azure.cn/zh-cn/dotnet/dotnet-sdk-azure-authenticate?view=azure-dotnet)

1. 运行以下命令，检查群集是否已成功创建：

    ```C#
    KustoManagementClient.Clusters.Get(resourceGroupName, clusterName);
    ```

如果结果包含带 `Succeeded` 值的 `ProvisioningState`，则表示已成功创建群集。

## <a name="create-the-database-in-the-azure-data-explorer-cluster"></a>在 Azure 数据资源管理器群集中创建数据库

1. 使用以下代码创建数据库：

    ```c#
    TimeSpan hotCachePeriod = new TimeSpan(3650, 0, 0, 0);
    TimeSpan softDeletePeriod = new TimeSpan(3650, 0, 0, 0);
    string databaseName = "mykustodatabase";
    Database database = new Database(location: location, softDeletePeriod: softDeletePeriod, hotCachePeriod: hotCachePeriod);
    
    KustoManagementClient.Databases.CreateOrUpdate(resourceGroupName, clusterName, databaseName, database);
    ```

   |**设置** | **建议的值** | **字段说明**|
   |---|---|---|
   | clusterName | *mykustocluster* | 将在其中创建数据库的群集的名称。|
   | databaseName | *mykustodatabase* | 数据库名称。|
   | resourceGroupName | *testrg* | 将在其中创建群集的资源组名称。 |
   | softDeletePeriod | *3650:00:00:00* | 供查询使用的数据的保留时间。 |
   | hotCachePeriod | *3650:00:00:00* | 数据将在缓存中保留的时间。 |

2. 若要查看已创建的数据库，请运行以下命令：

    ```c#
    KustoManagementClient.Databases.Get(resourceGroupName, clusterName, databaseName);
    ```

现在，你有了一个群集和一个数据库。

## <a name="clean-up-resources"></a>清理资源

* 如果计划学习我们的其他文章，请保留已创建的资源。
* 若要清理资源，请删除群集。 删除群集时，也会删除其中的所有数据库。 使用以下命令删除群集：

    ```C#
    KustoManagementClient.Clusters.Delete(resourceGroupName, clusterName);
    ```

## <a name="next-steps"></a>后续步骤

* [使用 Azure 数据资源管理器 .NET Standard SDK（预览版）引入数据](net-standard-ingest-data.md)
