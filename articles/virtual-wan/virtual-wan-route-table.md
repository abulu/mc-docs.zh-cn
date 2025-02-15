---
title: 创建一个 Azure 虚拟 WAN 虚拟中心路由表以定向到 NVA | Azure
description: 虚拟 WAN 虚拟中心路由表，用于将流量引导到网络虚拟设备。
services: virtual-wan
author: rockboyfor
ms.service: virtual-wan
ms.topic: conceptual
origin.date: 01/09/2019
ms.date: 06/28/2019
ms.author: v-yeche
Customer intent: As someone with a networking background, I want to work with routing tables for NVA.
ms.openlocfilehash: 4af4e826b2bcfabb28988c80f276f1365c64e7c3
ms.sourcegitcommit: 7ad1a199e9f81de27b9098b50062ef059099f765
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/28/2019
ms.locfileid: "67461879"
---
# <a name="create-a-virtual-hub-route-table-to-steer-traffic-to-a-network-virtual-appliance"></a>创建一个虚拟中心路由表来将流量引导到网络虚拟设备。

本文展示了如何将流量从虚拟中心引导到网络虚拟设备。 

![虚拟 WAN 示意图](./media/virtual-wan-route-table/vwanroute.png)

本文介绍如何执行以下操作：

* 创建 WAN
* 创建中心
* 创建中心虚拟网络连接
* 创建中心路由
* 创建路由表
* 应用路由表

## <a name="before-you-begin"></a>准备阶段

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

验证是否符合以下条件：

1. 已有一个网络虚拟设备 (NVA)。 它是所选的第三方软件，通常是通过虚拟网络中的 Azure 市场预配的。
2. 你向 NVA 网络接口分配了一个专用 IP。 
3. NVA 不能部署在虚拟中心内。 它必须部署在单独的 VNet 中。 在本文中，NVA VNet 称为“DMZ VNet”。
4. 可将一个或多个虚拟网络连接到“外围网络 VNet”。 在本文中，此 VNet 称为“间接辐射 VNet”。 这些 VNet 可以使用 VNet 对等互连连接到 DMZ VNet。
5. 验证已创建了 2 个 VNet。 它们将用作辐射 VNet。 对于本文，VNet 辐射地址空间是 10.0.2.0/24 和 10.0.3.0/24。 如果需要有关如何创建 VNet 的信息，请参阅[使用 PowerShell 创建虚拟网络](../virtual-network/quick-create-powershell.md)。
6. 请确保任何 VNet 中都没有虚拟网络网关。

<a name="signin"></a>
## <a name="1-sign-in"></a>1.登录

确保安装最新版本的资源管理器 PowerShell cmdlet。 有关安装 PowerShell cmdlet 的详细信息，请参阅[如何安装和配置 Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-az-ps)。 这很重要，因为早期版本的 cmdlet 不包含本练习所需的最新值。

1. 使用提升的权限打开 PowerShell 控制台，并登录到你的 Azure 帐户。 该 cmdlet 会提示你输入登录凭据。 登录后，它会下载帐户设置，供 Azure PowerShell 使用。

    ```powershell
    Connect-AzAccount -Environment AzureChinaCloud
    ```
2. 获取 Azure 订阅的列表。

    ```powershell
    Get-AzSubscription
    ```
3. 指定要使用的订阅。

    ```powershell
    Select-AzSubscription -SubscriptionName "Name of subscription"
    ```

<a name="rg"></a>
## <a name="2-create-resources"></a>2.创建资源

1. 创建资源组。

    ```powershell
    New-AzResourceGroup -Location "China North" -Name "testRG"
    ```
2. 创建虚拟 WAN。

    ```powershell
    $virtualWan = New-AzVirtualWan -ResourceGroupName "testRG" -Name "myVirtualWAN" -Location "China North"
    ```
3. 创建虚拟中心。

    ```powershell
    New-AzVirtualHub -VirtualWan $virtualWan -ResourceGroupName "testRG" -Name "chinanorthhub" -AddressPrefix "10.0.1.0/24"
    ```

<a name="connections"></a>
## <a name="3-create-connections"></a>3.创建连接

创建从间接辐射 VNet 和 DMZ VNet 到虚拟中心的中心虚拟网络连接。

  ```powershell
  $remoteVirtualNetwork1= Get-AzVirtualNetwork -Name "indirectspoke1" -ResourceGroupName "testRG"
  $remoteVirtualNetwork2= Get-AzVirtualNetwork -Name "indirectspoke2" -ResourceGroupName "testRG"
  $remoteVirtualNetwork3= Get-AzVirtualNetwork -Name "dmzvnet" -ResourceGroupName "testRG"

  New-AzVirtualHubVnetConnection -ResourceGroupName "testRG" -VirtualHubName "chinanorthhub" -Name  "testvnetconnection1" -RemoteVirtualNetwork $remoteVirtualNetwork1
  New-AzVirtualHubVnetConnection -ResourceGroupName "testRG" -VirtualHubName "chinanorthhub" -Name  "testvnetconnection2" -RemoteVirtualNetwork $remoteVirtualNetwork2
  New-AzVirtualHubVnetConnection -ResourceGroupName "testRG" -VirtualHubName "chinanorthhub" -Name  "testvnetconnection3" -RemoteVirtualNetwork $remoteVirtualNetwork3
  ```

<a name="route"></a>
## <a name="4-create-a-virtual-hub-route"></a>4.创建虚拟中心路由

对于本文，间接辐射 VNet 地址空间是 10.0.2.0/24 和 10.0.3.0/24，DMZ NVA 网络接口专用 IP 地址是 10.0.4.5。

```powershell
$route1 = New-AzVirtualHubRoute -AddressPrefix @("10.0.2.0/24", "10.0.3.0/24") -NextHopIpAddress "10.0.4.5"
```

<a name="applyroute"></a>
## <a name="5-create-a-virtual-hub-route-table"></a>5.创建虚拟中心路由表

创建一个虚拟中心路由表，然后将所创建的路由应用于它。

```powershell
$routeTable = New-AzVirtualHubRouteTable -Route @($route1)
```

<a name="commit"></a>
## <a name="6-commit-the-changes"></a>6.提交更改

将更改提交到虚拟中心。

```powershell
Update-AzVirtualHub -VirtualWanId $virtualWan.Id -ResourceGroupName "testRG" -Name "chinanorthhub" -RouteTable $routeTable
```

## <a name="next-steps"></a>后续步骤

若要详细了解虚拟 WAN，请参阅[虚拟 WAN 概述](virtual-wan-about.md)页。

<!--Update_Description: new articles on virtual wan route table -->
<!--ms.date: 07/01/2019-->