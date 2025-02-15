---
title: 管理 Linux 虚拟机的可用性 | Azure
description: 了解如何使用多个虚拟机来确保 Linux 应用程序在 Azure 中的高可用性
services: virtual-machines-linux
documentationcenter: ''
author: rockboyfor
manager: digimobile
editor: tysonn
tags: azure-resource-manager,azure-service-management
ms.assetid: 891c852a-84c0-4940-a61e-ada6e185bf37
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
origin.date: 03/27/2018
ms.date: 07/01/2019
ms.author: v-yeche
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 68cadf4fe7b7eaa7e360835245832abba94cbcbf
ms.sourcegitcommit: 5191c30e72cbbfc65a27af7b6251f7e076ba9c88
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/05/2019
ms.locfileid: "67570428"
---
# <a name="manage-the-availability-of-linux-virtual-machines"></a>管理 Linux 虚拟机的可用性

了解如何设置和管理多个虚拟机，以确保 Linux 应用程序在 Azure 中的高可用性。 也可以[管理 Windows 虚拟机的可用性](../windows/manage-availability.md?toc=%2fvirtual-machines%2fwindows%2ftoc.json)。

有关在 Resource Manager 部署模型中使用 CLI 创建可用性集的说明，请参阅 [azure availset：用于管理可用性集的命令](../azure-cli-arm-commands.md#azure-availset-commands-to-manage-your-availability-sets)。

[!INCLUDE [virtual-machines-common-manage-availability](../../../includes/virtual-machines-common-manage-availability.md)]

## <a name="next-steps"></a>后续步骤
若要了解对虚拟机进行负载均衡的详细信息，请参阅[对虚拟机进行负载均衡](../virtual-machines-linux-load-balance.md)。

<!--Update_Description: update meta properties -->