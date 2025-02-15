---
title: 使用虚拟网络扩展 HDInsight
description: 了解如何使用 Azure 虚拟网络将 HDInsight 连接到其他云资源或数据中心内的资源
services: hdinsight
author: hrasheed-msft
ms.assetid: 37b9b600-d7f8-4cb1-a04a-0b3a827c6dcc
ms.service: hdinsight
ms.custom: hdinsightactive
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: big-data
origin.date: 06/17/2019
ms.date: 07/22/2019
ms.author: v-yiso
ms.openlocfilehash: 65b1e76c9a8181c87c2dfea4c3f7c02949127052
ms.sourcegitcommit: f4351979a313ac7b5700deab684d1153ae51d725
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/12/2019
ms.locfileid: "67845135"
---
# <a name="extend-azure-hdinsight-using-an-azure-virtual-network"></a>使用 Azure 虚拟网络扩展 Azure HDInsight

了解如何将 HDInsight 与 [Azure 虚拟网络](../virtual-network/virtual-networks-overview.md)配合使用。 使用 Azure 虚拟网络可以实现以下方案：

* 直接从本地网络连接到 HDInsight。

* 在 Azure 虚拟网络中将 HDInsight 连接到数据存储。

* 直接访问无法通过 Internet 公开访问的 [Apache Hadoop](https://hadoop.apache.org/) 服务。 例如，[Apache Kafka](https://kafka.apache.org/) API 或 [Apache HBase](https://hbase.apache.org/) Java API。

> [!IMPORTANT]  
> 2019 年 2 月 28 日以后，在 VNET 中创建的新群集的网络资源（例如 NIC、LB 等）会在同一 HDInsight 群集资源组中进行预配。 以前，这些资源在 VNET 资源组中预配。 当前运行的群集以及那些在没有 VNET 的情况下创建的群集没有任何更改。

## <a name="prerequisites-for-code-samples-and-examples"></a>代码示例和示例的先决条件

* 了解 TCP/IP 网络。 如果不熟悉 TCP/IP 网络，则应在修改生产网络之前，与熟悉 TCP/IP 网络的人合作。
* 如果使用 PowerShell，你将需要 [AZ 模块](https://docs.microsoft.com/powershell/azure/overview)。
* 若要使用 Azure CLI 但尚未安装它，请参阅[安装 Azure CLI](/cli/install-azure-cli)。

> [!IMPORTANT]
> 如果正在查找有关如何使用 Azure 虚拟网络将 HDInsight 连接到本地网络的分步指南，请参阅[将 HDInsight 连接到本地网络](connect-on-premises-network.md)文档。

## <a name="planning"></a>规划

下面是计划在虚拟网络中安装 HDInsight 时必须回答的问题：

* 是否需要将 HDInsight 安装到现有的虚拟网络？ 或者，你是否在创建新的网络？

    如果使用现有的虚拟网络，则可能需要修改网络配置，然后才能安装 HDInsight。 有关详细信息，请参阅[将 HDInsight 添加到现有的虚拟网络](#existingvnet)部分。

* 是否需要将包含 HDInsight 的虚拟网络连接到另一个虚拟网络或本地网络？

    若要轻松地跨网络使用资源，可能需要创建自定义 DNS 并配置 DNS 转发。 有关详细信息，请参阅[连接多个网络](#multinet)部分。

* 是否需要限制/重定向 HDInsight 的入站或出站流量？

    HDInsight 必须与 Azure 数据中心的特定 IP 地址进行不受限制的通信。 此外还必须通过防火墙启用数个端口，以便进行客户端通信。 有关详细信息，请参阅[控制网络流量](#networktraffic)部分。

## <a id="existingvnet"></a>将 HDInsight 添加到现有的虚拟网络

使用本部分中的步骤，了解如何将 HDInsight 添加到现有 Azure 虚拟网络。

> [!NOTE]
> 无法将现有 HDInsight 群集添加到虚拟网络中。

1. 是否对虚拟网络使用经典或 Resource Manager 部署模型？

    HDInsight 3.4 及更高版本需要 Resource Manager 虚拟网络。 早期版本的 HDInsight 要求使用经典虚拟网络。

    如果现有网络是经典虚拟网络，则必须创建 Resource Manager 虚拟网络，然后将这两种网络连接起来。 [将经典 VNet 连接到新的 VNet](../vpn-gateway/vpn-gateway-connect-different-deployment-models-portal.md)。

    连接后，安装在 Resource Manager 网络中的 HDInsight 即可与经典网络中的资源交互。

2. 是否使用强制隧道？ 强制隧道是一种子网设置，可以强制出站 Internet 流量流向某个设备，以便进行检查和日志记录。 HDInsight 不支持强制隧道。 请在将 HDInsight 部署到现有子网之前删除强制隧道，或者为 HDInsight 创建没有强制隧道的新子网。

3. 是否使用网络安全组、用户定义路由或虚拟网络设备来限制进出虚拟网络的流量？

    作为托管服务，HDInsight 要求对 Azure 数据中心的多个 IP 地址进行不受限制的访问。 请更新任何现有的网络安全组或用户定义路由，以便与这些 IP 地址通信。

    HDInsight 托管多个服务，这些服务使用不同的端口。 不要阻止流向这些端口的流量。 有关虚拟设备防火墙的允许端口列表，请参阅“安全”一节。
    
    若要查找现有的安全配置，请使用以下 Azure PowerShell 或 Azure CLI 命令：

    * 网络安全组

        将 `RESOURCEGROUP` 替换为包含虚拟网络的资源组的名称，然后输入命令：
    
        ```powershell
        Get-AzNetworkSecurityGroup -ResourceGroupName  "RESOURCEGROUP"
        ```

        ```azurecli
        az network nsg list --resource-group RESOURCEGROUP
        ```

        有关详细信息，请参阅[排查网络安全组问题](../virtual-network/diagnose-network-traffic-filter-problem.md)文档。

        > [!IMPORTANT]
        > 网络安全组规则按规则优先级顺序应用。 将应用与流量模式匹配的第一条规则，而不应用该流量的其他规则。 权限级别从最高到最低排列的顺序规则。 有关详细信息，请参阅[使用网络安全组筛选网络流量](../virtual-network/security-overview.md)文档。

    * 用户定义路由

        将 `RESOURCEGROUP` 替换为包含虚拟网络的资源组的名称，然后输入命令：

        ```powershell
        Get-AzRouteTable -ResourceGroupName "RESOURCEGROUP"
        ```

        ```azurecli
        az network route-table list --resource-group RESOURCEGROUP
        ```

        有关详细信息，请参阅[排查路由问题](../virtual-network/diagnose-network-routing-problem.md)文档。

4. 创建一个 HDInsight 群集，并在配置过程中选择 Azure 虚拟网络。 使用以下文档中的步骤了解群集创建过程：

    * [Create HDInsight using the Azure portal](hdinsight-hadoop-create-linux-clusters-portal.md)（使用 Azure 门户创建 HDInsight）
    * [Create HDInsight using Azure PowerShell（使用 Azure PowerShell 创建 HDInsight）](hdinsight-hadoop-create-linux-clusters-azure-powershell.md)
    * [使用 Azure 经典 CLI 创建 HDInsight](hdinsight-hadoop-create-linux-clusters-azure-cli.md)
    * [使用 Azure 资源管理器模板创建 HDInsight](hdinsight-hadoop-create-linux-clusters-arm-templates.md)

   > [!IMPORTANT]
   > 向虚拟网络添加 HDInsight 是一项可选的配置步骤。 请确保在配置群集时选择虚拟网络。

## <a id="multinet"></a>连接多个网络

多网络配置的最大难题是在网络之间进行名称解析。

Azure 为安装在虚拟网络中的 Azure 服务提供名称解析。 此内置的名称解析功能允许 HDInsight 使用完全限定的域名 (FQDN) 连接到以下资源：

* 在 Internet 上提供的任何资源。 例如，microsoft.com、windowsupdate.com。

* 同一 Azure 虚拟网络中能够使用资源的内部 DNS 名称  连接的的任何资源。 例如，在使用默认的名称解析时，下面是分配给 HDInsight 工作节点的内部 DNS 名称示例：

  * wn0-hdinsi.0owcbllr5hze3hxdja3mqlrhhe.ex.internal.chinacloudapp.cn
  * wn2-hdinsi.0owcbllr5hze3hxdja3mqlrhhe.ex.internal.chinacloudapp.cn

    这两个节点均可使用内部 DNS 名称直接相互通信，以及与 HDInsight 中的其他节点通信。

 默认名称解析不允许 HDInsight 解析连接到虚拟网络的网络中的资源的名称。 例如，将本地网络加入虚拟网络很常见。 仅使用默认的名称解析时，HDInsight 不能通过名称访问本地网络中的资源。 反过来也是这样，本地网络中的资源不能通过名称访问虚拟网络中的资源。

> [!WARNING]
> 必须在创建 HDInsight 群集之前，先创建自定义 DNS 服务器并将虚拟网络配置为使用该服务器。

若要在虚拟网络和已连接网络中的资源之间启用名称解析，必须执行以下操作：

1. 在计划安装 HDInsight 的 Azure 虚拟网络中创建自定义 DNS 服务器。

2. 将虚拟网络配置为使用自定义 DNS 服务器。

3. 为虚拟网络找出 Azure 分配的 DNS 后缀。 该值类似于 `0owcbllr5hze3hxdja3mqlrhhe.ex.internal.chinacloudapp.cn`。 有关查找 DNS 后缀的信息，请参阅[示例：自定义 DNS](#example-dns) 部分。

4. 配置 DNS 服务器之间的转发。 此配置取决于远程网络的类型。

   * 如果远程网络为本地网络，请将 DNS 配置如下：

     *  自定义 DNS（位于虚拟网络中）：

         * 将虚拟网络 DNS 后缀的请求转发到 Azure 递归解析程序 (168.63.129.16)。 Azure 处理虚拟网络中资源的请求

         * 将其他所有请求转发到本地 DNS 服务器。 本地 DNS 处理所有其他的名称解析请求，甚至包括 Internet 资源（例如 Microsoft.com）的请求。

     * __本地 DNS__：将虚拟网络 DNS 后缀的请求转发到自定义 DNS 服务器。 然后，自定义 DNS 服务器转发给 Azure 递归解析程序。

       此配置将包含虚拟网络 DNS 后缀的完全限定的域名请求路由至自定义 DNS 服务器。 所有其他请求（甚至包括对公共 Internet 地址的请求）由本地 DNS 服务器处理。

   * 如果远程网络为另一 Azure 虚拟网络，请将 DNS 配置如下：

     *  自定义 DNS（位于每个虚拟网络中）：

         * 对虚拟网络 DNS 后缀的请求将转发到自定义 DNS 服务器。 每个虚拟网络中的 DNS 负责解析其网络中的资源。

         * 将所有其他请求转发到 Azure 递归解析程序。 递归解析程序负责解析本地资源和 Internet 资源。

       每个网络的 DNS 服务器根据 DNS 后缀将请求转发到其他服务器。 其他请求使用 Azure 递归解析程序进行解析。

     有关每个配置的示例，请参阅[示例：自定义 DNS](#example-dns) 部分。

有关详细信息，请参阅 [VM 和角色实例的名称解析](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md)文档。

## <a name="directly-connect-to-apache-hadoop-services"></a>直接连接到 Apache Hadoop 服务

可以通过 https://CLUSTERNAME.azurehdinsight.cn 连接到该群集。 此地址使用公共 IP，如果已使用 NSG 来限制来自 Internet 的传入流量，则可能无法访问此地址。 此外，在 VNet 中部署群集时，可以使用专用终结点 https://CLUSTERNAME-int.azurehdinsight.cn 访问它。 此终结点可解析为 VNet 中的专用 IP，以进行群集访问。

若要通过虚拟网络连接到 Apache Ambari 以及其他网页，请使用以下步骤：

1. 若要发现 HDInsight 群集节点的内部完全限定的域名 (FQDN)，请使用以下方法之一：

    将 `RESOURCEGROUP` 替换为包含虚拟网络的资源组的名称，然后输入命令：

    ```powershell
    $clusterNICs = Get-AzNetworkInterface -ResourceGroupName "RESOURCEGROUP" | where-object {$_.Name -like "*node*"}

    $nodes = @()
    foreach($nic in $clusterNICs) {
        $node = new-object System.Object
        $node | add-member -MemberType NoteProperty -name "Type" -value $nic.Name.Split('-')[1]
        $node | add-member -MemberType NoteProperty -name "InternalIP" -value $nic.IpConfigurations.PrivateIpAddress
        $node | add-member -MemberType NoteProperty -name "InternalFQDN" -value $nic.DnsSettings.InternalFqdn
        $nodes += $node
    }
    $nodes | sort-object Type
    ```

    ```azurecli
    az network nic list --resource-group RESOURCEGROUP --output table --query "[?contains(name,'node')].{NICname:name,InternalIP:ipConfigurations[0].privateIpAddress,InternalFQDN:dnsSettings.internalFqdn}"
    ```

    在返回的节点列表中，查找头节点的 FQDN，并使用这些 FQDN 连接到 Ambari 和其他 Web 服务。 例如，使用 `http://<headnode-fqdn>:8080` 访问 Ambari。

    > [!IMPORTANT]
    > 在头节点上托管的一些服务一次只能在一个节点上处于活动状态。 如果尝试在一个头节点上访问服务并且它返回 404 错误，请切换到其他头节点。

2. 若要确定服务可用的节点和端口，请参阅 [HDInsight 的 Hadoop 服务所用的端口](./hdinsight-hadoop-port-settings-for-services.md)一文。

## <a id="networktraffic"></a> 控制网络流量

### <a name="techniques-for-controlling-inbound-and-outbound-traffic-to-hdinsight-clusters"></a>控制 HDInsight 群集的入站和出站流量的技术

可以使用以下方法控制 Azure 虚拟网络中的网络流量：

*  网络安全组 (NSG)：用于筛选网络的入站和出站流量。 有关详细信息，请参阅[使用网络安全组筛选网络流量](../virtual-network/security-overview.md)文档。

* **网络虚拟设备** (NVA) 只能用于出站流量。 NVA 可复制设备（如防火墙和路由器）的功能。  有关详细信息，请参阅[网络设备](https://azure.microsoft.com/solutions/network-appliances)文档。

作为托管服务，HDInsight 需要对 HDInsight 运行状况和管理服务具有不受限制的访问权限，以处理从 VNET 传入和传出的流量。 使用 NSG 时，必须确保这些服务仍然可以与 HDInsight 群集进行通信。

![在 Azure 自定义 VNET 中创建的 HDInsight 实体示意图](./media/hdinsight-virtual-network-architecture/vnet-diagram.png)

### <a name="hdinsight-with-network-security-groups"></a>使用网络安全组的 HDInsight

如果计划使用**网络安全组**来控制网络流量，请在安装 HDInsight 之前执行以下操作：

1. 确定计划用于 HDInsight 的 Azure 区域。

2. 确定 HDInsight 所需的 IP 地址。 有关详细信息，请参阅 [HDInsight 所需的 IP 地址](#hdinsight-ip)部分。

3. 为计划将 HDInsight 安装到其中的子网创建或修改网络安全组。

    *  网络安全组：在端口 443 上允许来自 IP 地址的入站流量。   这将确保 HDInsight 管理服务可以从虚拟网络外部访问群集。

有关网络安全组的详细信息，请参阅[网络安全组概述](../virtual-network/security-overview.md)。

### <a name="controlling-outbound-traffic-from-hdinsight-clusters"></a>控制 HDInsight 群集的出站流量

有关控制 HDInsight 群集的出站流量的详细信息，请参阅[配置 Azure HDInsight 群集的出站网络流量限制](hdinsight-restrict-outbound-traffic.md)。

#### <a name="forced-tunneling-to-on-premise"></a>到本地的强制隧道

强制隧道是用户定义的路由配置，用于将子网中的所有流量强制流向特定的网络或位置，例如本地网络。 HDInsight 不  支持将流量通过强制隧道传输到本地网络。 

## <a id="hdinsight-ip"></a> 所需 IP 地址

如果使用网络安全组或用户定义的路由来控制流量，则必须允许来自 Azure 运行状况和管理服务的 IP 地址的流量，以便这些服务可与 HDInsight 群集进行通信。 一些 IP 地址特定于区域，而一些适用于所有的 Azure 区域。 如果使用的不是自定义 DNS，则可能还需要允许来自 Azure DNS 服务的流量。 还必须允许在子网内的 VM 之间传输流量。 使用以下步骤来查找必须允许的 IP 地址：

> [!Note]  
> 如果不使用网络安全组或用户定义的路由来控制流量，则可以忽略本部分。

1. 如果使用的是 Azure 提供的 DNS 服务，则允许通过端口 53 从 __168.63.129.16__ 进行访问。 有关详细信息，请参阅 [VM 和角色实例的名称解析](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md)文档。 如果使用的是自定义 DNS，请跳过此步骤。

2. 对于适用于所有 Azure 区域的 Azure 运行状况和管理服务，允许来自其以下 IP 地址的流量：

    | 源 IP 地址 | 目标  | 方向 |
    | ---- | ----- | ----- |
    | 168.61.49.99 | \*:443 | 入站 |
    | 23.99.5.239 | \*:443 | 入站 |
    | 168.61.48.131 | \*:443 | 入站 |
    | 138.91.141.162 | \*:443 | 入站 |

3. 对于位于资源所在特定区域中的 Azure 运行状况和管理服务，允许来自其下列 IP 地址的流量：

    > [!IMPORTANT]
    > 如果未列出所用的 Azure 区域，则仅使用步骤 1 中所列的四个 IP 地址。

    | 国家/地区 | 区域 | 允许的源 IP 地址 | 允许的目标 | 方向 |
    | ---- | ---- | ---- | ---- | ----- |
    | 中国 | 中国北部 | 42.159.96.170</br>139.217.2.219</br></br>42.159.198.178</br>42.159.234.157 | 443 | 入站 |
    | &nbsp; | 中国东部 | 42.159.198.178</br>42.159.234.157</br></br>42.159.96.170</br>139.217.2.219 | 443 | 入站 |
    | &nbsp; | 中国北部 2 | 40.73.37.141</br>40.73.38.172 | 443 | 入站 |

    若要获取用于 Azure 政府版的 IP 地址的信息，请参阅 [Azure 政府智能 + 分析](https://docs.microsoft.com/azure/azure-government/documentation-government-services-intelligenceandanalytics)文档。

有关详细信息，请参阅[控制网络流量](#networktraffic)部分。

如果使用用户定义的路由 (UDR)，则应当指定一个路由并允许来自 VNET 的出站流量到达下一跃点设置为“Internet”的上述 IP。
    
## <a id="hdinsight-ports"></a> 所需端口

如果计划使用**防火墙**并在特定端口上从外部访问群集，则需要允许你的方案所需的那些端口上的流量。 默认情况下，只要允许上一部分中介绍的 Azure 管理流量在端口 443 上到达群集，则不需要特地将端口列入白名单。

对于特定服务的端口列表，请参阅 [HDInsight 上的 Apache Hadoop 服务所用的端口](hdinsight-hadoop-port-settings-for-services.md)文档。

有关虚拟设备防火墙规则的详细信息，请参阅[虚拟设备方案](../virtual-network/virtual-network-scenario-udr-gw-nva.md)文档。

## <a id="hdinsight-nsg"></a>示例：网络安全组与 HDInsight

本部分的示例演示如何创建允许 HDInsight 与 Azure 管理服务通信的网络安全组规则。 在使用示例之前，请调整 IP 地址，使之与所用 Azure 区域的 IP 地址匹配。 有关此方面的信息，可参阅 [HDInsight 与网络安全组和用户定义路由](#hdinsight-ip)部分。

### <a name="azure-resource-management-template"></a>Azure 资源管理模板

以下资源管理模板创建一个虚拟网络，该网络限制入站流量，但允许来自 HDInsight 所需的 IP 地址的流量。 该模板还在虚拟网络中创建 HDInsight 群集。

* [部署安全的 Azure 虚拟网络和 HDInsight Hadoop 群集](https://azure.microsoft.com/resources/templates/101-hdinsight-secure-vnet/)

### <a name="azure-powershell"></a>Azure PowerShell

使用以下 PowerShell 脚本创建可限制入站流量的虚拟网络，但允许来自中国北部区域的 IP 地址的流量。

> [!IMPORTANT]  
> 更改此示例中的 `hdirule1` 和 `hdirule2` 的 IP 地址，使之与要使用的 Azure 区域匹配。 有关此方面的信息，可参阅 [HDInsight 与网络安全组和用户定义路由](#hdinsight-ip)部分。

```powershell
$vnetName = "Replace with your virtual network name"
$resourceGroupName = "Replace with the resource group the virtual network is in"
$subnetName = "Replace with the name of the subnet that you plan to use for HDInsight"
# Get the Virtual Network object
$vnet = Get-AzVirtualNetwork `
    -Name $vnetName `
    -ResourceGroupName $resourceGroupName
# Get the region the Virtual network is in.
$location = $vnet.Location
# Get the subnet object
$subnet = $vnet.Subnets | Where-Object Name -eq $subnetName
# Create a Network Security Group.
# And add exemptions for the HDInsight health and management services.
$nsg = New-AzNetworkSecurityGroup `
    -Name "hdisecure" `
    -ResourceGroupName $resourceGroupName `
    -Location $location `
    | Add-AzNetworkSecurityRuleConfig `
        -name "hdirule1" `
        -Description "HDI health and management address 52.164.210.96" `
        -Protocol "*" `
        -SourcePortRange "*" `
        -DestinationPortRange "443" `
        -SourceAddressPrefix "52.164.210.96" `
        -DestinationAddressPrefix "VirtualNetwork" `
        -Access Allow `
        -Priority 300 `
        -Direction Inbound `
    | Add-AzNetworkSecurityRuleConfig `
        -Name "hdirule2" `
        -Description "HDI health and management 13.74.153.132" `
        -Protocol "*" `
        -SourcePortRange "*" `
        -DestinationPortRange "443" `
        -SourceAddressPrefix "13.74.153.132" `
        -DestinationAddressPrefix "VirtualNetwork" `
        -Access Allow `
        -Priority 301 `
        -Direction Inbound `
    | Add-AzNetworkSecurityRuleConfig `
        -Name "hdirule3" `
        -Description "HDI health and management 168.61.49.99" `
        -Protocol "*" `
        -SourcePortRange "*" `
        -DestinationPortRange "443" `
        -SourceAddressPrefix "168.61.49.99" `
        -DestinationAddressPrefix "VirtualNetwork" `
        -Access Allow `
        -Priority 302 `
        -Direction Inbound `
    | Add-AzNetworkSecurityRuleConfig `
        -Name "hdirule4" `
        -Description "HDI health and management 23.99.5.239" `
        -Protocol "*" `
        -SourcePortRange "*" `
        -DestinationPortRange "443" `
        -SourceAddressPrefix "23.99.5.239" `
        -DestinationAddressPrefix "VirtualNetwork" `
        -Access Allow `
        -Priority 303 `
        -Direction Inbound `
    | Add-AzNetworkSecurityRuleConfig `
        -Name "hdirule5" `
        -Description "HDI health and management 168.61.48.131" `
        -Protocol "*" `
        -SourcePortRange "*" `
        -DestinationPortRange "443" `
        -SourceAddressPrefix "168.61.48.131" `
        -DestinationAddressPrefix "VirtualNetwork" `
        -Access Allow `
        -Priority 304 `
        -Direction Inbound `
    | Add-AzNetworkSecurityRuleConfig `
        -Name "hdirule6" `
        -Description "HDI health and management 138.91.141.162" `
        -Protocol "*" `
        -SourcePortRange "*" `
        -DestinationPortRange "443" `
        -SourceAddressPrefix "138.91.141.162" `
        -DestinationAddressPrefix "VirtualNetwork" `
        -Access Allow `
        -Priority 305 `
        -Direction Inbound `
# Set the changes to the security group
Set-AzNetworkSecurityGroup -NetworkSecurityGroup $nsg
# Apply the NSG to the subnet
Set-AzVirtualNetworkSubnetConfig `
    -VirtualNetwork $vnet `
    -Name $subnetName `
    -AddressPrefix $subnet.AddressPrefix `
    -NetworkSecurityGroup $nsg
$vnet | Set-AzVirtualNetwork
```

此示例演示如何添加规则，以便在所需的 IP 地址上允许入站流量。 它不包含限制从其他源进行入站访问的规则。 以下代码演示如何允许来自 Internet 的 SSH 访问：

```powershell
Get-AzNetworkSecurityGroup -Name hdisecure -ResourceGroupName RESOURCEGROUP |
Add-AzNetworkSecurityRuleConfig -Name "SSH" -Description "SSH" -Protocol "*" -SourcePortRange "*" -DestinationPortRange "22" -SourceAddressPrefix "*" -DestinationAddressPrefix "VirtualNetwork" -Access Allow -Priority 306 -Direction Inbound
```

### <a name="azure-cli"></a>Azure CLI

使用以下步骤创建一个虚拟网络，该网络限制入站流量，但允许来自 HDInsight 所需的 IP 地址的流量。

1. 使用以下命令创建名为 `hdisecure` 的新网络安全组。 将 `RESOURCEGROUP` 替换为包含 Azure 虚拟网络的资源组。 将 `LOCATION` 替换为组创建在的位置（区域）。

    ```azurecli
    az network nsg create -g RESOURCEGROUP -n hdisecure -l LOCATION
    ```

    在创建组后，会收到有关新组的信息。

2. 使用以下命令将规则添加新网络安全组，以允许从 Azure HDInsight 运行状况和管理服务通过端口 443 发起的入站通信。 将 `RESOURCEGROUP` 替换为包含 Azure 虚拟网络的资源组的名称。

    > [!IMPORTANT]  
    > 更改此示例中的 `hdirule1` 和 `hdirule2` 的 IP 地址，使之与要使用的 Azure 区域匹配。 有关此方面的信息，可参阅 [HDInsight 与网络安全组和用户定义路由](#hdinsight-ip)部分。

    ```azurecli
    az network nsg rule create -g RESOURCEGROUP --nsg-name hdisecure -n hdirule1 --protocol "*" --source-port-range "*" --destination-port-range "443" --source-address-prefix "52.164.210.96" --destination-address-prefix "VirtualNetwork" --access "Allow" --priority 300 --direction "Inbound"
    az network nsg rule create -g RESOURCEGROUP --nsg-name hdisecure -n hdirule2 --protocol "*" --source-port-range "*" --destination-port-range "443" --source-address-prefix "13.74.153.132" --destination-address-prefix "VirtualNetwork" --access "Allow" --priority 301 --direction "Inbound"
    az network nsg rule create -g RESOURCEGROUP --nsg-name hdisecure -n hdirule3 --protocol "*" --source-port-range "*" --destination-port-range "443" --source-address-prefix "168.61.49.99" --destination-address-prefix "VirtualNetwork" --access "Allow" --priority 302 --direction "Inbound"
    az network nsg rule create -g RESOURCEGROUP --nsg-name hdisecure -n hdirule4 --protocol "*" --source-port-range "*" --destination-port-range "443" --source-address-prefix "23.99.5.239" --destination-address-prefix "VirtualNetwork" --access "Allow" --priority 303 --direction "Inbound"
    az network nsg rule create -g RESOURCEGROUP --nsg-name hdisecure -n hdirule5 --protocol "*" --source-port-range "*" --destination-port-range "443" --source-address-prefix "168.61.48.131" --destination-address-prefix "VirtualNetwork" --access "Allow" --priority 304 --direction "Inbound"
    az network nsg rule create -g RESOURCEGROUP --nsg-name hdisecure -n hdirule6 --protocol "*" --source-port-range "*" --destination-port-range "443" --source-address-prefix "138.91.141.162" --destination-address-prefix "VirtualNetwork" --access "Allow" --priority 305 --direction "Inbound"
    ```

3. 若要检索此网络安全组的唯一标识符，请使用以下命令：

    ```azurecli
    az network nsg show -g RESOURCEGROUP -n hdisecure --query "id"
    ```

    此命令返回类似于以下文本的值：

        "/subscriptions/SUBSCRIPTIONID/resourceGroups/RESOURCEGROUP/providers/Microsoft.Network/networkSecurityGroups/hdisecure"

4. 使用以下命令将网络安全组应用于子网。 将 `GUID` 和 `RESOURCEGROUP` 值替换为从上一步骤中返回的值。 将 `VNETNAME` 和 `SUBNETNAME` 替换为要创建的虚拟网络名称和子网名称。

    ```azurecli
    az network vnet subnet update -g RESOURCEGROUP --vnet-name VNETNAME --name SUBNETNAME --set networkSecurityGroup.id="/subscriptions/GUID/resourceGroups/RESOURCEGROUP/providers/Microsoft.Network/networkSecurityGroups/hdisecure"
    ```

    此命令完成后，即可将 HDInsight 安装到虚拟网络中。


这些步骤只会实现对 Azure 云中 HDInsight 运行状况和管理服务的访问。 任何从虚拟网络外部对 HDInsight 群集的其他访问会被阻止。 若要从虚拟网络之外启用访问，必须添加其他的虚拟网络安全组规则。

以下代码演示如何允许来自 Internet 的 SSH 访问：

```azurecli
az network nsg rule create -g RESOURCEGROUP --nsg-name hdisecure -n ssh --protocol "*" --source-port-range "*" --destination-port-range "22" --source-address-prefix "*" --destination-address-prefix "VirtualNetwork" --access "Allow" --priority 306 --direction "Inbound"
```

## <a id="example-dns"></a> 示例：DNS 配置

### <a name="name-resolution-between-a-virtual-network-and-a-connected-on-premises-network"></a>在虚拟网络和连接的本地网络之间进行名称解析

此示例作出以下假设：

* 你有一个使用 VPN 网关连接到本地网络的 Azure 虚拟网络。

* 虚拟网络中的自定义 DNS 服务器运行 Linux 或 Unix 作为操作系统。

* [Bind](https://www.isc.org/downloads/bind/) 安装在自定义 DNS 服务器上。

在虚拟网络的自定义 DNS 服务器上，执行以下操作：

1. 使用 Azure PowerShell 或 Azure CLI 查找虚拟网络的 DNS 后缀：

    将 `RESOURCEGROUP` 替换为包含虚拟网络的资源组的名称，然后输入命令：

    ```powershell
    $NICs = Get-AzNetworkInterface -ResourceGroupName "RESOURCEGROUP"
    $NICs[0].DnsSettings.InternalDomainNameSuffix
    ```

    ```azurecli
    az network nic list --resource-group RESOURCEGROUP --query "[0].dnsSettings.internalDomainNameSuffix"
    ```

2. 在虚拟网络的自定义 DNS 服务器上，使用以下文本作为 `/etc/bind/named.conf.local` 文件的内容：

    ```
    // Forward requests for the virtual network suffix to Azure recursive resolver
    zone "0owcbllr5hze3hxdja3mqlrhhe.ex.internal.chinacloudapp.cn" {
        type forward;
        forwarders {168.63.129.16;}; # Azure recursive resolver
    };
    ```

    将 `0owcbllr5hze3hxdja3mqlrhhe.ex.internal.chinacloudapp.cn` 值替换为虚拟网络的 DNS 后缀。

    此配置将针对虚拟网络 DNS 后缀的所有 DNS 请求路由到 Azure 递归解析程序。

2. 在虚拟网络的自定义 DNS 服务器上，使用以下文本作为 `/etc/bind/named.conf.options` 文件的内容：

    ```
    // Clients to accept requests from
    // TODO: Add the IP range of the joined network to this list
    acl goodclients {
        10.0.0.0/16; # IP address range of the virtual network
        localhost;
        localnets;
    };

    options {
            directory "/var/cache/bind";

            recursion yes;

            allow-query { goodclients; };

            # All other requests are sent to the following
            forwarders {
                192.168.0.1; # Replace with the IP address of your on-premises DNS server
            };

            dnssec-validation auto;

            auth-nxdomain no;    # conform to RFC1035
            listen-on { any; };
    };
    ```

    * 将 `10.0.0.0/16` 值替换为虚拟网络的 IP 地址范围。 此项允许该范围内的名称解析请求地址。

    * 将本地网络的 IP 地址范围添加到 `acl goodclients { ... }` 部分。  此项允许本地网络中的资源发出的名称解析请求。

    * 将值 `192.168.0.1` 替换为本地 DNS 服务器的 IP 地址。 此项将所有其他的 DNS 请求路由到本地 DNS 服务器。

3. 若要使用配置，请重新启动 Bind。 例如，`sudo service bind9 restart`。

4. 向本地 DNS 服务器添加条件转发器。 配置条件转发器，将对步骤 1 中 DNS 后者的请求发送到自定义 DNS 服务器。

    > [!NOTE]
    > 请参阅 DNS 软件的文档，详细了解如何添加条件转发器。

完成这些步骤后，即可使用完全限定的域名 (FQDN) 连接到任一网络中的资源。 现在可以将 HDInsight 安装到虚拟网络。

### <a name="name-resolution-between-two-connected-virtual-networks"></a>在两个连接的虚拟网络之间进行名称解析

本示例做出如下假设：

* 你有两个使用 VPN 网关或对等互连进行连接的 Azure 虚拟网络。

* 两个网络中的自定义 DNS 服务器运行 Linux 或 Unix 作为操作系统。

* [Bind](https://www.isc.org/downloads/bind/) 安装在自定义 DNS 服务器上。

1. 使用 Azure PowerShell 或 Azure CLI 查找两个虚拟网络的 DNS 后缀：

    将 `RESOURCEGROUP` 替换为包含虚拟网络的资源组的名称，然后输入命令：

    ```powershell
    $NICs = Get-AzNetworkInterface -ResourceGroupName "RESOURCEGROUP"
    $NICs[0].DnsSettings.InternalDomainNameSuffix
    ```

    ```azurecli
    az network nic list --resource-group RESOURCEGROUP --query "[0].dnsSettings.internalDomainNameSuffix"
    ```

2. 在自定义 DNS 服务器上，使用以下文本作为 `/etc/bind/named.config.local` 文件的内容。 在两个虚拟网络中对自定义 DNS 服务器进行这种更改。

    ```
    // Forward requests for the virtual network suffix to Azure recursive resolver
    zone "0owcbllr5hze3hxdja3mqlrhhe.ex.internal.chinacloudapp.cn" {
        type forward;
        forwarders {10.0.0.4;}; # The IP address of the DNS server in the other virtual network
    };
    ```

    将 `0owcbllr5hze3hxdja3mqlrhhe.ex.internal.chinacloudapp.cn` 值替换为另一虚拟网络的 DNS 后缀。  此项将对远程网络 DNS 后缀的请求路由到该网络中的自定义 DNS。

3. 在两个虚拟网络的自定义 DNS 服务器上，使用以下文本作为 `/etc/bind/named.conf.options` 文件的内容：

    ```
    // Clients to accept requests from
    acl goodclients {
        10.1.0.0/16; # The IP address range of one virtual network
        10.0.0.0/16; # The IP address range of the other virtual network
        localhost;
        localnets;
    };

    options {
            directory "/var/cache/bind";

            recursion yes;

            allow-query { goodclients; };

            forwarders {
            168.63.129.16;   # Azure recursive resolver         
            };

            dnssec-validation auto;

            auth-nxdomain no;    # conform to RFC1035
            listen-on { any; };
    };
    ```

   * 将 `10.0.0.0/16` 和 `10.1.0.0/16` 值替换为虚拟网络的 IP 地址范围。 此项允许每个网络中的资源发出 DNS 服务器的请求。

     任何不是针对虚拟网络 DNS 后缀的请求（例如，microsoft.com）均由 Azure 递归解析程序处理。

4. 若要使用此配置，请重启 Bind。 例如，两个 DNS 服务器上的 `sudo service bind9 restart`。

完成这些步骤后，即可使用完全限定的域名 (FQDN) 连接到虚拟网络中的资源。 现在可以将 HDInsight 安装到虚拟网络。

## <a name="next-steps"></a>后续步骤

* 如需通过端到端示例来了解如何将 HDInsight 配置为连接到本地网络，请参阅[将 HDInsight 连接到本地网络](./connect-on-premises-network.md)。
* 要了解如何在 Azure 虚拟网络中配置 Apache Hbase 群集，请参阅[在 Azure 虚拟网络中的 HDInsight 上创建 Apache HBase 群集](hbase/apache-hbase-provision-vnet.md)。
* 要了解如何配置 Apache HBase 异地复制，请参阅[在 Azure 虚拟网络中设置 Apache HBase 群集复制](hbase/apache-hbase-replication.md)。
* 有关 Azure 虚拟网络的详细信息，请参阅 [Azure 虚拟网络概述](../virtual-network/virtual-networks-overview.md)。

* 有关网络安全组的详细信息，请参阅[网络安全组](../virtual-network/security-overview.md)。

* 有关用户定义路由的详细信息，请参阅[用户定义路由和 IP 转发](../virtual-network/virtual-networks-udr-overview.md)。
