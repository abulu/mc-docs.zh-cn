---
title: 如何计划 Azure-SSIS Integration Runtime | Microsoft Docs
description: 本文介绍如何使用 Azure 数据工厂计划 Azure-SSIS Integration Runtime 的启动和停止。
services: data-factory
documentationcenter: ''
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: ''
ms.devlang: powershell
ms.topic: conceptual
origin.date: 1/9/2019
ms.date: 07/08/2019
author: WenJason
ms.author: v-jay
ms.reviewer: douglasl
manager: digimobile
ms.openlocfilehash: 7a90ad25bc8b580ba303f81b20b5d262c06e5820
ms.sourcegitcommit: 5191c30e72cbbfc65a27af7b6251f7e076ba9c88
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/05/2019
ms.locfileid: "67570356"
---
# <a name="how-to-start-and-stop-azure-ssis-integration-runtime-on-a-schedule"></a>如何按计划启动和停止 Azure-SSIS Integration Runtime
本文介绍如何使用 Azure 数据工厂 (ADF) 计划 Azure-SSIS Integration Runtime (IR) 的启动和停止。 Azure-SSIS IR 是专用于执行 SQL Server Integration Services (SSIS) 包的 ADF 计算资源。 运行 Azure-SSIS IR 会产生相关成本。 因此，通常只有在需要在 Azure 中运行 SSIS 包时才运行 IR，而不再需要该包时则停止 IR。 可以使用 ADF 用户界面 (UI)/应用或 Azure PowerShell [手动启动或停止 IR](manage-azure-ssis-integration-runtime.md)。

或者，可以在 ADF 管道中创建 Web 活动，以按计划启动/停止 IR，例如，在早上执行每日 ETL 工作负载之前启动 IR，并在下午完成后将其停止。  还可以在启动和停止 IR 的两个 Web 活动之间链接一个执行 SSIS 包活动，这样 IR 就会按需在包执行之前/之后及时启动/停止。 有关执行 SSIS 包活动的详细信息，请参阅[在 ADF 管道中使用执行 SSIS 包活动运行 SSIS 包](how-to-invoke-ssis-package-ssis-activity.md)一文。

[!INCLUDE [requires-azurerm](../../includes/requires-azurerm.md)]

## <a name="prerequisites"></a>先决条件
如果尚未配置 Azure-SSIS IR，请按照[教程](tutorial-create-azure-ssis-runtime-portal.md)中的说明进行配置。 

## <a name="create-and-schedule-adf-pipelines-that-start-and-or-stop-azure-ssis-ir"></a>创建和安排启动和/或停止 Azure-SSIS IR 的 ADF 管道
本部分演示如何在 ADF 管道中使用 Web 活动按计划或按需启动/停止 Azure-SSIS IR。 我们将指导你创建三个管道： 

1. 第一个管道包含启动 Azure-SSIS IR 的 Web 活动。 
2. 第二个管道包含停止 Azure-SSIS IR 的 Web 活动。
3. 第三个管道包含一个执行 SSIS 包活动，该活动链接在两个启动/停止 Azure-SSIS IR 的 Web 活动之间。 

创建并测试管道后，可以创建一个计划触发器，并将其与任何管道相关联。 计划触发器定义了运行相关管道的计划。 

例如，可以创建两个触发器，第一个触发器计划在每天上午 6 点运行并与第一个管道相关联，而第二个触发器计划在每天晚上 6 点运行并与第二个管道相关联。  通过这种方式，IR 会在每天上午 6 点到晚上 6 点这一时段运行，可用于执行每日 ETL 工作负载。  

如果创建第三个触发器，计划在每天午夜运行并与第三个管道相关联，那么该管道将在每天午夜运行，在包执行前才启动 IR，随后执行包，然后在包执行后立即停止 IR，这样 IR 就不会空闲运行。

### <a name="create-your-adf"></a>创建 ADF

1. 登录到 [Azure 门户](https://portal.azure.cn/)。    
2. 在左侧菜单中单击“新建”，并依次单击“数据 + 分析”、“数据工厂”。    
   
   ![新建 -> DataFactory](./media/tutorial-create-azure-ssis-runtime-portal/new-data-factory-menu.png)
   
3. 在“新建数据工厂”页中，输入“MyAzureSsisDataFactory”作为名称    。 
      
   ![“新建数据工厂”页](./media/tutorial-create-azure-ssis-runtime-portal/new-azure-data-factory.png)
 
   ADF 的名称必须全局唯一。 如果收到错误，请更改数据工厂的名称（例如，yournameMyAzureSsisDataFactory），并重新尝试创建。 有关 ADF 项目的命名规则，请参阅[数据工厂 - 命名规则](naming-rules.md)一文。
  
   `Data factory name MyAzureSsisDataFactory is not available`
      
4. 选择要在其下创建 ADF 的 Azure 订阅  。 
5. 对于**资源组**，请执行以下步骤之一：
     
   - 选择“使用现有资源组”，并从下拉列表选择现有的资源组。  
   - 选择“新建”，并输入新资源组的名称。    
         
   若要了解有关资源组的详细信息，请参阅 [使用资源组管理 Azure 资源](../azure-resource-manager/resource-group-overview.md)一文。
   
6. 对于“版本”，选择“V2”   。
7. 对于“位置”，从下拉列表中选择 ADF 创建支持的其中一个位置  。
8. 选择“固定到仪表板”  。     
9. 单击**创建**。
10. 在 Azure 仪表板上，你会看状态如下的以下磁贴：**正在部署数据工厂**。 

    ![“正在部署数据工厂”磁贴](media/tutorial-create-azure-ssis-runtime-portal/deploying-data-factory.png)
   
11. 创建完成后，ADF 页面显示如下。
   
    ![数据工厂主页](./media/tutorial-create-azure-ssis-runtime-portal/data-factory-home-page.png)
   
12. 单击“创建者和监视器”，在单独的选项卡中启动 ADF UI/应用  。

### <a name="create-your-pipelines"></a>创建管道

1. 在“开始使用”页中，选择“创建管道”   。 

   ![“入门”页](./media/how-to-schedule-azure-ssis-integration-runtime/get-started-page.png)
   
2. 在“活动”工具箱中，展开“常规”菜单，并将“Web”活动拖放到管道设计器图面    。 在活动属性窗口的“常规”选项卡中，将活动名称更改为“startMyIR”   。 切换到“设置”选项卡，然后执行以下操作  。

    1. 对于“URL”，请为启动 Azure-SSIS IR 的 REST API 输入以下 URL，将 `{subscriptionId}`、`{resourceGroupName}`、`{factoryName}` 和 `{integrationRuntimeName}` 替换为 IR 的实际值  ：`https://management.chinacloudapi.cn/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DataFactory/factories/{factoryName}/integrationRuntimes/{integrationRuntimeName}/start?api-version=2018-06-01` 或者，也可以从 ADF UI/应用上的监视页面复制并粘贴 IR 的资源 ID，替换上述 URL 的以下部分：`/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DataFactory/factories/{factoryName}/integrationRuntimes/{integrationRuntimeName}`
    
       ![ADF SSIS IR 资源 ID](./media/how-to-schedule-azure-ssis-integration-runtime/adf-ssis-ir-resource-id.png)
  
    2. 对于“方法”，请选择“POST”。   
    3. 对于“正文”，请输入 `{"message":"Start my IR"}`。  
    4. 对于“身份验证”  ，请选择 **MSI** 以使用 ADF 的托管标识，有关详细信息，请参阅[数据工厂的托管标识](/data-factory/data-factory-service-identity)一文。
    5. 对于“资源”，请输入 `https://management.chinacloudapi.cn/`  。
    
       ![ADFWeb 活动计划 SSIS IR](./media/how-to-schedule-azure-ssis-integration-runtime/adf-web-activity-schedule-ssis-ir.png)
  
3. 克隆第一个管道以创建第二个管道，将活动名称更改为 stopMyIR 并替换以下属性  。

    1. 对于“URL”，请为停止 Azure-SSIS IR 的 REST API 输入以下 URL，将 `{subscriptionId}`、`{resourceGroupName}`、`{factoryName}` 和 `{integrationRuntimeName}` 替换为 IR 的实际值`https://management.chinacloudapi.cn/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DataFactory/factories/{factoryName}/integrationRuntimes/{integrationRuntimeName}/stop?api-version=2018-06-01`  ：
    
    2. 对于“正文”，请输入 `{"message":"Stop my IR"}`。  

4. 创建第三个管道，将“执行 SSIS 包”活动从“活动”工具箱拖放到管道设计器图面，然后按照[使用 ADF 中的执行 SSIS 包活动调用 SSIS 包](how-to-invoke-ssis-package-ssis-activity.md)一文中的说明配置 SSIS 包活动   。  或者，可以使用“存储过程”活动，并按照[使用 ADF 中的存储过程活动调用 SSIS 包](how-to-invoke-ssis-package-stored-procedure-activity.md)一文中的说明配置 SSIS 包活动  。  接下来，在启动/停止 IR 的两个 Web 活动之间链接执行 SSIS 包/存储过程活动，类似于第一个/第二个管道中的 Web 活动。

   ![ADF Web 活动按需 SSIS IR](./media/how-to-schedule-azure-ssis-integration-runtime/adf-web-activity-on-demand-ssis-ir.png)

5. 为 ADF 分配一个“参与者”角色的托管标识，因此其管道中的 Web 活动可以调用 REST API 来启动/停止在其中预配的 Azure-SSIS IR  。  在 Azure 门户的 ADF 页面上，单击“访问控制 (IAM)”，单击“+ 添加角色分配”，然后在“添加角色分配”边栏选项卡中，执行以下操作    。

    1. 对于“角色”  ，选择“参与者”  。 
    2. 对于“分配访问权限至”，选择“Azure AD 用户、组或服务主体”   。 
    3. 对于“选择”，搜索你的 ADF 名称并选择该 ADF  。 
    4. 单击“保存”  。
    
   ![ADF 托管标识角色分配](./media/how-to-schedule-azure-ssis-integration-runtime/adf-managed-identity-role-assignment.png)

6. 单击“工厂/管道”工具栏上的“验证所有/验证”，验证 ADF 和所有管道设置  。 单击 >> 按钮关闭“工厂/管道验证输出”   。  

   ![验证管道](./media/how-to-schedule-azure-ssis-integration-runtime/validate-pipeline.png)

### <a name="test-run-your-pipelines"></a>测试运行管道

1. 在工具栏上为每个管道选择“测试运行”，然后在底部窗格中查看“输出”窗口   。 

   ![测试运行](./media/how-to-schedule-azure-ssis-integration-runtime/test-run-output.png)
    
2. 若要测试第三个管道，请启动 SQL Server Management Studio (SSMS)。 在“连接到服务器”窗口中执行以下操作  。 

    1. 对于“服务器名称”，请输入 &lt;Azure SQL 数据库服务器名称&gt;.database.chinacloudapi.cn   。
    2. 选择“选项Options >>”。 
    3. 对于“连接到数据库”，请选择“SSISDB”。  
    4. 选择“连接”  。 
    5. 展开“Integration Services 目录” -> “SSISDB”-> 你的文件夹 ->“项目”-> 你的 SSIS 项目 ->“包”。     
    6. 右键单击指定的 SSIS 包，运行并选择“报告” -> “标准报告” -> “所有执行”    。 
    7. 验证是否已运行该包。 

   ![验证 SSIS 包是否已运行](./media/how-to-schedule-azure-ssis-integration-runtime/verify-ssis-package-run.png)

### <a name="schedule-your-pipelines"></a>计划管道

现在，管道按预期工作，可以创建触发器以按指定节奏运行管道。 有关如何将触发器与管道相关联的详细信息，请参阅[按计划触发管道](quickstart-create-data-factory-portal.md#trigger-the-pipeline-on-a-schedule)一文。

1. 在管道工具栏上，依次选择“触发器”、“新建/编辑”   。 

   ![“触发器”->“新建/编辑”](./media/how-to-schedule-azure-ssis-integration-runtime/trigger-new-menu.png)

2. 在“添加触发器”窗格中，选择“+ 新建”   。

   ![“添加触发器”-“新建”](./media/how-to-schedule-azure-ssis-integration-runtime/add-triggers-new.png)

3. 在“新建触发器”窗格中执行以下操作：  

    1. 对于“名称”，输入触发器的名称  。 在以下示例中，“每日运行”是触发器名称  。 
    2. 对于“类型”，请选择“计划”。   
    3. 对于“开始日期 (UTC)”，请在 UTC 中输入开始日期和时间  。 
    4. 对于“重复周期”，请输入触发器的频率  。 在以下示例中，频率为每日一次  。 
    5. 对于“结束”，请选择“不结束”或在选择“开始日期”后输入结束日期和时间    。 
    6. 发布整个 ADF 设置后，选择“激活”以立即激活触发器  。 
    7. 选择“**下一步**”。

   ![“触发器”->“新建/编辑”](./media/how-to-schedule-azure-ssis-integration-runtime/new-trigger-window.png)
    
4. 在“触发器运行参数”页中查看任何警告，然后选择“完成”   。 
5. 通过选择工厂工具栏中的“发布所有”，发布整个 ADF 设置  。 

   ![全部发布](./media/how-to-schedule-azure-ssis-integration-runtime/publish-all.png)

### <a name="monitor-your-pipelines-and-triggers-in-azure-portal"></a>在 Azure 门户中监视管道和触发器

1. 若要监视触发器运行和管道运行，请使用 ADF UI/app 左侧的“监视”选项卡  。 有关详细步骤，请参阅[监视管道](quickstart-create-data-factory-portal.md#monitor-the-pipeline)一文。

   ![管道运行](./media/how-to-schedule-azure-ssis-integration-runtime/pipeline-runs.png)

2. 若要查看与管道运行关联的活动运行，请在“操作”列中选择第一个链接（“查看活动运行”）   。 对于第三个管道，可看到三个活动在运行，每一个活动均用于管道中的每个链式活动（用于启动 IR 的 Web 活动、用于执行包的存储过程活动以及用于停止 IR 的 Web 活动）。 若要再次查看管道，请选择顶部的“管道”链接  。

   ![活动运行](./media/how-to-schedule-azure-ssis-integration-runtime/activity-runs.png)

3. 若要查看触发器运行，请从顶部“管道运行”下的下拉列表中选择“触发器运行”   。 

   ![触发器运行](./media/how-to-schedule-azure-ssis-integration-runtime/trigger-runs.png)

### <a name="monitor-your-pipelines-and-triggers-with-powershell"></a>使用 PowerShell 监视管道和触发器

使用如下示例脚本来监视管道和触发器。

1. 获取管道运行的状态。

   ```powershell
   Get-AzDataFactoryV2PipelineRun -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -PipelineRunId $myPipelineRun
   ```

2. 获取有关触发器的信息。

   ```powershell
   Get-AzDataFactoryV2Trigger -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name  "myTrigger"
   ```

3. 获取触发器运行的状态。

   ```powershell
   Get-AzDataFactoryV2TriggerRun -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -TriggerName "myTrigger" -TriggerRunStartedAfter "2018-07-15" -TriggerRunStartedBefore "2018-07-16"
   ```

## <a name="next-steps"></a>后续步骤
参阅 SSIS 文档中的以下文章： 

- [在 Azure 上部署、运行和监视 SSIS 包](https://docs.microsoft.com/sql/integration-services/lift-shift/ssis-azure-deploy-run-monitor-tutorial)   
- [连接到 Azure 上的 SSIS 目录](https://docs.microsoft.com/sql/integration-services/lift-shift/ssis-azure-connect-to-catalog-database)
- [在 Azure 上计划包执行](https://docs.microsoft.com/sql/integration-services/lift-shift/ssis-azure-schedule-packages)
- [使用 Windows 身份验证连接到本地数据源](https://docs.microsoft.com/sql/integration-services/lift-shift/ssis-azure-connect-with-windows-auth)
