---
title: 快速入门：通过 Azure 门户开始使用 Apache Hadoop 和 Apache Hive - Azure HDInsight
description: 了解如何使用 Azure 门户创建 HDInsight 群集，以及如何使用 Hive 查询数据。
keywords: hadoop 入门,hadoop linux,hadoop 快速入门,hive 入门,hive 快速入门
services: hdinsight
documentationcenter: ''
author: nitinme
manager: cgronlun
editor: cgronlun
tags: azure-portal
ms.service: hdinsight
ms.custom: hdinsightactive,hdiseo17may2017,mvc,seodec18
ms.devlang: na
ms.topic: quickstart
origin.date: 06/12/2019
ms.date: 07/22/2019
ms.author: v-yiso
ms.openlocfilehash: c44c6a8088dccc2a052cfc9e2c1bd4be3314103e
ms.sourcegitcommit: f4351979a313ac7b5700deab684d1153ae51d725
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/12/2019
ms.locfileid: "67845370"
---
# <a name="quickstart-create-apache-hadoop-cluster-in-azure-hdinsight-using-azure-portal"></a>快速入门：使用 Azure 门户在 Azure HDInsight 中创建 Apache Hadoop 群集

本文介绍如何使用 Azure 门户在 HDInsight 中创建 [Apache Hadoop](https://hadoop.apache.org/) 群集，以及如何在 HDInsight 中运行 Apache Hive 作业。 大部分 Hadoop 作业都是批处理作业。 创建群集，运行一些作业，并删除群集。 在本文中，将会执行所有这三项任务。

在此快速入门中，使用 Azure 门户创建 HDInsight Hadoop 群集。 还可以使用 [Azure 资源管理器模板](apache-hadoop-linux-tutorial-get-started.md)创建群集。

目前，HDInsight 附带[七个不同的群集类型](./apache-hadoop-introduction.md#cluster-types-in-hdinsight)。 每个群集类型都支持一组不同的组件。 所有群集类型都支持 Hive。 有关 HDInsight 中受支持组件的列表，请参阅 [HDInsight 提供的 Apache Hadoop 群集版本中有哪些新增功能？](../hdinsight-component-versioning.md)  

如果没有 Azure 订阅，请在开始前[创建一个试用帐户](https://www.azure.cn/pricing/1rmb-trial/)。

## <a name="create-an-apache-hadoop-cluster"></a>创建 Apache Hadoop 群集

在本部分中，使用 Azure 门户在 HDInsight 中创建 Hadoop 群集。 

1. 登录到 [Azure 门户](https://portal.azure.cn)。

1. 在 Azure 门户中，导航到“创建资源” > “Analytics” > “HDInsight”    。

    ![Azure 门户上的 Databricks](./media/apache-hadoop-linux-create-cluster-get-started-portal/create-hdinsight.png "Azure 门户上的 Databricks")

1. 在“HDInsight”   > “快速创建”   > “基本信息”  下，输入或选择以下值：

    |属性  |说明  |
    |---------|---------|
    |群集名称   | 输入 Hadoop 群集的名称。 由于 HDInsight 中的所有群集共享同一 DNS 命名空间，因此该名称必须唯一。 该名称最多可以有 59 个字符，包括字母、数字和连字符。 名称的第一个和最后一个字符不能为连字符。 |
    |订阅    |  选择 Azure 订阅。 |
    |群集类型     | 暂时跳过此步骤。 在此过程的下一步骤中提供此输入。|
    |群集登录用户名和密码    | 默认登录名为“admin”  。密码长度不得少于 10 个字符，且至少必须包含一个数字、一个大写字母和一个小写字母、一个非字母数字字符（' " ` \)字符除外）。 请确保不提供常见密码，如“Pass@word1”  。|
    |安全外壳 (SSH) 用户名 | 默认用户名为“sshuser”  。  可以提供其他名称作为 SSH 用户名。 |
    |对 SSH 使用群集登录密码| 选中此复选框，让 SSH 用户使用与提供给群集登录用户相同的密码。|
    |资源组     | 创建资源组，或选择现有资源组。  资源组是 Azure 组件的容器。  在此示例中，资源组包含 HDInsight 群集和依赖的 Azure 存储帐户。 |
    |位置    | 选择要在其中创建群集的 Azure 位置。  为获得更佳性能，请选择离你较近的位置。 |

    ![HDInsight Linux 入门提供群集基本值](./media/apache-hadoop-linux-create-cluster-get-started-portal/quick-create-basics.png "为创建 HDInsight 群集提供基本值")

1. 选择“群集类型”  以打开“群集配置”  页，然后提供以下值：

    |属性  |说明  |
    |---------|---------|
    |群集类型     | 选择“Hadoop”  |
    |版本     | 选择 Hadoop 2.7.3 (HDI 3.6) |

    ![HDInsight Linux 入门提供群集基本值](./media/apache-hadoop-linux-create-cluster-get-started-portal/cluster-configuration-hadoop.png "为创建 HDInsight 群集提供基本值")

    依次选择“选择”  、“下一步”  以转到存储设置。

1. 在“存储”  选项卡中，提供以下值：

    |属性  |说明  |
    |---------|---------|
    |主存储类型    | 对于本文，选择“Azure 存储”以使用 Azure 存储 Blob 作为默认存储帐户。 还可以将 Azure Data Lake Storage 用作默认存储。 |
    |选择方法     |  在本文中，选择“我的订阅”，使用 Azure 订阅中的存储帐户  。 若要使用其他订阅中的存储帐户，请选择“访问密钥”，然后提供该帐户的访问密钥  。 |
    |选择存储帐户   | 选择“选择存储帐户”  以选择现有存储帐户，或选择“新建”  。|

    接受所有其他默认值，然后选择“下一步”  以转到摘要页。

    ![HDInsight Linux 入门提供群集存储值](./media/apache-hadoop-linux-create-cluster-get-started-portal/quick-create-storage.png "为创建 HDInsight 群集提供存储值")

1. 在“摘要”选项卡中，确认之前步骤中所选的值  。

    ![HDInsight Linux 入门群集摘要](./media/apache-hadoop-linux-create-cluster-get-started-portal/quick-create-summary.png "HDInsight Linux 入门群集摘要")

1. 选择“创建”  。 创建群集大约需要 20 分钟时间。

1. 创建群集后，Azure 门户中会显示群集概述页。

    ![HDInsight Linux 入门群集设置](./media/apache-hadoop-linux-create-cluster-get-started-portal/cluster-overview.png "HDInsight 群集属性")    

    每个群集都有一个 [Azure 存储帐户](../hdinsight-hadoop-use-blob-storage.md)或 Azure Data Lake 帐户依赖项。 该帐户称为默认存储帐户。 HDInsight 群集及其默认存储帐户必须共存于同一个 Azure 区域中。 删除群集不会删除存储帐户。

    > [!NOTE]  
    > 如需其他群集创建方法或要了解本快速入门中使用的属性，请参阅[创建 HDInsight 群集](../hdinsight-hadoop-provision-linux-clusters.md)。

## <a name="run-apache-hive-queries"></a>运行 Apache Hive 查询

[Apache Hive](hdinsight-use-hive.md) 是 HDInsight 中最流行的组件。 有多种方法可以在 HDInsight 中运行 Hive 作业。 本快速入门使用门户中的 Ambari Hive 视图。 有关提交 Hive 作业的其他方法，请参阅[在 HDInsight 中使用 Hive](hdinsight-use-hive.md)。

1. 若要打开 Ambari，请从之前的屏幕截图中，选择“群集仪表板”  。  还可以浏览到 `https://ClusterName.azurehdinsight.cn`，其中 `ClusterName` 是你在上一部分中创建的群集。

    ![HDInsight Linux 入门群集仪表板](./media/apache-hadoop-linux-tutorial-get-started/hdinsight-linux-get-started-open-cluster-dashboard.png "HDInsight Linux 入门群集仪表板")

2. 输入在创建群集时指定的 Hadoop 用户名和密码。 默认的用户名为 **admin**。

3. 打开“Hive 视图”，如以下屏幕截图中所示： 
   
    ![选择 Ambari 视图](./media/apache-hadoop-linux-tutorial-get-started/selecthiveview.png "HDInsight Hive 查看器菜单")

4. 在“查询”  选项卡中，将以下 HiveQL 语句粘贴到工作表中：

    ```sql
    SHOW TABLES;
    ```

    ![HDInsight Hive 视图](./media/apache-hadoop-linux-tutorial-get-started/hiveview-1.png "HDInsight Hive 视图查询编辑器")

5. 选择“执行”  。 “结果”  选项卡将显示在“查询”  选项卡下面，并显示有关作业的信息。 
   
    完成查询后，“查询”  选项卡将显示操作结果。 应会看到一个名为 **hivesampletable** 的表。 所有 HDInsight 群集都随附了此示例 Hive 表。
   
    ![HDInsight Hive 视图](./media/apache-hadoop-linux-tutorial-get-started/hiveview.png "HDInsight Hive 视图查询编辑器")

6. 重复步骤 4 和 5，运行以下查询：

    ```sql
    SELECT * FROM hivesampletable;
    ```

7. 还可以保存查询的结果。 选择右侧的菜单按钮，并指定是要将结果下载为 CSV 文件，还是要将其存储到与群集关联的存储帐户。

    ![保存 Hive 查询的结果](./media/apache-hadoop-linux-tutorial-get-started/hdinsight-linux-hive-view-save-results.png "保存 Hive 查询的结果")

完成 Hive 作业后，可以[将结果导出到 Azure SQL 数据库或 SQL Server 数据库](apache-hadoop-use-sqoop-mac-linux.md)，还可以[使用 Excel 将结果可视化](apache-hadoop-connect-excel-power-query.md)。 有关在 HDInsight 中使用 Hive 的详细信息，请参阅[将 Apache Hive 和 HiveQL 与 HDInsight 中的 Apache Hadoop 配合使用以分析示例 Apache log4j 文件](hdinsight-use-hive.md)。

## <a name="clean-up-resources"></a>清理资源

完成本快速入门后，可以删除群集。 有了 HDInsight，便可以将数据存储在 Azure 存储中，因此可以在群集不用时安全地删除群集。 此外，还需要支付 HDInsight 群集费用，即使未使用。 由于群集费用高于存储空间费用数倍，因此在不使用群集时将其删除可以节省费用。

> [!NOTE]  
> 如果立即继续学习下一篇文章，以了解如何使用 Hadoop on HDInsight 运行 ETL 操作，建议保持群集运行  。 这是因为该教程中必须再次创建 Hadoop 群集。 但是，如果不立即学习下一篇文章，则必须立即删除该群集。

### <a name="to-delete-the-cluster-andor-the-default-storage-account"></a>删除群集和/或默认存储帐户

1. 返回到包含 Azure 门户的浏览器选项卡。 你应该在群集概览页上。 如果仅希望删除群集但保留默认的存储帐户，请选择“删除”  。

    ![删除 HDInsight 群集](./media/apache-hadoop-linux-tutorial-get-started/hdinsight-delete-cluster.png "删除 HDInsight 群集")

2. 如果希望删除群集和默认存储帐户，请选择资源组名称（之前的屏幕截图中已突出显示），打开资源组页。

3. 选择“删除资源组”，删除资源组（包括群集和默认存储帐户）  。 注意，删除资源组会删除存储帐户。 如果想要保留存储帐户，请选择仅删除群集。

## <a name="next-steps"></a>后续步骤

本快速入门介绍了如何使用资源管理器模板创建基于 Linux 的 HDInsight 群集，以及如何执行基本 Hive 查询。 下一篇文章将介绍如何使用 Hadoop on HDInsight 执行提取、转换和加载 (ETL) 操作。


[1]: ../HDInsight/apache-hadoop-visual-studio-tools-get-started.md

[hdinsight-provision]: hdinsight-provision-linux-clusters.md
[hdinsight-upload-data]: hdinsight-upload-data.md
[hdinsight-use-hive]: hdinsight-use-hive.md
[hdinsight-use-pig]: hdinsight-use-pig.md


