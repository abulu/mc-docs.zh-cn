---
title: 入站/出站 IP 地址 - Azure 应用服务
description: 介绍如何在应用服务中使用入站和出站 IP 地址，以及如何查找应用的此类信息。
services: app-service
documentationcenter: ''
author: cephalin
manager: cfowler
editor: ''
ms.service: app-service
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
origin.date: 04/20/2018
ms.date: 07/01/2019
ms.author: v-biyu
ms.custom: seodec18
ms.openlocfilehash: d8775cac0d995d55fc6692841e6e8d5145b04693
ms.sourcegitcommit: 153236e4ad63e57ab2ae6ff1d4ca8b83221e3a1c
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/18/2019
ms.locfileid: "67171350"
---
# <a name="inbound-and-outbound-ip-addresses-in-azure-app-service"></a>Azure 应用服务中的入站和出站 IP 地址

[Azure 应用服务](overview.md)是多租户服务。 应用与其他应用共享网络基础结构。 因此，应用的入站和出站 IP 地址可能不同，在某些情况下甚至可能会更改。 


## <a name="when-inbound-ip-changes"></a>入站 IP 更改时

不管横向扩展的实例数如何，每个应用只有一个入站 IP 地址。 执行以下操作之一时，入站 IP 地址可能会更改：

- 删除应用，然后在不同的资源组中重新创建它。
- 删除资源组和区域组合中的最后一个应用，然后重新创建它。 
- 在证书续订等操作期间删除现有的 SSL 绑定（请参阅[续订证书](app-service-web-tutorial-custom-ssl.md#renew-certificates)）。

## <a name="find-the-inbound-ip"></a>找到入站 IP

只需在本地终端中运行以下命令：

```bash
nslookup <app-name>.chinacloudsites.cn
```

## <a name="get-a-static-inbound-ip"></a>获取静态入站 IP

有时，你可能需要对应用使用专用静态 IP 地址。 若要获取静态入站 IP 地址，需要配置[基于 IP 的 SSL 绑定](app-service-web-tutorial-custom-ssl.md)。 如果并不真正需要使用 SSL 功能来保护应用，甚至可以上传一个自签名证书来实现此绑定。 在基于 IP 的 SSL 绑定中，证书将绑定到 IP 地址本身，因此，应用服务会预配一个静态 IP 地址来实现此目的。 

## <a name="when-outbound-ips-change"></a>出站 IP 更改时

不管横向扩展的实例数如何，每个应用在任意给定时间具有指定数目的出站 IP 地址。 从应用服务应用发起的任何出站连接（例如，与后端数据库的连接）使用某个出站 IP 地址作为源 IP 地址。 无法预先知道给定的应用实例要使用哪个 IP 地址来发起出站连接，因此，后端服务必须向应用的所有出站 IP 地址开放其防火墙。

在较低层（“基本”、“标准”和“高级”）之间缩放应用时，应用的出站 IP 地址集会发生更改。   

无论是哪个定价层，你都可以通过查找 `possibleOutboundIPAddresses` 属性或者在 Azure 门户的“属性”边栏选项卡中的“其他出站 IP 地址”  字段中查找你的应用可以使用的所有可能的出站 IP 地址。  请参阅[查找出站 IP](#find-outbound-ips)。

## <a name="find-outbound-ips"></a>查找出站 IP

若要在 Azure 门户中查找应用当前使用的出站 IP 地址，请单击应用左侧导航窗格中的“属性”。  它们列出在“出站 IP 地址”  字段中。

可通过在命令提示符窗口中运行以下命令来查找相同的信息。

```azurecli
az webapp show --resource-group <group_name> --name <app_name> --query outboundIpAddresses --output tsv
```

```azurepowershell
(Get-AzWebApp -ResourceGroup <group_name> -name <app_name>).OutboundIpAddresses
```

若要查找你的应用可能使用的所有出站 IP 地址（无论是哪个定价层），请在你的应用的左侧导航栏中单击“属性”。   它们列出在“其他出站 IP 地址”  字段中。

通过运行以下命令也可以找到相同的信息。

```azurecli
az webapp show --resource-group <group_name> --name <app_name> --query possibleOutboundIpAddresses --output tsv
```

```azurepowershell
(Get-AzWebApp -ResourceGroup <group_name> -name <app_name>).PossibleOutboundIpAddresses
```

## <a name="next-steps"></a>后续步骤

了解如何按源 IP 地址限制入站流量。

> [!div class="nextstepaction"]
> [静态 IP 限制](app-service-ip-restrictions.md)
