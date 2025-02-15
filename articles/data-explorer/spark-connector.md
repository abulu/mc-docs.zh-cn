---
title: 使用适用于 Apache Spark 的 Azure 数据资源管理器连接器在 Azure 数据资源管理器与 Spark 群集之间移动数据。
description: 本主题介绍如何在 Azure 数据资源管理器与 Apache Spark 群集之间移动数据。
author: orspod
ms.author: v-biyu
ms.reviewer: michazag
ms.service: data-explorer
ms.topic: conceptual
origin.date: 4/29/2019
ms.date: 07/22/2019
ms.openlocfilehash: 142bbfae2fb956a14c5496da078f0881b068c37c
ms.sourcegitcommit: ea5dc30371bc63836b3cfa665cc64206884d2b4b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/10/2019
ms.locfileid: "67717353"
---
# <a name="azure-data-explorer-connector-for-apache-spark-preview"></a>适用于 Apache Spark 的 Azure 数据资源管理器连接器（预览版）

[Apache Spark](https://spark.apache.org/) 是用于大规模数据处理的统一分析引擎。 Azure 数据资源管理器是一个快速、完全托管的数据分析服务，可用于实时分析大量数据。 

适用于 Spark 的 Azure 数据资源管理器连接器实施数据源和数据接收器在 Azure 数据资源管理器与 Spark 群集之间移动数据，以使用两者的功能。 使用 Azure 数据资源管理器和 Apache Spark，可以构建面向数据驱动型方案（如机器学习 (ML)、提取-转换-加载 (ETL) 和 Log Analytics）的可缩放快速应用程序。 可在批处理模式和流模式下将数据写入 Azure 数据资源管理器。
读取 Azure 数据资源管理器的操作支持列删除和谓词下推，这样，就可以通过过滤掉 Azure 数据资源管理器中的数据来减少传输的数据量。

Azure 数据资源管理器 Spark 连接器是可在任何 Spark 群集上运行的[开源项目](https://github.com/Azure/azure-kusto-spark)。

> [!NOTE]
> 尽管下面的某些示例提到了 [Azure Databricks](https://docs.azuredatabricks.net/) Spark 群集，但 Azure 数据资源管理器 Spark 连接器并不直接依赖于 Databricks 或任何其他 Spark 分发版。

## <a name="prerequisites"></a>先决条件

* [创建 Azure 数据资源管理器群集和数据库](/data-explorer/create-cluster-database-portal) 
* 创建 Spark 群集
* 安装 Azure 数据资源管理器连接器库，以及[依赖项](https://github.com/Azure/azure-kusto-spark#dependencies)中列出的库，包括以下 [Kusto Java SDK](https://docs.microsoft.com/zh-cn/azure/kusto/api/java/kusto-java-client-library) 库：
    * [Kusto 数据客户端](https://mvnrepository.com/artifact/com.microsoft.azure.kusto/kusto-data)
    * [Kusto 引入客户端](https://mvnrepository.com/artifact/com.microsoft.azure.kusto/kusto-ingest)
* [Spark 2.4、Scala 2.11](https://github.com/Azure/azure-kusto-spark/releases) 的预生成库

## <a name="how-to-build-the-spark-connector"></a>如何生成 Spark 连接器

可按下面的详述，从[源](https://github.com/Azure/azure-kusto-spark)生成 Spark 连接器。

> [!NOTE]
> 此步骤是可选的。 如果使用的是预生成库，请转到 [Spark 群集设置](#spark-cluster-setup)。

### <a name="build-prerequisites"></a>生成先决条件

* 已安装 Java 1.8 SDK
* 已安装 [Maven 3.x](https://maven.apache.org/download.cgi)
* Apache Spark 2.4.0 或更高版本

> [!TIP]
> 也支持版本 2.3.x，不过，这可能需要在 pom.xml 依赖项中进行某些更改。

对于使用 Maven 项目定义的 Scala/Java 应用程序，请将应用程序链接到以下项目（在最新版本中可能不同）：

```Maven
   <dependency>
     <groupId>com.microsoft.azure</groupId>
     <artifactId>spark-kusto-connector</artifactId>
     <version>1.0.0-Beta-02</version>
   </dependency>
```

### <a name="build-commands"></a>生成命令

生成 jar 并运行所有测试：

```
mvn clean package
```

生成 jar，运行所有测试，并将 jar 安装到本地 Maven 存储库：

```
mvn clean install
```

有关详细信息，请参阅[连接器用法](https://github.com/Azure/azure-kusto-spark#usage)。

## <a name="spark-cluster-setup"></a>Spark 群集设置

> [!NOTE]
> 建议使用最新的 Azure 数据资源管理器 Spark 连接器版本执行以下步骤：

1. 根据具体的 Azure Databricks 群集使用 Spark 2.4 和 Scala 2.11 设置以下 Spark 群集： 

    ![Databricks 群集设置](media/spark-connector/databricks-cluster.png)

1. 导入 Azure 数据资源管理器连接器库：

    ![导入 Azure 数据资源管理器库](media/spark-connector/db-create-library.png)

1. 添加其他依赖项：

    ![添加依赖项](media/spark-connector/db-dependencies.png)

    > [!TIP]
    > 在[此处](https://github.com/Azure/azure-kusto-spark#dependencies)可以找到每个 Spark 版本的正确 Java 发行版。

1. 验证是否已安装所有必需的库：

    ![验证是否已安装库](media/spark-connector/db-libraries-view.png)

## <a name="authentication"></a>身份验证

Azure 数据资源管理器 Spark 连接器允许使用 [Azure AD 应用程序](#azure-ad-application-authentication)、[Azure AD 访问令牌](https://github.com/Azure/azure-kusto-spark/blob/dev/docs/Authentication.md#direct-authentication-with-access-token)、[设备身份验证](https://github.com/Azure/azure-kusto-spark/blob/dev/docs/Authentication.md#device-authentication)（适用于非生产方案）或 [Azure Key Vault](https://github.com/Azure/azure-kusto-spark/blob/dev/docs/Authentication.md#key-vault) 在 Azure Active Directory (Azure AD) 中进行身份验证。 用户必须安装 azure-keyvault 包并提供应用程序凭据，才能访问 Key Vault 资源。

### <a name="azure-ad-application-authentication"></a>Azure AD 应用程序身份验证

最简单且最常用的身份验证方法。 使用 Azure 数据资源管理器 Spark 连接器时建议使用此方法。

|属性  |说明  |
|---------|---------|
|**KUSTO_AAD_CLIENT_ID**     |   Azure AD 应用程序（客户端）标识符。      |
|**KUSTO_AAD_AUTHORITY_ID**     |  Azure AD 身份验证颁发机构。 Azure AD 目录（租户）ID。        |
|**KUSTO_AAD_CLIENT_PASSWORD**    |    客户端的 Azure AD 应用程序密钥。     |

### <a name="azure-data-explorer-privileges"></a>Azure 数据资源管理器特权

必须授予对 Azure 数据资源管理器群集的以下特权：

* 对于读取（数据源）操作，Azure AD 应用程序必须对目标数据库拥有“查看者”特权，或者对目标表拥有“管理员”特权。  
* 对于写入（数据接收器）操作，Azure AD 应用程序必须对目标数据库拥有“引入者”特权。  此外，它必须对目标数据库拥有“用户”特权，这样才能创建新表。  如果目标表已存在，可以配置对目标表的“管理员”特权。 
 
有关 Azure 数据资源管理器主体角色的详细信息，请参阅[基于角色的授权](https://docs.microsoft.com/zh-cn/azure/kusto/management/access-control/role-based-authorization)。 有关如何管理安全角色，请参阅[安全角色管理](https://docs.microsoft.com/zh-cn/azure/kusto/management/security-roles)。

## <a name="spark-sink-writing-to-azure-data-explorer"></a>Spark 接收器：写入 Azure 数据资源管理器

1. 设置接收器参数：

     ```scala
    val KustoSparkTestAppId = dbutils.secrets.get(scope = "KustoDemos", key = "KustoSparkTestAppId")
    val KustoSparkTestAppKey = dbutils.secrets.get(scope = "KustoDemos", key = "KustoSparkTestAppKey")
 
    val appId = KustoSparkTestAppId
    val appKey = KustoSparkTestAppKey
    val authorityId = "72f988bf-86f1-41af-91ab-2d7cd011db47"
    val cluster = "Sparktest.eastus2"
    val database = "TestDb"
    val table = "StringAndIntTable"
    ```

1. 将 Spark 数据帧分批写入 Azure 数据资源管理器群集：

    ```scala
    df.write
      .format("com.microsoft.kusto.spark.datasource")
      .option(KustoOptions.KUSTO_CLUSTER, cluster)
      .option(KustoOptions.KUSTO_DATABASE, database)
      .option(KustoOptions.KUSTO_TABLE, table)
      .option(KustoOptions.KUSTO_AAD_CLIENT_ID, appId)
      .option(KustoOptions.KUSTO_AAD_CLIENT_PASSWORD, appKey) 
      .option(KustoOptions.KUSTO_AAD_AUTHORITY_ID, authorityId)
      .save()
    ```

1. 写入流数据：

    ```scala    
    import org.apache.spark.sql.streaming.Trigger
    import java.util.concurrent.TimeUnit
    
    // Set up a checkpoint and disable codeGen. Set up a checkpoint and disable codeGen as a workaround for an known issue 
    spark.conf.set("spark.sql.streaming.checkpointLocation", "/FileStore/temp/checkpoint")
    spark.conf.set("spark.sql.codegen.wholeStage","false")
    
    // Write to a Kusto table fro streaming source
    val kustoQ = csvDf
          .writeStream
          .format("com.microsoft.kusto.spark.datasink.KustoSinkProvider")
          .options(Map(
            KustoOptions.KUSTO_CLUSTER -> cluster,
            KustoOptions.KUSTO_TABLE -> table,
            KustoOptions.KUSTO_DATABASE -> database,
            KustoOptions.KUSTO_AAD_CLIENT_ID -> appId,
            KustoOptions.KUSTO_AAD_CLIENT_PASSWORD -> appKey,
            KustoOptions.KUSTO_AAD_AUTHORITY_ID -> authorityId))
          .trigger(Trigger.Once)
    
    kustoQ.start().awaitTermination(TimeUnit.MINUTES.toMillis(8))
    ```

## <a name="spark-source-reading-from-azure-data-explorer"></a>Spark 源：从 Azure 数据资源管理器读取数据

1. 读取少量的数据时，可定义数据查询：

    ```scala
    val conf: Map[String, String] = Map(
          KustoOptions.KUSTO_AAD_CLIENT_ID -> appId,
          KustoOptions.KUSTO_AAD_CLIENT_PASSWORD -> appKey,
          KustoOptions.KUSTO_QUERY -> s"$table | where (ColB % 1000 == 0) | distinct ColA"      
        )
    
    // Simplified syntax flavor
    import org.apache.spark.sql._
    import com.microsoft.kusto.spark.sql.extension.SparkExtension._
    import org.apache.spark.SparkConf
    
    val df = spark.read.kusto(cluster, database, "", conf)
    display(df)
    ```

1. 读取大量的数据时，必须提供暂用的 Blob 存储。 提供存储容器 SAS 密钥，或者存储帐户名称、帐户密钥和容器名称。 只有在当前 Spark 连接器预览版中才需要执行此步骤。

    ```scala
    // Use either container/account-key/account name, or container SaS
    val container = dbutils.secrets.get(scope = "KustoDemos", key = "blobContainer")
    val storageAccountKey = dbutils.secrets.get(scope = "KustoDemos", key = "blobStorageAccountKey")
    val storageAccountName = dbutils.secrets.get(scope = "KustoDemos", key = "blobStorageAccountName")
    // val storageSas = dbutils.secrets.get(scope = "KustoDemos", key = "blobStorageSasUrl")
    ```

    在以上示例中，我们不会使用连接器接口访问 Key Vault。 或者，我们可以通过一种更简单的方法使用 Databricks 机密。

1. 从 Azure 数据资源管理器读取数据：

    ```scala
    val df2 = spark.read.kusto(cluster, database, "ReallyBigTable", conf3)
    
    val dfFiltered = df2
      .where(df2.col("ColA").startsWith("row-2"))
      .filter("ColB > 12")
      .filter("ColB <= 21")
      .select("ColA")
    
    display(dfFiltered)
    ```
