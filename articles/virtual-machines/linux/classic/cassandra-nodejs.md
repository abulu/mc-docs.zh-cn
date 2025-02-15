---
title: 通过 Node.js 在 Azure 中的 Linux 上运行 Cassandra 群集
description: 如何使用 Node.js 应用在 Azure 虚拟机上通过 Linux 运行 Cassandra 群集
services: virtual-machines-linux
documentationcenter: nodejs
author: rockboyfor
manager: digimobile
editor: ''
tags: azure-service-management
ms.assetid: 30de1f29-e97d-492f-ae34-41ec83488de0
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
origin.date: 08/17/2017
ms.date: 11/26/2018
ms.author: v-yeche
ms.openlocfilehash: 8c3e7cc5f0b0b0996e973f6aaf06ffba91336d3e
ms.sourcegitcommit: 9e50dde3362b6e6b192761ead6cd3f434dfb2168
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/10/2019
ms.locfileid: "67725217"
---
# <a name="run-a-cassandra-cluster-on-linux-in-azure-with-nodejs"></a>使用 Node.js 在 Azure 中的 Linux 上运行 Cassandra 群集

> [!IMPORTANT] 
> Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器部署模型和经典部署模型](../../../resource-manager-deployment-model.md)。 本文介绍如何使用经典部署模型。 Azure 建议大多数新部署使用 Resource Manager 模型。 请参阅 [Datastax Enterprise](https://github.com/Azure/azure-quickstart-templates/tree/master/datastax) 的 Resource Manager 模板和 [CentOS 上 的Spark 群集和 Cassandra](https://github.com/Azure/azure-quickstart-templates/tree/master/spark-and-cassandra-on-centos/)。

## <a name="overview"></a>概述
Azure 是一个开放式云平台，可运行 Azure 软件和非 Azure 软件，包括操作系统、应用程序服务器、消息传递中间件，以及来自商业和开源模型的 SQL 数据库和 NoSQL 数据库。 在包括 Azure 在内的公共云上构建可复原的服务，需要针对应用程序服务器和存储层进行仔细规划，并精心设计体系结构。 Cassandra 的分布式存储体系结构，自然有助于构建高可用性的系统，此类系统在发生群集故障时容错性很强。 Cassandra 是 Apache Software Foundation 在 cassandra.apache.org 上维护的云规模 NoSQL 数据库。Cassandra 以 Java 编写。 因此，它可以在 Windows 和 Linux 平台上运行。
<!-- Notice: Change Microsoft to Azure-->

本文重点介绍如何在 Ubuntu 上使用 Azure 虚拟机和虚拟网络将 Cassandra 部署为单个和多个数据中心群集。 对群集进行部署以实现生产优化型工作负载不在本文讨论范围之内，因为这需要进行多磁盘节点配置、适当的环形拓扑设计和数据建模，以支持所需的复制、数据一致性、吞吐量并满足高可用性要求。

本文使用基本方法说明构建 Cassandra 群集所涉及的内容，并对 Docker、Chef 或 Puppet 进行比较，使基础结构部署更加简单。  

## <a name="the-deployment-models"></a>部署模型
Azure 网络允许部署独立的专用群集，并可对这些群集的访问进行限制，从而实现能够进行细化管理的网络安全性。  由于本文介绍 Cassandra 部署基础知识，因此不会重点讲解一致性级别以及如何针对吞吐量优化存储设计的问题。 以下是有关假设性群集的网络需求列表：

* 外部系统无法从 Azure 内部或外部访问 Cassandra 数据库
* Cassandra 群集必须位于负载均衡器之后，以便进行 Thrift 通信
* 可以将 Cassandra 节点部署在每个数据中心的两个组中，以便增强群集可用性
* 锁定该群集，使得只有应用程序服务器场可以直接访问数据库
* 除 SSH 之外，没有其他的公共网络终结点
* 每个 Cassandra 节点需要一个固定的内部 IP 地址

Cassandra 可以部署到单个或多个 Azure 区域，具体取决于工作负荷的分布式性质。 可以使用多区域部署模型，通过相同的 Cassandra 基础结构为靠近特定地理位置的最终用户提供服务。 Cassandra 的内置节点复制针对源自多个数据中心的多主机写入同步问题，可以为应用程序提供一致性的数据视图。 在出现较大范围的 Azure 服务中断的情况下，多区域部署还有助于降低风险。 可以调整 Cassandra 的一致性和复制拓扑，帮助满足应用程序的不同 RPO 需求。

### <a name="single-region-deployment"></a>单区域部署
我们首先学习单区域部署，然后运用所学的知识创建多区域模型。 将使用 Azure 虚拟网络创建独立的子网，以满足上述网络安全要求。  介绍的单区域部署创建过程使用 Ubuntu 14.04 LTS 和 Cassandra 2.08。 但是，可以轻松调整该过程，使其适用于其他 Linux 产品。 以下是单区域部署的部分系统特征。  

**高可用性：** 图 1 中所示的 Cassandra 节点已部署到两个可用性集，因此这些节点是分布到多个容错域的，可用性很高。 将标注每个可用性集的 VM 映射到 2 个容错域。 Azure 使用容错域概念来管理计划外停机（例如，硬件或软件故障）， 使用升级域（例如，主机或来宾 OS 修补/升级、应用程序升级）概念来管理计划内停机。 请参阅 [Azure 应用程序的灾难恢复和高可用性](https://msdn.microsoft.com/library/dn251004.aspx)，了解容错域和升级域在实现高可用性方面的作用。

![单区域部署](./media/cassandra-nodejs/cassandra-linux1.png)

图 1：单区域部署

请注意，在撰写本文时，Azure 并不允许将一组 VM 显式映射到特定容错域；因此，即使采用图 1 所示的部署模型，也极有可能会将所有虚拟机映射到两个容错域，而不是四个容错域。

**对 Thrift 流量进行负载均衡：** Web 服务器中的 Thrift 客户端库通过内部负载均衡器连接到群集。 在使用云服务托管 Cassandra 群集的情况下，这需要执行相关过程，以便将内部负载均衡器添加到“数据”子网（参见图 1）。 定义好内部负载均衡器以后，每个节点都需要添加进行过负载均衡的终结点，并使用以前定义的负载均衡器名称对负载均衡集进行标注。 有关详细信息，请参阅 [Azure 内部负载均衡](../../../load-balancer/load-balancer-internal-overview.md)。

**群集种子：** 必须选择可用性最高的节点作为种子，因为新节点需要与种子节点进行通信才能发现群集的拓扑。 会从每个可用性集中选择一个节点作为种子节点，以免出现单节点故障。

**复制因子和一致性级别：** Cassandra 固有的高可用性和数据耐用性通过复制因子（RF - 存储在群集中的每一行的副本数目）和一致性级别（在将结果返回到调用方之前需要读取/写入的副本数）来表示。 复制因子是在创建 KEYSPACE（类似于关系数据库）过程中指定的，而一致性级别则是在发出 CRUD 查询时指定的。 有关一致性的详细信息以及进行仲裁计算的公式，请参阅 Cassandra 文档： [针对一致性进行配置](https://docs.datastax.com/en/cassandra/3.0/cassandra/dml/dmlConfigConsistency.html) 。

Cassandra 支持两种类型的数据完整性模型 - 一致性和最终一致性；复制因子和一致性级别共同决定数据是在写操作完成后就表现出一致性，还是最终才表现出一致性。 例如，如果指定 QUORUM 作为一致性级别，则只要一致性级别低于需要写入的副本数，就会根据需要写入相应的副本数以满足 QUORUM（例如 1）结果，使得数据最终保持一致。

上面显示的 8 节点群集的复制因子为 3，读/写一致性级别为 QUORUM（读取或写入 2 个节点以确保一致性），因此理论上可以承受每个复制组最多丢失 1 个节点的故障，超出此数目应用程序才会注意到故障的存在。 这里假定所有密钥空间的读/写请求均已实现良好的平衡。  以下是用于已部署群集的参数：

单区域 Cassandra 群集配置：

| 群集参数 | Value | 备注 |
| --- | --- | --- |
| 节点数 (N) |8 |群集中节点总数 |
| 复制因子 (RF) |3 |给定行副本数 |
| 一致性级别（写入） |QUORUM [(RF/2) +1) = 2] 公式的结果向下舍入 |在将响应发送到调用方前，最多写入 2 个副本；第 3 个副本将采取最终一致性方式写入。 |
| 一致性级别（读取） |QUORUM [(RF/2) +1= 2] 公式结果向下舍入 |在将响应发送到调用方前读取 2 个副本。 |
| 复制策略 |NetworkTopologyStrategy 请参阅 Cassandra 文档中的 [数据复制](https://docs.datastax.com/en/cassandra/3.0/cassandra/architecture/archDataDistributeAbout.html) ，了解详细信息 |了解部署拓扑，并将副本置于节点上，以便确保最终不会让所有副本位于同一机架上 |
| Snitch |GossipingPropertyFileSnitch 请参阅 Cassandra 文档中的[开关](https://docs.datastax.com/en/cassandra/3.0/cassandra/architecture/archSnitchesAbout.html)了解详细信息 |NetworkTopologyStrategy 使用 snitch 概念了解拓扑。 将每个节点映射到数据中心和机架时，使用 GossipingPropertyFileSnitch 可以更好地进行控制。 然后，该群集使用 gossip 传播此信息。 相对于 PropertyFileSnitch，此方法在进行动态 IP 设置时更加简单 |

**针对 Cassandra 群集的 Azure 注意事项：** Azure 虚拟机功能使用 Azure Blob 存储以确保磁盘持久性；Azure 存储为每个磁盘保留 3 个副本以确保高耐用性。 这意味着插入 Cassandra 表中的每行数据已存储在三个副本中。 因此即使复制因子 (RF) 为 1。 复制因子为 1 的主要问题是，即使单个 Cassandra 节点发生故障，应用程序也会体验到停机。 不过，如果某个节点因 Azure 结构控制器检测到问题（例如，硬件故障、系统软件故障）而关闭，则会使用相同的存储驱动器预配一个新节点来代替旧节点。 预配一个新节点代替旧节点可能需要数分钟的时间。  同样，如果执行规划的维护活动（如来宾 OS 更改、Cassandra 升级和应用程序更改），Azure 结构控制器会在群集中对节点进行滚动升级。  滚动升级也会一次关闭数个节点，因此该群集会出现数个分区短暂停机的现象。 不过，由于固有的 Azure 存储冗余，数据不会丢失。  

对于部署到 Azure 但不需要高可用性（例如，约 99.9 的高可用性相当于 8.76 小时/年；有关详细信息，请参阅[高可用性](http://en.wikipedia.org/wiki/High_availability)）的系统，可以在 RF=1 且一致性级别=1 的情况下运行。  对于需要高可用性的应用程序，RF=3 且一致性级别=QUORUM 意味着系统可以承受一个节点的一个副本出现停机的情况。 RF=1 在传统部署（例如本地部署）中不能使用，因为如果出现磁盘故障之类的问题，就可能导致数据丢失。   

## <a name="multi-region-deployment"></a>多区域部署
以上所述的 Cassandra 数据中心感知型复制和一致性模型有助于进行多区域部署，不需任何外部工具。 这与传统的关系数据库不同，后者在针对多主机写入进行数据库监视设置时，可能需要完成复杂的操作。 多区域设置中的 Cassandra 有助于多种使用方案，包括：

**基于位置远近的部署：** 多租户应用程序如果进行了清楚的从租户用户到区域的映射，则适合采用多区域群集，因为后者的延迟较低。 例如，适合教育机构使用的学习管理系统可以在中国东部和中国北部区域部署分布式群集，为这两个区域的校园提供事务处理和分析服务。 数据在读取和写入时可以在本地保持一致，最终会在这两个地区保持一致。 此外，还有其他示例（如介质分发和电子商务等），只要其服务对象是集中在某个地区的用户群都适合此部署模型。

**高可用性：** 若要实现软硬件的高可用性，冗余很重要；有关详细信息，请参阅“在 Azure 中构建可靠的云系统”。 在 Azure 中，若要实现真正的冗余，唯一可靠的方式是部署多区域群集。 应用程序可以采用主动-主动或主动-被动模式进行部署。如果某个区域停机，Azure 流量管理器可以将流量重定向到活动区域。  如果单区域部署的可用性为 99.9，双区域部署可以实现 99.9999 的可用性，计算公式为：(1-(1-0.999) * (1-0.999))*100)；有关详细信息，请参阅前面的内容。

**灾难恢复：** 多区域 Cassandra 群集如果设计得当，可以承受灾难性的数据中心中断情况。 如果某个区域停机，可以通过部署到其他区域的应用程序为最终用户提供服务。 与任何其他业务连续性实施一样，该应用程序必须承受某种程度的数据丢失，因为数据位于异步管道中。 不过，与传统数据库恢复过程相比，Cassandra 恢复过程更快。 图 2 显示了典型多区域部署模型，每个区域有 8 个节点。 两个区域互为镜像以确保对称性；实际设计取决于工作负荷类型（例如，是事务性还是分析性）、RPO、RTO、数据一致性和可用性要求。

![多区域部署](./media/cassandra-nodejs/cassandra-linux2.png)

图 2：多区域 Cassandra 部署

### <a name="network-integration"></a>网络集成
部署到专用网络（位于两个区域）的虚拟机组使用 VPN 隧道互相通信。 VPN 隧道连接两个在网络部署过程中预配的软件网关。 针对“Web”和“数据”子网，两个区域具有类似的网络体系结构；Azure 网络可根据需要创建多个子网，并根据网络安全需要应用 ACL。 设计群集拓扑时，需要考虑数据中心之间的通信延迟，以及网络通信的经济影响。

### <a name="data-consistency-for-multi-data-center-deployment"></a>进行多数据中心部署时需要考虑数据一致性
进行分布式部署时，需要了解群集拓扑对吞吐量和高可用性的影响。 选择 RF 和一致性级别时，需要确保仲裁不依赖于所有数据中心的可用性。
对于需要高一致性的系统来说，如果一致性级别（针对读取和写入）为 LOCAL_QUORUM，则可以确保本地读取和写入能够从本地节点得到满足，而数据则会通过异步方式复制到远程数据中心。  表 2 汇总了后面介绍的多区域群集的配置详细信息。

**双区域 Cassandra 群集配置**

| 群集参数 | Value | 备注 |
| --- | --- | --- |
| 节点数 (N) |8 + 8 |群集中节点总数 |
| 复制因子 (RF) |3 |给定行副本数 |
| 一致性级别（写入） |LOCAL_QUORUM [(sum(RF)/2) +1) = 4] 公式结果向下舍入 |将 2 个节点同步写入第一个数据中心；满足仲裁所需的其余 2 个节点会通过异步方式写入第二个数据中心。 |
| 一致性级别（读取） |LOCAL_QUORUM ((RF/2) +1) = 2 公式结果向下舍入 |读取请求仅从一个区域满足；在将响应发送回客户端之前，读取 2 个节点。 |
| 复制策略 |NetworkTopologyStrategy 请参阅 Cassandra 文档中的 [数据复制](https://docs.datastax.com/en/cassandra/3.0/cassandra/architecture/archDataDistributeAbout.html) ，了解详细信息 |了解部署拓扑，并将副本置于节点上，以便确保最终不会让所有副本位于同一机架上 |
| Snitch |GossipingPropertyFileSnitch 请参阅 Cassandra 文档中的 [Snitch](https://docs.datastax.com/en/cassandra/3.0/cassandra/architecture/archSnitchesAbout.html) ，了解详细信息 |NetworkTopologyStrategy 使用 snitch 概念了解拓扑。 将每个节点映射到数据中心和机架时，使用 GossipingPropertyFileSnitch 可以更好地进行控制。 然后，该群集使用 gossip 传播此信息。 相对于 PropertyFileSnitch，此方法在进行动态 IP 设置时更加简单 |

## <a name="the-software-configuration"></a>软件配置
在部署过程中使用以下软件版本：

<table>
<tr><th>软件</th><th>Source</th><th>版本</th></tr>
<tr><td>JRE    </td><td><a href="https://docs.azure.cn/zh-cn/java/java-supported-jdk-runtime?view=azure-java-stable" data-raw-source="[JRE 8](https://docs.azure.cn/zh-cn/java/java-supported-jdk-runtime?view=azure-java-stable)">JRE 8</a> </td><td>8U5</td></tr>
<tr><td>JNA    </td><td><a href="https://github.com/twall/jna" data-raw-source="[JNA](https://github.com/twall/jna)">JNA</a> </td><td> 3.2.7</td></tr>
<tr><td>Cassandra</td><td><a href="http://www.apache.org/dist/cassandra/" data-raw-source="[Apache Cassandra 2.0.8](http://www.apache.org/dist/cassandra/)">Apache Cassandra 2.0.8</a></td><td> 2.0.8</td></tr>
<tr><td>Ubuntu    </td><td><a href="https://www.azure.cn/" data-raw-source="[Azure](https://www.azure.cn/)">Azure</a> </td><td>14.04 LTS</td></tr>
</table>

若要简化部署，请将全部所需软件下载至桌面。 然后将其上传到进行群集部署之前需要创建的 Ubuntu 模板映像中。

将以上软件下载到本地计算机上某个已知 的下载目录（例如，Windows 上的 %TEMP%/downloads 或者大多数 Linux 发行版或 Mac 上的 ~/Downloads）。

### <a name="create-ubuntu-vm"></a>创建 Ubuntu VM
在该过程的此步骤中，我们将使用必备软件创建 Ubuntu 映像，以便重复使用映像预配多个 Cassandra 节点。  

#### <a name="step-1-generate-ssh-key-pair"></a>步骤 1：生成 SSH 密钥对
Azure 在进行配置时需要用 PEM 或 DER 编码的 X509 公钥。 按照“如何在 Azure 上通过 Linux 使用 SSH”（可能为英文页面）上的说明进行操作来生成公/私钥对。 如果你打算在 Windows 或 Linux 上将 putty.exe 用作 SSH 客户端，则必须使用 puttygen.exe 将 PEM 编码的 RSA 私钥转换为 PPK 格式。 可在以上网页中找到有关此操作的说明。

#### <a name="step-2-create-ubuntu-template-vm"></a>步骤 2：创建 Ubuntu 模板 VM
若要创建模板 VM，请登录到 Azure 门户并按以下顺序操作：依次单击“新建”、“计算”、“虚拟机”、“从库中”、“UBUNTU”、“Ubuntu Server 14.04 LTS”、右键头。 有关介绍如何创建 Linux VM 的教程，请参阅“创建运行 Linux 的虚拟机”。

在“虚拟机配置”屏幕 #1 中输入以下信息：

<table>
<tr><th>字段名称              </td><td>       字段值               </td><td>         备注                </td><tr>
<tr><td>版本发布日期    </td><td> 从下拉列表中选择日期</td><td></td><tr>
<tr><td>虚拟机名称    </td><td> cass-template                   </td><td> 这是 VM 的主机名 </td><tr>
<tr><td>层                     </td><td> 标准                           </td><td> 保留默认值              </td><tr>
<tr><td>大小                     </td><td> A1                              </td><td>根据 IO 需求选择 VM；对于此用途，请保留默认值 </td><tr>
<tr><td> 新用户名             </td><td> localadmin                       </td><td> &quot;admin&quot; 是 Ubuntu 12.xx 及更高版本中保留的用户名</td><tr>
<tr><td> 身份验证         </td><td> 单击复选框                 </td><td>如果希望使用 SSH 密钥进行保护，则选中 </td><tr>
<tr><td> 证书             </td><td> 公钥证书的文件名 </td><td> 使用以前生成的公钥</td><tr>
<tr><td> 新密码    </td><td> 强密码 </td><td> </td><tr>
<tr><td> 确认密码    </td><td> 强密码 </td><td></td><tr>
</table>

在“虚拟机配置”屏幕 #2 中输入以下信息：

<table>
<tr><th>字段名称             </th><th> 字段值                       </th><th> 备注                                 </th></tr>
<tr><td> 云服务    </td><td> 创建新的云服务    </td><td>云服务是容器计算资源（例如虚拟机）</td></tr>
<tr><td> 云服务 DNS 名称    </td><td>ubuntu-template.chinacloudapp.cn    </td><td>为计算机提供不可知的负载均衡器名称</td></tr>
<tr><td> 区域/地缘组/虚拟网络 </td><td>    华北    </td><td> 选择你的 Web 应用程序从中访问 Cassandra 群集的区域</td></tr>
<tr><td>存储帐户 </td><td>    使用默认值    </td><td>使用特定区域中的默认存储帐户或预先创建的存储帐户</td></tr>
<tr><td>可用性集 </td><td>    无 </td><td>    将此字段留空</td></tr>
<tr><td>终结点    </td><td>使用默认值 </td><td>    使用默认 SSH 配置 </td></tr>
</table>
<!-- cloudapp.net  to chinacloudapp.cn(Correct) -->

单击右键头，保留屏幕 #3 中的默认设置。 单击&quot;勾选&quot;按钮完成 VM 预配过程。 几分钟后，名为“ubuntu-template”的 VM 应处于“正在运行”状态。

### <a name="install-the-necessary-software"></a>安装必要的软件
#### <a name="step-1-upload-tarballs"></a>步骤 1：上传 tarball
使用 scp 或 pscp，通过以下命令格式将以前下载的软件复制到 ~/downloads 目录：

##### <a name="pscp-server-jre-8u5-linux-x64targz-localadminhk-cas-templatechinacloudappcnhomelocaladmindownloadsserver-jre-8u5-linux-x64targz"></a>pscp server-jre-8u5-linux-x64.tar.gz localadmin@hk-cas-template.chinacloudapp.cn:/home/localadmin/downloads/server-jre-8u5-linux-x64.tar.gz
对 JRE 和 Cassandra 组件重复以上命令。

#### <a name="step-2-prepare-the-directory-structure-and-extract-the-archives"></a>步骤 2：准备目录结构并提取存档
使用以下 bash 脚本以超级用户身份登录到 VM，并创建目录结构并提取软件：

    #!/bin/bash
    CASS_INSTALL_DIR=&quot;/opt/cassandra&quot;
    JRE_INSTALL_DIR=&quot;/opt/java&quot;
    CASS_DATA_DIR=&quot;/var/lib/cassandra&quot;
    CASS_LOG_DIR=&quot;/var/log/cassandra&quot;
    DOWNLOADS_DIR=&quot;~/downloads&quot;
    JRE_TARBALL=&quot;server-jre-8u5-linux-x64.tar.gz&quot;
    CASS_TARBALL=&quot;apache-cassandra-2.0.8-bin.tar.gz&quot;
    SVC_USER=&quot;localadmin&quot;

    RESET_ERROR=1
    MKDIR_ERROR=2

    reset_installation ()
    {
       rm -rf $CASS_INSTALL_DIR 2&gt; /dev/null
       rm -rf $JRE_INSTALL_DIR 2&gt; /dev/null
       rm -rf $CASS_DATA_DIR 2&gt; /dev/null
       rm -rf $CASS_LOG_DIR 2&gt; /dev/null
    }
    make_dir ()
    {
       if [ -z &quot;$1&quot; ]
       then
          echo &quot;make_dir: invalid directory name&quot;
          exit $MKDIR_ERROR
       fi

       if [ -d &quot;$1&quot; ]
       then
          echo &quot;make_dir: directory already exists&quot;
          exit $MKDIR_ERROR
       fi

       mkdir $1 2&gt;/dev/null
       if [ $? != 0 ]
       then
          echo &quot;directory creation failed&quot;
          exit $MKDIR_ERROR
       fi
    }

    unzip()
    {
       if [ $# == 2 ]
       then
          tar xzf $1 -C $2
       else
          echo &quot;archive error&quot;
       fi

    }

    if [ -n &quot;$1&quot; ]
    then
       SVC_USER=$1
    fi

    reset_installation
    make_dir $CASS_INSTALL_DIR
    make_dir $JRE_INSTALL_DIR
    make_dir $CASS_DATA_DIR
    make_dir $CASS_LOG_DIR

    #unzip JRE and Cassandra
    unzip $HOME/downloads/$JRE_TARBALL $JRE_INSTALL_DIR
    unzip $HOME/downloads/$CASS_TARBALL $CASS_INSTALL_DIR

    #Change the ownership to the service credentials

    chown -R $SVC_USER:$GROUP $CASS_DATA_DIR
    chown -R $SVC_USER:$GROUP $CASS_LOG_DIR
    echo &quot;edit /etc/profile to add JRE to the PATH&quot;
    echo &quot;installation is complete&quot;

如果将此脚本粘贴到 vim 窗口中，请确保使用以下命令删除回车符 (&#39;\r&#39;)：

    tr -d &#39;\r&#39; &lt;infile.sh &gt;outfile.sh

#### <a name="step-3-edit-etcprofile"></a>步骤 3：编辑 etc/profile
将以下内容附加到结尾：

    JAVA_HOME=/opt/java/jdk1.8.0_05
    CASS_HOME= /opt/cassandra/apache-cassandra-2.0.8
    PATH=$PATH:$HOME/bin:$JAVA_HOME/bin:$CASS_HOME/bin
    export JAVA_HOME
    export CASS_HOME
    export PATH

#### <a name="step-4-install-jna-for-production-systems"></a>步骤 4：为生产系统安装 JNA
使用以下命令序列：以下命令会将 jna-3.2.7.jar 和 jna-platform-3.2.7.jar 安装到 /usr/share.java 目录 sudo apt-get install libjna-java

在 $CASS_HOME/lib 目录中创建符号链接，以便 Cassandra 启动脚本能够找到这些 jar：

    ln -s /usr/share/java/jna-3.2.7.jar $CASS_HOME/lib/jna.jar

    ln -s /usr/share/java/jna-platform-3.2.7.jar $CASS_HOME/lib/jna-platform.jar

#### <a name="step-5-configure-cassandrayaml"></a>步骤 5：配置 cassandra.yaml
编辑每个 VM 上的 cassandra.yaml，使之能够反映所有虚拟机所需的配置 [在实际预配过程中，我们会调整此配置]：

<table>
<tr><th>字段名称   </th><th> Value  </th><th>    备注 </th></tr>
<tr><td>cluster_name </td><td>    &quot;CustomerService&quot;    </td><td> 使用能够反映你的部署的名称</td></tr>
<tr><td>listen_address    </td><td>[将此字段留空]    </td><td> 删除 &quot;localhost&quot; </td></tr>
<tr><td>rpc_addres   </td><td>[将此字段留空]    </td><td> 删除 &quot;localhost&quot; </td></tr>
<tr><td>seeds    </td><td>&quot;10.1.2.4, 10.1.2.6, 10.1.2.8&quot;    </td><td>所有已指定为种子的 IP 地址的列表。</td></tr>
<tr><td>endpoint_snitch </td><td> org.apache.cassandra.locator.GossipingPropertyFileSnitch </td><td> 此字段由 NetworkTopologyStrateg 用来推断数据中心以及 VM 的机架</td></tr>
</table>

#### <a name="step-6-capture-the-vm-image"></a>步骤 6：捕获 VM 映像
使用以前创建的主机名 (hk-cas-template.chinacloudapp.cn) 和 SSH 私钥登录到虚拟机。 请参阅“如何在 Azure 上通过 Linux 使用 SSH”，以详细了解如何使用命令 ssh 或 putty.exe 登录。

执行以下顺序的操作以捕获映像：

##### <a name="1-deprovision"></a>1.预配
使用命令“sudo waagent -deprovision+user”删除虚拟机实例特定信息。 请参阅[如何捕获将用作模板的 Linux 虚拟机](capture-image-classic.md)，了解映像捕获过程的详细信息。

##### <a name="2-shut-down-the-vm"></a>2:关闭 VM
确保突出显示该虚拟机，并单击底部命令栏中的“关闭”链接。

##### <a name="3-capture-the-image"></a>3:捕获映像
确保突出显示该虚拟机，并单击底部命令栏中的“捕获”链接。 在下一屏幕中，请提供映像名称（例如 hk-cas-2-08-ub-14-04-2014071）、适当的映像说明，然后单击“勾选”标记完成捕获过程。

此过程需要几秒钟时间，映像应会出现在映像库中的“我的映像”部分。 成功捕获映像后，将自动删除源 VM。 

## <a name="single-region-deployment-process"></a>单区域部署过程
**步骤 1：创建虚拟网络**：登录 Azure 门户，再创建虚拟网络（经典），属性如下表所示。 请参阅[使用 Azure 门户创建虚拟网络（经典）](../../../virtual-network/virtual-networks-create-vnet-classic-pportal.md)，了解此过程的详细步骤。      

<table>
<tr><th>VM 属性名称</th><th>Value</th><th>备注</th></tr>
<tr><td>Name</td><td>vnet-cass-north-china</td><td></td></tr>
<tr><td>区域</td><td>中国北部</td><td></td></tr>
<tr><td>DNS 服务器</td><td>无</td><td>将其忽略，因为我们不使用 DNS 服务器</td></tr>
<tr><td>地址空间</td><td>10.1.0.0/16</td><td></td></tr><br/><tr><td>起始 IP</td><td>10.1.0.0</td><td></td></tr><br/><tr><td>CIDR </td><td>/16 (65531)</td><td></td></tr>
</table>

添加以下子网：

<table>
<tr><th>Name</th><th>起始 IP</th><th>CIDR</th><th>备注</th></tr>
<tr><td>Web</td><td>10.1.1.0</td><td>/24 (251)</td><td>Web 场的子网</td></tr>
<tr><td>数据</td><td>10.1.2.0</td><td>/24 (251)</td><td>数据库节点的子网</td></tr>
</table>

可以通过网络安全组保护数据和 Web 子网，此方面的内容不在本文讲述范围之内。  

**步骤 2：预配虚拟机**使用之前创建的映像，可以在云服务器“hk-c-svc-north”中创建以下虚拟机，并将其绑定到相应的子网，如下所示：

<table>
<tr><th>计算机名称    </th><th>子网    </th><th>IP 地址    </th><th>可用性集</th><th>DC/机架</th><th>种子？</th></tr>
<tr><td>hk-c1-north-china    </td><td>数据    </td><td>10.1.2.4    </td><td>hk-c-aset-1    </td><td>dc =NORTHCHINA rack =rack1 </td><td>是</td></tr>
<tr><td>hk-c2-north-china    </td><td>数据    </td><td>10.1.2.5    </td><td>hk-c-aset-1    </td><td>dc =NORTHCHINA rack =rack1    </td><td>否 </td></tr>
<tr><td>hk-c3-north-china    </td><td>数据    </td><td>10.1.2.6    </td><td>hk-c-aset-1    </td><td>dc =NORTHCHINA rack =rack2    </td><td>是</td></tr>
<tr><td>hk-c4-north-china    </td><td>数据    </td><td>10.1.2.7    </td><td>hk-c-aset-1    </td><td>dc =NORTHCHINA rack =rack2    </td><td>否 </td></tr>
<tr><td>hk-c5-north-china    </td><td>数据    </td><td>10.1.2.8    </td><td>hk-c-aset-2    </td><td>dc =NORTHCHINA rack =rack3    </td><td>是</td></tr>
<tr><td>hk-c6-north-china    </td><td>数据    </td><td>10.1.2.9    </td><td>hk-c-aset-2    </td><td>dc =NORTHCHINA rack =rack3    </td><td>否 </td></tr>
<tr><td>hk-c7-north-china    </td><td>数据    </td><td>10.1.2.10    </td><td>hk-c-aset-2    </td><td>dc =NORTHCHINA rack =rack4    </td><td>是</td></tr>
<tr><td>hk-c8-north-china    </td><td>数据    </td><td>10.1.2.11    </td><td>hk-c-aset-2    </td><td>dc =NORTHCHINA rack =rack4    </td><td>否 </td></tr>
<tr><td>hk-w1-north-china    </td><td>Web    </td><td>10.1.1.4    </td><td>hk-w-aset-1    </td><td>                       </td><td>不适用</td></tr>
<tr><td>hk-w2-north-china    </td><td>Web    </td><td>10.1.1.5    </td><td>hk-w-aset-1    </td><td>                       </td><td>不适用</td></tr>
</table>

创建以上 VM 列表需要完成以下过程：

1. 在特定区域创建空的云服务
2. 从之前捕获的映像创建 VM，并将其附加到之前创建的虚拟网络；对所有 VM 重复此过程
3. 将内部负载均衡器添加到云服务，并将其附加到“数据”子网
4. 对于以前创建的每个 VM，可以通过一个已连接到以前创建的内部负载均衡器的负载均衡集添加进行 Thrift 通信的负载均衡终结点

以上过程可以通过 Azure 门户来执行；使用 Windows 计算机（如果无法访问 Windows 计算机，则可使用 Azure 上的 VM）；使用以下 PowerShell 脚本自动预配所有 8 个 VM。

**列表 1：适用于预配虚拟机的 PowerShell 脚本**

        #Tested with Azure Powershell - November 2014
        #This powershell script deployes a number of VMs from an existing image inside an Azure region
        #Import your Azure subscription into the current Powershell session before proceeding
        #The process: 1. create Azure Storage account, 2. create virtual network, 3.create the VM template, 2. create a list of VMs from the template

        #fundamental variables - change these to reflect your subscription
        $country="china"; $region="north"; $vnetName = "your_vnet_name";$storageAccount="your_storage_account"
        $numVMs=8;$prefix = "hk-cass";$ilbIP="your_ilb_ip"
        $subscriptionName = "Azure_subscription_name";
        $vmSize="ExtraSmall"; $imageName="your_linux_image_name"
        $ilbName="ThriftInternalLB"; $thriftEndPoint="ThriftEndPoint"

        #generated variables
        $serviceName = "$prefix-svc-$region-$country"; $azureRegion = "$region $country"

        $vmNames = @()
        for ($i=0; $i -lt $numVMs; $i++)
        {
           $vmNames+=("$prefix-vm"+($i+1) + "-$region-$country" );
        }

        #select an Azure subscription already imported into Powershell session
        Select-AzureSubscription -SubscriptionName $subscriptionName -Current
        Set-AzureSubscription -SubscriptionName $subscriptionName -CurrentStorageAccountName $storageAccount

        #create an empty cloud service
        New-AzureService -ServiceName $serviceName -Label "hkcass$region" -Location $azureRegion
        Write-Host "Created $serviceName"

        $VMList= @()   # stores the list of azure vm configuration objects
        #create the list of VMs
        foreach($vmName in $vmNames)
        {
           $VMList += New-AzureVMConfig -Name $vmName -InstanceSize ExtraSmall -ImageName $imageName |
           Add-AzureProvisioningConfig -Linux -LinuxUser "localadmin" -Password "Local123" |
           Set-AzureSubnet "data"
        }

        New-AzureVM -ServiceName $serviceName -VNetName $vnetName -VMs $VMList

        #Create internal load balancer
        Add-AzureInternalLoadBalancer -ServiceName $serviceName -InternalLoadBalancerName $ilbName -SubnetName "data" -StaticVNetIPAddress "$ilbIP"
        Write-Host "Created $ilbName"
        #Add the thrift endpoint to the internal load balancer for all the VMs
        foreach($vmName in $vmNames)
        {
            Get-AzureVM -ServiceName $serviceName -Name $vmName |
                Add-AzureEndpoint -Name $thriftEndPoint -LBSetName "ThriftLBSet" -Protocol tcp -LocalPort 9160 -PublicPort 9160 -ProbePort 9160 -ProbeProtocol tcp -ProbeIntervalInSeconds 10 -InternalLoadBalancerName $ilbName |
                Update-AzureVM

            Write-Host "created $vmName"     
        }

**步骤 3：在每个 VM 上配置 Cassandra**

登录到 VM 并执行以下操作：

* 编辑 $CASS_HOME/conf/cassandra-rackdc.properties 以指定数据中心和机架属性：

       dc =EASTCHINA, rack =rack1
* 编辑 cassandra.yaml，将种子节点配置如下：

       Seeds: "10.1.2.4,10.1.2.6,10.1.2.8,10.1.2.10"

**步骤 4：启动 VM 并测试群集**

登录到其中一个节点（例如 hk-c1-north-china），然后运行以下命令以查看群集的状态：

       nodetool -h 10.1.2.4 -p 7199 status

对于 8 节点群集，显示内容应如下所示：

<table>
<tr><th>状态</th><th>地址    </th><th>加载    </th><th>令牌    </th><th>拥有 </th><th>主机 ID    </th><th>机架</th></tr>
<tr><th>UN    </td><td>10.1.2.4     </td><td>87.81 KB    </td><td>256    </td><td>38.0%    </td><td>Guid（已删除）</td><td>rack1</td></tr>
<tr><th>UN    </td><td>10.1.2.5     </td><td>41.08 KB    </td><td>256    </td><td>68.9%    </td><td>Guid（已删除）</td><td>rack1</td></tr>
<tr><th>UN    </td><td>10.1.2.6     </td><td>55.29 KB    </td><td>256    </td><td>68.8%    </td><td>Guid（已删除）</td><td>rack2</td></tr>
<tr><th>UN    </td><td>10.1.2.7     </td><td>55.29 KB    </td><td>256    </td><td>68.8%    </td><td>Guid（已删除）</td><td>rack2</td></tr>
<tr><th>UN    </td><td>10.1.2.8     </td><td>55.29 KB    </td><td>256    </td><td>68.8%    </td><td>Guid（已删除）</td><td>rack3</td></tr>
<tr><th>UN    </td><td>10.1.2.9     </td><td>55.29 KB    </td><td>256    </td><td>68.8%    </td><td>Guid（已删除）</td><td>rack3</td></tr>
<tr><th>UN    </td><td>10.1.2.10     </td><td>55.29 KB    </td><td>256    </td><td>68.8%    </td><td>Guid（已删除）</td><td>rack4</td></tr>
<tr><th>UN    </td><td>10.1.2.11     </td><td>55.29 KB    </td><td>256    </td><td>68.8%    </td><td>Guid（已删除）</td><td>rack4</td></tr>
</table>

## <a name="test-the-single-region-cluster"></a>测试单区域群集
使用以下步骤测试群集：

1. 使用 Powershell 命令 Get-AzureInternalLoadbalancer cmdlet 获取内部负载均衡器的 IP 地址（例如 10.1.2.101）。 该命令的语法如下所示：

        Get-AzureLoadbalancer -ServiceName "hk-c-svc-north-china" [displays the details of the internal load balancer along with its IP address]
2. 使用 Putty 或 ssh 登录到 Web 场 VM（例如 hk-w1-north-china）
3. 执行 $CASS_HOME/bin/cqlsh 10.1.2.101 9160
4. 使用以下 CQL 命令验证群集是否正常工作：

        CREATE KEYSPACE customers_ks WITH REPLICATION = { 'class' : 'SimpleStrategy', 'replication_factor' : 3 };
        USE customers_ks;
        CREATE TABLE Customers(customer_id int PRIMARY KEY, firstname text, lastname text);
        INSERT INTO Customers(customer_id, firstname, lastname) VALUES(1, 'John', 'Doe');
        INSERT INTO Customers(customer_id, firstname, lastname) VALUES (2, 'Jane', 'Doe');

        SELECT * FROM Customers;

应显示类似于下面的结果：

<table>
  <tr><th> customer_id </th><th> 名 </th><th> 姓 </th></tr>
  <tr><td> 1 </td><td> John </td><td> Doe </td></tr>
  <tr><td> 2 </td><td> Jane </td><td> Doe </td></tr>
</table>

在步骤 4 中创建的密钥空间使用 SimpleStrategy 并已将 a replication_factor 设置为 3。 建议使用 SimpleStrategy 进行单数据中心部署，使用 NetworkTopologyStrategy 进行多数据中心部署。 将 replication_factor 设置为 3 即可承受节点故障。

<a name="tworegion"></a>
## <a name="multi-region-deployment-process"></a>多区域部署过程
利用完成的单区域部署，并在安装第二个区域时重复相同的过程。 单区域部署和多区域部署的主要区别是 VPN 隧道，设置该隧道是为了进行区域间通信；首先安装网络，并完成 VM 预配和 Cassandra 配置。

### <a name="step-1-create-the-virtual-network-at-the-2nd-region"></a>步骤 1：在第二个区域创建虚拟网络
登录到 Azure 门户，并使用下表中的属性创建虚拟网络。 请参阅[在 Azure 门户中配置只使用云的虚拟网络](../../../virtual-network/virtual-networks-create-vnet-classic-pportal.md)，了解此过程的详细步骤。      

<table>
<tr><th>属性名称    </th><th>Value    </th><th>备注</th></tr>
<tr><td>Name    </td><td>vnet-cass-east-china</td><td></td></tr>
<tr><td>区域    </td><td>中国东部</td><td></td></tr>
<tr><td>DNS 服务器        </td><td></td><td>将其忽略，因为我们不使用 DNS 服务器</td></tr>
<tr><td>配置点到站点 VPN</td><td></td><td>        将其忽略</td></tr>
<tr><td>配置站点到站点 VPN</td><td></td><td>        将其忽略</td></tr>
<tr><td>地址空间    </td><td>10.2.0.0/16</td><td></td></tr>
<tr><td>起始 IP    </td><td>10.2.0.0    </td><td></td></tr>
<tr><td>CIDR    </td><td>/16 (65531)</td><td></td></tr>
</table>

添加以下子网：

<table>
<tr><th>Name    </th><th>起始 IP    </th><th>CIDR    </th><th>备注</th></tr>
<tr><td>Web    </td><td>10.2.1.0    </td><td>/24 (251)    </td><td>Web 场的子网</td></tr>
<tr><td>数据    </td><td>10.2.2.0    </td><td>/24 (251)    </td><td>数据库节点的子网</td></tr>
</table>

### <a name="step-2-create-local-networks"></a>步骤 2：创建本地网络
Azure 虚拟网络中的本地网络是映射到远程站点（包括私有云或其他 Azure 区域）的代理地址空间。 此代理地址空间绑定到远程网关，可以将网络路由到正确的网络目标。 请参阅[配置 VNet 到 VNet 连接](../../../vpn-gateway/virtual-networks-configure-vnet-to-vnet-connection.md)，获取建立 VNET 到 VNET 连接的说明。

按照以下详细信息创建两个本地网络：

| 网络名称 | VPN 网关地址 | 地址空间 | 备注 |
| --- | --- | --- | --- |
| hk-lnet-map-to-east-china |23.1.1.1 |10.2.0.0/16 |创建本地网络时，请提供占位符网关地址。 创建网关后，需要填充实际的网关地址。 请确保地址空间与相应的远程 VNET 完全匹配；在此示例中，该 VNET 在华东区域创建。 |
| hk-lnet-map-to-north-china |23.2.2.2 |10.1.0.0/16 |在创建本地网络时，请提供一个占位符网关地址。 创建网关后，需要填充实际的网关地址。 请确保地址空间与相应的远程 VNET 完全匹配；在此示例中，该 VNET 在华北区域创建。 |

### <a name="step-3-map-local-network-to-the-respective-vnets"></a>步骤 3：将“本地”网络映射到相应的 VNET
在 Azure 门户中，选择每个 VNET，单击“配置”，选中“连接到本地网络”，并按照以下详细信息选择本地网络：

| 虚拟网络 | 本地网络 |
| --- | --- |
| hk-vnet-north-china |hk-lnet-map-to-east-china |
| hk-vnet-east-china |hk-lnet-map-to-north-china |

### <a name="step-4-create-gateways-on-vnet1-and-vnet2"></a>步骤 4：在 VNET1 和 VNET2 上创建网关
在两个虚拟网络的仪表板中，单击“创建网关”，触发 VPN 网关预配过程。 几分钟后，每个虚拟网络的仪表板会显示实际网关地址。

### <a name="step-5-update-local-networks-with-the-respective-gateway-addresses"></a>步骤 5：使用相应的“网关”地址更新“本地”网络
编辑两个本地网络，将占位符网关 IP 地址替换为刚预配的网关的实际 IP 地址。 使用以下映射：

<table>
<tr><th>本地网络    </th><th>虚拟网络网关</th></tr>
<tr><td>hk-lnet-map-to-east-china </td><td>hk-vnet-north-china 的网关</td></tr>
<tr><td>hk-lnet-map-to-north-china </td><td>hk-vnet-east-china 的网关</td></tr>
</table>

### <a name="step-6-update-the-shared-key"></a>步骤 6：更新共享密钥
使用以下 Powershell 脚本更新每个 VPN 网关的 IPSec 密钥 [为两个网关使用安全的密钥]：

    Set-AzureVNetGatewayKey -VNetName hk-vnet-east-china -LocalNetworkSiteName hk-lnet-map-to-north-china -SharedKey D9E76BKK
    Set-AzureVNetGatewayKey -VNetName hk-vnet-north-china -LocalNetworkSiteName hk-lnet-map-to-east-china -SharedKey D9E76BKK

### <a name="step-7-establish-the-vnet-to-vnet-connection"></a>步骤 7：建立 VNET 到 VNET 连接
在 Azure 门户中，使用这两个虚拟网络的“仪表板”菜单建立网关到网关连接。 使用底部工具栏中的“连接”菜单项。 几分钟后，仪表板会以图形方式显示连接详细信息。

### <a name="step-8-create-the-virtual-machines-in-region-2"></a>步骤 8：在区域 #2 中创建虚拟机
按照相同步骤创建区域 #1 部署中描述的 Ubuntu 映像，或者将映像 VHD 文件复制到区域 #2 中的 Azure 存储帐户，然后创建该映像。 使用该映像，将下列虚拟机创建到新的云服务 hk-c-svc-east-china 中：

| 计算机名称 | 子网 | IP 地址 | 可用性集 | DC/机架 | 种子？ |
| --- | --- | --- | --- | --- | --- |
| hk-c1-east-china |数据 |10.2.2.4 |hk-c-aset-1 |dc =EASTCHINA rack =rack1 |是 |
| hk-c2-east-china |数据 |10.2.2.5 |hk-c-aset-1 |dc =EASTCHINA rack =rack1 |否 |
| hk-c3-east-china |数据 |10.2.2.6 |hk-c-aset-1 |dc =EASTCHINA rack =rack2 |是 |
| hk-c5-east-china |数据 |10.2.2.8 |hk-c-aset-2 |dc =EASTCHINA rack =rack3 |是 |
| hk-c6-east-china |数据 |10.2.2.9 |hk-c-aset-2 |dc =EASTCHINA rack =rack3 |否 |
| hk-c7-east-china |数据 |10.2.2.10 |hk-c-aset-2 |dc =EASTCHINA rack =rack4 |是 |
| hk-c8-east-china |数据 |10.2.2.11 |hk-c-aset-2 |dc =EASTCHINA rack =rack4 |否 |
| hk-w1-east-china |Web |10.2.1.4 |hk-w-aset-1 |不适用 |不适用 |
| hk-w2-east-china |Web |10.2.1.5 |hk-w-aset-1 |不适用 |不适用 |

遵循与区域 #1 相同的说明，但使用 10.2.xxx.xxx 地址空间。

### <a name="step-9-configure-cassandra-on-each-vm"></a>步骤 9：在每个 VM 上配置 Cassandra
登录到 VM 并执行以下操作：

1. 编辑 $CASS_HOME/conf/cassandra-rackdc.properties 以指定下述格式的数据中心和机架属性：dc =EASTCHINA rack =rack1
2. 编辑 cassandra.yaml 以配置种子节点：    种子："10.1.2.4,10.1.2.6,10.1.2.8,10.1.2.10,10.2.2.4,10.2.2.6,10.2.2.8,10.2.2.10"

### <a name="step-10-start-cassandra"></a>步骤 10：启动 Cassandra
登录到每个 VM，然后通过运行以下命令在后台启动 Cassandra：$CASS_HOME/bin/cassandra

## <a name="test-the-multi-region-cluster"></a>测试多区域群集
到目前为止，Cassandra 已部署到 16 个节点，每个 Azure 区域 8 个节点。 这些节点具有通用群集名称和种子节点配置，因此属于同一群集。 使用以下过程测试群集：

### <a name="step-1-get-the-internal-load-balancer-ip-for-both-the-regions-using-powershell"></a>步骤 1：使用 PowerShell 获取这两个区域的内部负载均衡器 IP
* Get-AzureInternalLoadbalancer -ServiceName "hk-c-svc-north-china"
* Get-AzureInternalLoadbalancer -ServiceName "hk-c-svc-east-china"  

    请注意显示的 IP 地址（例如 north - 10.1.2.101, east - 10.2.2.101）。

### <a name="step-2-execute-the-following-in-the-north-region-after-logging-into-hk-w1-china-north"></a>步骤 2：登录到 hk-w1-china-north 后，在 north 区域执行以下命令：
1. 执行 $CASS_HOME/bin/cqlsh 10.1.2.101 9160
2. 执行以下 CQL 命令：

        CREATE KEYSPACE customers_ks
        WITH REPLICATION = { 'class' : 'NetworkToplogyStrategy', 'NORTHCHINA' : 3, 'EASTCHINA' : 3};
        USE customers_ks;
        CREATE TABLE Customers(customer_id int PRIMARY KEY, firstname text, lastname text);
        INSERT INTO Customers(customer_id, firstname, lastname) VALUES(1, 'John', 'Doe');
        INSERT INTO Customers(customer_id, firstname, lastname) VALUES (2, 'Jane', 'Doe');
        SELECT * FROM Customers;

显示内容应如下所示：

| customer_id | 名 | 姓 |
| --- | --- | --- |
| 1 |John |Doe |
| 2 |Jane |Doe |

### <a name="step-3-execute-the-following-in-the-east-region-after-logging-into-hk-w1-east-china"></a>步骤 3：登录到 hk-w1-east-china 后，在东部区域执行以下命令：
1. 执行 $CASS_HOME/bin/cqlsh 10.2.2.101 9160
2. 执行以下 CQL 命令：

        USE customers_ks;
        CREATE TABLE Customers(customer_id int PRIMARY KEY, firstname text, lastname text);
        INSERT INTO Customers(customer_id, firstname, lastname) VALUES(1, 'John', 'Doe');
        INSERT INTO Customers(customer_id, firstname, lastname) VALUES (2, 'Jane', 'Doe');
        SELECT * FROM Customers;

显示内容与西部区域的显示内容应相同：

| customer_id | 名 | 姓 |
| --- | --- | --- |
| 1 |John |Doe |
| 2 |Jane |Doe |

执行一些插入操作，将这些插入内容复制到群集的 china-north 部分。

## <a name="test-cassandra-cluster-from-nodejs"></a>从 Node.js 测试 Cassandra 群集
使用以前在“Web”层创建的某个 Linux VM，可以执行一个简单的 Node.js 脚本，以便读取以前插入的数据

**步骤 1：安装 Node.js 和 Cassandra 客户端**

1. 安装 Node.js 和 npm
2. 使用 npm 安装节点包“cassandra-client”
3. 在显示检索数据的 json 字符串的 shell 提示符下，执行以下脚本：

        var pooledCon = require('cassandra-client').PooledConnection;
        var ksName = "custsupport_ks";
        var cfName = "customers_cf";
        var hostList = ['internal_loadbalancer_ip:9160'];
        var ksConOptions = { hosts: hostList,
                             keyspace: ksName, use_bigints: false };

        function createKeyspace(callback){
           var cql = 'CREATE KEYSPACE ' + ksName + ' WITH strategy_class=SimpleStrategy AND strategy_options:replication_factor=1';
           var sysConOptions = { hosts: hostList,  
                                 keyspace: 'system', use_bigints: false };
           var con = new pooledCon(sysConOptions);
           con.execute(cql,[],function(err) {
           if (err) {
             console.log("Failed to create Keyspace: " + ksName);
             console.log(err);
           }
           else {
             console.log("Created Keyspace: " + ksName);
             callback(ksConOptions, populateCustomerData);
           }
           });
           con.shutdown();
        }

        function createColumnFamily(ksConOptions, callback){
          var params = ['customers_cf','custid','varint','custname',
                        'text','custaddress','text'];
          var cql = 'CREATE COLUMNFAMILY ? (? ? PRIMARY KEY,? ?, ? ?)';
        var con =  new pooledCon(ksConOptions);
          con.execute(cql,params,function(err) {
              if (err) {
                 console.log("Failed to create column family: " + params[0]);
                 console.log(err);
              }
              else {
                 console.log("Created column family: " + params[0]);
                 callback();
              }
          });
          con.shutdown();
        }

        //populate Data
        function populateCustomerData() {
           var params = ['John','Infinity Dr, TX', 1];
           updateCustomer(ksConOptions,params);

           params = ['Tom','Fermat Ln, WA', 2];
           updateCustomer(ksConOptions,params);
        }

        //update also inserts the record if none exists
        function updateCustomer(ksConOptions,params)
        {
          var cql = 'UPDATE customers_cf SET custname=?,custaddress=? where custid=?';
          var con = new pooledCon(ksConOptions);
          con.execute(cql,params,function(err) {
              if (err) console.log(err);
              else console.log("Inserted customer : " + params[0]);
          });
          con.shutdown();
        }

        //read the two rows inserted above
        function readCustomer(ksConOptions)
        {
          var cql = 'SELECT * FROM customers_cf WHERE custid IN (1,2)';
          var con = new pooledCon(ksConOptions);
          con.execute(cql,[],function(err,rows) {
              if (err)
                 console.log(err);
              else
                 for (var i=0; i<rows.length; i++)
                    console.log(JSON.stringify(rows[i]));
            });
           con.shutdown();
        }

        //exectue the code
        createKeyspace(createColumnFamily);
        readCustomer(ksConOptions)

## <a name="conclusion"></a>结论
Azure 是一个灵活的平台，可运行 Azure 软件和开源软件，如本练习所演示。 将群集节点分散到多个容错域，可在单个数据中心部署高度可用的 Cassandra 群集。 也可以将 Cassandra 群集部署到多个地理距离遥远的 Azure 区域，以便建立防灾系统。 使用 Azure 和 Cassandra 可构建高度可扩展、高度可用且灾难恢复性强的云服务，满足当今 Internet 规模服务需求。  

## <a name="references"></a>参考
* [http://cassandra.apache.org](http://cassandra.apache.org)
* [http://www.datastax.com](http://www.datastax.com)
* [http://www.nodejs.org](http://www.nodejs.org)

<!--Update_Description: wording update, update link -->