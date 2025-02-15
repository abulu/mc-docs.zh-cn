---
title: 在 HDInsight 中将 Apache Hive 与 PowerShell 配合使用 - Azure
description: 使用 PowerShell 在 HDInsight 上的 Apache Hadoop 中运行 Hive 查询。
services: hdinsight
author: hrasheed-msft
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: big-data
origin.date: 04/23/2018
ms.date: 07/22/2019
ms.author: v-yiso
ms.openlocfilehash: 3fcb3554ba1e5f41857273fa5e01cd055aa03020
ms.sourcegitcommit: f4351979a313ac7b5700deab684d1153ae51d725
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/12/2019
ms.locfileid: "67845087"
---
# <a name="run-apache-hive-queries-using-powershell"></a>使用 PowerShell 运行 Apache Hive 查询
[!INCLUDE [hive-selector](../../../includes/hdinsight-selector-use-hive.md)]

本文档提供使用 Azure PowerShell 以 Azure 资源组模式在 Apache Hadoop on HDInsight 群集上运行 Hive 查询的示例。

> [!NOTE]  
> 本文档未详细描述示例中使用的 HiveQL 语句的作用。 有关此示例中使用的 HiveQL 的信息，请参阅[将 Apache Hive 与 HDInsight 上的 Apache Hadoop 配合使用](hdinsight-use-hive.md)。

## <a name="prerequisites"></a>先决条件

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

* 基于 Linux 的 Apache Hadoop on HDInsight 群集版本 3.4 或更高版本。

* 具有 Azure PowerShell 的客户端。

[!INCLUDE [upgrade-powershell](../../../includes/hdinsight-use-latest-powershell.md)]

## <a name="run-a-hive-query"></a>运行 Hive 查询

Azure PowerShell 提供 *cmdlet*，可让你在 HDInsight 上远程运行 Hive 查询。 cmdlet 在内部对 HDInsight 群集上的 [WebHCat](https://cwiki.apache.org/confluence/display/Hive/WebHCat) 进行 REST 调用。

在远程 HDInsight 群集上运行 Hive 查询时，使用以下 Cmdlet：

* `Connect-AzAccount`：在 Azure 订阅中进行 Azure PowerShell 身份验证。
* `New-AzHDInsightHiveJobDefinition`：使用指定的 HiveQL 语句创建作业定义  。
* `Start-AzHDInsightJob`：将作业定义发送到 HDInsight 并启动作业。 将返回作业对象  。
* `Wait-AzHDInsightJob`：使用作业对象来检查作业的状态。 它等到作业完成或超出等待时间。
* `Get-AzHDInsightJobOutput`：用于检索作业的输出。
* `Invoke-AzHDInsightHiveJob`：用于运行 HiveQL 语句。 此 cmdlet 将阻止查询完成，然后返回结果。
* `Use-AzHDInsightCluster`：设置要用于 `Invoke-AzHDInsightHiveJob` 命令的当前群集。

以下步骤演示了如何使用这些 Cmdlet 在 HDInsight 群集上运行作业：

1. 使用编辑器将以下代码保存为 `hivejob.ps1`。

    ```powershell
    # Login to your Azure subscription
    # Is there an active Azure subscription?
    $sub = Get-AzureRmSubscription -ErrorAction SilentlyContinue
    if(-not($sub))
    {
        Add-AzureRmAccount -EnvironmentName AzureChinaCloud
    }

    #Get cluster info
    $clusterName = Read-Host -Prompt "Enter the HDInsight cluster name"
    $creds=Get-Credential -Message "Enter the login for the cluster"

    #HiveQL
    #Note: set hive.execution.engine=tez; is not required for
    #      Linux-based HDInsight
    $queryString = "set hive.execution.engine=tez;" +
                "DROP TABLE log4jLogs;" +
                "CREATE EXTERNAL TABLE log4jLogs(t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string) ROW FORMAT DELIMITED FIELDS TERMINATED BY ' ' STORED AS TEXTFILE LOCATION 'wasbs:///example/data/';" +
                "SELECT * FROM log4jLogs WHERE t4 = '[ERROR]';"

    #Create an HDInsight Hive job definition
    $hiveJobDefinition = New-AzureRmHDInsightHiveJobDefinition -Query $queryString 

    #Submit the job to the cluster
    Write-Host "Start the Hive job..." -ForegroundColor Green

    $hiveJob = Start-AzureRmHDInsightJob -ClusterName $clusterName -JobDefinition $hiveJobDefinition -ClusterCredential $creds

    #Wait for the Hive job to complete
    Write-Host "Wait for the job to complete..." -ForegroundColor Green
    Wait-AzureRmHDInsightJob -ClusterName $clusterName -JobId $hiveJob.JobId -ClusterCredential $creds

    # Print the output
    Write-Host "Display the standard output..." -ForegroundColor Green
    Get-AzureRmHDInsightJobOutput `
        -Clustername $clusterName `
        -JobId $hiveJob.JobId `
        -HttpCredential $creds
    ```

2. 打开一个新的 **Azure PowerShell** 命令提示符。 将目录更改到 `hivejob.ps1` 文件的位置，并使用以下命令来运行脚本：

        .\hivejob.ps1

    脚本运行时，系统会提示输入群集名称和 HTTPS/群集管理员帐户凭据。 可能还会提示登录到 Azure 订阅。

3. 作业完成时，它会返回类似以下文本的信息：

        Display the standard output...
        2012-02-03      18:35:34        SampleClass0    [ERROR] incorrect       id
        2012-02-03      18:55:54        SampleClass1    [ERROR] incorrect       id
        2012-02-03      19:25:27        SampleClass4    [ERROR] incorrect       id

4. 如前所述，`Invoke-Hive` 可以用来运行查询，并等待响应。 使用以下脚本查看 Invoke-Hive 的工作原理：

    ```powershell
    # Login to your Azure subscription
    # Is there an active Azure subscription?
    $sub = Get-AzureRmSubscription -ErrorAction SilentlyContinue
    if(-not($sub))
    {
        Add-AzureRmAccount -EnvironmentName AzureChinaCloud
    }

    #Get cluster info
    $clusterName = Read-Host -Prompt "Enter the HDInsight cluster name"
    $creds=Get-Credential -Message "Enter the login for the cluster"

    # Set the cluster to use
    Use-AzureRmHDInsightCluster -ClusterName $clusterName -HttpCredential $creds

    $queryString = "set hive.execution.engine=tez;" +
                "DROP TABLE log4jLogs;" +
                "CREATE EXTERNAL TABLE log4jLogs(t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string) ROW FORMAT DELIMITED FIELDS TERMINATED BY ' ' STORED AS TEXTFILE LOCATION '/example/data/';" +
                "SELECT * FROM log4jLogs WHERE t4 = '[ERROR]';"
    Invoke-AzureRmHDInsightHiveJob `
        -StatusFolder "statusout" `
        -Query $queryString
    ```

    输出类似于以下文本：

        2012-02-03    18:35:34    SampleClass0    [ERROR]    incorrect    id
        2012-02-03    18:55:54    SampleClass1    [ERROR]    incorrect    id
        2012-02-03    19:25:27    SampleClass4    [ERROR]    incorrect    id

   > [!NOTE]
   > 对于较长的 HiveQL 查询，可以使用 Azure PowerShell **Here-Strings** cmdlet 或 HiveQL 脚本文件。 以下代码段显示了如何使用 `Invoke-Hive` cmdlet 来运行 HiveQL 脚本文件。 HiveQL 脚本文件必须上传到 wasb://。
   >
   > `Invoke-AzHDInsightHiveJob -File "wasb://<ContainerName>@<StorageAccountName>/<Path>/query.hql"`
   >
   > 有关 **Here-Strings** 的详细信息，请参阅<a href="https://technet.microsoft.com/library/ee692792.aspx" target="_blank">使用 Windows PowerShell Here-Strings</a>。

## <a name="troubleshooting"></a>故障排除

如果作业完成时未返回任何信息，请查看错误日志。 若要查看此作业的错误信息，请将以下内容添加到 `hivejob.ps1` 文件的末尾，保存并重新运行该文件。

```powershell
# Print the output of the Hive job.
Get-AzHDInsightJobOutput `
        -Clustername $clusterName `
        -JobId $job.JobId `
        -HttpCredential $creds `
        -DisplayOutputType StandardError
```

作业处理期间，此 cmdlet 返回写入到 STDERR 中的信息。

## <a name="summary"></a>摘要

如你所见，Azure PowerShell 提供了简单的方法让你在 HDInsight 群集上运行 Hive 查询，监视作业状态，以及检索输出。

## <a name="next-steps"></a>后续步骤

有关 HDInsight 中的 Hive 的一般信息：

* [将 Apache Hive 与 Apache Hadoop on HDInsight 配合使用](hdinsight-use-hive.md)

有关 HDInsight 上 Hadoop 的其他使用方法的信息：

* [将 Apache Pig 与 Apache Hadoop on HDInsight 配合使用](hdinsight-use-pig.md)
* [将 MapReduce 与 HDInsight 上的 Apache Hadoop 配合使用](hdinsight-use-mapreduce.md)
