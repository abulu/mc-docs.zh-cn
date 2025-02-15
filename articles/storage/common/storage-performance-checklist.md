---
title: Azure 存储性能和伸缩性清单 | Microsoft Docs
description: 在开发使用 Azure 存储的高性能应用程序时，一个经过验证的检查表。
services: storage
author: WenJason
ms.service: storage
ms.topic: article
origin.date: 06/07/2019
ms.date: 07/15/2019
ms.author: v-jay
ms.subservice: common
ms.openlocfilehash: 7ccf69a9792984d73b0def36959150d9524c6399
ms.sourcegitcommit: 80336a53411d5fce4c25e291e6634fa6bd72695e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/12/2019
ms.locfileid: "67844410"
---
# <a name="azure-storage-performance-and-scalability-checklist"></a>Azure 存储性能和可伸缩性清单

自 Azure 存储服务发布以来，Azure 已经积累了大量经过验证的做法，用于提高这些服务的使用效率。本文将其中最重要的一些做法进行了总结，并以“清单”的形式列出。 本文的目的在于确保应用程序开发人员在使用 Azure 存储时，采用的是经过验证的做法，并帮助他们确认其他经过验证的、可以考虑采用的做法。 本文不会全盘介绍所有可能的性能与伸缩性优化内容，那些影响不大或适用范围不广的内容不在本文所覆盖的范围之内。 必须在设计过程中确保应用程序的行为是可预测到的，因此应早些了解这些经过验证的做法，避免进行那些会引发性能问题的设计。  

每一位使用 Azure 存储的应用程序开发人员都应抽时间阅读本文，确保其应用程序的设计遵循下面列出的每一项经过验证的做法。  

## <a name="checklist"></a>清单

本文将这些经过验证的做法整理成以下类别。 经过验证的做法适用于：  

* 所有 Azure 存储服务（Blob、表、队列和文件）
* Blob
* 表
* 队列  

| 完成 | 区域 | Category | 问题 |
| --- | --- | --- | --- |
| &nbsp; | 所有服务 |可伸缩性目标 |[是否将应用程序设计为避免接近可伸缩性目标？](#subheading1) |
| &nbsp; | 所有服务 |可伸缩性目标 |[命名约定是否旨在实现更好的负载均衡？](#subheading47) |
| &nbsp; | 所有服务 |网络 |[客户端设备是否具有足够高的带宽和足够低的延迟，以实现所需的性能？](#subheading2) |
| &nbsp; | 所有服务 |网络 |[客户端设备是否具有足够高的高质量链接？](#subheading3) |
| &nbsp; | 所有服务 |网络 |[客户端应用程序的位置是否“靠近”存储帐户？](#subheading4) |
| &nbsp; | 所有服务 |内容分发 |[是否要使用 CDN 进行内容分发？](#subheading5) |
| &nbsp; | 所有服务 |直接客户端访问 |[是否要使用 SAS 和 CORS 以允许直接访问存储而非代理？](#subheading6) |
| &nbsp; | 所有服务 |缓存 |[应用程序是否会缓存反复使用且很少更改的数据？](#subheading7) |
| &nbsp; | 所有服务 |缓存 |[应用程序是否会对更新进行批处理（将更新缓存在客户端，等到数量较多时再批量上传）？](#subheading8) |
| &nbsp; | 所有服务 |.NET 配置 |[是否已将客户端配置为使用足够数量的并发连接？](#subheading9) |
| &nbsp; | 所有服务 |.NET 配置 |[是否已将 .NET 配置为使用足够数量的线程？](#subheading10) |
| &nbsp; | 所有服务 |.NET 配置 |[是否会使用 .NET 4.5 或更高版本，以此来改进垃圾收集？](#subheading11) |
| &nbsp; | 所有服务 |并行度 |[是否能够确保对并行度进行了恰当的界定，使客户端功能或可伸缩性目标不会出现过载现象？](#subheading12) |
| &nbsp; | 所有服务 |工具 |[是否会使用 Microsoft 提供的最新版客户端库和工具？](#subheading13) |
| &nbsp; | 所有服务 |重试 |[是否会对限制错误和超时使用指数性的回退重试策略？](#subheading14) |
| &nbsp; | 所有服务 |重试 |[对于不可重试的错误，应用程序是否会避免重试？](#subheading15) |
| &nbsp; | Blob |可伸缩性目标 |[是否拥有大量并发访问单个对象的客户端？](#subheading46) |
| &nbsp; | Blob |可伸缩性目标 |[应用程序是否会保持在针对单个 Blob 的带宽或操作可伸缩性目标范围之内？](#subheading16) |
| &nbsp; | Blob |复制 Blob |[Blob 复制操作方式是否高效？](#subheading17) |
| &nbsp; | Blob |复制 Blob |[是否会使用 AzCopy 对 Blob 进行批量复制？](#subheading18) |
| &nbsp; | Blob |复制 Blob |[是否会使用 Azure 导入/导出来传输大量的数据？](#subheading19) |
| &nbsp; | Blob |使用元数据 |[是否会将频繁使用的有关 Blob 的元数据存储在其元数据中？](#subheading20) |
| &nbsp; | Blob |快速上传 |[尝试快速上传一个 Blob 时，是否会以并行方式上传块？](#subheading21) |
| &nbsp; | Blob |快速上传 |[尝试快速上传许多 Blob 时，是否会以并行方式上传 Blob？](#subheading22) |
| &nbsp; | Blob |正确的 Blob 类型 |[是否会根据需要使用页 Blob 或块 Blob？](#subheading23) |
| &nbsp; | 表 |可伸缩性目标 |[是否在接近实体数/秒的可伸缩性目标？](#subheading24) |
| &nbsp; | 表 |配置 |[是否使用 JSON 进行表请求？](#subheading25) |
| &nbsp; | 表 |配置 |[是否已关闭 Nagle 以改进小型请求的性能？](#subheading26) |
| &nbsp; | 表 |表和分区 |[是否已对数据进行了适当的分区？](#subheading27) |
| &nbsp; | 表 |热分区 |[是否会避免使用仅追加和仅预置模式？](#subheading28) |
| &nbsp; | 表 |热分区 |[插入/更新的内容是否会分布在多个分区中？](#subheading29) |
| &nbsp; | 表 |查询范围 |[是否已将架构设计为允许在大多数情况下使用点查询，尽量少用表查询？](#subheading30) |
| &nbsp; | 表 |查询密度 |[查询是否通常只扫描和返回应用程序会使用的行？](#subheading31) |
| &nbsp; | 表 |限制返回的数据 |[是否使用筛选来避免返回不需要的实体？](#subheading32) |
| &nbsp; | 表 |限制返回的数据 |[是否使用投影来避免返回不需要的属性？](#subheading33) |
| &nbsp; | 表 |非规范化 |[是否已对数据实施非规范化，以此避免在尝试获取数据时无效的查询或多次读取请求？](#subheading34) |
| &nbsp; | 表 |插入/更新/删除 |[是否会对需要进行事务处理或可以同时完成的请求进行批处理，以此减少不必要的重复操作？](#subheading35) |
| &nbsp; | 表 |插入/更新/删除 |[是否会避免仅仅为了确定是否需要调用插入或更新而检索某个实体？](#subheading36) |
| &nbsp; | 表 |插入/更新/删除 |[是否考虑过将各种需要频繁检索的数据作为属性一起存储在单个实体中而非多个实体中？](#subheading37) |
| &nbsp; | 表 |插入/更新/删除 |[对于那些始终需要一起检索并可成批写入的实体（例如时序数据），是否考虑过使用 Blob 而非表？](#subheading38) |
| &nbsp; | 队列 |可伸缩性目标 |[是否在接近消息数/秒的可伸缩性目标？](#subheading39) |
| &nbsp; | 队列 |配置 |[是否已关闭 Nagle 以改进小型请求的性能？](#subheading40) |
| &nbsp; | 队列 |消息大小 |[消息是否经过压缩，以此来改进队列的性能？](#subheading41) |
| &nbsp; | 队列 |批量检索 |[是否会使用单个“Get”操作检索多个消息？](#subheading42) |
| &nbsp; | 队列 |轮询频率 |[是否会进行足够频繁的轮询，以减少感知到的应用程序的延迟？](#subheading43) |
| &nbsp; | 队列 |更新消息 |[是否会使用 UpdateMessage 来存储消息处理进度，这样，在发生错误时则无需重新处理整个消息？](#subheading44) |
| &nbsp; | 队列 |体系结构 |[是否会将长时间运行的工作负载置于关键路径之外，以便使用队列来提高整个应用程序的可伸缩性，再进行独立的伸缩？](#subheading45) |

## <a name="allservices"></a>所有服务

本部分列出的经过验证的做法适用于任何 Azure 存储服务（Blob、表、队列或文件）。  

### <a name="subheading1"></a>可伸缩性目标

Azure 存储本身的限制是在每个订阅/每个区域创建的存储帐户数不能超过 250 个。 如果达到该限制，将无法在该订阅/区域组合中创建更多的存储帐户。

每项 Azure 存储服务都有针对容量 (GB)、事务处理速率和带宽的可伸缩性目标。 如果应用程序接近或超过任何可伸缩性目标，则可能会出现事务处理延迟或限制越来越严重的现象。 当某个存储服务对应用程序进行限制时，该服务在进行部分存储事务处理时会开始返回“503 服务器忙”或“500 操作超时”错误代码。 本部分讨论处理可伸缩性目标的通用方法以及具体的带宽可伸缩性目标。 在后面讲述各个存储服务的部分中，会讨论使用该特定服务时的可伸缩性目标：  

* [Blob 带宽和请求数/秒](#subheading16)
* [表实体数/秒](#subheading24)
* [队列消息数/秒](#subheading39)  

#### <a name="sub1bandwidth"></a>所有服务的带宽可伸缩性目标

在撰写本文之际，中国异地冗余存储 (GRS) 帐户的入口（发送到存储帐户的数据）带宽目标为每秒 5 千兆位 (Gbps)，出口（从存储帐户发送的数据）带宽目标则为 10 Gbps。 对于本地冗余存储 (LRS) 帐户，限制更高 - 10 Gbps（入口）和 15 Gbps（出口）。  请参阅[可伸缩性目标页](https://docs.azure.cn/storage/common/storage-scalability-targets)。  有关存储冗余选项的详细信息，请参阅下面“有用的资源”中的链接。  

#### <a name="what-to-do-when-approaching-a-scalability-target"></a>接近可伸缩性目标时应怎么办

如果即将达到特定订阅/区域组合中的存储帐户限制，请评估应用程序和存储帐户的用量，并确定是否存在以下任一情况。

* 使用存储帐户作为非托管磁盘，并将这些磁盘添加到虚拟机。 对于这种情况，我们建议使用[托管磁盘](../../virtual-machines/windows/managed-disks-overview.md)，因为它们可以自动处理存储磁盘的可伸缩性，你无需创建和管理单个存储帐户。
* 对每个客户使用一个存储帐户，以实现数据隔离。 对于这种情况，我们建议对每个客户使用存储容器，而不要使用一个完整的存储帐户。 Azure 存储现在允许[按容器](storage-auth-aad-rbac-portal.md)指定基于角色的访问控制。
* 使用多个存储帐户进行分片，以提高流入量/流出量/IOPS/容量的可伸缩性。 对于这种情况，我们建议在可能的情况下，利用标准存储帐户的更高限制来减少工作负荷所需的存储帐户数量。

如果应用程序正接近单个存储帐户的可伸缩性目标，可考虑采用以下方法之一：  

* 重新考虑导致应用程序接近或超过可伸缩性目标的工作负载。 能否对其进行另外的设计，以便使用较少的带宽、容量或处理事务？
* 如果某个应用程序肯定会超出可伸缩性目标之一，则应创建多个存储帐户并将应用程序数据跨多个这样的存储帐户进行分区。 如果使用这种模式，则在设计应用程序时，必须确保能够在以后添加更多的存储帐户，以便进行负载均衡。 在撰写本文之际，每个 Azure 订阅最多可以有 100 个存储帐户。  存储帐户除了用于数据存储、事务处理或数据传输之外，并无其他开销。
* 如果应用程序达到了带宽目标，可考虑压缩客户端的数据，以便减少将数据发送到存储服务所需的带宽。  这虽然会节省带宽并改进网络性能，但也会带来某些负面影响。  因此应该评估一下此操作对性能的影响，因为在客户端中压缩和解压缩数据会有其他处理要求。 此外，存储压缩数据会使问题解决起来更加困难，因为使用标准工具查看存储的数据可能会更困难。
* 如果应用程序达到了可伸缩性目标，则请确保对重试使用指数性回退（请参阅[重试](#subheading14)）。  这会确保应用程序不会一直进行快速重试，以免加重限制，不过，最好还是使用上述方法之一来确保永远不会达到可伸缩性目标。  

#### <a name="useful-resources"></a>有用的资源

以下链接提供了有关可伸缩性目标的更多详细信息：

* 有关可伸缩性目标的信息，请参阅 [Azure 存储可伸缩性和性能目标](storage-scalability-targets.md)。
* 有关存储冗余选项的信息，请参阅 [Azure 存储复制](storage-redundancy.md)和博客文章 [Azure Storage Redundancy Options and Read Access Geo Redundant Storage](https://blogs.msdn.com/b/windowsazurestorage/archive/2013/12/11/introducing-read-access-geo-replicated-storage-ra-grs-for-windows-azure-storage.aspx)（Azure 存储冗余选项和读取访问异地冗余存储）。
* 有关 Azure 服务定价的最新信息，请参阅 [Azure 定价](https://www.azure.cn/pricing/overview/)。  

### <a name="subheading47"></a>分区命名约定

Azure 存储使用基于范围的分区方案来对系统进行缩放和负载均衡。 分区键（帐户+容器+Blob）用于将数据分区成多个范围，会在整个系统上对这些范围进行负载均衡。 这意味着，命名约定，如词汇顺序（例如 *mypayroll*、*myperformance*、*myemployees* 等）或使用时间戳（*log20160101*、*log20160102*、*log20160102* 等）将本身出租给可能共置于同一分区服务器中的分区，直到负载均衡操作将它们拆分成较小的范围。 例如，容器中的所有 Bob 可接受单个服务器的服务，直到这些 Blob 的负载需要进一步进行重新平衡分区范围。 同样，一组名称按词汇顺序排列的少量负载帐户可以接受单个服务器的服务，直到其中一个或所有帐户的负载请求它们拆分到多个分区服务器。 每个负载均衡操作可能在操作期间会影响存储调用的延迟。 系统处理某个分区流量激增的能力受到单个分区服务器可伸缩性的限制，直到负载均衡操作着手重新平衡分区键范围。

可以遵循一些最佳实践来降低此类操作的频率。  

* 可能情况下，请使用较大的放置 Blob 或放置块大小（标准帐户大于 4 MiB，高级帐户大于 256 KiB），以便激活高吞吐量块 Blob (HTBB)。 HTBB 提供不受分区命名影响的高性能引入。
* 仔细检查帐户、容器、Blob、表和队列使用的命名约定。 考虑使用最符合需求的哈希函数，在帐户、容器或 Blob 名前加上 3 位数哈希。  
* 如果使用时间戳或数字标识符组织数据，必须确保使用的不是仅附加在后（或仅在前面加上）的流量模式。 这些模式并不适合基于范围的分区系统，并且可能导致所有流量进入单个分区并限制系统进行有效的负载均衡。 例如，如果日常操作使用有时间戳的 Blob 对象，如 *yyyymmdd*，则该日常操作的所有流量都定向到由单个分区服务器服务的单个对象。 查看每个 Blob 的限制和每个分区的限制是否符合需求，考虑是否需要将此操作拆分成多个 Blob。 同样，如果在表中存储时序数据，则所有流量都可能定向到键命名空间的最后一个部分。 如果必须使用时间戳或数字 ID，请在 ID 前面加上 3 位数哈希，或在时间戳前面加上时间的秒部分，如 *ssyyyymmdd*。 如果定期执行列出和查询操作，请选择限制查询次数的哈希函数。 对于其他情况，使用随机前缀便已足够。  
* 有关 Azure 存储中使用的分区方案的其他信息，请参阅 [Azure 存储：具有高度一致性的高可用云存储服务](https://sigops.org/sosp/sosp11/current/2011-Cascais/printable/11-calder.pdf)。

### <a name="networking"></a>网络

虽然 API 调用有一定的影响，但通常情况下物理网络对应用程序的约束具有更大的性能影响。 以下描述了用户可能会遇到的某些限制。  

#### <a name="client-network-capability"></a>客户端网络功能

##### <a name="subheading2"></a>吞吐量

通常情况下，对带宽来说，问题在于客户端的功能。 例如，虽然单个存储帐户可以处理 10 Gbps 或以上的入口吞吐量（参阅[带宽可伸缩性目标](#sub1bandwidth)），但在“小型”Azure 辅助角色实例中，网络速度只能达到大约 100 Mbps。 较大的 Azure 实例的 NIC 具有较大的容量，因此如果需要提高单个计算机的网络限制，则应考虑使用较大的实例或更多 VM。 如果从本地应用程序访问存储服务，则可应用相同的规则：了解客户端设备的网络功能以及与 Azure 存储位置的网络连接情况，并根据需要对其进行改进，或者将应用程序设计为可在这种网络功能下工作。  

##### <a name="subheading3"></a>链接质量

请注意，因错误和数据包丢失而导致的网络状况会降低有效吞吐量，使用任何网络都是这样。  WireShark 或 NetMon 可用于诊断此问题。  

##### <a name="useful-resources"></a>有用的资源

有关虚拟机大小和分配带宽的详细信息，请参阅 [Windows VM 大小](../../virtual-machines/windows/sizes.md?toc=%2fvirtual-machines%2fwindows%2ftoc.json)或 [Linux VM 大小](../../virtual-machines/windows/sizes.md?toc=%2fvirtual-machines%2flinux%2ftoc.json)。  

#### <a name="subheading4"></a>位置

在任何分布式环境中，将客户端放置在服务器附近可提供最佳性能。 要以最低的延迟访问 Azure 存储，则最好是将客户端放置在同一 Azure 区域内。 例如，如果 Azure 网站使用 Azure 存储，则应将二者都放置在同一个区域（例如中国东部或中国北部）。 这会降低延迟和成本 — 在本文撰写之际，同一个区域的带宽使用是免费的。  

如果客户端应用程序不是托管在 Azure 中（例如使用的是移动设备应用或本地企业服务），则同样将存储帐户放置在靠近需要访问该帐户的设备的区域通常会降低延迟。 如果客户端分布广泛（例如，某些客户端在北京，某些客户端在上海），则应考虑使用多个存储帐户：一个位于中国北部区域，一个位于中国东部区域。 这有助于降低这两个区域的用户的延迟。 如果应用程序存储的数据是特定于各个用户的，不需要在存储帐户之间复制数据，则此方法更容易实施。  如果内容分发范围很广泛，则建议使用 CDN – 详见下一部分。  

### <a name="subheading5"></a>内容分发

有时，应用程序需要向位于同一区域或多个区域的许多用户提供相同的内容（例如网站主页中使用的产品演示视频）。 在这种情况下，应该使用内容分发网络 (CDN)，例如 Azure CDN，该 CDN 会使用 Azure 存储作为数据的源。 与存在于一个区域且无法以低延迟向其他区域交付内容的 Azure 存储帐户不同，Azure CDN 使用位于全世界多个数据中心的服务器。 此外，与单个存储帐户相比，CDN 通常可以支持更高的出口限制。  

有关 Azure CDN 的详细信息，请参阅 [Azure CDN](https://www.azure.cn/home/features/cdn/)。  

### <a name="subheading6"></a>使用 SAS 和 CORS

当需要在用户的 Web 浏览器或移动电话应用中对 JavaScript 之类的代码授权以访问 Azure 存储中的数据时，一种方法是使用代理形式的 Web 角色应用程序：用户的设备通过 Web 角色进行身份验证，从而授权对存储资源的访问。 这样，就可以避免在不安全的设备上公开存储帐户密钥。 但是，这会大大增加 Web 角色的开销，因为在用户设备和存储服务之间传输的所有数据都必须通过 Web 角色。 只需使用共享访问签名 (SAS) 便可避免使用 Web 角色作为存储服务的代理，有时候还需结合使用跨域资源共享标头 (CORS)。 使用 SAS，可以让用户的设备通过受限访问令牌直接向存储服务提出请求。 例如，如果某个用户想要将照片上传到应用程序，Web 角色就会生成一个 SAS 令牌并将其发送到用户的设备，从而授予用户在随后的 30 分钟内（此时间过后 SAS 令牌会失效）向特定 Blob 或容器执行写入操作的权限。

通常情况下，浏览器不会允许某个域上的网站所托管的页面中的 JavaScript 执行特定的操作，例如针对其他域的“PUT”操作。 例如，如果在“contosomarketing.chinacloudapp.cn”上托管了一个 Web 角色，同时想要使用客户端 JavaScript 将一个 Blob 上传到“contosoproducts.blob.core.chinacloudapi.cn”上的存储帐户，则浏览器的“同源策略”会禁止该操作。 CORS 是一种浏览器功能，该功能允许目标域（在本示例中为存储帐户）与符合后列条件的浏览器进行通信：目标域信任该浏览器从源域（在本示例中为 Web 角色）发出的请求。  

这两种技术都有助于避免 Web 应用程序上出现不必要的负载（和瓶颈）。  

#### <a name="useful-resources"></a>有用的资源

有关 SAS 的详细信息，请参阅[共享访问签名，第 1 部分：了解 SAS 模型](../storage-dotnet-shared-access-signature-part-1.md)。  

有关 CORS 的详细信息，请参阅 [Cross-Origin Resource Sharing (CORS) Support for the Azure Storage Services](https://msdn.microsoft.com/library/azure/dn535601.aspx)（对 Azure 存储服务的跨域资源共享 (CORS) 支持）。  

### <a name="caching"></a>缓存

#### <a name="subheading7"></a>获取数据

通常情况下，从服务获取一次数据比两次要好。 考虑这样一个示例：一个 MVC Web 应用程序以某个 Web 角色运行，该角色已从存储服务检索了 50 MB 的 Blob 并将其作为内容提供给了某个用户。 然后，该应用程序就可以在每次有用户请求该 Blob 时检索同一 Blob，也可以将其缓存在本地磁盘上，将缓存的版本重复用于后续用户请求。 此外，每当有用户请求该数据时，应用程序还可以发出一个带修改时间的条件性标头的 GET 调用，避免在没有对 Blob 进行修改的情况下获取整个 Blob。 使用表实体时，可以应用此同一个模式。  

在某些情况下，可以选择让应用程序假定 Blob 在检索后短时间内保持有效，在这段时间内应用程序不需要查看是否对该 Blob 进行了修改。

配置、查看以及始终由应用程序使用的其他数据都非常适合进行缓存。  

有关如何通过使用 .NET 获取 Blob 的属性来发现上次修改日期的示例，请参阅[设置和检索属性与元数据](../blobs/storage-properties-metadata.md)。 有关条件性下载的详细信息，请参阅 [Conditionally Refresh a Local Copy of a Blob](https://msdn.microsoft.com/library/azure/dd179371.aspx)（有条件地刷新 Blob 的本地副本）。  

#### <a name="subheading8"></a>批量上传数据

在某些应用程序方案中，可以将数据聚合在本地，然后定期批量进行上传，而不必立即上传每个数据片段。 例如，Web 应用程序可以保留一个有关活动的日志文件：应用程序可以在活动发生时以表实体的形式上传每项活动的详细信息（这需要许多存储操作），也可以将活动详细信息保存到本地日志文件中，然后定期将所有活动详细信息作为带分隔符的文件上传到某个 Blob。 如果每个日志条目的大小为 1 KB，则可以在单个“放置 Blob”事务处理中上传数千个这样的条目（可以在单个事务处理中上传一个最大大小为 64 MB 的 Blob）。 如果本地计算机在上传之前崩溃，则可能会丢失某些日志数据：应用程序开发人员必须针对可能发生的客户端设备故障或上传失败情况进行相应的设计。  如果活动数据需要在不同的时间范围进行下载（不仅仅是单个活动），则建议使用 Blob 而非表。

### <a name="net-configuration"></a>.NET 配置

如果使用的是 .NET Framework，则本部分列出的数种快速配置设置可以用于显著提高性能。  如果使用其他语言，则需查看类似的概念是否适用于所选择的语言。  

#### <a name="subheading9"></a>提高默认连接限制

在 .NET 中，以下代码可将默认的连接限制（通常在客户端环境中为 2，在服务器环境中为 10）提高到 100。 通常情况下，应将值大致设置为应用程序使用的线程数。  

```csharp
ServicePointManager.DefaultConnectionLimit = 100; //(Or More)  
```

在打开任何连接前必须设置连接限制。  

对于其他编程语言，请参阅该语言的文档以确定如何设置连接限制。  

有关详细信息，请参阅博客文章 [Web 服务：Concurrent Connections](https://blogs.msdn.com/b/darrenj/archive/2005/03/07/386655.aspx)（Web 服务：并发连接）。  

#### <a name="subheading10"></a>如果对异步任务使用同步代码，则请增加最小线程数

此代码会增加线程池中的最小线程数：  

```csharp
ThreadPool.SetMinThreads(100,100); //(Determine the right number for your application)  
```

有关详细信息，请参阅 [ThreadPool.SetMinThreads 方法](https://msdn.microsoft.com/library/system.threading.threadpool.setminthreads%28v=vs.110%29.aspx)。  

#### <a name="subheading11"></a>充分利用 .NET 4.5 及更高版本的垃圾回收功能

将 .NET 4.5 或更高版本用于客户端应用程序，以便充分利用在服务器垃圾回收方面的性能改进。

有关详细信息，请参阅以下文章： [.NET 4.5 性能改进概述](https://msdn.microsoft.com/magazine/hh882452.aspx)。  

### <a name="subheading12"></a>不受限制的并行度

虽然提高并行度可以大幅提高性能，但在使用不受限制的并行度（对线程数和/或并行请求数没有限制）来上传或下载数据，以及使用多个辅助角色来访问同一存储帐户中的多个分区（容器、队列或表分区）或访问同一分区中的多个项目时，应小心谨慎。 如果并行度不受限制，应用程序则可能会超出客户端设备的承受程度或超出存储帐户的可伸缩性目标，导致延迟和限制时间增长。  

### <a name="subheading13"></a>存储客户端库和工具

始终使用 Microsoft 提供的最新客户端库和工具。 在本文撰写之际，已发布针对 .NET、Windows Phone、Windows 运行时、Java 和 C++ 的客户端库，以及针对其他语言的预览库。 此外，Microsoft 还发布了适用于 Azure 存储的 PowerShell cmdlet 和 Azure CLI 命令。 Microsoft 正在积极开发这些以性能为主要考量的工具，并使用最新服务版本对其进行更新，确保这些工具可以在内部协调好许多经过验证的做法。  

### <a name="retries"></a>重试

#### <a name="subheading14"></a>限制和服务器繁忙错误

在某些情况下，存储服务可能会限制应用程序，或者因某些暂时性的状态而无法处理请求，因而会返回“503 服务器忙”或“500 超时”的消息。  如果应用程序正接近某个可伸缩性目标，或者系统正在对分区数据进行重新平衡以提高吞吐量，则可能会发生这种情况。  通常情况下，客户端应用程序应重试引发此类错误的操作：稍后尝试同一请求可能会成功。 不过，如果是存储服务因为应用程序超出可伸缩性目标而限制应用程序，或者是由其他某种原因导致服务无法处理请求，则过于频繁的重试通常会使问题更糟。 因此，应使用指数性回退（客户端库默认采取此行为）。 例如，应用程序可能会在 2 秒后、4 秒后、10 秒后，以及 30 秒后进行重试，最后彻底放弃重试。 此行为会使应用程序显著降低对服务的负载，而不会使问题更糟。  

连接错误可以立即重试，因为它不是限制造成的，而且应该是暂时性的。  

#### <a name="subheading15"></a>不可重试的错误

客户端库了解哪些错误可以重试，哪些不能。 不过，如果针对存储 REST API 编写自己的代码，则应记住某些错误不应重试：例如，400（请求错误）响应表示客户端应用程序发送的请求因格式不正确而无法处理。 重新发送此请求每次都会导致相同的响应，因此没必要重试。 如果针对存储 REST API 编写自己的代码，则需了解错误代码的意义，以及每个错误代码的合适重试方式（或者不进行重试）。  

#### <a name="useful-resources"></a>有用的资源

有关存储错误代码的详细信息，请参阅 Azure 网站上的 [Status and Error Codes](https://msdn.microsoft.com/library/azure/dd179382.aspx) （状态和错误代码）。  

## <a name="blobs"></a>Blob

除了前面所述的适用于[所有服务](#allservices)的经过验证的做法，还有以下经过验证的做法，这些做法尤其适用于 Blob 服务。  

### <a name="blob-specific-scalability-targets"></a>特定于 Blob 的可伸缩性目标

#### <a name="subheading46"></a>并发访问单个对象的多个客户端

如果拥有并发访问单个对象的大量客户端，则需要要考虑每个对象和存储帐户的可伸缩性目标。 可以访问单个对象的客户端的具体数量根据各种因素（如同时请求对象的客户端数量、对象的大小、网络连接等）而有所不同。

如果对象可以通过 CDN 来进行分发（如网站服务提供的图像或视频），则应该使用 CDN。 [请参阅](#subheading5)。

在其他情况下，例如在数据保密的科学模拟下，有两个选项。 第一个选项是错开工作负荷的访问，以确保在某个时间段内访问对象，而不是同时访问对象。 另一个选项是暂时将对象复制到多个存储帐户，从而增加每个对象和存储帐户内的 IOPS 总数。 经过有限的测试，我们发现大约 25 台 VM 可以同时并行下载 100 GB 的 Blob（每台 VM 使用 32 个线程并行化下载）。 如果有 100 个客户端需要访问某个对象，请先将该对象复制到另一个存储帐户，然后让前 50 台 VM 访问第一个 Blob，让剩下的 50 台 VM 访问第二个 Blob。 结果根据应用程序的行为而有所不同，因此应该在设计期间对此进行测试。 

#### <a name="subheading16"></a>每个 Blob 的带宽和操作

可以读取或写入单个 Blob，最大读/写速度为 60 MB/秒。这大约相当于 480 Mbps，超出了许多客户端网络（包括客户端设备上的物理 NIC）的承受能力。 此外，单个 Blob 每秒最多可支持 500 个请求。 如果有多个客户端需要读取同一 Blob，而且可能会超过这些限制，则应考虑使用 CDN 来分发该 Blob。  

有关 Blob 的目标吞吐量的详细信息，请参阅 [Azure 存储可伸缩性和性能目标](storage-scalability-targets.md)。  

### <a name="copying-and-moving-blobs"></a>复制和移动 Blob

#### <a name="subheading17"></a>复制 Blob

存储 REST API 2012-02-12 版引入了跨帐户复制 Blob 这一有用功能：客户端应用程序可以指示存储服务从其他源复制 Blob（可能属于不同的存储帐户），然后让服务异步执行复制。 在从其他存储帐户迁移数据时，这可以显著减少应用程序所需的带宽，因为不需要下载和上传数据。  

不过，需要考虑的一个问题是，在存储帐户之间进行复制时，无法确保复制的完成时间。 如果应用程序需要在控制之下快速完成 Blob 复制，则在复制该 Blob 时，最好是先将其下载到某个 VM，再将其上传到目标位置。  在这种情况下，若要实现完全可预测性，请确保复制由在同一 Azure 区域中运行的 VM 执行，否则网络状况可能会（极可能会）影响复制性能。  此外，还可以通过编程方式监视异步复制的进程。  

在同一存储帐户内的复制通常会快速完成。  

有关详细信息，请参阅 [Copy Blob](https://msdn.microsoft.com/library/azure/dd894037.aspx)（复制 Blob）。  

#### <a name="subheading18"></a>使用 AzCopy

Azure 存储团队发布了命令行工具“AzCopy”，该工具用于通过存储帐户来回批量传输多个 Blob，以及跨多个存储帐户进行批量传输。  该工具已针对此方案进行了优化，可以实现较高的传输速率。  建议将其用于需要批量上传、批量下载和批量复制的情况。 若要了解关于它的详细信息并下载它，请参阅 [使用 AzCopy 命令行实用程序传输数据](storage-use-azcopy.md)。  

#### <a name="subheading19"></a>Azure 导入/导出服务

对于大量的数据（超过 1 TB），Azure 存储提供了导入/导出服务，允许用户通过寄送硬盘驱动器的方式从 Blob 存储上传和下载数据。  用户可以将数据置于硬盘驱动器上，并将其寄送到 Azure 进行上传，也可以将空的硬盘驱动器寄给 Azure 来下载数据。  有关详细信息，请参阅[使用 Azure 导入/导出服务将数据传输到 Blob 存储中](../storage-import-export-service.md)。  对于这种大小的数据，此方式可能比通过网络上传/下载要高效得多。  

### <a name="subheading20"></a>使用元数据

Blob 服务支持 head 请求，这其中可能包含有关 Blob 的元数据。 例如，如果应用程序需要某张照片中的 EXIF 数据，则可以检索该照片，并从中提取数据。 为了节省带宽并改进性能，应用程序可能会在上传照片时将 EXIF 数据存储在 Blob 的元数据中：随后只需使用 HEAD 请求便可检索元数据中的 EXIF 数据，这样就可以在每次读取 Blob 时，显著节省带宽和提取 EXIF 数据所需的处理时间。 在只需元数据而不需要 Blob 的完整内容时，这种方法很有用。  每个 Blob 只能存储 8 KB 的元数据（该服务不会接受数据大小超过此要求的存储请求），因此如果数据大小不符合该要求，则可能无法使用此方法。  

有关如何使用 .NET 获取 Blob 的元数据的示例，请参阅[设置和检索属性与元数据](../blobs/storage-properties-metadata.md)。  

### <a name="rapid-uploading"></a>快速上传

若要快速上传 Blob，首要问题是：是要上传一个还是多个 Blob？  请参阅以下指南，根据具体情况确定要使用的正确方法。  

#### <a name="subheading21"></a>快速上传一个大型 Blob

若要快速上传单个大型 Blob，客户端应用程序应并行上传其块或页（需考虑各个 Blob 的可伸缩性目标并综合考虑存储帐户的情况）。  Microsoft 提供的正式的 RTM 存储客户端库（.NET、Java）具有此方面的功能。  每个库均应使用下面指定的对象/属性来设置并发级别：  

* .NET:将 BlobRequestOptions 对象上的 ParallelOperationThreadCount 设置为“used”。
* Java/Android：使用 BlobRequestOptions.setConcurrentRequestCount()
* Node.js：将 parallelOperationThreadCount 用于请求选项或 Blob 服务。
* C++：使用 blob_request_options::set_parallelism_factor 方法。

#### <a name="subheading22"></a>快速上传多个 Blob

若要快速上传多个 Blob，请以并行方式上传。 此方法要快于通过并行块上传方式一次上传一个 Blob，因为这种情况下上传会分布到存储服务的多个分区中。 单个 Blob 仅支持 60 MB/秒（大约 480 Mbps）的吞吐量。 在本文撰写之际，一个 LRS 帐户最多可支持 10 Gbps 的入口吞吐量，远远大于单个 Blob 支持的吞吐量。  [AzCopy](#subheading18) 以并行方式执行上传操作，建议用于此方案。  

### <a name="subheading23"></a>选择正确的 Blob 类型

Azure 存储支持两种类型的 Blob：页  Blob 和块  Blob。 在给定的使用方案中，对 Blob 类型的选择会影响到解决方案的执行情况和可伸缩性。 需要以高效方式上传大量数据时，适合选择块 Blob：例如，客户端应用程序可能需要将照片或视频上传到 Blob 存储中。 如果应用程序需要对数据执行随机写入，则应选择页 Blob：例如，Azure VHD 以页 Blob 方式存储。  

有关详细信息，请参阅 [Understanding Block Blobs, Append Blobs, and Page Blobs](https://msdn.microsoft.com/library/azure/ee691964.aspx)（了解块 Blob、追加 Blob 和页 Blob）。  

## <a name="tables"></a>表

除了前面所述的适用于 [所有服务](#allservices) 的经过验证的做法，还有以下经过验证的做法，这些做法尤其适用于表服务。  

### <a name="subheading24"></a>特定于表的可伸缩性目标

除了针对整个存储帐户的带宽限制，表还有以下特定的可伸缩性限制。  系统会在流量增加时进行负载均衡，但如果流量激增，吞吐量可能不会相应地突增。  如果使用的模式包含流量激增情况，则会在激增期间出现限制和/或超时现象，因为存储服务会自动将表负载均衡掉。  让流量缓慢增加通常会有更好的效果，因为这给系统提供了进行适当负载均衡的时间。  

#### <a name="entities-per-second-storage-account"></a>实体数/秒（存储帐户）

对于单个帐户来说，访问表时的可伸缩性限制高达每秒 20,000 个实体（每个实体 1 KB）。  一般情况下，每个被插入、更新、删除或扫描的实体都会计入此目标的计数。  因此，包含 100 个实体的批量插入计为 100 个实体。  一个查询扫描了 1000 个实体但只返回 5 个，则会将其计为 1000 个实体。  

#### <a name="entities-per-second-partition"></a>实体数/秒（分区）

在单个分区中，访问表时的可伸缩性目标为每秒 2,000 个实体（每个实体 1 KB），使用前面部分所述的相同计数方法。  

### <a name="configuration"></a>配置

本部分列出了多个快速配置设置，可以使用这些设置显著提高表服务的性能：  

#### <a name="subheading25"></a>使用 JSON

从存储服务 2013-08-15 版开始，表服务就支持使用 JSON 而非基于 XML 的 AtomPub 格式来传输表数据。 这最多可以减少 75% 的负载大小，可以显著改进应用程序的性能。

有关详细信息，请参阅文章 [Azure 表：JSON 简介](https://blogs.msdn.com/b/windowsazurestorage/archive/2013/12/05/windows-azure-tables-introducing-json.aspx)和[表服务操作的有效负载格式](https://msdn.microsoft.com/library/azure/dn535600.aspx)。

#### <a name="subheading26"></a>禁用 Nagle

Nagle 的算法已跨 TCP/IP 网络进行了广泛的实施，是一种改进网络性能的方法。 不过，该方法并非适用于所有情况（例如高度交互式的环境）。 对于 Azure 存储，Nagle 的算法会对表请求和队列服务请求的执行造成负面影响，因此应尽可能禁用。

### <a name="schema"></a>架构

数据的呈现和查询方式是影响表服务性能的单个最大因素。 虽然每个应用程序都不同，但本部分仍概要列出了一些通用的经过验证的做法，这些做法适用于：  

* 表设计
* 高效的查询
* 高效的数据更新  

#### <a name="subheading27"></a>表和分区

表划分为分区。 存储在分区中的每个实体共享相同的分区键，并具有唯一的行键，用于在该分区中标识自己。 分区具有好处，但也带来了可伸缩性限制。  

* 好处：可以在同一个分区中更新单个事务、原子事务和批处理事务的实体，每种事务最多包含 100 个单独的存储操作（总大小限制为 4 MB）。 此外，假定需要检索相同数量的实体，则在单个分区中查询数据要比跨多个分区查询数据更高效（不过，如果需要查询表数据，则请继续阅读以获取更进一步的建议）。
* 可伸缩性限制：对存储在单个分区中的实体的访问不能进行负载均衡，因为分区支持原子批处理事务。 因此，总体说来单个表分区的可伸缩性目标低于表服务的相应目标。  

考虑到表和分区的这些特点，应该采用以下设计原则：  

* 客户端应用程序在同一逻辑工作单元中频繁更新或查询的数据应位于同一分区。  这可能是因为应用程序需要聚合写入，或者是因为需要充分利用原子批处理操作。  此外，与跨分区的数据相比，可以更高效地对单个分区中的数据进行查询。
* 客户端应用程序不在同一逻辑工作单元中插入/更新或查询（单个查询或批量更新）的数据应位于不同的分区中。  必须指出的是，单个表中的分区键没有数量限制，因此即使设置数百万个分区键也不是问题，也不会影响性能。  例如，如果应用程序是一个需要用户登录的热门网站，不妨使用用户 ID 作为分区键。  

#### <a name="hot-partitions"></a>热分区

热分区是指这样一种分区，即收到了某个帐户的过多流量，但又无法对其进行负载均衡，因为该分区为单个分区。  一般情况下，热分区的创建有以下两种模式：  

##### <a name="subheading28"></a>仅追加和仅预置模式

“仅追加”模式是指流向某个给定 PK 的所有（或几乎所有）流量都会按当前时间增加或减少。  其中一个示例为，如果应用程序使用了当前日期作为日志数据的分区键，则会发生这种情况。  这会造成所有的插入都会进到表中的最后一个分区，而系统无法进行负载均衡，因为所有的写入都会追加到表的末尾。  如果进入该分区的流量超出分区级的可伸缩性目标，则会导致限制。  最好是确保将流量发送到多个分区，以便对跨表请求进行负载均衡。  

##### <a name="subheading29"></a>高流量数据

如果分区方案导致单个分区的数据较其他分区的数据使用更为频繁，则也可能会看到限制现象，因为该分区达到了单个分区的可伸缩性目标。  最好是确保分区方案不会导致单个分区接近可伸缩性目标。  

#### <a name="querying"></a>查询

本部分介绍用于查询表服务的经过验证的做法。  

##### <a name="subheading30"></a>查询范围

有多种方式可指定需要查询的实体的范围。  下面讨论如何使用每种方式。  

通常情况下，应避免进行扫描（大于单个实体的查询），但如果必须要进行扫描，则应尝试对数据进行组织，使扫描仅检索所需数据，避免扫描或返回大量不需要的实体。  

###### <a name="point-queries"></a>点查询

点查询正好只检索一个实体。 通过同时指定要检索的实体的分区键和行键来实现此功能。 此类查询非常高效，应尽可能使用。  

###### <a name="partition-queries"></a>分区查询

分区查询用于检索共享分区键的一组数据。 通常情况下，该查询会指定一系列行键值或者一系列用于某些实体属性和分区键的值。 此类查询效率不如点查询，应谨慎使用。  

###### <a name="table-queries"></a>表查询

表查询用于检索没有共享分区键的一组实体。 此类查询效率不高，应尽可能避免使用。  

##### <a name="subheading31"></a>查询密度

影响查询效率的另一关键因素是返回的实体数与查找返回的集合时扫描过的实体数的比率。 如果应用程序在执行表查询时使用了某个属性值的筛选器，而该属性值仅供 1% 的数据共享，则该查询需要扫描 100 个实体才会返回 1 个实体。 前面讨论的表可伸缩性目标均与所扫描的实体数相关，与返回的实体数无关：查询密度低很容易导致表服务限制应用程序，因为表服务在检索要查找的实体时需要扫描的实体过多。  请参阅下面有关 [非规范化](#subheading34) 的部分，以了解有关如何避免这种情况的详细信息。  

##### <a name="limiting-the-amount-of-data-returned"></a>限制返回的数据量

###### <a name="subheading32"></a>筛选

如果知道某个查询返回的实体并不是客户端应用程序所需要的，则应考虑使用筛选器来减少返回的集合的大小。 虽然没有返回到客户端的实体仍会计入可伸缩性限制，但应用程序的性能会提高，因为网络负载大小会下降，同时客户端应用程序必须处理的实体数会下降。  请参见上面有关[查询密度](#subheading31)的说明，然而，由于可伸缩性目标与扫描的实体数相关，因此查询在筛选掉许多实体后仍可能导致限制，即使返回很少的实体。  

###### <a name="subheading33"></a>投影

如果客户端应用程序只需表中实体提供的一组有限的属性，则可以使用投影来限制所返回数据集的大小。 就像使用筛选一样，这有助于减少网络负载和客户端处理。  

##### <a name="subheading34"></a>非规范化

与使用关系数据库不同，根据经过验证的做法，若要提高表数据的查询效率，需对数据进行非规范化。 也就是说，需要将相同的数据复制到多个实体中（一个实体对应一个用于查找数据的键）以尽量降低查询在查找客户端所需数据时必须扫描的实体数，这样就不必扫描大量实体来查找应用程序需要的数据。  例如，在电子商务网站中，可能希望通过两种方式查找订单：按客户 ID（提供此客户的订单）和按日期（提供某个日期的订单）。  在表存储中，最好是将实体（或者对实体的引用）存储两次 – 一次使用表名称、PK 和 RK 进行存储，以便能够按客户 ID 快速查找，另一次则通过日期来加快查找速度。  

#### <a name="insertupdatedelete"></a>插入/更新/删除

本部分介绍的经过验证的做法用于修改存储在表服务中的实体。  

##### <a name="subheading35"></a>批处理

批处理事务在 Azure 存储中也称为实体组事务 (ETG)；ETG 中的所有操作都必须位于单个表的单个分区中。 在可能的情况下，请使用 ETG 来批量执行插入、更新和删除操作。 这会减少客户端应用程序与服务器之间的往返操作次数、减少需要收费的事务数（一个 ETG 计为一个收费事务，最多可能包含 100 个存储操作），并启用原子更新（ETG 中的所有操作都成功或都失败）。 高延迟性的环境（例如移动设备）可以充分利用 ETG。  

##### <a name="subheading36"></a>Upsert

尽可能使用表的“Upsert”  操作。 有两种类型的“Upsert”  ，两种都可能比传统的“插入”  和“更新”  操作更高效：  

* **InsertOrMerge**：需要上传一部分实体的属性，但不确定该实体是否存在时，可使用此操作。 如果实体存在，则该调用会更新包含在“Upsert”  操作中的属性，保留所有现有的属性不变，而如果实体不存在，则会插入新的实体。 这类似于在查询中使用投影，因为只需上传在更改的属性。
* **InsertOrReplace**：需要上传某个全新的实体，却又不确定该实体是否存在时，可使用此操作。 仅当知道这个刚上传的实体完全正确时，才应使用此操作，因为该实体会完全覆盖旧实体。 例如，需要更新用于存储用户当前位置的实体，而不管应用程序以前是否存储过该用户的位置数据；新位置实体是完整的，不需要任何旧实体提供的任何信息。

##### <a name="subheading37"></a>将数据系列存储在单个实体中

有时候，应用程序会存储一系列需要频繁进行一次性检索的数据：例如，应用程序可能会跟踪一段时间内的 CPU 使用情况，以便绘制过去 24 小时内数据的滚动图表。 一种方法是每小时构建一个表实体，每个实体代表一个具体的小时，并存储该小时的 CPU 使用情况。 为了针对该数据绘图，应用程序需要检索保留过去 24 小时内数据的实体。  

此外，也可以让应用程序将每小时的 CPU 使用情况存储为单个实体的独立属性：更新每个小时的时候，应用程序可以使用单个 **InsertOrMerge Upsert** 调用来更新最近的一个小时的值。 针对数据进行绘图时，应用程序只需检索 1 个实体而非 24 个，这样的查询非常高效（请参阅上面有关[查询范围](#subheading30)的讨论）。

##### <a name="subheading38"></a>在 Blob 中存储结构化数据

有时候，本应将结构化数据置于表中，但由于各种实体始终是一起检索的，因此可以将其批量插入。  这方面的一个很好的示例是日志文件。  在这个示例中，可以批处理数分钟的日志，将其插入，然后又可以始终一次性检索数分钟的日志。  在这个示例中，使用 Blob 要比使用表的性能更好，因为可以显著降低写入/返回的对象数，并且通常情况下还可以显著降低需要发出的请求数。  

## <a name="queues"></a>队列

除了前面所述的适用于[所有服务](#allservices)的经过验证的做法，还有以下经过验证的做法，这些做法尤其适用于队列服务。  

### <a name="subheading39"></a>可伸缩性限制

单个队列可以处理大约 2,000 条消息（每条 1 KB）/秒（在这里，每个 AddMessage、GetMessage 和 DeleteMessage 均计为一条消息）。 如果这对应用程序来说还不够用，则应使用多个队列并将这些消息分散到队列中去。  

在 [Azure 存储可伸缩性和性能目标](storage-scalability-targets.md)中查看最新的可伸缩性目标。  

### <a name="subheading40"></a>禁用 Nagle

请参阅有关表配置的部分，其中讨论了 Nagle 算法 — Nagle 算法通常不适合执行队列请求，应禁用。  

### <a name="subheading41"></a>消息大小

队列性能和可伸缩性随消息增大而下降。 只应将接收方需要的信息置于消息中。  

### <a name="subheading42"></a>批量检索

可以在单个操作中从队列检索多达 32 条消息。 这可以降低客户端应用程序的往返操作数，尤其适合延迟性严重的环境，例如移动设备。  

### <a name="subheading43"></a>队列轮询间隔

大多数应用程序会轮询获取队列中的消息，队列可能是适合于该应用程序的事务的最大源之一。 理智选择轮询间隔：轮询过于频繁可能导致应用程序接近队列的可伸缩性目标。 不过，200,000 次事务仅收费 0.01 美元（在撰写本文时），一个处理器每秒轮询一次且持续一个月的成本不到 15 美分，因此成本通常不是影响选择轮询间隔的一项因素。  

如需最新成本信息，请参阅 [Azure 存储定价](https://www.azure.cn/pricing/details/storage/)。  

### <a name="subheading44"></a>更新消息

可以使用“更新消息”操作  来增加不可见性超时或更新消息的状态信息。 虽然此功能很强大，但请记住，每项“更新消息”  操作都会计入可伸缩性目标。 不过，与作业每完成一步就将其从一个队列传到下一个队列的工作流相比，此方法可能要高效得多。 使用“更新消息”  操作可以让应用程序将作业状态保存到消息，然后又可以继续工作，而不必在作业的每一步完成的时候，为了执行作业的下一步而将消息重新排队。  

有关详细信息，请参阅[如何：更改已排队消息的内容](../queues/storage-dotnet-how-to-use-queues.md#change-the-contents-of-a-queued-message)。  

### <a name="subheading45"></a>应用程序体系结构

应该使用队列，让应用程序体系结构具有可伸缩性。 下面列出了一些方法，可以通过这些方法，使用队列来提高应用程序的可伸缩性：  

* 可以使用队列来创建需要处理的积压工作，并消除应用程序中的工作负载。 例如，可以将用户发出的执行处理器密集型工作（例如，调整已上传图像的大小）的请求排队。
* 可以使用队列来分离应用程序的某些部分，以便对其进行独立的缩放操作。 例如，Web 前端可以将用户提交的调查结果置于某个队列中，留待以后分析和存储。 可以添加更多的辅助角色实例，使之按要求处理队列数据。  

## <a name="conclusion"></a>结论

本文讨论了部分最常用的、经过验证的做法，目的是优化使用 Azure 存储时的性能。 我们建议每一位应用程序开发人员对照以上每条做法，对自己的应用程序进行评估，并考虑按照建议要求进行操作，以便优化使用 Azure 存储的应用程序的性能。
