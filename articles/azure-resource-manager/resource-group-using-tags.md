---
title: 为逻辑组织标记 Azure 资源 | Azure
description: 演示如何应用标记来组织 Azure 资源进行计费和管理。
author: rockboyfor
ms.service: azure-resource-manager
ms.topic: conceptual
origin.date: 07/11/2019
ms.date: 07/22/2019
ms.author: v-yeche
ms.openlocfilehash: 377651e315a6856d88391b877e4cf236a2b5ac29
ms.sourcegitcommit: 5fea6210f7456215f75a9b093393390d47c3c78d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/19/2019
ms.locfileid: "68337439"
---
# <a name="use-tags-to-organize-your-azure-resources"></a>使用标记整理 Azure 资源

[!INCLUDE [resource-manager-governance-tags](../../includes/resource-manager-governance-tags.md)]

要向资源应用标记，用户必须具有对该资源类型的写权限。 要将标记应用于所有资源类型，请使用[参与者](../role-based-access-control/built-in-roles.md#contributor)角色。 要将标记仅应用于一种资源类型，请使用该资源的参与者角色。 例如，要将标记应用到虚拟机，请使用[虚拟机参与者](../role-based-access-control/built-in-roles.md#virtual-machine-contributor)。

[!INCLUDE [Handle personal data](../../includes/gdpr-intro-sentence.md)]

## <a name="policies"></a>策略

可以使用 [Azure 策略](../governance/policy/overview.md)来强制实施标记规则和约定。 通过创建策略，可以避免将与预期的组织标记不相符的资源部署到订阅。 无需手动应用标记或搜索不符合的资源，可以创建一个策略，在部署期间自动应用所需标记。 以下部分展示标记策略示例。

[!INCLUDE [Tag policies](../../includes/azure-policy-samples-general-tags.md)]

## <a name="powershell"></a>PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

若要查看*资源组*的现有标记，请使用：

```powershell
(Get-AzResourceGroup -Name examplegroup).Tags
```

该脚本返回以下格式：

```powershell
Name                           Value
----                           -----
Dept                           IT
Environment                    Test
```

若要查看具有指定资源 ID 的资源  的现有标记，请使用：

```powershell
(Get-AzResource -ResourceId /subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Storage/storageAccounts/<storage-name>).Tags
```

或者，若要查看具有指定名称和资源组的资源  的现有标记，请使用：

```powershell
(Get-AzResource -ResourceName examplevnet -ResourceGroupName examplegroup).Tags
```

若要获取具有特定标记的资源组，请使用： 

```powershell
(Get-AzResourceGroup -Tag @{ Dept="Finance" }).ResourceGroupName
```

若要获取具有特定标记的资源，请使用： 

```powershell
(Get-AzResource -Tag @{ Dept="Finance"}).Name
```

若要获取具有特定标记名称的资源，请使用： 

```powershell
(Get-AzResource -TagName Dept).Name
```

每次将标记应用到某个资源或资源组时，都会覆盖该资源或资源组中的现有标记。 因此，必须根据该资源或资源组是否包含现有标记来使用不同的方法。

若要将标记添加到*不包含现有标记的资源组*，请使用：

```powershell
Set-AzResourceGroup -Name examplegroup -Tag @{ Dept="IT"; Environment="Test" }
```

若要将标记添加到包含现有标记的资源组  ，请检索现有标记，添加新标记，然后重新应用标记：

```powershell
$tags = (Get-AzResourceGroup -Name examplegroup).Tags
$tags.Add("Status", "Approved")
Set-AzResourceGroup -Tag $tags -Name examplegroup
```

若要将标记添加到*不包含现有标记的资源*，请使用：

```powershell
$r = Get-AzResource -ResourceName examplevnet -ResourceGroupName examplegroup
Set-AzResource -Tag @{ Dept="IT"; Environment="Test" } -ResourceId $r.ResourceId -Force
```

若要将标记添加到包含现有标记的资源  ，请使用：

```powershell
$r = Get-AzResource -ResourceName examplevnet -ResourceGroupName examplegroup
$r.Tags.Add("Status", "Approved")
Set-AzResource -Tag $r.Tags -ResourceId $r.ResourceId -Force
```

若要将资源组中的所有标记应用于其资源，并且不保留资源上的现有标记  ，请使用以下脚本：

```powershell
$groups = Get-AzResourceGroup
foreach ($g in $groups)
{
    Get-AzResource -ResourceGroupName $g.ResourceGroupName | ForEach-Object {Set-AzResource -ResourceId $_.ResourceId -Tag $g.Tags -Force }
}
```

若要将资源组中的所有标记应用于其资源，并且保留资源上不重复的现有标记  ，请使用以下脚本：

```powershell
$group = Get-AzResourceGroup "examplegroup"
if ($null -ne $group.Tags) {
    $resources = Get-AzResource -ResourceGroupName $group.ResourceGroupName
    foreach ($r in $resources)
    {
        $resourcetags = (Get-AzResource -ResourceId $r.ResourceId).Tags
        if ($resourcetags)
        {
            foreach ($key in $group.Tags.Keys)
            {
                if (-not($resourcetags.ContainsKey($key)))
                {
                    $resourcetags.Add($key, $group.Tags[$key])
                }
            }
            Set-AzResource -Tag $resourcetags -ResourceId $r.ResourceId -Force
        }
        else
        {
            Set-AzResource -Tag $group.Tags -ResourceId $r.ResourceId -Force
        }
    }
}
```

若要删除所有标记，请传递一个空哈希表：

```powershell
Set-AzResourceGroup -Tag @{} -Name examplegroup
```

## <a name="azure-cli"></a>Azure CLI

若要查看*资源组*的现有标记，请使用：

```azurecli
az group show -n examplegroup --query tags
```

该脚本返回以下格式：

```json
{
  "Dept"        : "IT",
  "Environment" : "Test"
}
```

或者，若要查看具有指定名称、类型和资源组的资源的现有标记，请使用： 

```azurecli
az resource show -n examplevnet -g examplegroup --resource-type "Microsoft.Network/virtualNetworks" --query tags
```

循环访问资源集合时，可能想要按资源 ID 显示资源。 本文稍后介绍一个完整的示例。 若要查看具有指定资源 ID 的资源  的现有标记，请使用：

```azurecli
az resource show --id <resource-id> --query tags
```

若要获取具有特定标记的资源组，请使用 `az group list`：

```azurecli
az group list --tag Dept=IT
```

若要获取具有特定标记和值的所有资源，请使用 `az resource list`：

```azurecli
az resource list --tag Dept=Finance
```

每次将标记应用到某个资源或资源组时，都会覆盖该资源或资源组中的现有标记。 因此，必须根据该资源或资源组是否包含现有标记来使用不同的方法。

若要将标记添加到*不包含现有标记的资源组*，请使用：

```azurecli
az group update -n examplegroup --set tags.Environment=Test tags.Dept=IT
```

若要将标记添加到*不包含现有标记的资源*，请使用：

```azurecli
az resource tag --tags Dept=IT Environment=Test -g examplegroup -n examplevnet --resource-type "Microsoft.Network/virtualNetworks"
```

若要将标记添加到已带标记的资源，请检索现有标记，重新格式化该值，然后重新应用现有标记和新标记：

```azurecli
jsonrtag=$(az resource show -g examplegroup -n examplevnet --resource-type "Microsoft.Network/virtualNetworks" --query tags)
rt=$(echo $jsonrtag | tr -d '"{},' | sed 's/: /=/g')
az resource tag --tags $rt Project=Redesign -g examplegroup -n examplevnet --resource-type "Microsoft.Network/virtualNetworks"
```

若要将资源组中的所有标记应用于其资源，并且不保留资源上的现有标记  ，请使用以下脚本：

```azurecli
groups=$(az group list --query [].name --output tsv)
for rg in $groups
do
  jsontag=$(az group show -n $rg --query tags)
  t=$(echo $jsontag | tr -d '"{},' | sed 's/: /=/g')
  r=$(az resource list -g $rg --query [].id --output tsv)
  for resid in $r
  do
    az resource tag --tags $t --id $resid
  done
done
```

若要将资源组中的所有标记应用于其资源，并且保留资源上的现有标记  ，请使用以下脚本：

```azurecli
groups=$(az group list --query [].name --output tsv)
for rg in $groups
do
  jsontag=$(az group show -n $rg --query tags)
  t=$(echo $jsontag | tr -d '"{},' | sed 's/: /=/g')
  r=$(az resource list -g $rg --query [].id --output tsv)
  for resid in $r
  do
    jsonrtag=$(az resource show --id $resid --query tags)
    rt=$(echo $jsonrtag | tr -d '"{},' | sed 's/: /=/g')
    az resource tag --tags $t$rt --id $resid
  done
done
```

## <a name="templates"></a>模板

[!INCLUDE [resource-manager-tags-in-templates](../../includes/resource-manager-tags-in-templates.md)]

## <a name="portal"></a>门户

[!INCLUDE [resource-manager-tag-resource](../../includes/resource-manager-tag-resources.md)]

## <a name="rest-api"></a>REST API

Azure 门户和 PowerShell 均在后台使用[资源管理器 REST API](https://docs.microsoft.com/rest/api/resources/)。 如果需要将标记集成到其他环境中，可对资源 ID 使用 GET 以获取标记，并使用 PATCH 调用更新标记集。  

## <a name="tags-and-billing"></a>标记和计费

可使用标记对计费数据进行分组。 例如，如果针对不同组织运行多个 VM，可以使用标记根据成本中心对使用情况进行分组。 还可使用标记根据运行时环境（例如，在生产环境中运行的 VM 的计费使用情况）对成本进行分类。

可以通过使用情况逗号分隔值 (CSV) 文件检索有关标记的信息。 可从 [Azure 帐户门户](https://account.windowsazure.cn/)或 Azure 门户下载使用情况文件。

<!-- Not Available [Azure Billing REST API Reference](https://msdn.microsoft.com/library/azure/1ea5b323-54bb-423d-916f-190de96c6a3c)-->
<!-- Not Available [Azure Resource Usage and RateCard APIs](../billing/billing-usage-rate-card-overview.md) -->

有关 REST API 操作，请参阅 [Azure 计费 REST API 参考](https://docs.microsoft.com/rest/api/billing/)。

## <a name="next-steps"></a>后续步骤

* 并非所有资源类型都支持标记。 若要确定是否可以将标记应用到资源类型，请参阅 [Azure 资源的标记支持](tag-support.md)。
* 有关使用门户的说明，请参阅[使用 Azure 门户管理 Azure 资源](manage-resource-groups-portal.md)。

<!--Update_Description: update meta properties, wording update -->