---
title: Azure CLI 脚本示例 - 使用 NLB 创建 Linux VM | Azure
description: Azure CLI 脚本示例 - 使用 NLB 创建 Linux VM
services: virtual-machines-linux
documentationcenter: virtual-machines
author: rockboyfor
manager: digimobile
editor: tysonn
tags: azure-service-management
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
origin.date: 02/27/2017
ms.date: 02/18/2019
ms.author: v-yeche
ms.custom: mvc
ms.openlocfilehash: ca487ce352c3d4303f497809e742a2797dfb8c01
ms.sourcegitcommit: 9e50dde3362b6e6b192761ead6cd3f434dfb2168
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/10/2019
ms.locfileid: "67725222"
---
# <a name="create-a-highly-available-vm"></a>创建高度可用 VM

此脚本示例创建运行多个 Ubuntu 虚拟机（使用高度可用且负载均衡的配置进行配置）所需的所有项。 运行脚本后，即可拥有已加入到 Azure 可用性集并可通过 Azure 负载均衡器访问的 3 个虚拟机。

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>示例脚本

```azurecli
#!/bin/bash

# Create a resource group.
az group create --name myResourceGroup --location chinanorth

# Create a virtual network.
az network vnet create --resource-group myResourceGroup --location chinanorth --name myVnet --subnet-name mySubnet

# Create a public IP address.
az network public-ip create --resource-group myResourceGroup --name myPublicIP

# Create an Azure Load Balancer.
az network lb create --resource-group myResourceGroup --name myLoadBalancer --public-ip-address myPublicIP \
  --frontend-ip-name myFrontEndPool --backend-pool-name myBackEndPool

# Creates an LB probe on port 80.
az network lb probe create --resource-group myResourceGroup --lb-name myLoadBalancer \
  --name myHealthProbe --protocol tcp --port 80

# Creates an LB rule for port 80.
az network lb rule create --resource-group myResourceGroup --lb-name myLoadBalancer --name myLoadBalancerRuleWeb \
  --protocol tcp --frontend-port 80 --backend-port 80 --frontend-ip-name myFrontEndPool \
  --backend-pool-name myBackEndPool --probe-name myHealthProbe

# Create three NAT rules for port 22.
for i in `seq 1 3`; do
  az network lb inbound-nat-rule create \
    --resource-group myResourceGroup --lb-name myLoadBalancer \
    --name myLoadBalancerRuleSSH$i --protocol tcp \
    --frontend-port 422$i --backend-port 22 \
    --frontend-ip-name myFrontEndPool
done

# Create a network security group
az network nsg create --resource-group myResourceGroup --name myNetworkSecurityGroup

# Create a network security group rule for port 22.
az network nsg rule create --resource-group myResourceGroup --nsg-name myNetworkSecurityGroup --name myNetworkSecurityGroupRuleSSH \
  --protocol tcp --direction inbound --source-address-prefix '*' --source-port-range '*'  \
  --destination-address-prefix '*' --destination-port-range 22 --access allow --priority 1000

# Create a network security group rule for port 80.
az network nsg rule create --resource-group myResourceGroup --nsg-name myNetworkSecurityGroup --name myNetworkSecurityGroupRuleHTTP \
--protocol tcp --direction inbound --priority 1001 --source-address-prefix '*' --source-port-range '*' \
--destination-address-prefix '*' --destination-port-range 80 --access allow --priority 2000

# Create three virtual network cards and associate with public IP address and NSG.
for i in `seq 1 3`; do
  az network nic create \
    --resource-group myResourceGroup --name myNic$i \
    --vnet-name myVnet --subnet mySubnet \
    --network-security-group myNetworkSecurityGroup --lb-name myLoadBalancer \
    --lb-address-pools myBackEndPool --lb-inbound-nat-rules myLoadBalancerRuleSSH$i
done

# Create an availability set.
az vm availability-set create --resource-group myResourceGroup --name myAvailabilitySet --platform-fault-domain-count 3 --platform-update-domain-count 3

# Create three virtual machines, this creates SSH keys if not present.
for i in `seq 1 3`; do
  az vm create \
    --resource-group myResourceGroup \
    --name myVM$i \
    --availability-set myAvailabilitySet \
    --nics myNic$i \
    --image UbuntuLTS \
    --generate-ssh-keys \
    --no-wait
done
```

## <a name="clean-up-deployment"></a>清理部署

运行以下命令来删除资源组、VM 和所有相关资源。

```azurecli
az group delete --name myResourceGroup
```

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令创建资源组、虚拟机、可用性集、负载均衡器和所有相关资源。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 注释 |
|---|---|
| [az group create](https://docs.azure.cn/zh-cn/cli/group?view=azure-cli-latest#az-group-create) | 创建用于存储所有资源的资源组。 |
| [az network vnet create](https://docs.azure.cn/zh-cn/cli/network/vnet?view=azure-cli-latest#az-network-vnet-create) | 创建 Azure 虚拟网络和子网。 |
| [az network public-ip create](https://docs.azure.cn/zh-cn/cli/network/public-ip?view=azure-cli-latest#az-network-public-ip-create) | 使用静态 IP 地址和关联的 DNS 名称创建公共 IP 地址。 |
| [az network lb create](https://docs.azure.cn/zh-cn/cli/network/lb?view=azure-cli-latest#az-network-lb-create) | 创建 Azure 网络负载均衡器 (NLB)。 |
| [az network lb probe create](https://docs.azure.cn/zh-cn/cli/network/lb/probe?view=azure-cli-latest#az-network-lb-probe-create) | 创建 NLB 探测。 NLB 探测用于监视 NLB 集中的每个 VM。 如果任何 VM 无法访问，流量不会路由到该 VM。 |
| [az network lb rule create](https://docs.azure.cn/zh-cn/cli/network/lb/rule?view=azure-cli-latest#az-network-lb-rule-create) | 创建 NLB 规则。 在此示例中，为端口 80 创建一个规则。 当 HTTP 流量到达 NLB 时，它会路由到 NLB 集中的一个 VM 的端口 80。 |
| [az network lb inbound-nat-rule create](https://docs.azure.cn/zh-cn/cli/network/lb/inbound-nat-rule?view=azure-cli-latest#az-network-lb-inbound-nat-rule-create) | 创建 NLB 网络地址转换 (NAT) 规则。  NAT 规则将 NLB 的端口映射到 VM 上的端口。 此示例为发往 NLB 集中的每个 VM 的 SSH 流量创建 NAT 规则。  |
| [az network nsg create](https://docs.azure.cn/zh-cn/cli/network/nsg?view=azure-cli-latest#az-network-nsg-create) | 创建网络安全组 (NSG)，这是 Internet 和虚拟机之间的安全边界。 |
| [az network nsg rule create](https://docs.azure.cn/zh-cn/cli/network/nsg/rule?view=azure-cli-latest#az-network-nsg-rule-create) | 创建 NSG 规则以允许入站流量。 在此示例中，为 SSH 流量打开端口 22。 |
| [az network nic create](https://docs.azure.cn/zh-cn/cli/network/nic?view=azure-cli-latest#az-network-nic-create) | 创建虚拟网卡并将其连接到虚拟网络、子网和 NSG。 |
| [az vm availability-set create](https://docs.azure.cn/zh-cn/cli/network/lb/rule?view=azure-cli-latest#az-network-lb-rule-create) | 创建可用性集。 可用性集通过将虚拟机分布到各个物理资源上（以便发生故障时，不会影响整个集）来确保应用程序运行时间。 |
| [az vm create](https://docs.azure.cn/zh-cn/cli/vm/availability-set?view=azure-cli-latest#az-vm-availability-set-create) | 创建虚拟机并将其连接到网卡、虚拟网络、子网和 NSG。 此命令还指定要使用的虚拟机映像和管理凭据。  |
| [az group delete](https://docs.azure.cn/zh-cn/cli/group?view=azure-cli-latest#az-group-delete) | 删除资源组，包括所有嵌套的资源。 |

## <a name="next-steps"></a>后续步骤

有关 Azure CLI 的详细信息，请参阅 [Azure CLI 文档](https://docs.azure.cn/zh-cn/cli/index?view=azure-cli-latest)。

可以在 [Azure Linux VM 文档](../linux/cli-samples.md?toc=%2fvirtual-machines%2flinux%2ftoc.json)中找到其他虚拟机 CLI 脚本示例。

<!--Update_Description: update meta properties, update link -->