---
title: Azure 认可的 Linux 分发版 | Azure
description: 了解 Azure 认可的分发中的 Linux，包括 Ubuntu、CentOS、Oracle 和 SUSE 的指南。
services: virtual-machines-linux
documentationcenter: ''
author: rockboyfor
manager: digimobile
editor: tysonn
tags: azure-service-management,azure-resource-manager
ms.assetid: 2777a526-c260-4cb9-a31a-bdfe1a55fffc
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
origin.date: 05/16/2019
ms.date: 07/01/2019
ms.author: v-yeche
ms.openlocfilehash: 58d2b3e9beedc3e2ea8783ce46efa88b87393e8e
ms.sourcegitcommit: 5191c30e72cbbfc65a27af7b6251f7e076ba9c88
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/05/2019
ms.locfileid: "67570623"
---
# <a name="endorsed-linux-distributions-on-azure"></a>Azure 认可的 Linux 分发版
合作伙伴在 Azure 市场中提供了 Linux 映像。 我们与各大 Linux 社区合作以便在认可的发行版列表中添加更多成员。 在此期间，对于市场未提供的分发，用户始终可以按照[创建并上传包含 Linux 操作系统的虚拟硬盘](/virtual-machines/linux/create-upload-generic)中的准则安装自己的 Linux。

## <a name="supported-distributions-and-versions"></a>支持的发行版和版本
下表列出了 Azure 支持的 Linux 分发和版本。 有关 Azure 中支持 Linux 和开源代码技术的更多详细信息，请参阅 [Azure 中对 Linux 映像的支持](https://support.microsoft.com/help/2941892/support-for-linux-and-open-source-technology-in-azure)。

Hyper-V 和 Azure 的 Linux 集成服务 (LIS) 驱动程序是 Azure 直接为上游 Linux 内核提供的内核模块。  默认情况下，某些 LIS 驱动程序已内置在发行版的内核中。 [适用于 Hyper-V 和 Azure 的 Linux Integration Services 版本 4.2](https://www.microsoft.com/download/details.aspx?id=55106) 中提供了基于 CentOS 的旧式发行版供单独下载。 有关 LIS 驱动程序的详细信息，请参阅 [Linux 内核要求](create-upload-generic.md#linux-kernel-requirements)。

<!-- Not Avaiable on Red Hat Enterprise (RHEL) -->

Azure Linux 代理已预安装在 Azure 市场映像中，通常可从分发的包存储库中获得。 源代码可在 [GitHub](https://github.com/azure/walinuxagent)上找到。

| 分发 | 版本 | 驱动程序 | Agent |
| --- | --- | --- | --- |
| CentOS |CentOS 6.3+、7.0+ |CentOS 6.3：[LIS 下载](https://www.microsoft.com/download/details.aspx?id=55106)<p>CentOS 6.4+：在内核中 |包：在“WALinuxAgent”下的[存储库](http://olcentgbl.trafficmanager.net/openlogic/6/openlogic/x86_64/RPMS/)中 <br/>源代码：[GitHub](https://github.com/Azure/WALinuxAgent) |
| [CoreOS](https://coreos.com/docs/running-coreos/cloud-providers/azure/) |494.4.0+ |在内核中 |源代码：[GitHub](https://github.com/coreos/coreos-overlay/tree/master/app-emulation/wa-linux-agent) |
| Debian |Debian 7.9+、8.2+ |在内核中 |包：在“waagent”下的存储库中 <br/>源代码：[GitHub](https://github.com/Azure/WALinuxAgent) |
| SUSE Linux Enterprise |SLES/SLES for SAP<br />11 SP4<br />12 SP1+<br />15|在内核中 |包：<p> 对于 11，在 [Cloud:Tools](https://build.opensuse.org/project/show/Cloud:Tools) 存储库中<br />对于 12，包含在“公有云”模块中的“python-azure-agent”下<br/>源代码：[GitHub](https://go.microsoft.com/fwlink/p/?LinkID=250998) |
| openSUSE |openSUSE Leap 42.2+ |在内核中 |包：在“python-azure-agent”下的 [Cloud:Tools](https://build.opensuse.org/project/show/Cloud:Tools) 存储库中 <br/>源代码：[GitHub](https://github.com/Azure/WALinuxAgent) |
| Ubuntu |Ubuntu 12.04+ **<sup>1</sup>** |在内核中 |包：在“walinuxagent”下的存储库中 <br/>源代码：[GitHub](https://github.com/Azure/WALinuxAgent) |

  - **<sup>1</sup>** 对 Ubuntu 12.04 和 14.04 的扩展支持有关的信息可在此处找到：[Ubuntu 扩展安全维护](https://www.ubuntu.com/esm)。

<!--Not Available on Oracle Linux, Red Hat Enterprise Linux -->

## <a name="image-update-cadence"></a>映像的更新节奏
Azure 要求受认可 Linux 分发版的发布者在每个季度或以更快的节奏，定期使用最新修补程序和安全修复程序，更新其在 Azure 市场中的映像。 在 Azure 市场中更新的映像将自动以新版映像 SKU 的形式提供给客户使用。 有关如何查找 Linux 映像的详细信息：[在 Azure 市场中查找 Linux VM 映像](/virtual-machines/linux/cli-ps-findimage)。

### <a name="additional-links"></a>其他链接
 - [SUSE 公有云映像生命周期](https://www.suse.com/c/suse-public-cloud-image-life-cycle/)

## <a name="azure-tuned-kernels"></a>Azure 优化的内核

Azure 与众多认可的 Linux 分发商密切合作，以优化他们在 Azure 市场中发布的映像。 这种协作的一个方面体现在：开发针对 Azure 平台优化的 Linux 内核，并以完全受支持组件的形式交付 Linux 分发版。 Azure 优化的内核整合了新的功能和性能改进，并且与分发版中提供的默认或常规内核相比，其发布节奏更快（通常每季度发布一次）。

在大多数情况下，你会发现这些内核已预装在 Azure 市场中的默认映像内，因此，Azure 客户可以立即获得这些优化内核的优势。 可在以下链接中找到有关这些 Azure 优化内核的详细信息：

 - CentOS Azure 优化内核 - 通过 CentOS 虚拟化 SIG 提供 - [详细信息](https://wiki.centos.org/SpecialInterestGroup/Virtualization)
 - Debian 云内核 - 适用于 Azure 上的 Debian 10 和 Debian 9“后向移植”映像 - [详细信息](https://wiki.debian.org/Cloud/MicrosoftAzure)
 - SLES Azure 优化内核 - [详细信息](https://www.suse.com/c/a-different-builtin-kernel-for-azure-on-demand-images/)
 - Ubuntu Azure 优化内核 - [详细信息](https://blog.ubuntu.com/2017/09/21/microsoft-and-canonical-increase-velocity-with-azure-tailored-kernel)

## <a name="partners"></a>合作伙伴

### <a name="coreos"></a>CoreOS
[https://coreos.com/docs/running-coreos/cloud-providers/azure/](https://coreos.com/docs/running-coreos/cloud-providers/azure/)

从 CoreOS 网站：

*CoreOS 在设计时就已考虑到了安全性、一致性和可靠性。CoreOS 使用 Linux 容器在更高的抽象级别管理服务，而不是通过 yum 或 apt 来安装程序包。单个服务的代码和所有依赖项都打包在一个容器中，该容器可以在一个或多个 CoreOS 计算机上运行。*

### <a name="credativ"></a>Credativ
[https://www.credativ.co.uk/credativ-blog/debian-images-microsoft-azure](https://www.credativ.co.uk/credativ-blog/debian-images-microsoft-azure)

Credativ 是一家独立的咨询和服务公司，致力于通过免费软件开发和实施专业解决方案。 Credativ 是获得国际认可的开源领域专业先行者，为许多公司的 IT 部门提供支持。 Credativ 与 Azure 合作，目前正在为 Debian 8 (Jessie) 以及 Debian 7 (Wheezy) 之前的版本准备相应的 Debian 映像。 这些映像经过专门的设计，可以在 Azure 上运行并可通过该平台轻松进行管理。 Credativ 还会通过其开源支持中心为 Azure 的 Debian 映像的维护和更新提供长期支持。

<!-- Not Available on ### Oracle -->
<!-- Not Available on ### Red Hat -->
### <a name="suse"></a>SUSE
[https://www.suse.com/suse-linux-enterprise-server-on-azure](https://www.suse.com/suse-linux-enterprise-server-on-azure)

Azure 上的 SUSE Linux Enterprise Server 是一个已验证的平台，该平台为云计算提供了高级可靠性和安全性。 SUSE 的通用 Linux 平台可与 Azure 云服务无缝集成，以便交付易于管理的云环境。 借助 1,800 多个独立软件供应商提供的适用于 SUSE Linux Enterprise Server 的 9,200 多个认证应用程序，SUSE 可确保满怀信心地在 Azure 上部署数据中心内支持的运行负载。

### <a name="canonical"></a>Canonical
[https://www.ubuntu.com/cloud/azure](https://www.ubuntu.com/cloud/azure)

Canonical 工程和开放社区监管对 Ubuntu 在客户端、服务器和云计算（包括用户的个人云服务）方面获得成功起到了推动作用。 Canonical 期望使用 Ubuntu 开发一个统一的免费平台（从手机到云），为手机、平板电脑、电视和台式机提供一致的接口系列。 这个愿景使得 Ubuntu 成为各种机构（从公有云提供商到消费类电子产品制造商）的首选以及各个技术专家的最爱。

借助其遍布全球的开发人员和工程中心，Canonical 在与硬件制造商、内容提供商和软件开发人员合作以将 Ubuntu 解决方案推向市场（从电脑到服务器和手持设备）方面拥有独特的优势。

<!--Update_Description: wording update, update meta properties -->