---
title: 使用 VNet 到 VNet 连接将 Azure 虚拟网络连接到另一 VNet：PowerShell | Microsoft Docs
description: 使用 VNet 到 VNet 连接和 PowerShell 将虚拟网络连接起来。
services: vpn-gateway
author: WenJason
ms.service: vpn-gateway
ms.topic: conceptual
origin.date: 02/15/2019
ms.date: 03/04/2019
ms.author: v-jay
ms.openlocfilehash: 580fb5205ce3dc170f76eb663c367f98df36cc6a
ms.sourcegitcommit: 5fc46672ae90b6598130069f10efeeb634e9a5af
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/19/2019
ms.locfileid: "67236654"
---
# <a name="configure-a-vnet-to-vnet-vpn-gateway-connection-using-powershell"></a>使用 PowerShell 配置 VNet 到 VNet VPN 网关连接

本文介绍如何使用 VNet 到 VNet 连接类型来连接虚拟网络。 虚拟网络可以位于相同或不同的区域中。

本文中的步骤适用于 Resource Manager 部署模型并使用 PowerShell。 也可使用不同的部署工具或部署模型创建此配置，方法是从以下列表中选择另一选项：

> [!div class="op_single_selector"]
> * [Azure 门户](vpn-gateway-howto-vnet-vnet-resource-manager-portal.md)
> * [PowerShell](vpn-gateway-vnet-vnet-rm-ps.md)
> * [Azure CLI](vpn-gateway-howto-vnet-vnet-cli.md)
> * [Azure 门户（经典）](vpn-gateway-howto-vnet-vnet-portal-classic.md)
> * [连接不同的部署模型 - Azure 门户](vpn-gateway-connect-different-deployment-models-portal.md)
> * [连接不同的部署模型 - PowerShell](vpn-gateway-connect-different-deployment-models-powershell.md)

## <a name="about"></a>关于连接 VNet

可通过多种方式来连接 VNet。 以下各节介绍了如何通过不同方式来连接虚拟网络。

### <a name="vnet-to-vnet"></a>VNet 到 VNet

配置一个 VNet 到 VNet 连接即可轻松地连接 VNet。 使用 VNet 到 VNet 连接类型 (VNet2VNet) 将一个虚拟网络连接到另一个虚拟网络类似于创建到本地位置的站点到站点 IPsec 连接。  这两种连接类型都使用 VPN 网关来提供使用 IPsec/IKE 的安全隧道，二者在通信时使用同样的方式运行。 连接类型的差异在于本地网关的配置方式。 创建 VNet 到 VNet 连接时，看不到本地网关地址空间。 它是自动创建并填充的。 如果更新一个 VNet 的地址空间，另一个 VNet 会自动知道路由到更新的地址空间。 与在 VNet 之间创建站点到站点连接相比，创建 VNet 到 VNet 连接通常速度更快且更容易。

### <a name="site-to-site-ipsec"></a>站点到站点 (IPsec)

如果要进行复杂的网络配置，则与使用 VNet 到 VNet 步骤相比，使用[站点到站点](vpn-gateway-create-site-to-site-rm-powershell.md)步骤来连接 VNet 会更好。 使用站点到站点步骤时，可以手动创建和配置本地网关。 每个 VNet 的本地网关都将其他 VNet 视为本地站点。 这样可以为本地网关指定路由流量所需的其他地址空间。 如果 VNet 的地址空间更改，则需根据更改更新相应的本地网关。 它不自动进行更新。

### <a name="vnet-peering"></a>VNet 对等互连

可以考虑使用 VNet 对等互连来连接 VNet。 VNet 对等互连不使用 VPN 网关，并且有不同的约束。 另外，[VNet 对等互连定价](https://www.azure.cn/pricing/details/virtual-network)的计算不同于 [VNet 到 VNet VPN 网关定价](https://www.azure.cn/pricing/details/vpn-gateway)的计算。 有关详细信息，请参阅 [VNet 对等互连](../virtual-network/virtual-network-peering-overview.md)。

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

  ![v2v 示意图](./media/vpn-gateway-vnet-vnet-rm-ps/v2vrmps.png)

## <a name="samesub"></a>如何连接相同订阅中的 VNet

### <a name="before-you-begin"></a>准备阶段

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

如果更想本地安装最新版本的 Azure PowerShell 模块，请参阅[如何安装和配置 Azure PowerShell](/powershell/azure/overview)。

### <a name="Step1"></a>步骤 1 - 规划 IP 地址范围

以下步骤将创建两个虚拟网络，以及它们各自的网关子网和配置。 然后在两个 VNet 之间创建 VPN 连接。 必须规划好网络配置的 IP 地址范围。 请记住，必须确保没有任何 VNet 范围或本地网络范围存在任何形式的重叠。 在这些示例中，我们没有包括 DNS 服务器。 如果需要虚拟网络的名称解析，请参阅[名称解析](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md)。

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
* 连接（1 到 5）：VNet1 到 VNet5（适用于不同订阅中的 VNet）
* 连接类型：VNet2VNet

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
* 连接：VNet4 到 VNet1
* 连接类型：VNet2VNet


### <a name="Step2"></a>步骤 2 - 创建并配置 TestVNet1

1. 验证订阅设置。

   连接到帐户。

   ```azurepowershell
   Connect-AzAccount -Environment AzureChinaCloud
   ```

   检查该帐户的订阅。

   ```azurepowershell
   Get-AzSubscription
   ```

   如果有多个订阅，请指定要使用的订阅。

   ```azurepowershell
   Select-AzSubscription -SubscriptionName nameofsubscription
   ```
2. 声明变量。 此示例使用本练习中的值来声明变量。 在大多数情况下，应该将这些值替换为自己的值。 但是，如果执行这些步骤是为了熟悉此类型的配置，则可以使用这些变量。 根据需要修改变量，并将其复制并粘贴到 PowerShell 控制台中。

   ```azurepowershell
   $RG1 = "TestRG1"
   $Location1 = "China North"
   $VNetName1 = "TestVNet1"
   $FESubName1 = "FrontEnd"
   $BESubName1 = "Backend"
   $GWSubName1 = "GatewaySubnet"
   $VNetPrefix11 = "10.11.0.0/16"
   $VNetPrefix12 = "10.12.0.0/16"
   $FESubPrefix1 = "10.11.0.0/24"
   $BESubPrefix1 = "10.12.0.0/24"
   $GWSubPrefix1 = "10.12.255.0/27"
   $GWName1 = "VNet1GW"
   $GWIPName1 = "VNet1GWIP"
   $GWIPconfName1 = "gwipconf1"
   $Connection14 = "VNet1toVNet4"
   $Connection15 = "VNet1toVNet5"
   ```
3. 创建资源组。

   ```powershell
   New-AzResourceGroup -Name $RG1 -Location $Location1
   ```
4. 创建 TestVNet1 的子网配置。 本示例创建一个名为 TestVNet1 的虚拟网络和三个子网：一个名为 GatewaySubnet、一个名为 FrontEnd，还有一个名为 Backend。 替换值时，请务必始终将网关子网特意命名为 GatewaySubnet。 如果命名为其他名称，网关创建会失败。

   以下示例使用前面设置的变量。 在本示例中，网关子网使用 /27。 尽管创建的网关子网最小可为 /29，但建议至少选择 /28 或 /27，创建包含更多地址的更大子网。 这样便可以留出足够的地址，满足将来可能需要使用的其他配置。

   ```powershell
   $fesub1 = New-AzVirtualNetworkSubnetConfig -Name $FESubName1 -AddressPrefix $FESubPrefix1
   $besub1 = New-AzVirtualNetworkSubnetConfig -Name $BESubName1 -AddressPrefix $BESubPrefix1
   $gwsub1 = New-AzVirtualNetworkSubnetConfig -Name $GWSubName1 -AddressPrefix $GWSubPrefix1
   ```
5. 创建 TestVNet1。

   ```powershell
   New-AzVirtualNetwork -Name $VNetName1 -ResourceGroupName $RG1 `
   -Location $Location1 -AddressPrefix $VNetPrefix11,$VNetPrefix12 -Subnet $fesub1,$besub1,$gwsub1
   ```
6. 请求一个公共 IP 地址，以分配给要为 VNet 创建的网关。 请注意，AllocationMethod 为 Dynamic。 无法指定要使用的 IP 地址。 它会动态分配到网关。 

   ```powershell
   $gwpip1 = New-AzPublicIpAddress -Name $GWIPName1 -ResourceGroupName $RG1 `
   -Location $Location1 -AllocationMethod Dynamic
   ```
7. 创建网关配置。 网关配置定义要使用的子网和公共 IP 地址。 使用该示例创建网关配置。

   ```powershell
   $vnet1 = Get-AzVirtualNetwork -Name $VNetName1 -ResourceGroupName $RG1
   $subnet1 = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet1
   $gwipconf1 = New-AzVirtualNetworkGatewayIpConfig -Name $GWIPconfName1 `
   -Subnet $subnet1 -PublicIpAddress $gwpip1
   ```
8. 为 TestVNet1 创建网关。 本步骤为 TestVNet1 创建虚拟网络网关。 VNet 到 VNet 配置需要基于路由的 VPN 类型。 创建网关通常需要 45 分钟或更长的时间，具体取决于所选网关 SKU。

   ```powershell
   New-AzVirtualNetworkGateway -Name $GWName1 -ResourceGroupName $RG1 `
   -Location $Location1 -IpConfigurations $gwipconf1 -GatewayType Vpn `
   -VpnType RouteBased -GatewaySku VpnGw1
   ```

### <a name="step-3---create-and-configure-testvnet4"></a>步骤 3 - 创建并配置 TestVNet4

配置 TestVNet1 后，请创建 TestVNet4。 遵循以下步骤，并根据需要替换为自己的值。

1. 连接并声明变量。 请务必将值替换为要用于配置的值。

   ```powershell
   $RG4 = "TestRG4"
   $Location4 = "China North"
   $VnetName4 = "TestVNet4"
   $FESubName4 = "FrontEnd"
   $BESubName4 = "Backend"
   $GWSubName4 = "GatewaySubnet"
   $VnetPrefix41 = "10.41.0.0/16"
   $VnetPrefix42 = "10.42.0.0/16"
   $FESubPrefix4 = "10.41.0.0/24"
   $BESubPrefix4 = "10.42.0.0/24"
   $GWSubPrefix4 = "10.42.255.0/27"
   $GWName4 = "VNet4GW"
   $GWIPName4 = "VNet4GWIP"
   $GWIPconfName4 = "gwipconf4"
   $Connection41 = "VNet4toVNet1"
   ```
2. 创建资源组。

   ```powershell
   New-AzResourceGroup -Name $RG4 -Location $Location4
   ```
3. 创建 TestVNet4 的子网配置。

   ```powershell
   $fesub4 = New-AzVirtualNetworkSubnetConfig -Name $FESubName4 -AddressPrefix $FESubPrefix4
   $besub4 = New-AzVirtualNetworkSubnetConfig -Name $BESubName4 -AddressPrefix $BESubPrefix4
   $gwsub4 = New-AzVirtualNetworkSubnetConfig -Name $GWSubName4 -AddressPrefix $GWSubPrefix4
   ```
4. 创建 TestVNet4。

   ```powershell
   New-AzVirtualNetwork -Name $VnetName4 -ResourceGroupName $RG4 `
   -Location $Location4 -AddressPrefix $VnetPrefix41,$VnetPrefix42 -Subnet $fesub4,$besub4,$gwsub4
   ```
5. 请求公共 IP 地址。

   ```powershell
   $gwpip4 = New-AzPublicIpAddress -Name $GWIPName4 -ResourceGroupName $RG4 `
   -Location $Location4 -AllocationMethod Dynamic
   ```
6. 创建网关配置。

   ```powershell
   $vnet4 = Get-AzVirtualNetwork -Name $VnetName4 -ResourceGroupName $RG4
   $subnet4 = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet4
   $gwipconf4 = New-AzVirtualNetworkGatewayIpConfig -Name $GWIPconfName4 -Subnet $subnet4 -PublicIpAddress $gwpip4
   ```
7. 创建 TestVNet4 网关。 创建网关通常需要 45 分钟或更长的时间，具体取决于所选网关 SKU。

   ```powershell
   New-AzVirtualNetworkGateway -Name $GWName4 -ResourceGroupName $RG4 `
   -Location $Location4 -IpConfigurations $gwipconf4 -GatewayType Vpn `
   -VpnType RouteBased -GatewaySku VpnGw1
   ```

### <a name="step-4---create-the-connections"></a>步骤 4 - 创建连接

1. 获取两个虚拟网关。

   ```powershell
   $vnet1gw = Get-AzVirtualNetworkGateway -Name $GWName1 -ResourceGroupName $RG1
   $vnet4gw = Get-AzVirtualNetworkGateway -Name $GWName4 -ResourceGroupName $RG4
   ```
2. 创建 TestVNet1 到 TestVNet4 的连接。 本步骤创建从 TestVNet1 到 TestVNet4 的连接。 示例中引用了共享密钥。 可以对共享密钥使用自己的值。 共享密钥必须与两个连接匹配，这一点非常重要。 创建连接可能需要简短的一段时间才能完成。

   ```powershell
   New-AzVirtualNetworkGatewayConnection -Name $Connection14 -ResourceGroupName $RG1 `
   -VirtualNetworkGateway1 $vnet1gw -VirtualNetworkGateway2 $vnet4gw -Location $Location1 `
   -ConnectionType Vnet2Vnet -SharedKey 'AzureA1b2C3'
   ```
3. 创建 TestVNet4 到 TestVNet1 的连接。 此步骤类似上面的步骤，只不过是创建 TestVNet4 到 TestVNet1 的连接。 确保共享密钥匹配。 几分钟后会建立连接。

   ```powershell
   New-AzVirtualNetworkGatewayConnection -Name $Connection41 -ResourceGroupName $RG4 `
   -VirtualNetworkGateway1 $vnet4gw -VirtualNetworkGateway2 $vnet1gw -Location $Location4 `
   -ConnectionType Vnet2Vnet -SharedKey 'AzureA1b2C3'
   ```

## <a name="verify"></a>如何验证连接

[!INCLUDE [vpn-gateway-no-nsg-include](../../includes/vpn-gateway-no-nsg-include.md)]

[!INCLUDE [verify connections powershell](../../includes/vpn-gateway-verify-connection-ps-rm-include.md)]

## <a name="faq"></a>VNet 到 VNet 常见问题解答

[!INCLUDE [vpn-gateway-vnet-vnet-faq](../../includes/vpn-gateway-faq-vnet-vnet-include.md)]

## <a name="next-steps"></a>后续步骤

* 连接完成后，即可将虚拟机添加到虚拟网络。 有关详细信息，请参阅[虚拟机文档](https://docs.azure.cn/)。
* 有关 BGP 的信息，请参阅 [BGP 概述](vpn-gateway-bgp-overview.md)和[如何配置 BGP](vpn-gateway-bgp-resource-manager-ps.md)。
