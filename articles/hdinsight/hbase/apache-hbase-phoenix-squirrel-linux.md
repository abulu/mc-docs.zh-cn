---
title: 快速入门：在 Azure HDInsight 中查询 Apache HBase - Apache Phoenix
description: 了解如何在 HDInsight 中使用 Apache Phoenix。 此外，了解如何在计算机上安装和设置 SQLLine 以连接到 HDInsight 中的 HBase 群集。
author: hrasheed-msft
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: quickstart
origin.date: 06/12/2019
ms.date: 07/22/2019
ms.author: v-yiso
ms.openlocfilehash: d50764f2a3c18cfc311704b5e690ebfb7a08ea47
ms.sourcegitcommit: f4351979a313ac7b5700deab684d1153ae51d725
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/12/2019
ms.locfileid: "67845239"
---
# <a name="quickstart-query-apache-hbase-in-azure-hdinsight-with-apache-phoenix"></a>快速入门：使用 Apache Phoenix 在 Azure HDInsight 中查询 Apache HBase

在本快速入门中，你将学习如何使用 Apache Phoeni 在 Azure HDInsight 中运行 HBase 查询。 Apache Phoenix 是 Apache HBase 的 SQL 查询引擎。 该引擎以 JDBC 驱动程序的形式供用户访问，并且支持使用 SQL 来查询和管理 HBase 表。 [SQLLine](http://sqlline.sourceforge.net/) 是用于执行 SQL 的命令行实用工具。

如果没有 Azure 订阅，可以在开始前创建一个[免费帐户](https://www.azure.cn/pricing/1rmb-trial)。

## <a name="prerequisites"></a>先决条件

* Apache HBase 群集。 若要创建 HDInsight 群集，请参阅[创建群集](../hadoop/apache-hadoop-linux-tutorial-get-started.md#create-cluster)。  确保选择 **HBase** 群集类型。

* SSH 客户端。 有关详细信息，请参阅[使用 SSH 连接到 HDInsight (Apache Hadoop)](../hdinsight-hadoop-linux-use-ssh-unix.md)。

## <a name="identify-a-zookeeper-node"></a>识别 ZooKeeper 节点

在连接到 HBase 群集时，需要连接到 Apache ZooKeeper 节点之一。 每个 HDInsight 群集都有三个 ZooKeeper 节点。 可以使用 Curl 来快速识别 ZooKeeper 节点。 编辑以下 curl 命令，将 `PASSWORD` 和 `CLUSTERNAME` 替换为相关的值，然后在命令提示符下输入该命令：

```cmd
curl -u admin:PASSWORD -sS -G https://CLUSTERNAME.azurehdinsight.cn/api/v1/clusters/CLUSTERNAME/services/ZOOKEEPER/components/ZOOKEEPER_SERVER
```

输出的一部分将类似于以下内容：

```output
    {
      "href" : "http://hn1-brim.432dc3rlshou3ocf251eycoapa.bx.internal.chinacloudapp.cn:8080/api/v1/clusters/myCluster/hosts/zk0-brim.432dc3rlshou3ocf251eycoapa.bx.internal.cloudapp.net/host_components/ZOOKEEPER_SERVER",
      "HostRoles" : {
        "cluster_name" : "myCluster",
        "component_name" : "ZOOKEEPER_SERVER",
        "host_name" : "zk0-brim.432dc3rlshou3ocf251eycoapa.bx.internal.chinacloudapp.cn"
      }
```

记下 `host_name` 的值供以后使用。

## <a name="create-a-table-and-manipulate-data"></a>创建表并操作数据

可以使用 SSH 连接到 HBase 群集，然后使用 Apache Phoenix 来创建 HBase 表以及插入和查询数据。

1. 使用 `ssh` 命令连接到 HBase 群集。 编辑以下命令，将 `CLUSTERNAME` 替换为群集的名称，然后输入该命令：

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.cn
    ```

2. 将目录更改到 Phoenix 客户端。 输入以下命令：

    ```bash
    cd /usr/hdp/current/phoenix-client/bin
    ```

3. 启动 [SQLLine](http://sqlline.sourceforge.net/)。 编辑以下命令，将 `ZOOKEEPER` 替换为群集的名称，然后输入该命令：

    ```bash
    ./sqlline.py ZOOKEEPER:2181:/hbase-unsecure
    ```

4. 创建一个 HBase 表。 输入以下命令：

    ```sql
    CREATE TABLE Company (company_id INTEGER PRIMARY KEY, name VARCHAR(225));
    ```

5. 使用 SQLLine `!tables` 命令列出 HBase 中的所有表。 输入以下命令：

    ```sqlline
    !tables
    ```

6. 在表中插入值。 输入以下命令：

    ```sql
    UPSERT INTO Company VALUES(1, 'Microsoft');
    UPSERT INTO Company VALUES(2, 'Apache');
    ```

7. 查询表。 输入以下命令：

    ```sql
    SELECT * FROM Company;
    ```

8. 删除记录。 输入以下命令：

    ```sql
    DELETE FROM Company WHERE COMPANY_ID=1;
    ```

9. 删除表。 输入以下命令：

    ```hbase
    DROP TABLE Company;
    ```

10. 使用 SQLLine `!quit` 命令退出 SQLLine。 输入以下命令：

    ```sqlline
    !quit
    ```

## <a name="clean-up-resources"></a>清理资源

完成本快速入门后，可以删除群集。 有了 HDInsight，便可以将数据存储在 Azure 存储中，因此可以在群集不用时安全地删除群集。 此外，还需要支付 HDInsight 群集费用，即使未使用。 由于群集费用高于存储空间费用数倍，因此在不使用群集时将其删除可以节省费用。

若要删除群集，请参阅[使用浏览器、PowerShell 或 Azure CLI 删除 HDInsight 群集](../hdinsight-delete-cluster.md)。

## <a name="next-steps"></a>后续步骤

在本快速入门中，你已学习了如何使用 Apache Phoenix 在 Azure HDInsight 中运行 HBase 查询。 若要详细了解 Apache Phoenix，下一篇文章将提供更深层次的介绍。

> [!div class="nextstepaction"]
> [HDInsight 中的 Apache Phoenix](../hdinsight-phoenix-in-hdinsight.md)
