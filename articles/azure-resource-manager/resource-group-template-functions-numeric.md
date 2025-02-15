---
title: Azure Resource Manager 模板函数 - 数值 | Azure
description: 介绍可在 Azure Resource Manager 模板中使用的用于处理数值的函数。
author: rockboyfor
ms.service: azure-resource-manager
ms.topic: reference
origin.date: 11/08/2017
ms.date: 07/22/2019
ms.author: v-yeche
ms.openlocfilehash: d47594c1452d0e048efdaaf20ed6395d73f3a6f0
ms.sourcegitcommit: 5fea6210f7456215f75a9b093393390d47c3c78d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/19/2019
ms.locfileid: "68337433"
---
# <a name="numeric-functions-for-azure-resource-manager-templates"></a>用于 Azure Resource Manager 模板的数值函数

Resource Manager 提供以下用于处理整数的函数：

* [添加](#add)
* [copyIndex](#copyindex)
* [div](#div)
* [float](#float)
* [int](#int)
* [max](#max)
* [min](#min)
* [mod](#mod)
* [mul](#mul)
* [sub](#sub)

<a name="add" />

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="add"></a>添加
`add(operand1, operand2)`

返回提供的两个整数的总和。

### <a name="parameters"></a>parameters

| 参数 | 必须 | 类型 | 说明 |
|:--- |:--- |:--- |:--- | 
|operand1 |是 |int |被加数。 |
|operand2 |是 |int |加数。 |

### <a name="return-value"></a>返回值

一个整数，包含参数的总和。

### <a name="example"></a>示例

以下[示例模板](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/add.json)将添加两个参数。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "first": {
            "type": "int",
            "defaultValue": 5,
            "metadata": {
                "description": "First integer to add"
            }
        },
        "second": {
            "type": "int",
            "defaultValue": 3,
            "metadata": {
                "description": "Second integer to add"
            }
        }
    },
    "resources": [
    ],
    "outputs": {
        "addResult": {
            "type": "int",
            "value": "[add(parameters('first'), parameters('second'))]"
        }
    }
}
```

上述示例中使用默认值的输出为：

| Name | 类型 | Value |
| ---- | ---- | ----- |
| addResult | int | 8 |

要使用 Azure CLI 部署此示例模板，请使用：

```azurecli
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/add.json
```

要使用 PowerShell 部署此示例模板，请使用：

```powershell
New-AzResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/add.json 
```

<a name="copyindex" />

## <a name="copyindex"></a>copyIndex
`copyIndex(loopName, offset)`

返回一个迭代循环的索引。 

### <a name="parameters"></a>parameters

| 参数 | 必须 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| loopName | 否 | string | 用于获取迭代的循环的名称。 |
| offset |否 |int |要添加到从零开始的迭代值的数。 |

### <a name="remarks"></a>备注

此函数始终用于 **copy** 对象。 如果没有提供任何值作为 **偏移量**，则返回当前迭代值。 迭代值从零开始。 定义资源或变量时，你可以使用迭代循环。

**loopName** 属性可用于指定 copyIndex 是引用资源迭代还是引用属性迭代。 如果没有为 **loopName** 提供值，则将使用当前的资源类型迭代。 在属性上迭代时，请为 **loopName** 提供值。 

有关如何使用 **copyIndex** 的完整说明，请参阅 [Create multiple instances of resources in Azure Resource Manager](resource-group-create-multiple.md)（在 Azure Resource Manager 中创建多个资源实例）。

有关定义变量时使用“copyIndex”的示例  ，请参阅[变量](resource-group-authoring-templates.md#variables)。

### <a name="example"></a>示例

以下示例显示名称中包含 copy 循环和索引值。 

```json
"resources": [ 
  { 
    "name": "[concat('examplecopy-', copyIndex())]", 
    "type": "Microsoft.Web/sites", 
    "copy": { 
      "name": "websitescopy", 
      "count": "[parameters('count')]" 
    }, 
    ...
  }
]
```

### <a name="return-value"></a>返回值

一个表示迭代的当前索引的整数。

<a name="div" />

## <a name="div"></a>div
`div(operand1, operand2)`

返回提供的两个整数在整除后的商。

### <a name="parameters"></a>parameters

| 参数 | 必须 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| operand1 |是 |int |被除数。 |
| operand2 |是 |int |除数。 不能为 0。 |

### <a name="return-value"></a>返回值

一个表示商的整数。

### <a name="example"></a>示例

以下[示例模板](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/div.json)将一个参数除以另一个参数。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "first": {
            "type": "int",
            "defaultValue": 8,
            "metadata": {
                "description": "Integer being divided"
            }
        },
        "second": {
            "type": "int",
            "defaultValue": 3,
            "metadata": {
                "description": "Integer used to divide"
            }
        }
    },
    "resources": [
    ],
    "outputs": {
        "divResult": {
            "type": "int",
            "value": "[div(parameters('first'), parameters('second'))]"
        }
    }
}
```

上述示例中使用默认值的输出为：

| Name | 类型 | Value |
| ---- | ---- | ----- |
| divResult | int | 2 |

要使用 Azure CLI 部署此示例模板，请使用：

```azurecli
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/div.json
```

要使用 PowerShell 部署此示例模板，请使用：

```powershell
New-AzResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/div.json 
```

<a name="float" />

## <a name="float"></a>float
`float(arg1)`

将值转换为浮点数。 仅当将自定义参数传递给应用程序（例如，逻辑应用）时，才使用此函数。

### <a name="parameters"></a>parameters

| 参数 | 必须 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| arg1 |是 |字符串或整数 |要转换为浮点数的值。 |

### <a name="return-value"></a>返回值
一个浮点数。

### <a name="example"></a>示例

以下示例演示如何使用 float 将参数传递给逻辑应用：

```json
{
    "type": "Microsoft.Logic/workflows",
    "properties": {
        ...
        "parameters": {
            "custom1": {
                "value": "[float('3.0')]"
            },
            "custom2": {
                "value": "[float(3)]"
            },
```

<a name="int" />

## <a name="int"></a>int
`int(valueToConvert)`

将指定的值转换为整数。

### <a name="parameters"></a>parameters

| 参数 | 必须 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| valueToConvert |是 |string 或 int |要转换为整数的值。 |

### <a name="return-value"></a>返回值

转换后的值的整数。

### <a name="example"></a>示例

以下[示例模板](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/int.json)将用户提供的参数值转换为整数。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "stringToConvert": { 
            "type": "string",
            "defaultValue": "4"
        }
    },
    "resources": [
    ],
    "outputs": {
        "intResult": {
            "type": "int",
            "value": "[int(parameters('stringToConvert'))]"
        }
    }
}
```

上述示例中使用默认值的输出为：

| Name | 类型 | Value |
| ---- | ---- | ----- |
| intResult | int | 4 |

要使用 Azure CLI 部署此示例模板，请使用：

```azurecli
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/int.json
```

要使用 PowerShell 部署此示例模板，请使用：

```powershell
New-AzResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/int.json
```

<a name="max" />

## <a name="max"></a>max
`max (arg1)`

返回整数数组或逗号分隔的整数列表中的最大值。

### <a name="parameters"></a>parameters

| 参数 | 必须 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| arg1 |是 |整数数组或逗号分隔的整数列表 |要获取最大值的集合。 |

### <a name="return-value"></a>返回值

一个整数，表示集合中的最大值。

### <a name="example"></a>示例

以下[示例模板](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/max.json)演示如何对整数数组和整数列表使用 max：

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "arrayToTest": {
            "type": "array",
            "defaultValue": [0,3,2,5,4]
        }
    },
    "resources": [],
    "outputs": {
        "arrayOutput": {
            "type": "int",
            "value": "[max(parameters('arrayToTest'))]"
        },
        "intOutput": {
            "type": "int",
            "value": "[max(0,3,2,5,4)]"
        }
    }
}
```

上述示例中使用默认值的输出为：

| Name | 类型 | Value |
| ---- | ---- | ----- |
| arrayOutput | int | 5 |
| intOutput | int | 5 |

要使用 Azure CLI 部署此示例模板，请使用：

```azurecli
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/max.json
```

要使用 PowerShell 部署此示例模板，请使用：

```powershell
New-AzResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/max.json
```

<a name="min" />

## <a name="min"></a>min
`min (arg1)`

返回整数数组或逗号分隔的整数列表中的最小值。

### <a name="parameters"></a>parameters

| 参数 | 必须 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| arg1 |是 |整数数组或逗号分隔的整数列表 |要获取最小值的集合。 |

### <a name="return-value"></a>返回值

一个整数，表示集合中的最小值。

### <a name="example"></a>示例

以下[示例模板](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/min.json)演示如何对整数数组和整数列表使用 min：

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "arrayToTest": {
            "type": "array",
            "defaultValue": [0,3,2,5,4]
        }
    },
    "resources": [],
    "outputs": {
        "arrayOutput": {
            "type": "int",
            "value": "[min(parameters('arrayToTest'))]"
        },
        "intOutput": {
            "type": "int",
            "value": "[min(0,3,2,5,4)]"
        }
    }
}
```

上述示例中使用默认值的输出为：

| Name | 类型 | Value |
| ---- | ---- | ----- |
| arrayOutput | int | 0 |
| intOutput | int | 0 |

要使用 Azure CLI 部署此示例模板，请使用：

```azurecli
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/min.json
```

要使用 PowerShell 部署此示例模板，请使用：

```powershell
New-AzResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/min.json
```

<a name="mod" />

## <a name="mod"></a>mod
`mod(operand1, operand2)`

返回使用提供的两个整数整除后的余数。

### <a name="parameters"></a>parameters

| 参数 | 必须 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| operand1 |是 |int |被除数。 |
| operand2 |是 |int |除数，不能为 0。 |

### <a name="return-value"></a>返回值
一个表示余数的整数。

### <a name="example"></a>示例

以下[示例模板](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/mod.json)将返回一个参数除以另一个参数后所得的余数。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "first": {
            "type": "int",
            "defaultValue": 7,
            "metadata": {
                "description": "Integer being divided"
            }
        },
        "second": {
            "type": "int",
            "defaultValue": 3,
            "metadata": {
                "description": "Integer used to divide"
            }
        }
    },
    "resources": [
    ],
    "outputs": {
        "modResult": {
            "type": "int",
            "value": "[mod(parameters('first'), parameters('second'))]"
        }
    }
}
```

上述示例中使用默认值的输出为：

| Name | 类型 | Value |
| ---- | ---- | ----- |
| modResult | int | 1 |

要使用 Azure CLI 部署此示例模板，请使用：

```azurecli
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/mod.json
```

要使用 PowerShell 部署此示例模板，请使用：

```powershell
New-AzResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/mod.json
```

<a name="mul" />

## <a name="mul"></a>mul
`mul(operand1, operand2)`

返回提供的两个整数的积。

### <a name="parameters"></a>parameters

| 参数 | 必须 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| operand1 |是 |int |被乘数。 |
| operand2 |是 |int |乘数。 |

### <a name="return-value"></a>返回值

一个表示积的整数。

### <a name="example"></a>示例

以下[示例模板](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/mul.json)将一个参数乘以另一个参数。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "first": {
            "type": "int",
            "defaultValue": 5,
            "metadata": {
                "description": "First integer to multiply"
            }
        },
        "second": {
            "type": "int",
            "defaultValue": 3,
            "metadata": {
                "description": "Second integer to multiply"
            }
        }
    },
    "resources": [
    ],
    "outputs": {
        "mulResult": {
            "type": "int",
            "value": "[mul(parameters('first'), parameters('second'))]"
        }
    }
}
```

上述示例中使用默认值的输出为：

| Name | 类型 | Value |
| ---- | ---- | ----- |
| mulResult | int | 15 |

要使用 Azure CLI 部署此示例模板，请使用：

```azurecli
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/mul.json
```

要使用 PowerShell 部署此示例模板，请使用：

```powershell
New-AzResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/mul.json
```

<a name="sub" />

## <a name="sub"></a>sub
`sub(operand1, operand2)`

返回提供的两个整数在相减后的结果。

### <a name="parameters"></a>parameters

| 参数 | 必须 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| operand1 |是 |int |被减数。 |
| operand2 |是 |int |减数。 |

### <a name="return-value"></a>返回值
一个表示减后结果的整数。

### <a name="example"></a>示例

以下[示例模板](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/sub.json)将一个参数与另一个参数相减。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "first": {
            "type": "int",
            "defaultValue": 7,
            "metadata": {
                "description": "Integer subtracted from"
            }
        },
        "second": {
            "type": "int",
            "defaultValue": 3,
            "metadata": {
                "description": "Integer to subtract"
            }
        }
    },
    "resources": [
    ],
    "outputs": {
        "subResult": {
            "type": "int",
            "value": "[sub(parameters('first'), parameters('second'))]"
        }
    }
}
```

上述示例中使用默认值的输出为：

| Name | 类型 | Value |
| ---- | ---- | ----- |
| subResult | int | 4 |

要使用 Azure CLI 部署此示例模板，请使用：

```azurecli
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/sub.json
```

要使用 PowerShell 部署此示例模板，请使用：

```powershell
New-AzResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/sub.json
```

## <a name="next-steps"></a>后续步骤
* 有关 Azure 资源管理器模板中各部分的说明，请参阅[创作 Azure 资源管理器模板](resource-group-authoring-templates.md)。
* 若要合并多个模板，请参阅[将链接的模板与 Azure Resource Manager 配合使用](resource-group-linked-templates.md)。
* 若要在创建资源类型时迭代指定的次数，请参阅[在 Azure Resource Manager 中创建多个资源实例](resource-group-create-multiple.md)。
* 若要查看如何部署已创建的模板，请参阅[使用 Azure Resource Manager 模板部署应用程序](resource-group-template-deploy.md)。

<!--Update_Description: update meta properties, wording update, update powershell az cmdlet -->