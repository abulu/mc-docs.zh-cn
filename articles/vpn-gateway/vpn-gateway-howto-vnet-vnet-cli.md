---
title: 使用 VNet 到 VNet 连接将虚拟网络连接到另一 VNet：Azure CLI | Microsoft Docs
description: 使用 VNet 到 VNet 连接和 Azure CLI 将虚拟网络连接起来。
services: vpn-gateway
documentationcenter: na
author: WenJason
manager: digimobile
editor: ''
tags: azure-resource-manager
ms.assetid: 0683c664-9c03-40a4-b198-a6529bf1ce8b
ms.service: vpn-gateway
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
origin.date: 02/14/2018
ms.date: 06/24/2018
ms.author: v-jay
ms.openlocfilehash: f199bdcfee5b91e36876cb8847724792acf57fa9
ms.sourcegitcommit: 5fc46672ae90b6598130069f10efeeb634e9a5af
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/19/2019
ms.locfileid: "67236482"
---
# <a name="configure-a-vnet-to-vnet-vpn-gateway-connection-using-azure-cli"></a>使用 Azure CLI 配置 VNet 到 VNet 的 VPN 网关连接

本文介绍如何使用 VNet 到 VNet 连接类型来连接虚拟网络。 这些虚拟网络可以位于同一区域。

本文中的步骤适用于资源管理器部署模型并使用 Azure CLI。 也可使用不同的部署工具或部署模型创建此配置，方法是从以下列表中选择另一选项：

> [!div class="op_single_selector"]
> * [Azure 门户](vpn-gateway-howto-vnet-vnet-resource-manager-portal.md)
> * [PowerShell](vpn-gateway-vnet-vnet-rm-ps.md)
> * [Azure CLI](vpn-gateway-howto-vnet-vnet-cli.md)
> * [Azure 门户（经典）](vpn-gateway-howto-vnet-vnet-portal-classic.md)
> * [连接不同的部署模型 - Azure 门户](vpn-gateway-connect-different-deployment-models-portal.md)
> * [连接不同的部署模型 - PowerShell](vpn-gateway-connect-different-deployment-models-powershell.md)
>
>

## <a name="about"></a>关于连接 VNet

可通过多种方式来连接 VNet。 以下各节介绍了如何通过不同方式来连接虚拟网络。

### <a name="vnet-to-vnet"></a>VNet 到 VNet

配置一个 VNet 到 VNet 连接即可轻松地连接 VNet。 使用 VNet 到 VNet 连接类型将一个虚拟网络连接到另一个虚拟网络类似于创建到本地位置的站点到站点 IPsec 连接。 这两种连接类型都使用 VPN 网关来提供使用 IPsec/IKE 的安全隧道，二者在通信时使用同样的方式运行。 连接类型的差异在于本地网关的配置方式。 创建 VNet 到 VNet 连接时，看不到本地网关地址空间。 它是自动创建并填充的。 如果更新一个 VNet 的地址空间，另一个 VNet 会自动知道路由到更新的地址空间。 与在 VNet 之间创建站点到站点连接相比，创建 VNet 到 VNet 连接通常速度更快且更容易。

### <a name="connecting-vnets-using-site-to-site-ipsec-steps"></a>使用站点到站点 (IPsec) 步骤连接 VNet

如果要进行复杂的网络配置，则与使用 VNet 到 VNet 步骤相比，使用[站点到站点](vpn-gateway-howto-site-to-site-resource-manager-cli.md)步骤来连接 VNet 会更好。 使用站点到站点步骤时，可以手动创建和配置本地网关。 每个 VNet 的本地网关都将其他 VNet 视为本地站点。 这样可以为本地网关指定路由流量所需的其他地址空间。 如果 VNet 的地址空间更改，则需根据更改手动更新相应的本地网关。 它不自动进行更新。

### <a name="vnet-peering"></a>VNet 对等互连

可以考虑使用 VNet 对等互连来连接 VNet。 VNet 对等互连不使用 VPN 网关，并且有不同的约束。 另外，[VNet 对等互连定价](https://azure.cn/pricing/details/virtual-network)的计算不同于 [VNet 到 VNet VPN 网关定价](https://www.azure.cn/pricing/details/vpn-gateway)的计算。 有关详细信息，请参阅 [VNet 对等互连](../virtual-network/virtual-network-peering-overview.md)。

## <a name="why"></a>为何创建 VNet 到 VNet 连接？

你可能会出于以下原因而使用 VNet 到 VNet 连接来连接虚拟网络：

* **跨区域地域冗余和地域存在**

  * 可以使用安全连接设置自己的异地复制或同步，而无需借助于面向 Internet 的终结点。
  * 使用 Azure 流量管理器和负载均衡器，可以设置支持跨多个 Azure 区域实现异地冗余的高可用性工作负荷。 一个重要的示例就是对分布在多个 Azure 区域中的可用性组设置 SQL Always On。
* **具有隔离或管理边界的区域多层应用程序**

  * 在同一区域中，由于存在隔离或管理要求，可以设置具有多个虚拟网络的多层应用程序，这些虚拟网络相互连接在一起。

可以将 VNet 到 VNet 通信与多站点配置组合使用。 这样，便可以建立将跨界连接与虚拟网络间连接相结合的网络拓扑。

## <a name="steps"></a>应使用哪些 VNet 到 VNet 步骤？

此配置的步骤使用 TestVNet1 和 TestVNet4。

  ![v2v 示意图](./media/vpn-gateway-howto-vnet-vnet-cli/v2vrmps.png)

## <a name="samesub"></a>连接同一订阅中的 VNet

### <a name="before-you-begin"></a>准备阶段

在开始之前，请安装最新版本的 CLI 命令（2.0 或更高版本）。 有关安装 CLI 命令的信息，请参阅[安装 Azure CLI](/cli/install-azure-cli)。

### <a name="Plan"></a>计划 IP 地址范围

以下步骤将创建两个虚拟网络，以及它们各自的网关子网和配置。 然后在两个 VNet 之间创建 VPN 连接。 必须计划用于网络配置的 IP 地址范围。 请记住，必须确保没有任何 VNet 范围或本地网络范围存在任何形式的重叠。 在这些示例中，我们没有包括 DNS 服务器。 如果需要虚拟网络的名称解析，请参阅[名称解析](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md)。

示例中使用了以下值：

**TestVNet1 的值：**

* VNet 名称：TestVNet1
* 资源组：TestRG1
* 位置：中国北部
* TestVNet1：10.11.0.0/16 和 10.12.0.0/16
* FrontEnd：10.11.0.0/24
* BackEnd：10.12.0.0/24
* GatewaySubnet：10.12.255.0/27
* GatewayName：VNet1GW
* 公共 IP：VNet1GWIP
* VPNType：RouteBased
* 连接（1 到 4）：VNet1 到 VNet4

**TestVNet4 的值：**

* VNet 名称：TestVNet4
* TestVNet2：10.41.0.0/16 和 10.42.0.0/16
* FrontEnd：10.41.0.0/24
* BackEnd：10.42.0.0/24
* GatewaySubnet：10.42.255.0/27
* 资源组：TestRG4
* 位置：中国北部
* GatewayName：VNet4GW
* 公共 IP：VNet4GWIP
* VPNType：RouteBased
* 连接：VNet4toVNet1

### <a name="Connect"></a>步骤 1 - 连接到订阅

[!INCLUDE [CLI login](../../includes/vpn-gateway-cli-login-numbers-include.md)]

### <a name="TestVNet1"></a>步骤 2 - 创建并配置 TestVNet1

1. 创建资源组。

   ```azurecli
   az group create -n TestRG1  -l chinanorth
   ```
2. 创建 TestVNet1 及其子网。 以下示例创建名为“TestVNet1”的虚拟网络和名为“FrontEnd”的子网。

   ```azurecli
   az network vnet create -n TestVNet1 -g TestRG1 --address-prefix 10.11.0.0/16 -l chinanorth --subnet-name FrontEnd --subnet-prefix 10.11.0.0/24
   ```
3. 为后端子网创建额外的地址空间。 请注意，这一步指定此前创建的地址空间，以及需要添加的额外地址空间。 这是因为，[az network vnet update](/cli/network/vnet) 命令覆盖以前的设置。 请确保在使用此命令时指定所有地址前缀。

   ```azurecli
   az network vnet update -n TestVNet1 --address-prefixes 10.11.0.0/16 10.12.0.0/16 -g TestRG1
   ```
4. 创建后端子网。
  
   ```azurecli
   az network vnet subnet create --vnet-name TestVNet1 -n BackEnd -g TestRG1 --address-prefix 10.12.0.0/24 
   ```
5. 创建网关子网。 请注意，网关子网命名为“GatewaySubnet”。 此名称是必需的。 在本示例中，网关子网使用 /27。 尽管创建的网关子网最小可为 /29，但建议至少选择 /28 或 /27，创建包含更多地址的更大子网。 这样便可以留出足够的地址，满足将来可能需要使用的其他配置。

   ```azurecli 
   az network vnet subnet create --vnet-name TestVNet1 -n GatewaySubnet -g TestRG1 --address-prefix 10.12.255.0/27
   ```
6. 请求一个公共 IP 地址，以分配给要为 VNet 创建的网关。 请注意，AllocationMethod 为 Dynamic。 无法指定要使用的 IP 地址。 它会动态分配到网关。

   ```azurecli
   az network public-ip create -n VNet1GWIP -g TestRG1 --allocation-method Dynamic
   ```
7. 为 TestVNet1 创建虚拟网络网关。 VNet 到 VNet 配置需要基于路由的 VPN 类型。 如果使用“--no-wait”参数运行该命令，则不会显示任何反馈或输出。 “--no-wait”参数允许在后台创建网关， 但并不意味着 VPN 网关会立即创建完毕。 创建网关通常需要 45 分钟或更长的时间，具体取决于所使用的网关 SKU。

   ```azurecli
   az network vnet-gateway create -n VNet1GW -l chinanorth --public-ip-address VNet1GWIP -g TestRG1 --vnet TestVNet1 --gateway-type Vpn --sku VpnGw1 --vpn-type RouteBased --no-wait
   ```

### <a name="TestVNet4"></a>步骤 3 - 创建并配置 TestVNet4

1. 创建资源组。

   ```azurecli
   az group create -n TestRG4  -l chinanorth
   ```
2. 创建 TestVNet4。

   ```azurecli
   az network vnet create -n TestVNet4 -g TestRG4 --address-prefix 10.41.0.0/16 -l chinanorth --subnet-name FrontEnd --subnet-prefix 10.41.0.0/24
   ```

3. 为 TestVNet4 创建额外的子网。

   ```azurecli
   az network vnet update -n TestVNet4 --address-prefixes 10.41.0.0/16 10.42.0.0/16 -g TestRG4 
   az network vnet subnet create --vnet-name TestVNet4 -n BackEnd -g TestRG4 --address-prefix 10.42.0.0/24 
   ```
4. 创建网关子网。

   ```azurecli
   az network vnet subnet create --vnet-name TestVNet4 -n GatewaySubnet -g TestRG4 --address-prefix 10.42.255.0/27
   ```
5. 请求公共 IP 地址。

   ```azurecli
   az network public-ip create -n VNet4GWIP -g TestRG4 --allocation-method Dynamic
   ```
6. 创建 TestVNet4 虚拟网关。

   ```azurecli
   az network vnet-gateway create -n VNet4GW -l chinanorth --public-ip-address VNet4GWIP -g TestRG4 --vnet TestVNet4 --gateway-type Vpn --sku VpnGw1 --vpn-type RouteBased --no-wait
   ```

### <a name="createconnect"></a>步骤 4 - 创建连接

现在有两个带 VPN 网关的 VNet。 下一步是创建虚拟网络网关之间的 VPN 网关连接。 如果使用了上面的示例，则 VNet 网关位于不同的资源组。 如果网关位于不同的资源组中，则在进行连接时需标识并指定每个网关的资源 ID。 如果 VNet 位于同一资源组中，则可使用[第二组说明](#samerg)，因为不需指定资源 ID。

### <a name="diffrg"></a>若要连接驻留在不同资源组中的 VNet，请执行以下步骤

1. 从以下命令的输出中获取 VNet1GW 的资源 ID：

   ```azurecli
   az network vnet-gateway show -n VNet1GW -g TestRG1
   ```

   在输出中，找到“id:”行。 引号中的值是在下一部分创建连接所必需的。 将这些值复制到文本编辑器（例如记事本），这样就可以在创建连接时轻松地粘贴它们。

   示例输出：

   ```
   "activeActive": false, 
   "bgpSettings": { 
    "asn": 65515, 
    "bgpPeeringAddress": "10.12.255.30", 
    "peerWeight": 0 
   }, 
   "enableBgp": false, 
   "etag": "W/\"ecb42bc5-c176-44e1-802f-b0ce2962ac04\"", 
   "gatewayDefaultSite": null, 
   "gatewayType": "Vpn", 
   "id": "/subscriptions/d6ff83d6-713d-41f6-a025-5eb76334fda9/resourceGroups/TestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW", 
   "ipConfigurations":
   ```

   复制引号中 "id": 后面的值。 

   ```
   "id": "/subscriptions/d6ff83d6-713d-41f6-a025-5eb76334fda9/resourceGroups/TestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW"
   ```

2. 获取 VNet4GW 的资源 ID 并将值复制到文本编辑器。

   ```azurecli
   az network vnet-gateway show -n VNet4GW -g TestRG4
   ```

3. 创建 TestVNet1 到 TestVNet4 的连接。 本步骤创建从 TestVNet1 到 TestVNet4 的连接。 示例中引用了一个共享密钥。 可以对共享密钥使用自己的值。 共享密钥必须与两个连接匹配，这一点非常重要。 创建连接短时间即可完成。

   ```azurecli
   az network vpn-connection create -n VNet1ToVNet4 -g TestRG1 --vnet-gateway1 /subscriptions/d6ff83d6-713d-41f6-a025-5eb76334fda9/resourceGroups/TestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW -l chinanorth --shared-key "aabbcc" --vnet-gateway2 /subscriptions/d6ff83d6-713d-41f6-a025-5eb76334fda9/resourceGroups/TestRG4/providers/Microsoft.Network/virtualNetworkGateways/VNet4GW 
   ```
4. 创建 TestVNet4 到 TestVNet1 的连接。 此步骤类似上面的步骤，只不过是创建 TestVNet4 到 TestVNet1 的连接。 确保共享密钥匹配。 建立连接需要数分钟的时间。

   ```azurecli
   az network vpn-connection create -n VNet4ToVNet1 -g TestRG4 --vnet-gateway1 /subscriptions/d6ff83d6-713d-41f6-a025-5eb76334fda9/resourceGroups/TestRG4/providers/Microsoft.Network/virtualNetworkGateways/VNet4GW -l chinanorth --shared-key "aabbcc" --vnet-gateway2 /subscriptions/d6ff83d6-713d-41f6-a025-5eb76334fda9/resourceGroups/TestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1G
   ```
5. 验证连接。 请参阅[验证连接](#verify)。

### <a name="samerg"></a>若要连接驻留在同一资源组中的 VNet，请执行以下步骤

1. 创建 TestVNet1 到 TestVNet4 的连接。 本步骤创建从 TestVNet1 到 TestVNet4 的连接。 请注意，示例中的资源组是相同的。 还可以看到示例中引用了共享密钥。 可以对共享密钥使用你自己的值，但两个连接的共享密钥必须匹配。 创建连接短时间即可完成。

   ```azurecli
   az network vpn-connection create -n VNet1ToVNet4 -g TestRG1 --vnet-gateway1 VNet1GW -l chinanorth --shared-key "eeffgg" --vnet-gateway2 VNet4GW
   ```
2. 创建 TestVNet4 到 TestVNet1 的连接。 此步骤类似上面的步骤，只不过是创建 TestVNet4 到 TestVNet1 的连接。 确保共享密钥匹配。 建立连接需要数分钟的时间。

   ```azurecli
    az network vpn-connection create -n VNet4ToVNet1 -g TestRG1 --vnet-gateway1 VNet4GW -l chinanorth --shared-key "eeffgg" --vnet-gateway2 VNet1GW
   ```
3. 验证连接。 请参阅[验证连接](#verify)。


## <a name="verify"></a>验证连接
[!INCLUDE [vpn-gateway-no-nsg-include](../../includes/vpn-gateway-no-nsg-include.md)]

[!INCLUDE [verify connections](../../includes/vpn-gateway-verify-connection-cli-rm-include.md)]

## <a name="faq"></a>VNet 到 VNet 常见问题解答
[!INCLUDE [vpn-gateway-vnet-vnet-faq](../../includes/vpn-gateway-faq-vnet-vnet-include.md)]

## <a name="next-steps"></a>后续步骤

* 连接完成后，即可将虚拟机添加到虚拟网络。 有关详细信息，请参阅[虚拟机文档](https://docs.azure.cn/)。
* 有关 BGP 的信息，请参阅 [BGP 概述](vpn-gateway-bgp-overview.md)和[如何配置 BGP](vpn-gateway-bgp-resource-manager-ps.md)。

<!--Update_Description: wording update-->
