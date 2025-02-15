---
title: include 文件
description: include 文件
services: networking
author: rockboyfor
ms.service: networking
ms.topic: include
origin.date: 04/10/2019
ms.date: 06/10/2019
ms.author: v-yeche
ms.custom: include file
ms.openlocfilehash: c6aa4a37071866d6489694f212bc119c239fd037
ms.sourcegitcommit: ab87d30f4435c3b7c03f7edd33c9f374b7fe88c9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/03/2019
ms.locfileid: "67540085"
---
<a name="virtual-networking-limits-classic"></a>以下限制仅适用于每个订阅通过经典部署模型托管的网络资源。 了解如何[针对订阅限制查看当前资源使用情况](../articles/networking/check-usage-against-limits.md)。

| Resource | 默认限制 | 最大限制 |
| --- | --- | --- |
| 虚拟网络 |50 |100 |
| 本地网络站点 |20 个 |请联系支持人员。 |
| 每个虚拟网络的 DNS 服务器数 |20 个 |20 个 |
| 每个虚拟网络的专用 IP 地址数 |4,096 |4,096 |
| 虚拟机或角色实例的单 NIC 并发 TCP 或 UDP 流数 |如果 NIC 至少有两个，则为 500,000（至多 1,000,000）。 |如果 NIC 至少有两个，则为 500,000（至多 1,000,000）。 |
| 网络安全组 (NSG) |100 |200 |
| 每个 NSG 的 NSG 规则数 |200 |1,000 |
| 用户定义路由表数 |100 |200 |
| 每个路由表的用户定义的路由数 |100 |400 |
| 公共 IP 地址 (动态) |5 |请联系支持人员。 |
| 保留的公共 IP 地址 |20 个 |请联系支持人员。 |
| 每个部署的公共 VIP |5 |请联系支持人员。 |
| 每个部署的专用 VIP（内部负载均衡） |1 |1 |
| 终结点访问控制列表 (ACL) |50 |50 |

<a name="azure-resource-manager-virtual-networking-limits"></a>
#### <a name="networking-limits---azure-resource-manager"></a>网络限制 - Azure 资源管理器
以下限制仅适用于通过每个订阅的每个区域的 Azure Resource Manager 进行管理的网络资源。 了解如何[针对订阅限制查看当前资源使用情况](../articles/networking/check-usage-against-limits.md)。

> [!NOTE]
> 我们最近将所有默认限制提高到了最大限制。 如果没有最大限制列，则资源没有可调整的限制。 如果过去已通过客户支持提高了这些上限，因此在以下表中看不到更新的限制，可[免费提交联机客户支持请求](../articles/azure-resource-manager/resource-manager-quota-errors.md)

|                                 Resource                                  |                                             默认限制/最大限制                                              |
|---------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------|
|                             虚拟网络                              |                                                  1,000                                                  |
|                        每个虚拟网络的子网数                        |                                                  3,000                                                  |
|               每个虚拟网络的虚拟网络对等互连数                |                                                   500                                                   |
|                      每个虚拟网络的 DNS 服务器数                      |                                                   20 个                                                    |
|                 每个虚拟网络的专用 IP 地址数                  |                                                 65,536                                                  |
|                每个网络接口的专用 IP 地址数                 |                                                   256                                                   |
|                 每个虚拟机的专用 IP 地址数                  |                                                   256                                                   |
| 虚拟机或角色实例的单 NIC 并发 TCP 或 UDP 流数 |                                                 500,000                                                 |
|                          网络接口卡数                          |                                                 65,536                                                  |
|                          网络安全组                          |                                                  5,000                                                  |
|                             每个 NSG 的 NSG 规则数                             |                                                  1,000                                                  |
|                        应用程序安全组                        |                                                  3,000                                                  |
|                         用户定义路由表数                         |                                                   200                                                   |
|                    每个路由表的用户定义的路由数                    |                                                   400                                                   |
|           每个 Azure VPN 网关的点到站点根证书数           |                                                   20 个                                                    |
| 虚拟网络 TAP |100 |
| 每个虚拟网络 TAP 的网络接口 TAP 配置 |100 |

<!-- Not Available on |             IP addresses and ranges specified for source or destination in a security rule              |-->
<!-- Not Available on |                        Application security groups per IP configuration, per NIC                        |-->
<!-- Not Available on |                            IP configurations per application security group                             |-->
<!-- Not Available on | Application security groups that can be specified within all security rules of a network security group |-->

<a name="publicip-address"></a>
#### <a name="public-ip-address-limits"></a>公共 IP 地址限制
| Resource | 默认限制 | 最大限制 |
| --- | --- | --- |
| 公共 IP 地址数 - 动态 | 基本版为 1,000。 |请联系支持人员。  |
| 公共 IP 地址数 - 静态  | 基本版为 1,000。 |请联系支持人员。  |
| 公共 IP 地址数 - 静态  | 标准版为 200。|请联系支持人员。  |
| 公共 IP 前缀大小         | /28              | 请联系支持人员。 |

<a name="load-balancer"></a>
#### <a name="load-balancer-limits"></a>负载均衡器限制
以下限制仅适用于通过每个订阅的每个区域的 Azure Resource Manager 进行管理的网络资源。 了解如何[针对订阅限制查看当前资源使用情况](../articles/networking/check-usage-against-limits.md)。


|         Resource          | 默认限制 |
|---------------------------|---------------|
|      负载均衡器       |     1,000     |
| 每个资源的规则数，基本 |      250      |
| 前端 IP 配置，基本 | 200 |
| 后端池，基本 | 100，单个可用性集 |

<!-- Not Avaiable on Rules per resource, Standard  -->
<!-- Not Avaiable on Rules per IP configuration -->
<!-- Not Avaiable on Frontend IP configurations, Standard -->
<!-- Not Avaiable on | Backend pool, Standard | 1000, single VNet | contact support | -->
<!-- Not Avaiable on | Backend resources per Load Balancer, Standard * | 150 | 150 |-->
<!-- Not Avaiable on | HA Ports, Standard | 1 per internal frontend | - | -->

<!--MOONCAKE: Not Available on Backend resources per Load Balancer, Standard * | 150 |-->
<!--Not Available on <sup>1</sup>The limit is up to 150 resources, in any combination of standalone virtual machine resources, availability set resources, and virtual machine scale-set resources.-->

