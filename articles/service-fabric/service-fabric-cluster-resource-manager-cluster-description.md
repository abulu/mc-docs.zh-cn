---
title: 群集资源管理器群集的描述 | Azure
description: 通过为群集资源管理器指定容错域、升级域、节点属性和节点容量来描述 Service Fabric 群集。
services: service-fabric
documentationcenter: .net
author: rockboyfor
manager: digimobile
editor: ''
ms.assetid: 55f8ab37-9399-4c9a-9e6c-d2d859de6766
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
origin.date: 08/18/2017
ms.date: 07/08/2019
ms.author: v-yeche
ms.openlocfilehash: fac163abba5861d2134c126f4a2902c965d2447b
ms.sourcegitcommit: 8f49da0084910bc97e4590fc1a8fe48dd4028e34
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/12/2019
ms.locfileid: "67844751"
---
# <a name="describing-a-service-fabric-cluster"></a>描述 Service Fabric 群集
Service Fabric 群集 Resource Manager 提供多种用于描述群集的的机制。 在运行时，群集 Resource Manager 使用此信息来确保群集中运行的服务的高可用性。 实施这些重要规则时，群集资源管理器还会尝试优化群集中的资源消耗。

## <a name="key-concepts"></a>关键概念
群集 Resource Manager 支持多种用于描述群集的功能：

* 容错域
* 升级域
* 节点属性
* 节点容量

## <a name="fault-domains"></a>容错域
容错域是协调故障的任何区域。 单一计算机就是一个容错域（因为它本身可能出于各种原因而发生故障，包括电源故障、驱动器故障、NIC 固件错误，等等）。 连接到同一以太网交换机的计算机位于同一个容错域中，共享一个电源或一个位置的计算机也是如此。 硬件故障在性质上是重叠的，因此容错域原本就有层次性，在 Service Fabric 中以 URI 的形式表示。

对容错域进行正确设置很重要，因为 Service Fabric 根据此信息来安全放置服务。 Service Fabric 不希望在放置服务后，容错域的丢失（由某个组件的故障而导致）会造成服务关闭。 在 Azure 环境中，Service Fabric 利用环境提供的容错域信息，代表用户正确配置群集中的节点。 对于独立的 Service Fabric，将在设置群集时定义容错域 

> [!WARNING]
> 请务必确保提供给 Service Fabric 的容错域信息准确无误。 例如，假设 Service Fabric 群集的节点在 5 个物理主机上运行的 10 台虚拟机中运行。 在此情况下，即使有 10 个虚拟机，也只有 5 个不同的（顶层）容错域。 共享同一物理主机会导致 VM 共享同一个根容错域，因此如果物理主机发生故障，共享主机的 VM 也会同时相应发生故障。  
>
> Service Fabric 预期节点的容错域不变。 确保 VM 高可用性的其他机制（例如 [HA-VM](https://technet.microsoft.com/library/cc967323.aspx)）可能会导致与 Service Fabric 冲突，因为它们使用主机间的 VM 透明迁移。 这些机制不会重新配置或通知 VM 内运行的代码。 因此，**不支持**这些机制作为运行 Service Fabric 群集的环境。 应使用 Service Fabric 作为唯一的高可用性技术。 实时 VM 迁移、SAN 等其他机制不是必要的机制。 如果将这些机制与 Service Fabric 一同使用，会降低应用程序的可用性和可靠性，因为这些机制会增加复杂性、加大导致并发故障的风险，并且这些机制所使用的可靠性和可用性策略可能会与 Service Fabric 中的策略相冲突  。 
>
>

下图中，我们将所有导致容错域的实体着色，并列出生成的所有不同容错域。 本示例列出了数据中心 (DC)、机架 (R) 和刀片服务器 (B)。 可以想象，如果每个刀片服务器包含多个虚拟机，则容错域层次结构中可能有另一个层。

<center>

![通过容错域组织的节点][Image1]
</center>

在运行时，Service Fabric 群集资源管理器会考虑群集中的容错域，并规划布局。 它会分发给定服务的有状态副本和无状态副本，使这些副本位于不同的容错域中。 容错域在层次结构的任一级别发生故障时，通过跨容错域分发服务可确保服务可用性不受影响。

Service Fabric 群集资源管理器不会考虑容错域层次结构中有多少层。 不过，它会尝试确保丢失层次结构的任何一部分不至于影响在其中运行的服务。 

最好是在容错域层次结构中的每个深度级别部署相同数目的节点。 如果群集中的容错域“树”不均衡，群集资源管理器会更难找到最佳的服务分配。 容错域布局失衡意味着，丢失某些域对服务可用性造成的影响比丢失其他域更严重。 因此，群集资源管理器难以在两个目标之间作出取舍：是要通过将服务放置在该“繁重”域中来使用计算机，还是要将服务放置在其他域中来避免域丢失造成问题。 

失衡的域是什么样子？ 下图显示了两种不同的群集布局。 在第一个示例中，节点均匀分散到容错域。 在第二个示例中，某一容错域包含的节点比其他容错域多出许多。 

<center>

![两种不同的群集布局][Image2]
</center>

在 Azure 中，系统会为用户选择哪个容错域包含节点。 但是，根据设置的节点数目，仍然可能出现某容错域比其他容错域承载更多节点的情况。 例如，假设群集中有 5 个容错域，但针对给定的 NodeType 预配了 7 个节点。 在这种情况下，前两个容错域会承载更多的节点。 如果继续使用几个实例部署更多的节点类型，问题会变得更严重。 因此，建议将每个节点类型中的节点数配置为容错域数的倍数。

## <a name="upgrade-domains"></a>升级域
升级域是另一项功能，可帮助 Service Fabric 群集资源管理器了解群集的布局。 升级域定义同时升级的节点集。 升级域可帮助群集资源管理器了解和安排诸如升级等管理操作。

升级域非常类似于容错域，但有几个重要的差异。 首先，容错域由协调性硬件故障区域定义。 而升级域由策略定义。 可以确定所需的数量，而不是由环境描述所需的数量。 升级域数量可以与节点数量相同。 容错域和升级域之间的另一个差异在于，升级域不是分层的。 相反，升级域更像是一个简单标记。 

下图显示了跨三个容错域条带化的三个升级域。 该图还显示了有状态服务的三个不同副本的一种可能的放置方式，即三个副本位于不同的容错域和升级域中。 这种放置方式容许在服务升级过程中丢失一个容错域，在这种情况下仍可运行一个代码和数据副本。  

<center>

![包含容错域和升级域的布局][Image3]
</center>

使用大量升级域既有利也有弊。 更多升级域意味着升级的每个步骤更细微，因此会给较少的节点或服务造成影响。 这样一来，每次只需移动较少的服务，进而降低系统中的流动。 这往往会提高可靠性，因为在升级过程中发生的任何问题所影响到的服务更少。 更多升级域也意味着处理升级的影响时，在其他节点上占用的缓冲区更少。 例如，如果有五个升级域，每个域中的节点处理大约 20% 的流量。 如果需要关闭升级域进行升级，则通常需要将负载转移到某个位置。 由于剩余有 4 个升级域，因此每个升级域必须具有可供 5% 的总流量使用的空间。 更多升级域意味着群集中节点上所需占用的缓冲区更少。 例如，考虑一下有 10 个升级域的情况。 在此情况下，每个 UD 仅处理约 10% 的总流量。 当开始升级群集时，每个域只需具有约 1.1% 的总流量空间即可。 使用更多升级域通常能够以更高的利用率运行节点，因为需要保留的容量更少。 这同样适用于容错域。  

拥有多个升级域的缺点是升级需要花费的时间更长。 当一个升级域完成升级后，Service Fabric 会等待一小段时间，并在开始升级下一个域之前执行检查。 通过这一延迟时间，可在继续升级之前检测升级带来的问题。 这种折衷是可接受的，因为可以防止错误的更改一次性影响过多的服务。

升级域太少会产生许多负面影响 - 当每个升级域关闭并进行升级时，大部分的整体容量均不可用。 例如，如果只有三个升级域，则一次只能采用整体服务或群集容量的 1/3。 让如此多的服务同时停止并非理想的结果，因为必须在其余的群集中拥有足够的容量才能处理工作负荷。 维护该缓冲区意味着在正常工作期间，这些节点比其他情况下的负载更少。 这会增加运行服务的成本。

环境中容错域或升级域的总数没有实际限制，对于它们如何重叠也没有约束。 尽管如此，却有几种常见模式：

- 容错域和升级域按 1:1 映射
- 每个节点一个升级域（物理或虚拟 OS 实例）
- “条带化”或“矩阵”模型，其中的容错域和升级域构成了通常沿对角线分布的计算机矩阵

<center>

![容错域和升级域布局][Image4]
</center>

至于要选择哪种布局并没有最佳答案，每种做法各有利弊。 例如，1FD:1UD 模型很容易设置。 最常见的模型是每个节点 1 个升级域。 升级过程中会独立更新每个节点。 这类似于过去手动升级少量计算机的方式。 

最常见的模型是 FD/UD 矩阵，其中 FD 和 UD 构成一个表，节点沿着对角线开始放置。 这是 Azure 中 Service Fabric 群集默认使用的模型。 对于具有多个节点的群集，最终都会形成与上述密集矩阵模式类似的模式。

## <a name="fault-and-upgrade-domain-constraints-and-resulting-behavior"></a>容错域与升级域约束及最终行为
### <a name="default-approach"></a>*默认方法*
默认情况下，群集资源管理器维持服务在故障域和升级域之间的平衡。 这将建模为[约束](service-fabric-cluster-resource-manager-management-integration.md)。 容错域和升级域约束如下所述：“对于给定的服务分区，同一层次结构级别上的任意两个域之间的服务对象（无状态服务实例或有状态服务副本）数量之差应永远不能大于 1”。 让我们假设此约束提供“最大差值”保证。 故障域和容错域约束会阻止违反上述规则的某些移动或排列。 

让我们看一个示例。 假设有一个 6 节点群集，其中配置了 5 个容错域和 5 个升级域。

|  | FD0 | FD1 | FD2 | FD3 | FD4 |
| --- |:---:|:---:|:---:|:---:|:---:|
| **UD0** |N1 | | | | |
| **UD1** |N6 |N2 | | | |
| **UD2** | | |N3 | | |
| **UD3** | | | |N4 | |
| **UD4** | | | | |N5 |

*配置 1*

现在，假设要创建一个 TargetReplicaSetSize（如果是无状态服务，则为 InstanceCount） 为 5 的服务。 副本驻留在 N1-N5 上。 事实上，无论创建多少个此类服务，都不会用到 N6。 为什么？ 看看当前布局和选择 N6 时所发生情况之间的差异。

下面是获得的布局，以及每个容错域和升级域的副本总数：

|  | FD0 | FD1 | FD2 | FD3 | FD4 | UDTotal |
| --- |:---:|:---:|:---:|:---:|:---:|:---:|
| **UD0** |R1 | | | | |1 |
| **UD1** | |R2 | | | |1 |
| **UD2** | | |R3 | | |1 |
| **UD3** | | | |R4 | |1 |
| **UD4** | | | | |R5 |1 |
| **FDTotal** |1 |1 |1 |1 |1 |- |

*布局 1*

此布局在每个容错域与升级域的节点数目上是均衡的。 并且在每个容错域和升级域的副本数目上也是均衡的。 每个域都拥有相同数量的节点，以及相同数量的副本。

现在让我们看看，如果不使用 N2，而是改用 N6 会发生什么情况。 副本会如何分布？

|  | FD0 | FD1 | FD2 | FD3 | FD4 | UDTotal |
| --- |:---:|:---:|:---:|:---:|:---:|:---:|
| **UD0** |R1 | | | | |1 |
| **UD1** |R5 | | | | |1 |
| **UD2** | | |R2 | | |1 |
| **UD3** | | | |R3 | |1 |
| **UD4** | | | | |R4 |1 |
| **FDTotal** |2 |0 |1 |1 |1 |- |

*布局 2*

此布局违反了容错域约束的“最大差值”保证的定义。 FD0 有 2 个副本，而 FD1 有 0 个，FD0 与 FD1 之间的总差为 2，这个总差已大于最大差值 1。 由于违反了约束，群集资源管理器不允许这种排列方式。 同样，如果选择 N2 和 N6（而不是 N1 和 N2），则会得到：

|  | FD0 | FD1 | FD2 | FD3 | FD4 | UDTotal |
| --- |:---:|:---:|:---:|:---:|:---:|:---:|
| **UD0** | | | | | |0 |
| **UD1** |R5 |R1 | | | |2 |
| **UD2** | | |R2 | | |1 |
| **UD3** | | | |R3 | |1 |
| **UD4** | | | | |R4 |1 |
| **FDTotal** |1 |1 |1 |1 |1 |- |

*布局 3*

就容错域而言，此布局是均衡。 但是，现在违反了升级域约束，因为 UD0 有 0 个副本，而 UD1 有 2 个。 因此，此布局也是无效的，群集资源管理器不会选取此布局。

这种有状态副本或无状态实例的分布方法提供了最佳容错能力。 在一个域关闭的情况下，丢失的副本/实例数最少。 

另一方面，此方法过于严格，不允许群集使用所有资源。 对于某些群集配置，某些节点不可用。 这可能导致 Service Fabric 无法放置服务，从而出现警告消息。 在之前的示例中，某些群集节点不可用（给定示例中的 N6）。 即使将节点添加到该群集 (N7 - N10)，由于容错域和升级域约束，副本/实例也只能放置在 N1 - N5 上。 

|  | FD0 | FD1 | FD2 | FD3 | FD4 |
| --- |:---:|:---:|:---:|:---:|:---:|
| **UD0** |N1 | | | |N10 |
| **UD1** |N6 |N2 | | | |
| **UD2** | |N7 |N3 | | |
| **UD3** | | |N8 |N4 | |
| **UD4** | | | |N9 |N5 |

*配置 2*

### <a name="alternative-approach"></a>*替换方法*

群集资源管理器支持另一版本的故障域和升级域约束，该版本在允许放置服务的同时仍然保证最低级别的安全。 备用容错和升级域约束如下所述：“对于给定的服务分区，跨域的副本分布应保证分区不会遭受仲裁丢失”。 让我们假设此约束提供“仲裁安全”保证。 

> [!NOTE]
>对于有状态服务，如果大部分分区副本同时关闭，则表示“仲裁丢失”  。 例如，如果 TargetReplicaSetSize 为 5，则任意三个副本一组表示仲裁。 同样，如果 TargetReplicaSetSize 为 6，则仲裁必须要 4 个副本。 如果分区想要继续正常运行，则在这两种情况下，不能同时关闭超过两个副本。 对于无状态服务，即使是大部分实例同时关闭，也没有类似“仲裁丢失”的情况，因为无状态服务可以继续正常运行  。 因此，文章的其余部分将着重介绍有状态服务。
>

让我们返回到上一个示例。 因为具有“仲裁安全”版本的约束，给定的所有三个布局都是有效的。 这是因为，即使第二个布局中的 FD0 或第三个布局中的 UD1 故障，该分区仍有仲裁（分区中的大部分副本仍可运行）。 由于该版本的约束，几乎始终使用 N6。

“仲裁安全”方法比“最大差值”方法更具灵活性，因为更容易在几乎所有群集拓扑中找到有效的副本分布。 但是，此方法不能保证最佳容错特性，因为有些故障比其他故障更为严重。 在最坏的情况下，大部分副本会因为某个域和某个额外的副本故障而丢失。 例如，相比 5 个副本或实例要求其中 3 个故障才丢失仲裁，现在仅 2 个故障就可能丢失大部分副本。 

### <a name="adaptive-approach"></a>*自适应方法*
由于这两种方法都有优缺点，现在我们将介绍结合了两种策略的自适应方法。

> [!NOTE]
> 从 Service Fabric 版本 6.2 起，这将成为默认行为。 
> 
> 自适应方法默认使用“最大差值”逻辑，并且仅在必要时切换为“仲裁安全”逻辑。 群集资源管理器通过查看群集和服务的配置方式，自动辨别必要的策略。 对于给定的服务：如果 TargetReplicaSetSize 能被故障域数和升级域数整除，并且节点数少于或等于容错域数乘以升级域数的积，则群集资源管理器应针对该服务使用“基于仲裁”逻辑 ** 。 请注意，群集资源管理器对无状态和有状态服务都将使用此方法，尽管仲裁丢失与无状态服务无关。

让我们返回到上一个示例，假设群集现在有 8 个节点（群集仍然配置了 5 个容错域和 5 个升级域，在该群集上托管的服务的 TargetReplicaSetSize 仍为 5）。 

|  | FD0 | FD1 | FD2 | FD3 | FD4 |
| --- |:---:|:---:|:---:|:---:|:---:|
| **UD0** |N1 | | | | |
| **UD1** |N6 |N2 | | | |
| **UD2** | |N7 |N3 | | |
| **UD3** | | |N8 |N4 | |
| **UD4** | | | | |N5 |

*配置 3*

由于满足所有必要条件，群集资源管理器将在分布服务时使用“基于仲裁”逻辑。 这将使用 N6 - N8。 本例中的一个可能的服务分布如下所示：

|  | FD0 | FD1 | FD2 | FD3 | FD4 | UDTotal |
| --- |:---:|:---:|:---:|:---:|:---:|:---:|
| **UD0** |R1 | | | | |1 |
| **UD1** |R2 | | | | |1 |
| **UD2** | |R3 |R4 | | |2 |
| **UD3** | | | | | |0 |
| **UD4** | | | | |R5 |1 |
| **FDTotal** |2 |1 |1 |0 |1 |- |

*布局 4*

如果服务的 TargetReplicaSetSize 减到 4（例如），群集资源管理器会注意到该变化并恢复使用“最大差值”逻辑，因为 TargetReplicaSetSize 不再能被 FD 数和 UD 数整除。 因此，为了将余下的 4 个副本分布在节点 N1-N5 上，将移动部分副本，这样才不会违反“最大差值”版本的容错域和升级域逻辑。 

回顾第四个布局，其 TargetReplicaSetSize 为 5。 如果 N1 从群集中删除，则升级域数变为 4。 同样，由于服务的 TargetReplicaSetSize 不再能被 UD 数整除，群集资源管理器开始使用“最大差值”逻辑。 因此，再次构建副本 R1 时，它必须位于 N4 上，这样才不会违反故障域和升级域约束。

|  | FD0 | FD1 | FD2 | FD3 | FD4 | UDTotal |
| --- |:---:|:---:|:---:|:---:|:---:|:---:|
| **UD0** |不适用 |不适用 |不适用 |不适用 |不适用 |不适用 |
| **UD1** |R2 | | | | |1 |
| **UD2** | |R3 |R4 | | |2 |
| **UD3** | | | |R1 | |1 |
| **UD4** | | | | |R5 |1 |
| **FDTotal** |1 |1 |1 |1 |1 |- |

*布局 5*

## <a name="configuring-fault-and-upgrade-domains"></a>配置容错域和升级域
对容错域和升级域的定义在 Azure 托管的 Service Fabric 部署中自动完成。 Service Fabric 从 Azure 中选择并使用环境信息。

如果要创建自己的群集（或者要在开发环境中运行特定的拓扑），则可自行提供容错域和升级域信息。 在本示例中，我们定义了一个 9 节点本地开发群集，该群集跨 3 个“数据中心”（每个数据中心有 3 个机架）。 该群集还有跨这三个数据中心条带化的三个升级域。 下面是一个配置示例： 

ClusterManifest.xml

```xml
  <Infrastructure>
    <!-- IsScaleMin indicates that this cluster runs on one-box /one single server -->
    <WindowsServer IsScaleMin="true">
      <NodeList>
        <Node NodeName="Node01" IPAddressOrFQDN="localhost" NodeTypeRef="NodeType01" FaultDomain="fd:/DC01/Rack01" UpgradeDomain="UpgradeDomain1" IsSeedNode="true" />
        <Node NodeName="Node02" IPAddressOrFQDN="localhost" NodeTypeRef="NodeType02" FaultDomain="fd:/DC01/Rack02" UpgradeDomain="UpgradeDomain2" IsSeedNode="true" />
        <Node NodeName="Node03" IPAddressOrFQDN="localhost" NodeTypeRef="NodeType03" FaultDomain="fd:/DC01/Rack03" UpgradeDomain="UpgradeDomain3" IsSeedNode="true" />
        <Node NodeName="Node04" IPAddressOrFQDN="localhost" NodeTypeRef="NodeType04" FaultDomain="fd:/DC02/Rack01" UpgradeDomain="UpgradeDomain1" IsSeedNode="true" />
        <Node NodeName="Node05" IPAddressOrFQDN="localhost" NodeTypeRef="NodeType05" FaultDomain="fd:/DC02/Rack02" UpgradeDomain="UpgradeDomain2" IsSeedNode="true" />
        <Node NodeName="Node06" IPAddressOrFQDN="localhost" NodeTypeRef="NodeType06" FaultDomain="fd:/DC02/Rack03" UpgradeDomain="UpgradeDomain3" IsSeedNode="true" />
        <Node NodeName="Node07" IPAddressOrFQDN="localhost" NodeTypeRef="NodeType07" FaultDomain="fd:/DC03/Rack01" UpgradeDomain="UpgradeDomain1" IsSeedNode="true" />
        <Node NodeName="Node08" IPAddressOrFQDN="localhost" NodeTypeRef="NodeType08" FaultDomain="fd:/DC03/Rack02" UpgradeDomain="UpgradeDomain2" IsSeedNode="true" />
        <Node NodeName="Node09" IPAddressOrFQDN="localhost" NodeTypeRef="NodeType09" FaultDomain="fd:/DC03/Rack03" UpgradeDomain="UpgradeDomain3" IsSeedNode="true" />
      </NodeList>
    </WindowsServer>
  </Infrastructure>
```

通过 ClusterConfig.json 实现独立部署

```json
"nodes": [
  {
    "nodeName": "vm1",
    "iPAddress": "localhost",
    "nodeTypeRef": "NodeType0",
    "faultDomain": "fd:/dc1/r0",
    "upgradeDomain": "UD1"
  },
  {
    "nodeName": "vm2",
    "iPAddress": "localhost",
    "nodeTypeRef": "NodeType0",
    "faultDomain": "fd:/dc1/r0",
    "upgradeDomain": "UD2"
  },
  {
    "nodeName": "vm3",
    "iPAddress": "localhost",
    "nodeTypeRef": "NodeType0",
    "faultDomain": "fd:/dc1/r0",
    "upgradeDomain": "UD3"
  },
  {
    "nodeName": "vm4",
    "iPAddress": "localhost",
    "nodeTypeRef": "NodeType0",
    "faultDomain": "fd:/dc2/r0",
    "upgradeDomain": "UD1"
  },
  {
    "nodeName": "vm5",
    "iPAddress": "localhost",
    "nodeTypeRef": "NodeType0",
    "faultDomain": "fd:/dc2/r0",
    "upgradeDomain": "UD2"
  },
  {
    "nodeName": "vm6",
    "iPAddress": "localhost",
    "nodeTypeRef": "NodeType0",
    "faultDomain": "fd:/dc2/r0",
    "upgradeDomain": "UD3"
  },
  {
    "nodeName": "vm7",
    "iPAddress": "localhost",
    "nodeTypeRef": "NodeType0",
    "faultDomain": "fd:/dc3/r0",
    "upgradeDomain": "UD1"
  },
  {
    "nodeName": "vm8",
    "iPAddress": "localhost",
    "nodeTypeRef": "NodeType0",
    "faultDomain": "fd:/dc3/r0",
    "upgradeDomain": "UD2"
  },
  {
    "nodeName": "vm9",
    "iPAddress": "localhost",
    "nodeTypeRef": "NodeType0",
    "faultDomain": "fd:/dc3/r0",
    "upgradeDomain": "UD3"
  }
],
```

> [!NOTE]
> 通过 Azure 资源管理器定义群集时，由 Azure 分配容错域和升级域。 因此，Azure 资源管理器模板中节点类型和虚拟机规模集的定义不包含容错域或升级域信息。
>

## <a name="node-properties-and-placement-constraints"></a>节点属性和放置约束
有时（事实上是大多数情况下），需要确保只在群集中特定类型的节点上运行某些工作负荷。 例如，某些工作负荷可能需要 GPU 或 SSD，而有些则不用。 将特定硬件用于特定工作负载的一个很好的示例是 n 层体系结构（几乎每个这样的体系结构都可看作是一个好例子）。 某些计算机充当应用程序的前端或 API 服务端，并向客户端或在 Internet 上公开。 其他一些计算机（通常具有不同的硬件资源）处理计算或存储层的工作。 通常不会直接向客户端或 Internet 公开这些计算机  。 Service Fabric 预期存在特定工作负荷需要在特定硬件配置上运行的情况。 例如：

* 现有的 n 层应用程序已“提升并迁移”到 Service Fabric 环境
* 出于性能、规模或安全性隔离原因，某个工作负荷需要在特定硬件上运行
* 出于策略或资源消耗原因，某个工作负荷应该与其他工作负荷隔离

为了支持这种配置，Service Fabric 提供可用于节点的一级标记概念。 这些标记称为**节点属性**。 **放置约束**是附加到单个服务的语句，这些服务专供 1 个或多个节点属性选择。 放置约束定义服务运行的位置。 约束集可扩展 - 任何键/值对都适用。 

<center>

![群集布局中的不同工作负荷][Image5]
</center>

### <a name="built-in-node-properties"></a>内置节点属性
Service Fabric 定义了一些默认节点属性，无需用户进行定义，系统即会自动使用这些属性。 在每个节点上定义的默认属性是 **NodeType** 和 **NodeName**。 因此举例而言，可以将放置约束编写为 `"(NodeType == NodeType03)"`。 通常来说，我们发现 NodeType 是最常用的属性之一。 它很有用，因为它与计算机的类型之间存在一一对应关系。 每种计算机类型都与一种传统 n 层应用程序的工作负荷类型相对应。

<center>

![放置约束和节点属性][Image6]
</center>

## <a name="placement-constraint-and-node-property-syntax"></a>放置约束和节点属性语法 
节点属性中指定的值可以是字符串、布尔值或带符号的长型值。 服务处的语句称为放置 *约束* ，它可以约束服务在群集中的运行位置。 约束可以是针对群集中的不同节点属性运行的任何布尔值语句。 这些布尔值语句中的有效选择器包括：

1) 用于创建特定语句的条件检查

| 语句 | 语法 |
| --- |:---:|
| “等于” | "==" |
| “不等于” | "!=" |
| “大于” | ">" |
| “大于等于” | ">=" |
| “小于” | "<" |
| “小于等于” | "<=" |

2) 用于分组和逻辑运算的布尔值语句

| 语句 | 语法 |
| --- |:---:|
| “和” | "&&" |
| “或” | "&#124;&#124;" |
| “非” | "!" |
| “分组为单个语句” | "()" |

下面是基本约束语句的一些示例。

  * `"Value >= 5"`
  * `"NodeColor != green"`
  * `"((OneProperty < 100) || ((AnotherProperty == false) && (OneProperty >= 100)))"`

只有整个放置约束语句求值为“True”的节点才能放置服务。 未定义属性的节点不匹配包含该属性的任何放置约束。

假设为给定节点类型定义了以下节点属性：

ClusterManifest.xml

```xml
    <NodeType Name="NodeType01">
      <PlacementProperties>
        <Property Name="HasSSD" Value="true"/>
        <Property Name="NodeColor" Value="green"/>
        <Property Name="SomeProperty" Value="5"/>
      </PlacementProperties>
    </NodeType>
```

通过 ClusterConfig.json 进行独立部署或将 Template.json 用于 Azure 托管群集。 

> [!NOTE]
> 在 Azure 资源管理器模板中，节点类型通常已参数化。 它类似于“[parameters('vmNodeType1Name')]”而不是“NodeType01”。
>

```json
"nodeTypes": [
    {
        "name": "NodeType01",
        "placementProperties": {
            "HasSSD": "true",
            "NodeColor": "green",
            "SomeProperty": "5"
        },
    }
],
```

可以针对服务创建服务放置 *约束* ，如下所示：

C#

```csharp
FabricClient fabricClient = new FabricClient();
StatefulServiceDescription serviceDescription = new StatefulServiceDescription();
serviceDescription.PlacementConstraints = "(HasSSD == true && SomeProperty >= 4)";
// add other required servicedescription fields
//...
await fabricClient.ServiceManager.CreateServiceAsync(serviceDescription);
```

Powershell：

```powershell
New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceType -Stateful -MinReplicaSetSize 3 -TargetReplicaSetSize 3 -PartitionSchemeSingleton -PlacementConstraint "HasSSD == true && SomeProperty >= 4"
```

如果 NodeType01 的所有节点都有效，则也可以使用约束“(NodeType == NodeType01)”选择该节点类型。

服务放置约束的一个突出优点是它们可以在运行时动态更新。 因此如果需要，可以在群集中移动服务、添加和删除要求，等等。Service Fabric 负责确保即使进行了这些类型的更改，服务仍保持运行且可供使用。

C#：

```csharp
StatefulServiceUpdateDescription updateDescription = new StatefulServiceUpdateDescription();
updateDescription.PlacementConstraints = "NodeType == NodeType01";
await fabricClient.ServiceManager.UpdateServiceAsync(new Uri("fabric:/app/service"), updateDescription);
```

Powershell：

```powershell
Update-ServiceFabricService -Stateful -ServiceName $serviceName -PlacementConstraints "NodeType == NodeType01"
```

放置约束是针对每个不同的命名服务实例指定的。 更新始终会取代（覆盖）以前指定的值。

群集定义用于定义节点上的属性。 更改节点属性需要升级群集配置。 升级节点属性需要重启每个受影响的节点以报告其新属性。 这些滚动升级由 Service Fabric 管理。

## <a name="describing-and-managing-cluster-resources"></a>描述和管理群集资源
任何协调器的最重要作业之一是帮助管理群集中的资源消耗。 而管理群集资源时需要注意的事项有所不同。 首先，确保计算机不会过载。 这是指确保计算机运行的服务数不超过其可处理的服务数。 其次，需要权衡和优化与高效运行服务息息相关的因素。 经济有效型或性能敏感型服务产品不允许某些节点处于热状态，而其他节点处于冷状态。 热节点会导致资源争用和性能不佳，而冷节点意味着资源浪费和成本增加。 

Service Fabric 使用 `Metrics`表示资源。 指标是要向 Service Fabric 描述的任何逻辑或物理资源。 指标的示例是诸如“WorkQueueDepth”或“MemoryInMb”的参数。 若要了解 Service Fabric 可在节点上调控的物理资源，请参阅[资源调控](service-fabric-resource-governance.md)。 有关配置自定义指标及其用法的信息，请参阅[此文](service-fabric-cluster-resource-manager-metrics.md)

指标与放置约束和节点属性不同。 节点属性是节点自身的静态描述符。 指标描述节点所含资源，以及当服务在节点上运行时服务所消耗的资源。 节点属性可能为“HasSSD”，可设置为 true 或 false。 该 SSD 上的可用空间量和服务消耗的空间量是类似于“DriveSpaceInMb”的指标。 

请注意，与放置约束和节点属性相同，Service Fabric 群集 Resource Manager 不理解指标名称的含义。 指标名称只是字符串。 建议不明确时，将单位声明为创建的指标名称的一部分。

## <a name="capacity"></a>容量
如果关闭所有的资源均衡功能，Service Fabric 群集资源管理器仍会确保最终不会有任何节点超出其容量  。 对容量溢出进行管理是可能的，除非群集过于饱和或工作负荷大于任何节点。 容量是群集 Resource Manager 用来了解节点包含的资源量的另一个 *约束* 。 还会针对整个群集来追踪剩余容量。 服务级别的容量和消耗量均以指标来表示。 因此举例而言，指标可能是“ClientConnections”，给定的节点可能拥有 32768 个单位的“ClientConnections”容量。 其他节点可能有其他限制。在该节点上运行的某一服务可以声明其当前正在消耗 32256 个单位的指标“ClientConnections”。

在运行时，群集资源管理器会跟踪群集中和节点上的剩余容量。 群集资源管理器通过从运行服务的节点容量中减去每个服务的使用量来跟踪容量。 使用此信息，Service Fabric 群集资源管理器可找出要放置或移动副本的位置，使节点不会超过容量。

<center>

![群集节点和容量][Image7]
</center>

C#：

```csharp
StatefulServiceDescription serviceDescription = new StatefulServiceDescription();
ServiceLoadMetricDescription metric = new ServiceLoadMetricDescription();
metric.Name = "ClientConnections";
metric.PrimaryDefaultLoad = 1024;
metric.SecondaryDefaultLoad = 0;
metric.Weight = ServiceLoadMetricWeight.High;
serviceDescription.Metrics.Add(metric);
await fabricClient.ServiceManager.CreateServiceAsync(serviceDescription);
```

Powershell：

```powershell
New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceTypeName -Stateful -MinReplicaSetSize 3 -TargetReplicaSetSize 3 -PartitionSchemeSingleton -Metric @("ClientConnections,High,1024,0)
```

可以在群集清单中看到定义的容量：

ClusterManifest.xml

```xml
    <NodeType Name="NodeType03">
      <Capacities>
        <Capacity Name="ClientConnections" Value="65536"/>
      </Capacities>
    </NodeType>
```

通过 ClusterConfig.json 进行独立部署或将 Template.json 用于 Azure 托管群集。 

```json
"nodeTypes": [
    {
        "name": "NodeType03",
        "capacities": {
            "ClientConnections": "65536",
        }
    }
],
```

通常，服务负载会以动态方式发生更改。 假设副本的“ClientConnections”负载从 1024 更改为 2048，但是当时正在运行该副本的节点只剩余了 512 个单位的该指标容量。 现在副本或实例的位置无效，因为该节点上没有足够的空间。 群集资源管理器必须介入，使节点上的容量消耗低于阈值。 这可通过将 1 个或多个副本或实例从该节点转移到其他节点，来减少超出容量的节点上的负载。 移动副本时，群集资源管理器会尝试将移动成本降至最低。 [此文](service-fabric-cluster-resource-manager-movement-cost.md)介绍了移动成本；若要深入了解群集资源管理器的重新均衡策略和规则，请参阅[此文](service-fabric-cluster-resource-manager-metrics.md)。

## <a name="cluster-capacity"></a>群集容量
那么，Service Fabric 群集资源管理器如何防止整个群集过于饱和？ 对于动态负载，基本上没有有效的解决方法。 服务可使自己的负载高峰独立于群集 Resource Manager 所执行的操作。 因此，群集当前或许拥有足够的容量，但将来需要扩大规模时，可能就不够用了。 尽管如此，仍有一些控件可以防止问题。 我们可做的第一件事是防止创建导致群集空间变满的新工作负荷。

假设要创建一个无状态服务，并且它有某些关联的负载。 假设服务需考虑“DiskSpaceInMb”指标。 并且假设服务的每个实例使用 5 个单位的“DiskSpaceInMb”。 需要创建服务的 3 个实例。 很好！ 这意味着我们需要群集中有 15 个单位的“DiskSpaceInMb”才能创建这些服务实例。 群集资源管理器会持续计算容量和每个指标的消耗量，因此它可以确定群集中的剩余容量。 如果没有足够的空间，群集资源管理器会拒绝创建服务调用。

因为只要求有 15 个可用单位，所以可以使用不同的方式分配此空间。 例如，可能是在 15 个不同节点上各有一个剩余单位的容量，或是在 5 个不同节点上各有三个剩余单位的容量。 如果群集资源管理器能够重新排列服务，在 3 个节点上提供 5 个单位，则可放置服务。 重新排列群集通常是可行的，除非群集几乎已满，或者出于某种原因无法合并现有服务。

## <a name="buffered-capacity"></a>缓冲容量
缓冲容量是群集资源管理器的另一项功能。 使用此功能可以保留总节点容量的一部分。 此容量缓冲区仅用于在升级期间和发生节点故障时放置服务。 缓冲容量是全局指定的，即针对所有节点按指标指定的。 为保留容量选择的值取决于群集中的容错域和升级域数目。 较多的容错域和升级域意味着可以选择较少数值的缓冲处理容量。 域越多，则升级和故障过程中无法使用的群集数量越少。 指定缓冲容量只有在同时指定了指标的节点容量时才有意义。

以下示例演示如何指定缓冲容量：

ClusterManifest.xml

```xml
        <Section Name="NodeBufferPercentage">
            <Parameter Name="SomeMetric" Value="0.15" />
            <Parameter Name="SomeOtherMetric" Value="0.20" />
        </Section>
```

通过用于独立部署的 ClusterConfig.json 或用于 Azure 托管群集的 Template.json：

```json
"fabricSettings": [
  {
    "name": "NodeBufferPercentage",
    "parameters": [
      {
          "name": "SomeMetric",
          "value": "0.15"
      },
      {
          "name": "SomeOtherMetric",
          "value": "0.20"
      }
    ]
  }
]
```

群集用于某个指标的缓冲容量不足时，创建新服务会失败。 通过防止创建新服务来保留缓冲区，可确保升级和故障不会造成节点超出容量。 缓冲容量是可选项，但建议为定义了指标容量的所有群集启用。

群集资源管理器会公开此负载信息。 对于每个指标，此信息包括： 
  - 缓冲容量设置
  - 总容量
  - 当前消耗量
  - 每项指标是否视为均衡
  - 有关标准偏差的统计信息
  - 负载最大和最小的节点  

下面提供了该输出的示例：

```powershell
PS C:\Users\user> Get-ServiceFabricClusterLoadInformation
LastBalancingStartTimeUtc : 9/1/2016 12:54:59 AM
LastBalancingEndTimeUtc   : 9/1/2016 12:54:59 AM
LoadMetricInformation     :
                            LoadMetricName        : Metric1
                            IsBalancedBefore      : False
                            IsBalancedAfter       : False
                            DeviationBefore       : 0.192450089729875
                            DeviationAfter        : 0.192450089729875
                            BalancingThreshold    : 1
                            Action                : NoActionNeeded
                            ActivityThreshold     : 0
                            ClusterCapacity       : 189
                            ClusterLoad           : 45
                            ClusterRemainingCapacity : 144
                            NodeBufferPercentage  : 10
                            ClusterBufferedCapacity : 170
                            ClusterRemainingBufferedCapacity : 125
                            ClusterCapacityViolation : False
                            MinNodeLoadValue      : 0
                            MinNodeLoadNodeId     : 3ea71e8e01f4b0999b121abcbf27d74d
                            MaxNodeLoadValue      : 15
                            MaxNodeLoadNodeId     : 2cc648b6770be1bc9824fa995d5b68b1
```

## <a name="next-steps"></a>后续步骤
* 有关群集资源管理器中的体系结构和信息流的信息，请参阅[此文](service-fabric-cluster-resource-manager-architecture.md)
* 定义碎片整理指标是合并（而不是分散）节点上负载的一种方式。若要了解如何配置重整，请参阅[此文](service-fabric-cluster-resource-manager-defragmentation-metrics.md)
* 从头开始并[获取 Service Fabric 群集 Resource Manager 简介](service-fabric-cluster-resource-manager-introduction.md)
* 若要了解群集 Resource Manager 如何管理和均衡群集中的负载，请查看有关[均衡负载](service-fabric-cluster-resource-manager-balancing.md)的文章

[Image1]:./media/service-fabric-cluster-resource-manager-cluster-description/cluster-fault-domains.png
[Image2]:./media/service-fabric-cluster-resource-manager-cluster-description/cluster-uneven-fault-domain-layout.png
[Image3]:./media/service-fabric-cluster-resource-manager-cluster-description/cluster-fault-and-upgrade-domains-with-placement.png
[Image4]:./media/service-fabric-cluster-resource-manager-cluster-description/cluster-fault-and-upgrade-domain-layout-strategies.png
[Image5]:./media/service-fabric-cluster-resource-manager-cluster-description/cluster-layout-different-workloads.png
[Image6]:./media/service-fabric-cluster-resource-manager-cluster-description/cluster-placement-constraints-node-properties.png
[Image7]:./media/service-fabric-cluster-resource-manager-cluster-description/cluster-nodes-and-capacity.png

<!--Update_Description: update meta properties， wording update -->