---
title: Azure 实例元数据服务 | Azure
description: 一个 RESTful 接口，用于获取有关 Windows VM 计算、网络和即将发生的维护事件的信息。
services: virtual-machines-windows
documentationcenter: ''
author: rockboyfor
manager: digimobile
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
origin.date: 04/25/2019
ms.date: 07/01/2019
ms.author: v-yeche
ms.reviewer: azmetadata
ms.openlocfilehash: 6a7ca1988a2840aa1836bbb42b9b0bc309ce60bc
ms.sourcegitcommit: 9e50dde3362b6e6b192761ead6cd3f434dfb2168
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/10/2019
ms.locfileid: "67725221"
---
# <a name="azure-instance-metadata-service"></a>Azure 实例元数据服务

Azure 实例元数据服务提供有关可用于管理和配置虚拟机的正在运行的虚拟机实例的信息。
这包括 SKU、网络配置和即将发生的维护事件等信息。 若要详细了解可用信息类型，请参阅[元数据 API](#metadata-apis)。

Azure 的实例元数据服务是一个 REST 终结点，所有创建的 IaaS VM 可通过 [Azure Resource Manager](https://docs.microsoft.com/rest/api/resources/) 访问此终结点。
该终结点位于已知不可路由的 IP 地址 (`169.254.169.254`)，该地址只能从 VM 中访问。

> [!IMPORTANT]
> 此服务在所有 Azure 区域中提供有正式版  。  它会定期接收更新，发布有关虚拟机实例的新信息。 本页反映了最新可用的[元数据 API](#metadata-apis)。

## <a name="service-availability"></a>服务可用性

此服务在所有 Azure 区域中提供有可用的正式版。 并非所有 API 版本在所有 Azure 区域中可用。

Regions                                        | 可用性？                                 | 支持的版本
-----------------------------------------------|-----------------------------------------------|-----------------
[全球所有公开上市的 Azure 区域](https://azure.microsoft.com/regions/)     | 正式版   | 2017-04-02、2017-08-01、2017-12-01、2018-02-01、2018-04-02、2018-10-01
[Azure 美国政府版](https://azure.microsoft.com/overview/clouds/government/)              | 正式版 | 2017-04-02、2017-08-01、2017-12-01、2018-02-01、2018-04-02、2018-10-01
[Azure 中国](https://www.azure.cn/)                                                           | 正式版 | 2017-04-02、2017-08-01、2017-12-01、2018-02-01、2018-04-02、2018-10-01
[Azure 德国](https://azure.microsoft.com/overview/clouds/germany/)                    | 正式版 | 2017-04-02、2017-08-01、2017-12-01、2018-02-01、2018-04-02、2018-10-01

<!-- [All Generally Available Global Azure Regions] Should be https://azure.microsoft.com/regions/ -->
<!--Not Available on 2019-02-01-->

当有服务更新且/或有可用的新支持版本时，此表将更新。

<!--Not Available on 2019-02-01-->

若要试用实例元数据服务，请在上述区域中从 [Azure 资源管理器](https://docs.microsoft.com/rest/api/resources/)或 [Azure 门户](https://portal.azure.cn)创建一个 VM，并按照以下示例操作。

## <a name="usage"></a>使用情况

### <a name="versioning"></a>版本控制

已对实例元数据服务进行了版本控制。 版本是必需的，全局 Azure 上的当前版本为 `2018-10-01`。 当前支持的版本为（2017-04-02、2017-08-01、2017-12-01、2018-02-01、2018-04-02、2018-10-01）。

在添加更新的版本时，早期版本仍可供访问以保持兼容性（如果脚本依赖于特定的数据格式）。

如果未指定版本，则会返回错误并列出受支持的最新版本。

> [!NOTE]
> 此响应是 JSON 字符串。 以下示例响应显示清晰，可供阅读。

**请求**

```bash
curl -H Metadata:true "http://169.254.169.254/metadata/instance"
```

**响应**

```json
{
    "error": "Bad request. api-version was not specified in the request. For more information refer to aka.ms/azureimds",
    "newest-versions": [
        "2018-10-01",
        "2018-04-02",
        "2018-02-01"
    ]
}
```

### <a name="using-headers"></a>使用标头

查询实例元数据服务时，必须提供标头 `Metadata: true` 以确保不会意外重定向请求。

### <a name="retrieving-metadata"></a>检索元数据

实例元数据可用于运行使用 [Azure Resource Manager](https://docs.microsoft.com/rest/api/resources/) 创建/管理的 VM。 使用以下请求访问虚拟机实例的所有数据类别：

```bash
curl -H Metadata:true "http://169.254.169.254/metadata/instance?api-version=2017-08-01"
```

> [!NOTE]
> 所有实例元数据查询都区分大小写。

### <a name="data-output"></a>数据输出

默认情况下，实例元数据服务会返回 JSON 格式的数据 (`Content-Type: application/json`)。 但是，如果提出请求，不同 API 可以其他格式返回数据。
下表是有关 API 可支持的其他数据格式的参考。

API | 默认数据格式 | 其他格式
--------|---------------------|--------------
/instance | json | text
/scheduledevents | json | 无
/attested | json | 无

若要访问非默认响应格式，请在请求中将所请求的格式指定为查询字符串参数。 例如：

```bash
curl -H Metadata:true "http://169.254.169.254/metadata/instance?api-version=2017-08-01&format=text"
```

> [!NOTE]
> 对于叶节点，`format=json` 不起作用。 对于这些查询，如果默认格式是 JSON，则需要显式指定 `format=text`。

### <a name="security"></a>安全性

此实例元数据服务终结点只能从不可路由的 IP 地址上正在运行的虚拟机实例中访问。 此外，任何包含`X-Forwarded-For`标头的请求都会被服务拒绝。
请求必须包含 `Metadata: true` 标头，以确保实际请求是直接计划好的，而不是无意重定向的一部分。

### <a name="error"></a>错误

如果找不到某个数据元素，或者请求的格式不正确，则实例元数据服务返回标准 HTTP 错误。 例如：

HTTP 状态代码 | 原因
----------------|-------
200 正常 |
400 错误的请求 | 查询叶节点时缺少 `Metadata: true` 标头或缺少格式
404 未找到 | 请求的元素不存在
不允许 405 方法 | 仅支持 `GET` 和 `POST` 请求
429 请求次数过多 | 该 API 当前支持每秒最多 5 个查询
500 服务错误     | 请稍后重试

### <a name="examples"></a>示例

> [!NOTE]
> 所有 API 响应都是 JSON 字符串。 以下所有响应示例都以美观的形式输出以提高可读性。

#### <a name="retrieving-network-information"></a>检索网络信息

**请求**

```bash
curl -H Metadata:true "http://169.254.169.254/metadata/instance/network?api-version=2017-08-01"
```

**响应**

> [!NOTE]
> 此响应是 JSON 字符串。 以下示例响应显示清晰，可供阅读。

```json
{
  "interface": [
    {
      "ipv4": {
        "ipAddress": [
          {
            "privateIpAddress": "10.1.0.4",
            "publicIpAddress": "X.X.X.X"
          }
        ],
        "subnet": [
          {
            "address": "10.1.0.0",
            "prefix": "24"
          }
        ]
      },
      "ipv6": {
        "ipAddress": []
      },
      "macAddress": "000D3AF806EC"
    }
  ]
}

```

#### <a name="retrieving-public-ip-address"></a>检索公共 IP 地址

```bash
curl -H Metadata:true "http://169.254.169.254/metadata/instance/network/interface/0/ipv4/ipAddress/0/publicIpAddress?api-version=2017-08-01&format=text"
```

#### <a name="retrieving-all-metadata-for-an-instance"></a>检索实例的所有元数据

**请求**

```bash
curl -H Metadata:true "http://169.254.169.254/metadata/instance?api-version=2018-10-01"
```

**响应**

> [!NOTE]
> 此响应是 JSON 字符串。 以下示例响应显示清晰，可供阅读。

<!--MOONCAKE: CUSTOMIZE-->

```json
{
  "compute": {
    "azEnvironment": "AZURECHINACLOUD",
    "location": "chinanorth",
    "name": "jubilee",
    "offer": "Windows-10",
    "osType": "Windows",
    "placementGroupId": "",
    "plan": {
        "name": "",
        "product": "",
        "publisher": ""
    },
    "platformFaultDomain": "1",
    "platformUpdateDomain": "1",
    "provider": "Microsoft.Compute",
    "publicKeys": [],
    "publisher": "MicrosoftWindowsDesktop",
    "resourceGroupName": "myrg",
    "sku": "rs4-pro",
    "subscriptionId": "xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx",
    "tags": "Department:IT;Environment:Prod;Role:WorkerRole",
    "version": "17134.345.59",
    "vmId": "13f56399-bd52-4150-9748-7190aae1ff21",
    "vmScaleSetName": "",
    "vmSize": "Standard_D1",
    "zone": ""
  },
  "network": {
    "interface": [
      {
        "ipv4": {
          "ipAddress": [
            {
              "privateIpAddress": "10.1.2.5",
              "publicIpAddress": "X.X.X.X"
            }
          ],
          "subnet": [
            {
              "address": "10.1.2.0",
              "prefix": "24"
            }
          ]
        },
        "ipv6": {
          "ipAddress": []
        },
        "macAddress": "000D3A36DDED"
      }
    ]
  }
}
```

#### <a name="retrieving-metadata-in-windows-virtual-machine"></a>在 Windows 虚拟机中检索元数据

**请求**

可通过 `curl` 程序在 Windows 中检索实例元数据：

```bash
curl -H @{'Metadata'='true'} http://169.254.169.254/metadata/instance?api-version=2018-10-01 | select -ExpandProperty Content
```

还可以通过 `Invoke-RestMethod` PowerShell cmdlet 检索：

```powershell

Invoke-RestMethod -Headers @{"Metadata"="true"} -URI http://169.254.169.254/metadata/instance?api-version=2018-10-01 -Method get
```

**响应**

> [!NOTE]
> 此响应是 JSON 字符串。 以下示例响应显示清晰，可供阅读。

<!--MOONCAKE: CUSTOMIZE-->

```json
{
  "compute": {
    "azEnvironment": "AZURECHINACLOUD",
    "location": "chinanorth",
    "name": "SQLTest",
    "offer": "SQL2016SP1-WS2016",
    "osType": "Windows",
    "placementGroupId": "",
    "plan": {
        "name": "",
        "product": "",
        "publisher": ""
    },
    "platformFaultDomain": "0",
    "platformUpdateDomain": "0",
    "provider": "Microsoft.Compute",
    "publicKeys": [],
    "publisher": "MicrosoftSQLServer",
    "resourceGroupName": "myrg",
    "sku": "Enterprise",
    "subscriptionId": "xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx",
    "tags": "Department:IT;Environment:Test;Role:WebRole",
    "version": "13.0.400110",
    "vmId": "453945c8-3923-4366-b2d3-ea4c80e9b70e",
    "vmScaleSetName": "",
    "vmSize": "Standard_DS2",
    "zone": ""
  },
  "network": {
    "interface": [
      {
        "ipv4": {
          "ipAddress": [
            {
              "privateIpAddress": "10.0.1.4",
              "publicIpAddress": "X.X.X.X"
            }
          ],
          "subnet": [
            {
              "address": "10.0.1.0",
              "prefix": "24"
            }
          ]
        },
        "ipv6": {
          "ipAddress": []
        },
        "macAddress": "002248020E1E"
      }
    ]
  }
}
```

## <a name="metadata-apis"></a>元数据 API

#### <a name="the-following-apis-are-available-through-the-metadata-endpoint"></a>可以通过元数据终结点使用以下 API：

数据 | 说明 | 引入的版本
-----|-------------|-----------------------
attested | 请参阅[证明数据](#attested-data) | 2018-10-01
instance | 请参阅[实例 API](#instance-api) | 2017-04-02
scheduledevents | 请参阅[计划事件](scheduled-events.md) | 2017-08-01

<!--Not Available on identity | Managed identities for Azure resources. See [acquire an access token](../../active-directory/managed-identities-azure-resources/how-to-use-vm-token.md)-->

#### <a name="instance-api"></a>实例 API
##### <a name="the-following-compute-categories-are-available-through-the-instance-api"></a>可以通过实例 API 使用以下计算类别：

> [!NOTE]
> 在元数据终结点中通过实例/计算访问以下类别

数据 | 说明 | 引入的版本
-----|-------------|-----------------------
azEnvironment | VM 运行时所在的 Azure 环境 | 2018-10-01
location | VM 在其中运行的 Azure 区域 | 2017-04-02
name | VM 的名称 | 2017-04-02
offer | 提供 VM 映像的信息，仅适用于从 Azure 映像库部署的映像 | 2017-04-02
osType | Linux 或 Windows | 2017-04-02
计划 | [计划](https://docs.microsoft.com/rest/api/compute/virtualmachines/createorupdate#plan)包含 VM 的名称、产品和发布者（如果是 Azure 市场映像） | 2018-04-02
platformUpdateDomain |  正在运行 VM 的[更新域](manage-availability.md) | 2017-04-02
platformFaultDomain | 正在运行 VM 的[容错域](manage-availability.md) | 2017-04-02
provider | VM 的提供商 | 2018-10-01
publicKeys | [公钥的集合](https://docs.microsoft.com/rest/api/compute/virtualmachines/createorupdate#sshpublickey)，已分配给 VM 和路径 | 2018-04-02
publisher | VM 映像的发布者 | 2017-04-02
resourceGroupName | 虚拟机的[资源组](../../azure-resource-manager/resource-group-overview.md) | 2017-08-01
sku | VM 映像的特定 SKU | 2017-04-02
subscriptionId | 虚拟机的 Azure 订阅 | 2017-08-01
标记 | 虚拟机的[标记](../../azure-resource-manager/resource-group-using-tags.md)  | 2017-08-01
版本 | VM 映像的版本 | 2017-04-02
vmId | VM 的[唯一标识符](https://azure.microsoft.com/blog/accessing-and-using-azure-vm-unique-id/) | 2017-04-02
vmScaleSetName | 虚拟机规模集的[虚拟机规模集名称](../../virtual-machine-scale-sets/virtual-machine-scale-sets-overview.md) | 2017-12-01
vmSize | [VM 大小](sizes.md) | 2017-04-02

<!--Not Availablle on customData | See [Custom Data](#custom-data)-->
<!--Not Availablle on placementGroupId | [Placement Group](../../virtual-machine-scale-sets/virtual-machine-scale-sets-placement-groups.md)-->
<!--Not Availablle on zone | [Availability Zone](../../availability-zones/az-overview.md)-->

##### <a name="the-following-network-categories-are-available-through-the-instance-api"></a>可以通过实例 API 使用以下网络类别：

> [!NOTE]
> 在元数据终结点中通过实例/网络/接口访问以下类别

数据 | 说明 | 引入的版本
-----|-------------|-----------------------
ipv4/privateIpAddress | VM 的本地 IPv4 地址 | 2017-04-02
ipv4/publicIpAddress | VM 的公共 IPv4 地址 | 2017-04-02
subnet/address | VM 的子网地址 | 2017-04-02
subnet/prefix | 子网前缀，例如 24 | 2017-04-02
macAddress | VM mac 地址 | 2017-04-02

<!-- Not Available on ipv6/ipAddress | Local IPv6 address of the VM | 2017-04-02 -->

## <a name="attested-data"></a>证明数据

实例元数据在 http 终结点上响应 169.254.169.254。 实例元数据服务提供的部分方案是为了保证响应的数据来自 Azure。 我们对此信息的一部分进行签名，以便市场映像可以确保其映像在 Azure 上运行。

### <a name="example-attested-data"></a>示例证明数据

> [!NOTE]
> 所有 API 响应都是 JSON 字符串。 以下示例响应显示清晰，可供阅读。

**请求**

```bash
curl -H Metadata:true "http://169.254.169.254/metadata/attested/document?api-version=2018-10-01&nonce=1234567890"

```

Api-version 是必填字段，证明数据支持的版本为 2018-10-01。
Nonce 是一个可选的 10 位字符串。 Nonce 可用于跟踪请求，如果未提供，则响应编码字符串中会返回当前 UTC 时间戳。

**响应**

> [!NOTE]
> 此响应是 JSON 字符串。 以下示例响应显示清晰，可供阅读。

 ```json
{
 "encoding":"pkcs7","signature":"MIIEEgYJKoZIhvcNAQcCoIIEAzCCA/8CAQExDzANBgkqhkiG9w0BAQsFADCBugYJKoZIhvcNAQcBoIGsBIGpeyJub25jZSI6IjEyMzQ1NjY3NjYiLCJwbGFuIjp7Im5hbWUiOiIiLCJwcm9kdWN0IjoiIiwicHVibGlzaGVyIjoiIn0sInRpbWVTdGFtcCI6eyJjcmVhdGVkT24iOiIxMS8yMC8xOCAyMjowNzozOSAtMDAwMCIsImV4cGlyZXNPbiI6IjExLzIwLzE4IDIyOjA4OjI0IC0wMDAwIn0sInZtSWQiOiIifaCCAj8wggI7MIIBpKADAgECAhBnxW5Kh8dslEBA0E2mIBJ0MA0GCSqGSIb3DQEBBAUAMCsxKTAnBgNVBAMTIHRlc3RzdWJkb21haW4ubWV0YWRhdGEuYXp1cmUuY29tMB4XDTE4MTEyMDIxNTc1N1oXDTE4MTIyMDIxNTc1NlowKzEpMCcGA1UEAxMgdGVzdHN1YmRvbWFpbi5tZXRhZGF0YS5henVyZS5jb20wgZ8wDQYJKoZIhvcNAQEBBQADgY0AMIGJAoGBAML/tBo86ENWPzmXZ0kPkX5dY5QZ150mA8lommszE71x2sCLonzv4/UWk4H+jMMWRRwIea2CuQ5RhdWAHvKq6if4okKNt66fxm+YTVz9z0CTfCLmLT+nsdfOAsG1xZppEapC0Cd9vD6NCKyE8aYI1pliaeOnFjG0WvMY04uWz2MdAgMBAAGjYDBeMFwGA1UdAQRVMFOAENnYkHLa04Ut4Mpt7TkJFfyhLTArMSkwJwYDVQQDEyB0ZXN0c3ViZG9tYWluLm1ldGFkYXRhLmF6dXJlLmNvbYIQZ8VuSofHbJRAQNBNpiASdDANBgkqhkiG9w0BAQQFAAOBgQCLSM6aX5Bs1KHCJp4VQtxZPzXF71rVKCocHy3N9PTJQ9Fpnd+bYw2vSpQHg/AiG82WuDFpPReJvr7Pa938mZqW9HUOGjQKK2FYDTg6fXD8pkPdyghlX5boGWAMMrf7bFkup+lsT+n2tRw2wbNknO1tQ0wICtqy2VqzWwLi45RBwTGB6DCB5QIBATA/MCsxKTAnBgNVBAMTIHRlc3RzdWJkb21haW4ubWV0YWRhdGEuYXp1cmUuY29tAhBnxW5Kh8dslEBA0E2mIBJ0MA0GCSqGSIb3DQEBCwUAMA0GCSqGSIb3DQEBAQUABIGAld1BM/yYIqqv8SDE4kjQo3Ul/IKAVR8ETKcve5BAdGSNkTUooUGVniTXeuvDj5NkmazOaKZp9fEtByqqPOyw/nlXaZgOO44HDGiPUJ90xVYmfeK6p9RpJBu6kiKhnnYTelUk5u75phe5ZbMZfBhuPhXmYAdjc7Nmw97nx8NnprQ="
}
```

> [!NOTE]
> 签名 Blob 是 [pkcs7](https://aka.ms/pkcs7) 签名的文档版本。 它包含用于签名的证书以及 VM 详情，如 vmId、nonce、文档创建和到期的时间戳以及关于映像的计划信息。 计划信息只针对 Azure 市场映像填充。 证书可从响应中提取，用于验证响应是否有效、是否来自 Azure。

#### <a name="retrieving-attested-metadata-in-windows-virtual-machine"></a>检索 Windows 虚拟机中的证明元数据

**请求**

可以通过 Powershell 实用工具 `curl` 在 Windows 中检索实例元数据：

```bash
curl -H @{'Metadata'='true'} "http://169.254.169.254/metadata/attested/document?api-version=2018-10-01&nonce=1234567890" | select -ExpandProperty Content
```

或通过 `Invoke-RestMethod`cmdlet：

```powershell
Invoke-RestMethod -Headers @{"Metadata"="true"} -URI "http://169.254.169.254/metadata/attested/document?api-version=2018-10-01&nonce=1234567890" -Method get
```

Api-version 是必填字段，证明数据支持的版本为 2018-10-01。
Nonce 是一个可选的 10 位字符串。 Nonce 可用于跟踪请求，如果未提供，则响应编码字符串中会返回当前 UTC 时间戳。

**响应**

> [!NOTE]
> 此响应是 JSON 字符串。 以下示例响应显示清晰，可供阅读。

```json
{
 "encoding":"pkcs7","signature":"MIIEEgYJKoZIhvcNAQcCoIIEAzCCA/8CAQExDzANBgkqhkiG9w0BAQsFADCBugYJKoZIhvcNAQcBoIGsBIGpeyJub25jZSI6IjEyMzQ1NjY3NjYiLCJwbGFuIjp7Im5hbWUiOiIiLCJwcm9kdWN0IjoiIiwicHVibGlzaGVyIjoiIn0sInRpbWVTdGFtcCI6eyJjcmVhdGVkT24iOiIxMS8yMC8xOCAyMjowNzozOSAtMDAwMCIsImV4cGlyZXNPbiI6IjExLzIwLzE4IDIyOjA4OjI0IC0wMDAwIn0sInZtSWQiOiIifaCCAj8wggI7MIIBpKADAgECAhBnxW5Kh8dslEBA0E2mIBJ0MA0GCSqGSIb3DQEBBAUAMCsxKTAnBgNVBAMTIHRlc3RzdWJkb21haW4ubWV0YWRhdGEuYXp1cmUuY29tMB4XDTE4MTEyMDIxNTc1N1oXDTE4MTIyMDIxNTc1NlowKzEpMCcGA1UEAxMgdGVzdHN1YmRvbWFpbi5tZXRhZGF0YS5henVyZS5jb20wgZ8wDQYJKoZIhvcNAQEBBQADgY0AMIGJAoGBAML/tBo86ENWPzmXZ0kPkX5dY5QZ150mA8lommszE71x2sCLonzv4/UWk4H+jMMWRRwIea2CuQ5RhdWAHvKq6if4okKNt66fxm+YTVz9z0CTfCLmLT+nsdfOAsG1xZppEapC0Cd9vD6NCKyE8aYI1pliaeOnFjG0WvMY04uWz2MdAgMBAAGjYDBeMFwGA1UdAQRVMFOAENnYkHLa04Ut4Mpt7TkJFfyhLTArMSkwJwYDVQQDEyB0ZXN0c3ViZG9tYWluLm1ldGFkYXRhLmF6dXJlLmNvbYIQZ8VuSofHbJRAQNBNpiASdDANBgkqhkiG9w0BAQQFAAOBgQCLSM6aX5Bs1KHCJp4VQtxZPzXF71rVKCocHy3N9PTJQ9Fpnd+bYw2vSpQHg/AiG82WuDFpPReJvr7Pa938mZqW9HUOGjQKK2FYDTg6fXD8pkPdyghlX5boGWAMMrf7bFkup+lsT+n2tRw2wbNknO1tQ0wICtqy2VqzWwLi45RBwTGB6DCB5QIBATA/MCsxKTAnBgNVBAMTIHRlc3RzdWJkb21haW4ubWV0YWRhdGEuYXp1cmUuY29tAhBnxW5Kh8dslEBA0E2mIBJ0MA0GCSqGSIb3DQEBCwUAMA0GCSqGSIb3DQEBAQUABIGAld1BM/yYIqqv8SDE4kjQo3Ul/IKAVR8ETKcve5BAdGSNkTUooUGVniTXeuvDj5NkmazOaKZp9fEtByqqPOyw/nlXaZgOO44HDGiPUJ90xVYmfeK6p9RpJBu6kiKhnnYTelUk5u75phe5ZbMZfBhuPhXmYAdjc7Nmw97nx8NnprQ="
}
```

> 签名 Blob 是 [pkcs7](https://aka.ms/pkcs7) 签名的文档版本。 它包含用于签名的证书以及 VM 详情，如 vmId、nonce、文档创建和到期的时间戳以及关于映像的计划信息。 计划信息只针对 Azure 市场映像填充。 证书可从响应中提取，用于验证响应是否有效、是否来自 Azure。

## <a name="example-scenarios-for-usage"></a>用法的示例方案  

### <a name="tracking-vm-running-on-azure"></a>跟踪 Azure 上正在运行的 VM

作为服务提供商，可能需要跟踪运行软件的 VM 数目，或者代理需要跟踪 VM 的唯一性。 为了能够获取 VM 的唯一 ID，请使用实例元数据服务中的 `vmId` 字段。

**请求**

```bash
curl -H Metadata:true "http://169.254.169.254/metadata/instance/compute/vmId?api-version=2017-08-01&format=text"
```

**响应**

```text
5c08b38e-4d57-4c23-ac45-aca61037f084
```

### <a name="placement-of-containers-data-partitions-based-faultupdate-domain"></a>基于容错/更新域放置容器、数据分区 

对于某些方案，不同数据副本的放置至关重要。 例如，对于 [HDFS 副本放置](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html#Replica_Placement:_The_First_Baby_Steps)或者对于通过 [orchestrator](https://kubernetes.io/docs/user-guide/node-selection/) 放置容器，可能需要知道正在运行 VM 的 `platformFaultDomain` 和 `platformUpdateDomain`。

<!-- Not Available on [Availability Zones](../../availability-zones/az-overview.md) -->

可以直接通过实例元数据服务查询此数据。

**请求**

```bash
curl -H Metadata:true "http://169.254.169.254/metadata/instance/compute/platformFaultDomain?api-version=2017-08-01&format=text"
```

**响应**

```text
0
```

### <a name="getting-more-information-about-the-vm-during-support-case"></a>在支持案例期间获取有关 VM 的详细信息

作为服务提供商，你可能会接到支持电话，了解有关 VM 的详细信息。 请求客户共享计算元数据可以提供基本信息，以支持专业人员了解有关 Azure 上的 VM 类型。 

**请求**

```bash
curl -H Metadata:true "http://169.254.169.254/metadata/instance/compute?api-version=2017-08-01"
```

**响应**

> [!NOTE]
> 此响应是 JSON 字符串。 以下示例响应显示清晰，可供阅读。

```json
{
  "compute": {
    "location": "chinanorth",
    "name": "IMDSCanary",
    "offer": "RHEL",
    "osType": "Linux",
    "platformFaultDomain": "0",
    "platformUpdateDomain": "0",
    "publisher": "RedHat",
    "sku": "7.2",
    "version": "7.2.20161026",
    "vmId": "5c08b38e-4d57-4c23-ac45-aca61037f084",
    "vmSize": "Standard_DS2"
  }
}
```

### <a name="getting-azure-environment-where-the-vm-is-running"></a>获取 VM 所在的 Azure 环境

Azure 具有各种主权云，如 Azure 中国云。 有时你需要使用 Azure 环境来做出一些运行时决策。 以下示例显示了如何实现此行为。

<!--Not Available on [Azure China Cloud](https://azure.microsoft.com/overview/clouds/government/)-->

**请求**
```bash
curl -H Metadata:true "http://169.254.169.254/metadata/instance/compute/azEnvironment?api-version=2018-10-01&format=text"
```

**响应**
```bash
AZURECHINACLOUD
```

### <a name="getting-the-tags-for-the-vm"></a>获取 VM 的标记

系统可能已将标记应用到 Azure VM，以便按逻辑方式将其组织成分类。 可使用以下请求检索分配给 VM 的标记。

**请求**

```bash
curl -H Metadata:true "http://169.254.169.254/metadata/instance/compute/tags?api-version=2018-10-01&format=text"
```

**响应**

```text
Department:IT;Environment:Test;Role:WebRole
```

> [!NOTE]
> 标记以分号分隔。 如果编写的分析器以编程方式提取标记，则标记名称和值不应包含分号，这样分析器才能正常工作。

### <a name="validating-that-the-vm-is-running-in-azure"></a>验证 VM 是否在 Azure 中运行

市场供应商希望确保其软件仅许可在 Azure 中运行。 如果有人将 VHD 复制到本地，则应当有能力检测到这一情况。 通过调用实例元数据服务，市场供应商可以获得签名数据，以保证响应仅来自 Azure。

> [!NOTE]
> 需要安装 jq。

**请求**

```bash
# Get the signature
curl  --silent -H Metadata:True http://169.254.169.254/metadata/attested/document?api-version=2018-10-01 | jq -r '.["signature"]' > signature
# Decode the signature
base64 -d signature > decodedsignature
#Get PKCS7 format
openssl pkcs7 -in decodedsignature -inform DER -out sign.pk7
# Get Public key out of pkc7
openssl pkcs7 -in decodedsignature -inform DER  -print_certs -out signer.pem
#Get the intermediate certificate
wget -q -O intermediate.cer "$(openssl x509 -in signer.pem -text -noout | grep " CA Issuers -" | awk -FURI: '{print $2}')"
openssl x509 -inform der -in intermediate.cer -out intermediate.pem
#Verify the contents
openssl smime -verify -in sign.pk7 -inform pem -noverify
```

**响应**

```json
Verification successful
{"nonce":"20181128-001617",
  "plan":
    {
     "name":"",
     "product":"",
     "publisher":""
    },
"timeStamp":
  {
    "createdOn":"11/28/18 00:16:17 -0000",
    "expiresOn":"11/28/18 06:16:17 -0000"
  },
"vmId":"d3e0e374-fda6-4649-bbc9-7f20dc379f34"
}
```

数据 | 说明
-----|------------
nonce | 用户提供了带有请求的可选字符串。 如果请求中未提供 nonce，则返回当前 UTC 时间戳
计划 | 在 Azure 市场映像中 VM 的[计划](https://docs.microsoft.com/rest/api/compute/virtualmachines/createorupdate#plan)，包含名称、产品和发布者
timestamp/createdOn | 创建第一个签名文档的时间戳
timestamp/expiresOn | 签名文档到期的时间戳
vmId |  VM 的[唯一标识符](https://azure.microsoft.com/blog/accessing-and-using-azure-vm-unique-id/)

#### <a name="verifying-the-signature"></a>验证签名

获得上面的签名后，可以验证签名是否来自世纪互联。 还可以验证中间证书和证书链。

> [!NOTE]
> 公有云和主权云的证书将有所不同。

 Regions | 证书
---------|-----------------
[全球所有公开上市的 Azure 区域](https://www.azure.cn/support/service-dashboard/)     | metadata.azure.com
[Azure 美国政府云](https://azure.microsoft.com/overview/clouds/government/)              | metadata.azure.us
[Azure 中国](https://www.azure.cn/)                                                           | metadata.azure.cn
[Azure 德国](https://azure.microsoft.com/overview/clouds/germany/)                    | metadata.microsoftazure.de

```bash

# Verify the subject name for the main certificate
openssl x509 -noout -subject -in signer.pem
# Verify the issuer for the main certificate
openssl x509 -noout -issuer -in signer.pem
#Validate the subject name for intermediate certificate
openssl x509 -noout -subject -in intermediate.pem
# Verify the issuer for the intermediate certificate
openssl x509 -noout -issuer -in intermediate.pem
# Verify the certificate chain
openssl verify -verbose -CAfile /etc/ssl/certs/Baltimore_CyberTrust_Root.pem -untrusted intermediate.pem signer.pem
```

如果由于验证期间出现网络限制，导致中间证书无法下载，可以固定中间证书。 但是，Azure 会根据标准的 PKI 做法滚动更新证书。 发生滚动更新时，需要更新固定的证书。 每当规划某项更改来更新中间证书，就会更新 Azure 博客并向 Azure 客户发出通知。 [此处](https://www.microsoft.com/pki/mscorp/cps/default.htm)可找到中间证书。 每个区域的中间证书可能并不相同。

### <a name="failover-clustering-in-windows-server"></a>Windows Server 中的故障转移群集

在某些情况下，在使用故障转移群集查询实例元数据服务时，必须向路由表添加路由。

1. 使用管理员特权打开命令提示符。

2. 运行以下命令，并记下 IPv4 路由表中网络目标 (`0.0.0.0`) 接口的地址。

    ```bat
    route print
    ```

    > [!NOTE] 
    > 为简单起见，启用了故障转移群集的 Windows Server VM 的以下示例输出仅包含 IPv4 路由表。

    ```bat
    IPv4 Route Table
    ===========================================================================
    Active Routes:
    Network Destination        Netmask          Gateway       Interface  Metric
              0.0.0.0          0.0.0.0         10.0.1.1        10.0.1.10    266
             10.0.1.0  255.255.255.192         On-link         10.0.1.10    266
            10.0.1.10  255.255.255.255         On-link         10.0.1.10    266
            10.0.1.15  255.255.255.255         On-link         10.0.1.10    266
            10.0.1.63  255.255.255.255         On-link         10.0.1.10    266
            127.0.0.0        255.0.0.0         On-link         127.0.0.1    331
            127.0.0.1  255.255.255.255         On-link         127.0.0.1    331
      127.255.255.255  255.255.255.255         On-link         127.0.0.1    331
          169.254.0.0      255.255.0.0         On-link     169.254.1.156    271
        169.254.1.156  255.255.255.255         On-link     169.254.1.156    271
      169.254.255.255  255.255.255.255         On-link     169.254.1.156    271
            224.0.0.0        240.0.0.0         On-link         127.0.0.1    331
            224.0.0.0        240.0.0.0         On-link     169.254.1.156    271
            224.0.0.0        240.0.0.0         On-link         10.0.1.10    266
      255.255.255.255  255.255.255.255         On-link         127.0.0.1    331
      255.255.255.255  255.255.255.255         On-link     169.254.1.156    271
      255.255.255.255  255.255.255.255         On-link         10.0.1.10    266
    ```

3. 运行以下命令并使用网络目标 (`0.0.0.0`) 接口的地址，在此示例中为 `10.0.1.10`。

```bat
route add 169.254.169.254/32 10.0.1.10 metric 1 -p
```

<!--Not Available on ### Custom Data-->
<!--MOONCAKE: Custom Data is valid on Api Version 2019-02-01-->

#### <a name="retrieving-custom-data-in-virtual-machine"></a>在虚拟机中检索自定义数据
实例元数据服务向 VM 提供 base64 编码形式的自定义数据。 以下示例解码 base64 编码的字符串。

> [!NOTE]
> 此示例中的自定义数据解释为 ASCII 字符串，其内容是“My custom data.”。

**请求**

```bash
curl -H "Metadata:true" "http://169.254.169.254/metadata/instance/compute/customData?api-version=2019-02-01&&format=text" | base64 --decode
```

**响应**

```text
My custom data.
```

### <a name="examples-of-calling-metadata-service-using-different-languages-inside-the-vm"></a>使用 VM 中的不同语言调用元数据服务的示例 

语言 | 示例
---------|----------------
Ruby     | https://github.com/Microsoft/azureimds/blob/master/IMDSSample.rb
Go  | https://github.com/Microsoft/azureimds/blob/master/imdssample.go
Python   | https://github.com/Microsoft/azureimds/blob/master/IMDSSample.py
C++      | https://github.com/Microsoft/azureimds/blob/master/IMDSSample-windows.cpp
C# | https://github.com/Microsoft/azureimds/blob/master/IMDSSample.cs
Javascript | https://github.com/Microsoft/azureimds/blob/master/IMDSSample.js
PowerShell | https://github.com/Microsoft/azureimds/blob/master/IMDSSample.ps1
Bash       | https://github.com/Microsoft/azureimds/blob/master/IMDSSample.sh
Perl       | https://github.com/Microsoft/azureimds/blob/master/IMDSSample.pl
Java       | https://github.com/Microsoft/azureimds/blob/master/imdssample.java
Visual Basic | https://github.com/Microsoft/azureimds/blob/master/IMDSSample.vb
Puppet | https://github.com/keirans/azuremetadata

## <a name="faq"></a>常见问题

1. 我收到错误 `400 Bad Request, Required metadata header not specified`。 这是什么意思呢？
    * 实例元数据服务需要在请求中传递标头 `Metadata: true`。 将该标头传入 REST 调用将允许访问实例元数据服务。
2. 为什么我无法获取我的 VM 的计算信息？
    * 当前实例元数据服务仅支持 Azure Resource Manager 创建的实例。 将来可能会添加对云服务 VM 的支持。
3. 我刚才通过 Azure Resource Manager 创建了我的虚拟机。 为什么我无法看到计算元数据信息？
    * 对于 2016 年 9 月之后创建的所有 VM，请添加[标记](../../azure-resource-manager/resource-group-using-tags.md)以开始查看计算元数据。 对于早期 VM（在 2016 年 9 月之前创建），请在 VM 中添加/删除扩展或数据磁盘以刷新元数据。
4. 我看不到为新版本填充的任何数据
    * 对于 2016 年 9 月之后创建的所有 VM，请添加[标记](../../azure-resource-manager/resource-group-using-tags.md)以开始查看计算元数据。 对于早期 VM（在 2016 年 9 月之前创建），请在 VM 中添加/删除扩展或数据磁盘以刷新元数据。
5. 我为什么会收到错误 `500 Internal Server Error`？
    * 请根据指数后退系统重试请求。 如果问题持续出现，请联系 Azure 支持部门。
6. 在何处共享其他问题/评论？
    * 在 https://support.azure.cn/en-us/support/contact 上发送评论。
7. 这是否适用于虚拟机规模集实例？
    * 是的，元数据服务可用于规模集实例。
8. 如何获取服务支持？
    * 若要获取该服务的支持，请针对长时间重试后仍无法获取元数据响应的 VM，在 Azure 门户中创建相关支持问题。
9. 调用服务时请求超时？
    * 必须从分配给 VM 的网卡的主 IP 地址进行元数据调用，此外，在已更改路由的情况下，网卡外必须存在地址 169.254.0.0/16 的路由。
10. 我更新了虚拟机规模集中的标记，但与 VM 不同，它们未显示在实例中，这是怎么回事？
    * 目前，对于规模集，仅在重启/重置映像/或对实例的磁盘更改时，向 VM 显示标记。

    <!-- Not Available on ![Instance Metadata Support](./media/instance-metadata-service/InstanceMetadata-support.png)-->

## <a name="next-steps"></a>后续步骤

- 详细了解[计划事件](scheduled-events.md)

<!--Update_Description: update meta properties, wording update, update link  -->