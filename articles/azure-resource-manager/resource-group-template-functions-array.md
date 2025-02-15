---
title: Azure Resource Manager 模板函数 - 数组和对象 | Azure
description: 介绍可在 Azure Resource Manager 模板中用来处理数组和对象的函数。
author: rockboyfor
ms.service: azure-resource-manager
ms.topic: reference
origin.date: 11/08/2018
ms.date: 07/22/2019
ms.author: v-yeche
ms.openlocfilehash: b8c357d4f519536b2a48a157d36a50264df3bfcc
ms.sourcegitcommit: 5fea6210f7456215f75a9b093393390d47c3c78d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/19/2019
ms.locfileid: "68337436"
---
# <a name="array-and-object-functions-for-azure-resource-manager-templates"></a>用于 Azure Resource Manager 模板的数组和对象函数

Resource Manager 提供以下用于处理数组和对象的函数。

* [array](#array)
* [coalesce](#coalesce)
* [concat](#concat)
* [contains](#contains)
* [createArray](#createarray)
* [empty](#empty)
* [first](#first)
* [intersection](#intersection)
* [json](#json)
* [last](#last)
* [length](#length)
* [max](#max)
* [min](#min)
* [range](#range)
* [skip](#skip)
* [take](#take)
* [union](#union)

若要获取由某个值分隔的字符串值数组，请参阅 [split](resource-group-template-functions-string.md#split)。

<a name="array" />

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="array"></a>array
`array(convertToArray)`

将值转换为数组。

### <a name="parameters"></a>parameters

| 参数 | 必须 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| convertToArray |是 |整数、字符串、数组或对象 |要转换为数组的值。 |

### <a name="return-value"></a>返回值

一个数组。

### <a name="example"></a>示例

以下[示例模板](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/array.json)演示如何对不同的类型使用 array 函数。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "intToConvert": {
            "type": "int",
            "defaultValue": 1
        },
        "stringToConvert": {
            "type": "string",
            "defaultValue": "efgh"
        },
        "objectToConvert": {
            "type": "object",
            "defaultValue": {"a": "b", "c": "d"}
        }
    },
    "resources": [
    ],
    "outputs": {
        "intOutput": {
            "type": "array",
            "value": "[array(parameters('intToConvert'))]"
        },
        "stringOutput": {
            "type": "array",
            "value": "[array(parameters('stringToConvert'))]"
        },
        "objectOutput": {
            "type": "array",
            "value": "[array(parameters('objectToConvert'))]"
        }
    }
}
```

上述示例中使用默认值的输出为：

| Name | 类型 | Value |
| ---- | ---- | ----- |
| intOutput | Array | [1] |
| stringOutput | Array | ["efgh"] |
| objectOutput | Array | [{"a": "b", "c": "d"}] |

要使用 Azure CLI 部署此示例模板，请使用：

```azurecli
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/array.json
```

要使用 PowerShell 部署此示例模板，请使用：

```powershell
New-AzResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/array.json
```

<a name="coalesce" />

## <a name="coalesce"></a>coalesce
`coalesce(arg1, arg2, arg3, ...)`

从参数中返回第一个非 null 值。 空字符串、空数组和空对象不为 null。

### <a name="parameters"></a>parameters

| 参数 | 必须 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| arg1 |是 |int、string、array 或 object |要测试是否为 null 的第一个值。 |
| 其他参数 |否 |整数、字符串、数组或对象 |要测试是否为 null 的其他值。 |

### <a name="return-value"></a>返回值

第一个非 null 参数的值，可以是字符串、整数、数组或对象。 如果所有参数都为 null，则为 null。 

### <a name="example"></a>示例

以下[示例模板](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/coalesce.json)显示 coalesce 不同用法的输出。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "objectToTest": {
            "type": "object",
            "defaultValue": {
                "null1": null, 
                "null2": null,
                "string": "default",
                "int": 1,
                "object": {"first": "default"},
                "array": [1]
            }
        }
    },
    "resources": [
    ],
    "outputs": {
        "stringOutput": {
            "type": "string",
            "value": "[coalesce(parameters('objectToTest').null1, parameters('objectToTest').null2, parameters('objectToTest').string)]"
        },
        "intOutput": {
            "type": "int",
            "value": "[coalesce(parameters('objectToTest').null1, parameters('objectToTest').null2, parameters('objectToTest').int)]"
        },
        "objectOutput": {
            "type": "object",
            "value": "[coalesce(parameters('objectToTest').null1, parameters('objectToTest').null2, parameters('objectToTest').object)]"
        },
        "arrayOutput": {
            "type": "array",
            "value": "[coalesce(parameters('objectToTest').null1, parameters('objectToTest').null2, parameters('objectToTest').array)]"
        },
        "emptyOutput": {
            "type": "bool",
            "value": "[empty(coalesce(parameters('objectToTest').null1, parameters('objectToTest').null2))]"
        }
    }
}
```

上述示例中使用默认值的输出为：

| Name | 类型 | Value |
| ---- | ---- | ----- |
| stringOutput | String | 默认值 |
| intOutput | int | 1 |
| objectOutput | Object | {"first": "default"} |
| arrayOutput | Array | [1] |
| emptyOutput | Bool | True |

要使用 Azure CLI 部署此示例模板，请使用：

```azurecli
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/coalesce.json
```

要使用 PowerShell 部署此示例模板，请使用：

```powershell
New-AzResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/coalesce.json
```

<a name="concat" />

## <a name="concat"></a>concat
`concat(arg1, arg2, arg3, ...)`

合并多个数组并返回串联的数组，或合并多个字符串值并返回串联的字符串。 

### <a name="parameters"></a>parameters

| 参数 | 必须 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| arg1 |是 |数组或字符串 |要串联的第一个数组或字符串。 |
| 其他参数 |否 |数组或字符串 |按顺序排列的串联的其他数组或字符串。 |

此函数可以使用任意数量的参数，并可接受字符串或数组作为参数。

### <a name="return-value"></a>返回值
由串联值构成的字符串或数组。

### <a name="example"></a>示例

以下[示例模板](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/concat-array.json)演示如何组合两个数组。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": { 
        "firstArray": { 
            "type": "array", 
            "defaultValue": [ 
                "1-1", 
                "1-2", 
                "1-3" 
            ] 
        },
        "secondArray": {
            "type": "array", 
            "defaultValue": [ 
                "2-1", 
                "2-2",
                "2-3" 
            ] 
        }
    },
    "resources": [
    ],
    "outputs": {
        "return": {
            "type": "array",
            "value": "[concat(parameters('firstArray'), parameters('secondArray'))]"
        }
    }
}
```

上述示例中使用默认值的输出为：

| Name | 类型 | Value |
| ---- | ---- | ----- |
| return | Array | ["1-1", "1-2", "1-3", "2-1", "2-2", "2-3"] |

要使用 Azure CLI 部署此示例模板，请使用：

```azurecli
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/concat-array.json
```

要使用 PowerShell 部署此示例模板，请使用：

```powershell
New-AzResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/concat-array.json
```

以下[示例模板](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/concat-string.json)演示如何组合两个字符串值并返回串联的字符串。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "prefix": {
            "type": "string",
            "defaultValue": "prefix"
        }
    },
    "resources": [],
    "outputs": {
        "concatOutput": {
            "value": "[concat(parameters('prefix'), '-', uniqueString(resourceGroup().id))]",
            "type" : "string"
        }
    }
}
```

上述示例中使用默认值的输出为：

| Name | 类型 | Value |
| ---- | ---- | ----- |
| concatOutput | String | prefix-5yj4yjf5mbg72 |

要使用 Azure CLI 部署此示例模板，请使用：

```azurecli
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/concat-string.json
```

要使用 PowerShell 部署此示例模板，请使用：

```powershell
New-AzResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/concat-string.json
```

<a name="contains" />

## <a name="contains"></a>contains
`contains(container, itemToFind)`

检查数组是否包含某个值、某个对象是否包含某个键，或者某个字符串是否包含某个子字符串。 字符串比较区分大小写。 但在测试某个对象是否包含某个键时，该比较不区分大小写。

### <a name="parameters"></a>parameters

| 参数 | 必须 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| container |是 |数组、对象或字符串 |包含要查找的值的值。 |
| itemToFind |是 |字符串或整数 |要查找的值。 |

### <a name="return-value"></a>返回值

如果找到该项，则为 **True**；否则为 **False**。

### <a name="example"></a>示例

以下[示例模板](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/contains.json)演示如何对不同的类型使用 contains：

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "stringToTest": {
            "type": "string",
            "defaultValue": "OneTwoThree"
        },
        "objectToTest": {
            "type": "object",
            "defaultValue": {"one": "a", "two": "b", "three": "c"}
        },
        "arrayToTest": {
            "type": "array",
            "defaultValue": ["one", "two", "three"]
        }
    },
    "resources": [
    ],
    "outputs": {
        "stringTrue": {
            "type": "bool",
            "value": "[contains(parameters('stringToTest'), 'e')]"
        },
        "stringFalse": {
            "type": "bool",
            "value": "[contains(parameters('stringToTest'), 'z')]"
        },
        "objectTrue": {
            "type": "bool",
            "value": "[contains(parameters('objectToTest'), 'one')]"
        },
        "objectFalse": {
            "type": "bool",
            "value": "[contains(parameters('objectToTest'), 'a')]"
        },
        "arrayTrue": {
            "type": "bool",
            "value": "[contains(parameters('arrayToTest'), 'three')]"
        },
        "arrayFalse": {
            "type": "bool",
            "value": "[contains(parameters('arrayToTest'), 'four')]"
        }
    }
}
```

上述示例中使用默认值的输出为：

| Name | 类型 | Value |
| ---- | ---- | ----- |
| stringTrue | Bool | True |
| stringFalse | Bool | False |
| objectTrue | Bool | True |
| objectFalse | Bool | False |
| arrayTrue | Bool | True |
| arrayFalse | Bool | False |

要使用 Azure CLI 部署此示例模板，请使用：

```azurecli
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/contains.json
```

要使用 PowerShell 部署此示例模板，请使用：

```powershell
New-AzResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/contains.json
```

<a name="createarray" />

## <a name="createarray"></a>createarray
`createArray (arg1, arg2, arg3, ...)`

从参数创建数组。

### <a name="parameters"></a>parameters

| 参数 | 必须 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| arg1 |是 |字符串、整数、数组或对象 |数组中的第一个值。 |
| 其他参数 |否 |字符串、整数、数组或对象 |数组中的其他值。 |

### <a name="return-value"></a>返回值

一个数组。

### <a name="example"></a>示例

以下[示例模板](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/createarray.json)演示如何对不同的类型使用 createArra：

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "objectToTest": {
            "type": "object",
            "defaultValue": {"one": "a", "two": "b", "three": "c"}
        },
        "arrayToTest": {
            "type": "array",
            "defaultValue": ["one", "two", "three"]
        }
    },
    "resources": [
    ],
    "outputs": {
        "stringArray": {
            "type": "array",
            "value": "[createArray('a', 'b', 'c')]"
        },
        "intArray": {
            "type": "array",
            "value": "[createArray(1, 2, 3)]"
        },
        "objectArray": {
            "type": "array",
            "value": "[createArray(parameters('objectToTest'))]"
        },
        "arrayArray": {
            "type": "array",
            "value": "[createArray(parameters('arrayToTest'))]"
        }
    }
}
```

上述示例中使用默认值的输出为：

| Name | 类型 | Value |
| ---- | ---- | ----- |
| stringArray | Array | ["a", "b", "c"] |
| intArray | Array | [1, 2, 3] |
| objectArray | Array | [{"one": "a", "two": "b", "three": "c"}] |
| arrayArray | Array | [["one", "two", "three"]] |

要使用 Azure CLI 部署此示例模板，请使用：

```azurecli
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/createarray.json
```

要使用 PowerShell 部署此示例模板，请使用：

```powershell
New-AzResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/createarray.json
```

<a name="empty" />

## <a name="empty"></a>empty

`empty(itemToTest)`

确定数组、对象或字符串是否为空。

### <a name="parameters"></a>parameters

| 参数 | 必须 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| itemToTest |是 |数组、对象或字符串 |要检查是否为空的值。 |

### <a name="return-value"></a>返回值

如果该值为空，则返回 **True**；否则返回 **False**。

### <a name="example"></a>示例

以下[示例模板](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/empty.json)检查某个数组、对象和字符串是否为空。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "testArray": {
            "type": "array",
            "defaultValue": []
        },
        "testObject": {
            "type": "object",
            "defaultValue": {}
        },
        "testString": {
            "type": "string",
            "defaultValue": ""
        }
    },
    "resources": [
    ],
    "outputs": {
        "arrayEmpty": {
            "type": "bool",
            "value": "[empty(parameters('testArray'))]"
        },
        "objectEmpty": {
            "type": "bool",
            "value": "[empty(parameters('testObject'))]"
        },
        "stringEmpty": {
            "type": "bool",
            "value": "[empty(parameters('testString'))]"
        }
    }
}
```

上述示例中使用默认值的输出为：

| Name | 类型 | Value |
| ---- | ---- | ----- |
| arrayEmpty | Bool | True |
| objectEmpty | Bool | True |
| stringEmpty | Bool | True |

要使用 Azure CLI 部署此示例模板，请使用：

```azurecli
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/empty.json
```

要使用 PowerShell 部署此示例模板，请使用：

```powershell
New-AzResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/empty.json
```

<a name="first" />

## <a name="first"></a>first
`first(arg1)`

返回数组的第一个元素，或字符串的第一个字符。

### <a name="parameters"></a>parameters

| 参数 | 必须 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| arg1 |是 |数组或字符串 |要检索第一个元素或字符的值。 |

### <a name="return-value"></a>返回值

数组中第一个元素的类型（字符串、整数、数组或对象），或者字符串的第一个字符。

### <a name="example"></a>示例

以下[示例模板](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/first.json)演示如何对数组和字符串使用 first 函数。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "arrayToTest": {
            "type": "array",
            "defaultValue": ["one", "two", "three"]
        }
    },
    "resources": [
    ],
    "outputs": {
        "arrayOutput": {
            "type": "string",
            "value": "[first(parameters('arrayToTest'))]"
        },
        "stringOutput": {
            "type": "string",
            "value": "[first('One Two Three')]"
        }
    }
}
```

上述示例中使用默认值的输出为：

| Name | 类型 | Value |
| ---- | ---- | ----- |
| arrayOutput | String | one |
| stringOutput | String | O |

要使用 Azure CLI 部署此示例模板，请使用：

```azurecli
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/first.json
```

要使用 PowerShell 部署此示例模板，请使用：

```powershell
New-AzResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/first.json
```

<a name="intersection" />

## <a name="intersection"></a>intersection
`intersection(arg1, arg2, arg3, ...)`

返回包含参数中通用元素的单个数组或对象。

### <a name="parameters"></a>parameters

| 参数 | 必须 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| arg1 |是 |数组或对象 |用于查找通用元素的第一个值。 |
| arg2 |是 |数组或对象 |用于查找通用元素的第二个值。 |
| 其他参数 |否 |数组或对象 |用于查找通用元素的其他值。 |

### <a name="return-value"></a>返回值

包含通用元素的数组或对象。

### <a name="example"></a>示例

以下[示例模板](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/intersection.json)演示如何对数组和对象使用 intersection：

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "firstObject": {
            "type": "object",
            "defaultValue": {"one": "a", "two": "b", "three": "c"}
        },
        "secondObject": {
            "type": "object",
            "defaultValue": {"one": "a", "two": "z", "three": "c"}
        },
        "firstArray": {
            "type": "array",
            "defaultValue": ["one", "two", "three"]
        },
        "secondArray": {
            "type": "array",
            "defaultValue": ["two", "three"]
        }
    },
    "resources": [
    ],
    "outputs": {
        "objectOutput": {
            "type": "object",
            "value": "[intersection(parameters('firstObject'), parameters('secondObject'))]"
        },
        "arrayOutput": {
            "type": "array",
            "value": "[intersection(parameters('firstArray'), parameters('secondArray'))]"
        }
    }
}
```

上述示例中使用默认值的输出为：

| Name | 类型 | Value |
| ---- | ---- | ----- |
| objectOutput | 对象 | {"one": "a", "three": "c"} |
| arrayOutput | Array | ["two", "three"] |

要使用 Azure CLI 部署此示例模板，请使用：

```azurecli
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/intersection.json
```

要使用 PowerShell 部署此示例模板，请使用：

```powershell
New-AzResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/intersection.json
```

## <a name="json"></a>json
`json(arg1)`

返回一个 JSON 对象。

### <a name="parameters"></a>parameters

| 参数 | 必须 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| arg1 |是 |string |要转换为 JSON 的值。 |

### <a name="return-value"></a>返回值

指定字符串中的 JSON 对象，或指定 null 时的空对象  。

### <a name="remarks"></a>备注

如需将某个参数值或变量包含在 JSON 对象中，可使用 [concat](resource-group-template-functions-string.md#concat) 函数创建要传递到此函数的字符串。

### <a name="example"></a>示例

以下[示例模板](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/json.json)演示如何对数组和对象使用 json 函数：

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "testValue": {
            "type": "string",
            "defaultValue": "demo value"
        }
    },
    "resources": [
    ],
    "outputs": {
        "jsonOutput": {
            "type": "object",
            "value": "[json('{\"a\": \"b\"}')]"
        },
        "nullOutput": {
            "type": "bool",
            "value": "[empty(json('null'))]"
        },
        "paramOutput": {
            "type": "object",
            "value": "[json(concat('{\"a\": \"', parameters('testValue'), '\"}'))]"
        }
    }
}
```

上述示例中使用默认值的输出为：

| Name | 类型 | Value |
| ---- | ---- | ----- |
| jsonOutput | Object | {"a": "b"} |
| nullOutput | 布尔 | True |
| paramOutput | Object | {"a": "demo value"}

要使用 Azure CLI 部署此示例模板，请使用：

```azurecli
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/json.json
```

要使用 PowerShell 部署此示例模板，请使用：

```powershell
New-AzResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/json.json
```

<a name="last" />

## <a name="last"></a>last
`last (arg1)`

返回数组的最后一个元素，或字符串的最后一个字符。

### <a name="parameters"></a>parameters

| 参数 | 必须 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| arg1 |是 |数组或字符串 |要检索最后一个元素或字符的值。 |

### <a name="return-value"></a>返回值

数组中最后一个元素的类型（字符串、整数、数组或对象），或者字符串的最后一个字符。

### <a name="example"></a>示例

以下[示例模板](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/last.json)演示如何对数组和字符串使用 last 函数。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "arrayToTest": {
            "type": "array",
            "defaultValue": ["one", "two", "three"]
        }
    },
    "resources": [
    ],
    "outputs": {
        "arrayOutput": {
            "type": "string",
            "value": "[last(parameters('arrayToTest'))]"
        },
        "stringOutput": {
            "type": "string",
            "value": "[last('One Two Three')]"
        }
    }
}
```

上述示例中使用默认值的输出为：

| Name | 类型 | Value |
| ---- | ---- | ----- |
| arrayOutput | String | three |
| stringOutput | String | e |

要使用 Azure CLI 部署此示例模板，请使用：

```azurecli
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/last.json
```

要使用 PowerShell 部署此示例模板，请使用：

```powershell
New-AzResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/last.json
```

<a name="length" />

## <a name="length"></a>length
`length(arg1)`

返回数组中的元素数，或字符串中的字符数。

### <a name="parameters"></a>parameters

| 参数 | 必须 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| arg1 |是 |数组或字符串 |用于获取元素数的数组，或用于获取字符数的字符串。 |

### <a name="return-value"></a>返回值

一个整数。 

### <a name="example"></a>示例

以下[示例模板](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/length.json)演示如何对数组和字符串使用 length：

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "arrayToTest": {
            "type": "array",
            "defaultValue": [
                "one",
                "two",
                "three"
            ]
        },
        "stringToTest": {
            "type": "string",
            "defaultValue": "One Two Three"
        }
    },
    "resources": [],
    "outputs": {
        "arrayLength": {
            "type": "int",
            "value": "[length(parameters('arrayToTest'))]"
        },
        "stringLength": {
            "type": "int",
            "value": "[length(parameters('stringToTest'))]"
        }
    }
}
```

上述示例中使用默认值的输出为：

| Name | 类型 | Value |
| ---- | ---- | ----- |
| arrayLength | int | 3 |
| stringLength | int | 13 |

要使用 Azure CLI 部署此示例模板，请使用：

```azurecli
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/length.json
```

要使用 PowerShell 部署此示例模板，请使用：

```powershell
New-AzResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/length.json
```

创建资源时，可在数组中使用此函数指定迭代数。 在以下示例中，参数 **siteNames** 引用创建网站时要使用的名称数组。

```json
"copy": {
    "name": "websitescopy",
    "count": "[length(parameters('siteNames'))]"
}
```

有关在数组中使用此函数的详细信息，请参阅 [Create multiple instances of resources in Azure Resource Manager](resource-group-create-multiple.md)（在 Azure Resource Manager 中创建多个资源实例）。

<a name="max" />

## <a name="max"></a>max
`max(arg1)`

返回整数数组或逗号分隔的整数列表中的最大值。

### <a name="parameters"></a>parameters

| 参数 | 必须 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| arg1 |是 |整数数组或逗号分隔的整数列表 |要获取最大值的集合。 |

### <a name="return-value"></a>返回值

表示最大值的整数。

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
`min(arg1)`

返回整数数组或逗号分隔的整数列表中的最小值。

### <a name="parameters"></a>parameters

| 参数 | 必须 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| arg1 |是 |整数数组或逗号分隔的整数列表 |要获取最小值的集合。 |

### <a name="return-value"></a>返回值

表示最小值的整数。

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

<a name="range" />

## <a name="range"></a>range
`range(startingInteger, numberOfElements)`

从起始整数创建整数数组并包含一些项。

### <a name="parameters"></a>parameters

| 参数 | 必须 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| startingInteger |是 |int |数组中的第一个整数。 |
| numberofElements |是 |int |数组中的整数个数。 |

### <a name="return-value"></a>返回值

整数数组。

### <a name="example"></a>示例

以下[示例模板](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/range.json)演示如何使用 range 函数:

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "startingInt": {
            "type": "int",
            "defaultValue": 5
        },
        "numberOfElements": {
            "type": "int",
            "defaultValue": 3
        }
    },
    "resources": [],
    "outputs": {
        "rangeOutput": {
            "type": "array",
            "value": "[range(parameters('startingInt'),parameters('numberOfElements'))]"
        }
    }
}
```

上述示例中使用默认值的输出为：

| Name | 类型 | Value |
| ---- | ---- | ----- |
| rangeOutput | Array | [5, 6, 7] |

要使用 Azure CLI 部署此示例模板，请使用：

```azurecli
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/range.json
```

要使用 PowerShell 部署此示例模板，请使用：

```powershell
New-AzResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/range.json
```

<a name="skip" />

## <a name="skip"></a>skip
`skip(originalValue, numberToSkip)`

返回一个数组，其中包含数组中指定数字后面的所有元素；或返回一个字符串，其中包含字符串中指定数后面的所有字符。

### <a name="parameters"></a>parameters

| 参数 | 必须 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| originalValue |是 |数组或字符串 |用于跳过的数组或字符串。 |
| numberToSkip |是 |int |要跳过的元素数或字符数。 如果此值小于或等于 0，则返回值中的所有元素或字符。 如果此值大于数组或字符串的长度，则返回空数组或字符串。 |

### <a name="return-value"></a>返回值

数组或字符串。

### <a name="example"></a>示例

以下[示例模板](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/skip.json)跳过数组中指定数目的元素，以及字符串中指定数目的字符。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "testArray": {
            "type": "array",
            "defaultValue": [
                "one",
                "two",
                "three"
            ]
        },
        "elementsToSkip": {
            "type": "int",
            "defaultValue": 2
        },
        "testString": {
            "type": "string",
            "defaultValue": "one two three"
        },
        "charactersToSkip": {
            "type": "int",
            "defaultValue": 4
        }
    },
    "resources": [],
    "outputs": {
        "arrayOutput": {
            "type": "array",
            "value": "[skip(parameters('testArray'),parameters('elementsToSkip'))]"
        },
        "stringOutput": {
            "type": "string",
            "value": "[skip(parameters('testString'),parameters('charactersToSkip'))]"
        }
    }
}
```

上述示例中使用默认值的输出为：

| Name | 类型 | Value |
| ---- | ---- | ----- |
| arrayOutput | Array | ["three"] |
| stringOutput | String | two three |

要使用 Azure CLI 部署此示例模板，请使用：

```azurecli
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/skip.json
```

要使用 PowerShell 部署此示例模板，请使用：

```powershell
New-AzResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/skip.json
```

<a name="take" />

## <a name="take"></a>take
`take(originalValue, numberToTake)`

返回一个数组，其中包含从数组开头位置算起的指定数目的元素；或返回一个字符串，其中包含从字符串开头位置算起的指定数目的字符。

### <a name="parameters"></a>parameters

| 参数 | 必须 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| originalValue |是 |数组或字符串 |要从中提取元素的数组或字符串。 |
| numberToTake |是 |int |要提取的元素或字符数。 如果此值为 0 或更小，则返回一个空数组或字符串。 如果此值大于给定数组或字符串的长度，则返回数组或字符串中的所有元素。 |

### <a name="return-value"></a>返回值

数组或字符串。

### <a name="example"></a>示例

以下[示例模板](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/take.json)从数组中提取指定数目的元素，并从字符串中提取指定数目的字符。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "testArray": {
            "type": "array",
            "defaultValue": [
                "one",
                "two",
                "three"
            ]
        },
        "elementsToTake": {
            "type": "int",
            "defaultValue": 2
        },
        "testString": {
            "type": "string",
            "defaultValue": "one two three"
        },
        "charactersToTake": {
            "type": "int",
            "defaultValue": 2
        }
    },
    "resources": [],
    "outputs": {
        "arrayOutput": {
            "type": "array",
            "value": "[take(parameters('testArray'),parameters('elementsToTake'))]"
        },
        "stringOutput": {
            "type": "string",
            "value": "[take(parameters('testString'),parameters('charactersToTake'))]"
        }
    }
}
```

上述示例中使用默认值的输出为：

| Name | 类型 | Value |
| ---- | ---- | ----- |
| arrayOutput | Array | ["one", "two"] |
| stringOutput | String | on |

要使用 Azure CLI 部署此示例模板，请使用：

```azurecli
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/take.json
```

要使用 PowerShell 部署此示例模板，请使用：

```powershell
New-AzResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/take.json
```

<a name="union" />

## <a name="union"></a>union
`union(arg1, arg2, arg3, ...)`

返回包含参数中所有元素的单个数组或对象。 重复的值或键仅包含一次。

### <a name="parameters"></a>parameters

| 参数 | 必须 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| arg1 |是 |数组或对象 |用于联接元素的第一个值。 |
| arg2 |是 |数组或对象 |用于联接元素的第二个值。 |
| 其他参数 |否 |数组或对象 |用于联接元素的其他值。 |

### <a name="return-value"></a>返回值

数组或对象。

### <a name="example"></a>示例

以下[示例模板](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/union.json)演示如何对数组和对象使用 union：

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "firstObject": {
            "type": "object",
            "defaultValue": {"one": "a", "two": "b", "three": "c1"}
        },
        "secondObject": {
            "type": "object",
            "defaultValue": {"three": "c2", "four": "d", "five": "e"}
        },
        "firstArray": {
            "type": "array",
            "defaultValue": ["one", "two", "three"]
        },
        "secondArray": {
            "type": "array",
            "defaultValue": ["three", "four"]
        }
    },
    "resources": [
    ],
    "outputs": {
        "objectOutput": {
            "type": "object",
            "value": "[union(parameters('firstObject'), parameters('secondObject'))]"
        },
        "arrayOutput": {
            "type": "array",
            "value": "[union(parameters('firstArray'), parameters('secondArray'))]"
        }
    }
}
```

上述示例中使用默认值的输出为：

| Name | 类型 | Value |
| ---- | ---- | ----- |
| objectOutput | Object | {"one": "a", "two": "b", "three": "c2", "four": "d", "five": "e"} |
| arrayOutput | Array | ["one", "two", "three", "four"] |

要使用 Azure CLI 部署此示例模板，请使用：

```azurecli
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/union.json
```

要使用 PowerShell 部署此示例模板，请使用：

```powershell
New-AzResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/union.json
```

## <a name="next-steps"></a>后续步骤
* 有关 Azure 资源管理器模板中各部分的说明，请参阅[创作 Azure 资源管理器模板](resource-group-authoring-templates.md)。
* 若要合并多个模板，请参阅[将链接的模板与 Azure Resource Manager 配合使用](resource-group-linked-templates.md)。
* 若要在创建资源类型时迭代指定的次数，请参阅[在 Azure Resource Manager 中创建多个资源实例](resource-group-create-multiple.md)。
* 若要查看如何部署已创建的模板，请参阅[使用 Azure Resource Manager 模板部署应用程序](resource-group-template-deploy.md)。

<!--Update_Description: update meta properties, wording update, update cmdlet-->