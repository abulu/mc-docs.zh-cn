---
title: 应用服务中的操作系统功能 - Azure
description: 了解 Azure Web 应用上可供应用服务、移动应用后端和 API 应用使用的 OS 功能
services: app-service
documentationcenter: ''
author: cephalin
manager: erikre
editor: mollybos
ms.assetid: 39d5514f-0139-453a-b52e-4a1c06d8d914
ms.service: app-service
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
origin.date: 07/01/2016
ms.date: 07/01/2019
ms.author: v-biyu
ms.custom: seodec18
ms.openlocfilehash: f429a8ef025aac005ce58e4925b786ee56f35c60
ms.sourcegitcommit: 153236e4ad63e57ab2ae6ff1d4ca8b83221e3a1c
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/18/2019
ms.locfileid: "67171311"
---
# <a name="operating-system-functionality-on-azure-app-service"></a>Azure 应用服务上的操作系统功能
本文介绍了可供在 [Azure 应用服务](overview.md)上运行的所有 Windows 应用使用的常见基准操作系统功能。 这些功能包括文件、网络和注册表访问以及诊断日志和事件。 

<a id="tiers"></a>

## <a name="app-service-plan-tiers"></a>应用服务计划层
应用服务在多租户托管环境中运行客户应用。 部署在“免费”和“共享”层中的应用在共享虚拟机上的辅助进程中运行，而部署在“标准”和“高级”层中的应用在专用于与单个客户关联的应用的虚拟机上运行。    

[!INCLUDE [app-service-dev-test-note](../../includes/app-service-dev-test-note.md)]

由于应用服务支持不同层之间的无缝缩放体验，因此，为应用服务应用实施的安全配置保持不变。 这可以确保应用服务计划在切换不同的层时，应用不会突然发生行为上的变化，并且不会以意外的方式失败。

<a id="developmentframeworks"></a>

## <a name="development-frameworks"></a>开发框架
应用服务定价层控制可用于应用的计算资源（CPU、磁盘存储、内存和网络出口）的数量。 但是，可用于应用的框架功能范围保持不变，而与缩放层无关。

应用服务支持多种开发框架，包括 ASP.NET、经典 ASP、node.js、PHP 和 Python – 它们全都作为 IIS 中的扩展运行。 为了简化和标准化安全配置，应用服务应用通常使用其默认设置运行不同的开发框架。 用于配置应用的一个方法可能是为每个单独的开发框架自定义 API 外围应用和功能。 而应用服务则是通过实现操作系统功能的公共基准，采用更通用的方法，与应用的开发框架无关。

以下部分概述了可用于应用服务应用的一般类型的操作系统功能。

<a id="FileAccess"></a>

## <a name="file-access"></a>文件访问
应用服务中存在各种不同的驱动器，包括本地驱动器和网络驱动器。

<a id="LocalDrives"></a>

### <a name="local-drives"></a>本地驱动器
就其核心而言，应用服务是在 Azure PaaS（平台即服务）基础结构的基础上运行的服务。 因此，“附加到”虚拟机的本地驱动器是可用于在 Azure 中运行的任何辅助角色的相同驱动器类型。 这包括：

- 操作系统驱动器 (D:\ drive)
- 包含应用服务独占使用（且客户不可访问）的 Azure 包 cspkg 文件的应用程序驱动器
- “user”驱动器 (C:\ drive)，其大小因虚拟机大小而异。 

随着应用程序增长，请务必监视磁盘利用率。 如果达到了磁盘配额，可能会对应用程序产生负面影响。 例如： 

- 应用可能会引发错误，指示磁盘上没有足够的空间。
- 浏览到 Kudu 控制台时，可能会看到磁盘错误。
- 从 Azure DevOps 或 Visual Studio 进行部署可能会失败并显示 `ERROR_NOT_ENOUGH_DISK_SPACE: Web deployment task failed. (Web Deploy detected insufficient space on disk)`。
- 你的应用可能会出现性能下降。

<a id="NetworkDrives"></a>

### <a name="network-drives-aka-unc-shares"></a>网络驱动器（即 UNC 共享）
应用服务中有一个独具特色的方面能够简化应用的部署和维护，这就是所有用户内容都存储在一组 UNC 共享中。 此模型很好地映射到具有多个负载均衡服务器的本地 Web 托管环境所用内容存储的公共模式。 

在应用服务内，每个数据中心都创建了许多 UNC 共享。 在每个数据中心针对所有客户的某个百分比的用户内容将分配给各 UNC 共享。 此外，单个客户的订阅的所有文件内容将始终置于相同的 UNC 共享中。 

由于 Azure 服务的工作方式，负责承载 UNC 共享的特定虚拟机将随着时间而更改。 应确保由不同虚拟机装入 UNC 共享，因为在正常 Azure 操作过程中它们会启动和关闭。 因此，应用应该永远不会作出这样的硬编码的假定，即 UNC 文件路径中的计算机信息会在一段时间后保持不变。 相反，它们应使用应用服务提供的方便的 *faux* 绝对路径 **D:\home\site**。 此 faux 绝对路径为引用自己的网站提供可移植的应用到用户未知方法。 通过使用 **D:\home\site**，可以在应用之间传输共享文件，而不必为每次传输都配置新的绝对路径。

<a id="TypesOfFileAccess"></a>

### <a name="types-of-file-access-granted-to-an-app"></a>向应用授予的文件访问的类型
每个客户的订阅都在一个数据中心内的特定 UNC 共享上具有保留的目录结构。 客户可以在特定数据中心内创建多个应用，因此，属于单个客户订阅的所有目录都在同一个 UNC 共享上创建。 该共享可以包含目录（例如针对内容、错误和诊断日志的目录）以及源代码管理创建的应用的更早版本。 按照预期，客户的应用目录可用于在运行时由应用的应用程序代码进行读写访问。

在附加到运行应用的虚拟机的本地驱动器上，应用服务在 C:\ 驱动器上为特定于应用的临时本地存储预留一处空间。 尽管应用对自己的临时本地存储具有完全读/写访问权限，但该存储实际上并不旨在直接供应用程序代码使用。 而是用于为 IIS 和 Web 应用程序框架提供临时文件存储。 应用服务还限制可用于每个应用的临时本地存储量，以免单个应用占用过多的本地文件存储量。

两个说明应用服务如何使用临时本地存储的示例分别针对的是临时 ASP.NET 文件的目录和 IIS 压缩文件的目录。 ASP.NET 编译系统使用“临时 ASP.NET 文件”目录作为临时编译缓存位置。 IIS 使用“IIS 临时压缩文件”目录存储压缩的响应输出。 在应用服务中，这两种类型的文件使用（以及其他使用）都重新映射到按应用临时本地存储。 此重新映射确保该功能按预期延续。

应用服务中的每个应用作为随机的唯一低权限辅助进程标识运行，该标识名为“应用程序池标识”，以下网页做了进一步的介绍：[https://www.iis.net/learn/manage/configuring-security/application-pool-identities](https://www.iis.net/learn/manage/configuring-security/application-pool-identities)。 应用程序代码将此标识由于对操作系统驱动器（D:\ 驱动器）的基本的只读访问。 这意味着应用程序代码可以列出公共目录结构并且读取操作系统驱动器上的公共文件。 尽管这可能看上去就好像是一种较为广泛的访问级别，但在 Azure 托管服务中设置某一辅助角色并且读取驱动器内容时，相同的目录和文件是可访问的。 

<a name="multipleinstances"></a>

### <a name="file-access-across-multiple-instances"></a>跨多个实例的文件访问
主目录包含应用的内容，并且应用程序代码可以写入该目录。 如果应用在多个实例上运行，则主目录在所有实例间共享，以便所有实例都看到同一个目录。 所以，举例来说，如果应用将上传的文件保存到主目录，则所有实例都可以立即使用那些文件。 

<a id="NetworkAccess"></a>

## <a name="network-access"></a>网络访问
应用程序代码可以使用基于 TCP/IP 和 UDP 的协议建立与公开外部服务的 Internet 可访问终结点的出站网络连接。 应用可以使用这些相同协议连接到 Azure 内的服务&#151;例如，建立与 SQL 数据库的 HTTPS 连接即可实现此目的。

还有有限容量以便为应用建立一个本地环回连接，并且让应用侦听该本地环回套接字。 此功能存在主要是为了实现作为其功能的一部分侦听本地环回套接字的应用。 每个应用监视一个“专用”环回连接。 应用“A”无法侦听应用“B”创建的本地环回套接字。

还支持命名管道作为共同运行应用的不同进程之间的进程间通信 (IPC) 机制。 例如，IIS FastCGI 模块依赖命名管道协调运行 PHP 页的单独进程。

<a id="Code"></a>

## <a name="code-execution-processes-and-memory"></a>代码执行、进程和内存
如前所述，应用使用随机应用程序池标识在低权限辅助进程内运行。 应用程序代码有权访问与辅助进程相关联的内存空间，以及可由 CGI 处理器或其他应用程序生成的任何子进程。 但是，一个应用不能访问另一个应用的内存或数据，即使它们位于同一个虚拟机上。

应用可以运行使用支持的 Web 开发框架编写的脚本或页面。 应用服务不将任何 Web 框架设置配置为更受限制的模式。 例如，在应用服务上运行的 ASP.NET 应用以“完全”信任运行，与更受限制的信任模式相反。 Web 框架（包括经典 ASP 和 ASP.NET）可以调用进程中 COM 组件（但不能调用进程外 COM 组件），例如在 Windows 操作系统上默认注册的 ADO（ActiveX 数据对象）。

应用可以生成和运行任意代码。 允许应用执行诸如生成命令外壳程序或运行 PowerShell 脚本之类的任务。 但是，即使可以从应用生成任意代码和进程，可执行程序和脚本仍会被限制为授予父应用程序池的权限。 例如，应用可以生成发出出站 HTTP 调用的可执行文件，但同一个可执行文件不能尝试从其 NIC 取消绑定某个虚拟机的 IP 地址。 允许向低权限的代码发出出站网络调用，但尝试在虚拟机上重新配置网络设置要求管理权限。

<a id="Diagnostics"></a>

## <a name="diagnostics-logs-and-events"></a>诊断日志和事件
日志信息是某些应用尝试访问的另外一组数据。 可用于在应用服务中运行的代码的日志信息类型包括应用生成的诊断和日志信息，这些信息对于应用而言也是可以轻松进行访问的。 

例如，某一活动应用生成的 W3C HTTP 日志既可以在为该应用创建的网络共享位置中的日志目录上提供，也可以在 blob 存储中提供（如果客户已经将 W3C 日志记录设置到存储中）。 后者能够收集大量日志，而没有超出与某一网络共享相关联的文件存储限制的风险。

在类似情况下，还可以使用 .NET 跟踪和诊断基础结构将来自 .NET 应用的实时诊断信息记入日志，并且可以选择是将跟踪信息写入应用的网络共享还是写入 blob 存储位置。

诊断日志记录和跟踪中不可用于应用的领域是 Windows ETW 事件以及常见的 Windows 事件日志（例如系统、应用程序和安全事件日志）。 因为 ETW 跟踪信息可能在计算机范围中是可查看的（具有正确的 ACL），所以，将阻止对 ETW 事件的读写访问。 开发人员可能会注意到，用于读取和写入 ETW 事件和常见 Windows 事件日志的 API 调用好像在起作用，但这是因为应用服务在“伪装”这些调用，让它们看起来很成功。 实际上，应用程序代码对于此事件数据没有访问权限。

<a id="RegistryAccess"></a>

## <a name="registry-access"></a>注册表访问
应用对于它们在其上运行的虚拟机的注册表的大部分内容（尽管不是全部内容）具有只读访问权限。 实际上，这意味着应用可以访问允许对本地用户组进行只读访问的注册表项。 注册表中当前不支持读写访问的一个区域是 HKEY\_CURRENT\_USER Hive。

对注册表的写访问被阻止，包括对任何按用户注册表项的访问。 从应用角度来说，对注册表的写访问永远不应依赖于 Azure 环境，因为应用可以（并且也是这样做的）跨不同虚拟机进行迁移。 应用可依赖的唯一持久可写入存储是在应用服务 UNC 共享上存储的按应用内容目录结构。 

## <a name="remote-desktop-access"></a>远程桌面访问

应用服务不提供对 VM 实例的远程桌面访问。

## <a name="more-information"></a>详细信息

[Azure 应用服务沙盒](https://github.com/projectkudu/kudu/wiki/Azure-Web-App-sandbox) - 有关应用服务的执行环境的最新信息。 直接由应用服务开发团队维护此页。

