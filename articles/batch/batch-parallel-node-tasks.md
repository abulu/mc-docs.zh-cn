---
title: 以并行方式运行任务以高效使用计算资源 - Azure Batch | Microsoft 文档
description: 通过减少所用的计算节点数并在 Azure Batch 池的每个节点上运行并发任务，来提高效率并降低成本
services: batch
documentationcenter: .net
author: lingliw
manager: digimobile
editor: ''
ms.assetid: 538a067c-1f6e-44eb-a92b-8d51c33d3e1a
ms.service: batch
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: ''
ms.workload: big-compute
ms.date: 04/17/2019
ms.author: v-lingwu
ms.openlocfilehash: d454131de17b9421a5f7e4546ef8938dba318cb0
ms.sourcegitcommit: f4351979a313ac7b5700deab684d1153ae51d725
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/12/2019
ms.locfileid: "67845267"
---
# <a name="run-tasks-concurrently-to-maximize-usage-of-batch-compute-nodes"></a>以并发方式运行任务以最大程度地利用 Batch 计算节点 

通过在 Azure Batch 池中的每个计算节点上同时运行多个任务，可在池中的较少节点上最大程度利用资源。 对于某些工作负荷，这可以缩短作业时间并降低成本。

尽管在某些情况下，将一个节点的所有资源专用于单个任务会更有利，但在多数情况下，最好是让多个任务共享这些资源：

- **尽量减少数据传输**：适用于任务可以共享数据的情况。 在此方案中，将共享数据复制到较小数目的节点并在每个节点上并行执行任务可以大大减少数据传输费用， 尤其是在复制到每个节点的数据必须跨地理区域传输的情况下。
- **尽量增加内存使用**：适用于任务需要大量的内存，但这种需要仅在执行过程中短时出现且时间不固定的情况。 可以减少计算节点的数量但增加其大小，同时提供更多的内存，以便有效地应对此类高峰负载。 这些节点会在每个节点上并行运行多个任务，而每个任务都会充分利用节点在不同时间的大量内存。
- **减少节点数目限制** ：适用于需要在池中进行节点间通信的情况。 目前，经过配置可以进行节点间通信的池仅限 50 个计算节点。 如果此类池中的每个节点都可以并行执行任务，则可同时执行更大数目的任务。
- **复制本地计算群集**：适用于首次将计算环境移至 Azure 等情况。 如果当前的本地解决方案在每个计算节点上执行多个任务，则可通过增大节点任务的最大数目来更紧密地完成该配置的镜像操作。

## <a name="example-scenario"></a>示例方案
为了举例说明并行任务执行的好处，假设根据任务应用程序的 CPU 和内存要求，[Standard\_D1](../cloud-services/cloud-services-sizes-specs.md) 节点是足够的。 但若要在所需时间内完成作业，则需使用 1,000 个这样的节点。

如果不使用具有 1 个 CPU 内核的 Standard\_D1 节点，则可使用每个具有 16 个内核的 [Standard\_D14](../cloud-services/cloud-services-sizes-specs.md) 节点，同时允许并行执行任务。 因此，可以使用 *1/16 的节点*，即只需使用 63 个节点，而无需使用 1,000 个节点。 此外，如果每个节点需要大型应用程序文件或引用数据，作业持续时间和效率将再次得到提升，因为数据仅复制到 63 个节点。

## <a name="enable-parallel-task-execution"></a>允许并行执行任务
可以对计算节点进行配置，在池级别并行执行任务。 使用 Batch .NET 库，在池创建过程中在请求正文中设置 [CloudPool.MaxTasksPerComputeNode][maxtasks_net] property when you create a pool. If you are using the Batch REST API, set the [maxTasksPerNode][rest_addpool] 元素。

Azure Batch 允许你将每个节点的任务数最多设置为（4 倍）核心节点数。 例如，如果将池的节点大小配置为“大型”（四核），则可将 `maxTasksPerNode` 设置为 16。 但是，无论节点有多少个核心，每个节点的任务数都不能超过 256 个。 有关每个节点大小的核心数的详细信息，请参阅[云服务的大小](../cloud-services/cloud-services-sizes-specs.md)。 有关服务限制的详细信息，请参阅 [Azure Batch 服务的配额和限制](batch-quota-limit.md)。

> [!TIP]
> 为池构造[自动缩放公式][enable_autoscaling]时，请务必考虑 `maxTasksPerNode` 值。 例如，如果增加每个节点的任务数，则可能会极大地影响对 `$RunningTasks` 求值的公式。 有关详细信息，请参阅[自动缩放 Azure Batch 池中的计算节点](batch-automatic-scaling.md)。
>
>

## <a name="distribution-of-tasks"></a>任务分发
当池中的计算节点可以并行执行任务时，请务必指定任务在池中各节点之间的分布方式。

可以通过 [CloudPool.TaskSchedulingPolicy][task_schedule] 属性指定任务应在池中所有节点之间平均分配（“散布式”）。 或者，先给池中的每个节点分配尽量多的任务，此后再将任务分配给池中的其他节点（“装箱式”）。

为了举例说明此功能如何重要，考虑为配置了 [CloudPool.MaxTasksPerComputeNode][maxtasks_net] value of 16. If the [CloudPool.TaskSchedulingPolicy][task_schedule] 的 [Standard\_D14](../cloud-services/cloud-services-sizes-specs.md) 节点池（在上例中）配置 *Pack* 的 [ComputeNodeFillType][fill_type]，这会最大程度地使用每个节点的所有 16 个核心，并且允许[自动缩放池](batch-automatic-scaling.md)删除池中未使用的节点（没有分配任何任务的节点）。 这可以最大程度地减少资源使用量并节省资金。

## <a name="batch-net-example"></a>Batch .NET 示例
此 [Batch .NET][api_net] API code snippet shows a request to create a pool that contains four nodes with a maximum of four tasks per node. It specifies a task scheduling policy that will fill each node with tasks prior to assigning tasks to another node in the pool. For more information on adding pools by using the Batch .NET API, see [BatchClient.PoolOperations.CreatePool][poolcreate_net]。

```csharp
CloudPool pool =
    batchClient.PoolOperations.CreatePool(
        poolId: "mypool",
        targetDedicatedComputeNodes: 4
        virtualMachineSize: "standard_d1_v2",
        cloudServiceConfiguration: new CloudServiceConfiguration(osFamily: "5"));

pool.MaxTasksPerComputeNode = 4;
pool.TaskSchedulingPolicy = new TaskSchedulingPolicy(ComputeNodeFillType.Pack);
pool.Commit();
```

## <a name="batch-rest-example"></a>Batch REST 示例
此 [Batch REST][api_rest] API snippet shows a request to create a pool that contains two large nodes with a maximum of four tasks per node. For more information on adding pools by using the REST API, see [Add a pool to an account][rest_addpool]。

```json
{
  "odata.metadata":"https://myaccount.myregion.batch.chinacloudapi.cn/$metadata#pools/@Element",
  "id":"mypool",
  "vmSize":"large",
  "cloudServiceConfiguration": {
    "osFamily":"4",
    "targetOSVersion":"*",
  }
  "targetDedicatedComputeNodes":2,
  "maxTasksPerNode":4,
  "enableInterNodeCommunication":true,
}
```

> [!NOTE]
> 只能在创建池时设置 `maxTasksPerNode` 元素和 [MaxTasksPerComputeNode][maxtasks_net] 属性。 创建完池以后，不能对上述元素和属性进行修改。
>
>

## <a name="code-sample"></a>代码示例
[ParallelNodeTasks][parallel_tasks_sample] project on GitHub illustrates the use of the [CloudPool.MaxTasksPerComputeNode][maxtasks_net] 属性。

此 C# 控制台应用程序使用 [Batch .NET][api_net] 库创建包含一个或多个计算节点的池。 并在这些节点上执行其数量可以配置的任务，以便模拟可变负荷。 应用程序的输出指定了哪些节点执行了每个任务。 该应用程序还提供了作业参数和持续时间的摘要。 下面显示了同一个应用程序运行两次后的输出摘要部分。

```
Nodes: 1
Node size: large
Max tasks per node: 1
Tasks: 32
Duration: 00:30:01.4638023
```

第一次执行示例应用程序时，结果显示，在池中只有一个节点且使用默认的一个节点一个任务设置的情况下，作业持续时间超过 30 分钟。

```
Nodes: 1
Node size: large
Max tasks per node: 4
Tasks: 32
Duration: 00:08:48.2423500
```

第二次运行示例应用程序时，显示作业持续时间显著缩短。 这是因为该池已被配置为每个节点四个任务，因此可以并行执行任务，使得作业可以在大约四分之一的时间内完成。

> [!NOTE]
> 上述摘要中的作业持续时间不包括创建池的时间。 上述每个作业都提交到此前已创建的池，这些池的计算节点在提交时处于 *空闲* 状态。
>
>

## <a name="next-steps"></a>后续步骤
### <a name="batch-explorer-heat-map"></a>Batch 资源管理器热度地图
[Batch Explorer][batch_labs] 是一个功能丰富的免费独立客户端工具，可帮助创建、调试和监视 Azure Batch 应用程序。 Batch Explorer 包含“热度地图”  功能，可提供任务执行的可视化效果。 执行 [ParallelTasks][parallel_tasks_sample] 示例应用程序时，可以使用“热度地图”功能轻松直观显示每个节点上并行任务的执行。


[api_net]: http://msdn.microsoft.com/library/azure/mt348682.aspx
[api_rest]: http://msdn.microsoft.com/library/azure/dn820158.aspx
[batch_labs]: https://azure.github.io/BatchExplorer/
[cloudpool]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.aspx
[enable_autoscaling]: https://msdn.microsoft.com/library/azure/dn820173.aspx
[fill_type]: https://msdn.microsoft.com/library/microsoft.azure.batch.common.computenodefilltype.aspx
[github_samples]: https://github.com/Azure/azure-batch-samples
[maxtasks_net]: http://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.maxtaskspercomputenode.aspx
[rest_addpool]: https://msdn.microsoft.com/library/azure/dn820174.aspx
[parallel_tasks_sample]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/ParallelTasks
[poolcreate_net]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.pooloperations.createpool.aspx
[task_schedule]: https://msdn.microsoft.com/library/microsoft.azure.batch.cloudpool.taskschedulingpolicy.aspx


