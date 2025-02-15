---
title: 使用资源提供程序 REST API 创建 Azure IoT 中心 | Azure
description: 如何使用资源提供程序 REST API 创建 IoT 中心。
services: iot-hub
documentationcenter: .net
author: dominicbetts
manager: timlt
editor: ''
ms.assetid: 52814ee5-bc10-4abe-9eb2-f8973096c2d8
ms.service: iot-hub
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
origin.date: 08/08/2017
ms.author: v-yiso
ms.date: 07/15/2019
ms.openlocfilehash: 521d8a767a94cd96ee711336759ca15c37ada487
ms.sourcegitcommit: 5191c30e72cbbfc65a27af7b6251f7e076ba9c88
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/05/2019
ms.locfileid: "67569713"
---
# <a name="create-an-iot-hub-using-the-resource-provider-rest-api-net"></a>使用资源提供程序 REST API 创建 IoT 中心 (.NET)
[!INCLUDE [iot-hub-resource-manager-selector](../../includes/iot-hub-resource-manager-selector.md)]

可以通过编程方式使用 [IoT 中心资源提供程序 REST API](https://docs.microsoft.com/rest/api/iothub/iothubresource) 创建和管理 Azure IoT 中心。 本教程介绍如何使用 IoT 中心资源提供程序 REST API 通过 C# 程序创建 IoT 中心。

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

要完成本教程，需要以下各项：

* Visual Studio。

* 有效的 Azure 帐户。 <br/>如果没有帐户，只需花费几分钟就能创建一个[帐户][lnk-free-trial]。
* [Azure PowerShell 1.0][lnk-powershell-install] 或更高版本。

[!INCLUDE [iot-hub-prepare-resource-manager](../../includes/iot-hub-prepare-resource-manager.md)]

## <a name="prepare-your-visual-studio-project"></a>准备 Visual Studio 项目
1. 在 Visual Studio 中，使用“控制台应用(.NET Framework)”  项目模板创建 Visual C# Windows 经典桌面项目。 将该项目命名为 **CreateIoTHubREST**。
2. 在解决方案资源管理器中右键单击项目，然后单击“**管理 NuGet 包**”。
3. 在 NuGet 包管理器中，选中“包括预发行版”，并在“浏览”页上搜索 **Microsoft.Azure.Management.ResourceManager**   。 选择该包，单击“安装”  ，在“审阅更改”  中单击“确定”  ，并单击“我接受”  以接受许可证。
4. 在 NuGet 包管理器中，搜索 **Microsoft.IdentityModel.Clients.ActiveDirectory**。  单击“**安装**”，在“**审阅更改**”中单击“**确定**”，并单击“**我接受**”以接受许可证。
5. 在 Program.cs 中，将现有 **using** 语句替换为以下代码：

    ```csharp
    using System;
    using System.Net.Http;
    using System.Net.Http.Headers;
    using System.Text;
    using Microsoft.Azure.Management.ResourceManager;
    using Microsoft.Azure.Management.ResourceManager.Models;
    using Microsoft.IdentityModel.Clients.ActiveDirectory;
    using Newtonsoft.Json;
    using Microsoft.Rest;
    using System.Linq;
    using System.Threading;
    ```

6. 在 Program.cs 中，将占位符值替换为以下静态变量。 在本教程前面的介绍中，已记下 **ApplicationId**、**SubscriptionId**、**TenantId** 和 **Password**。 资源组名称  是创建 IoT 中心时要使用的资源组名称。 可以使用现有的资源组或新资源组。 “IoT 中心名称”  是你创建的 IoT 中心的名称，例如“MyIoTHub”  。 IoT 中心的名称必须全局唯一。 **部署名称**是部署的名称，例如 **Deployment_01**。

    ```csharp
    static string applicationId = "{Your ApplicationId}";
    static string subscriptionId = "{Your SubscriptionId}";
    static string tenantId = "{Your TenantId}";
    static string password = "{Your application Password}";

    static string rgName = "{Resource group name}";
    static string iotHubName = "{IoT Hub name including your initials}";
    ```
   [!INCLUDE [iot-hub-pii-note-naming-hub](../../includes/iot-hub-pii-note-naming-hub.md)]

[!INCLUDE [iot-hub-get-access-token](../../includes/iot-hub-get-access-token.md)]

## <a name="use-the-resource-provider-rest-api-to-create-an-iot-hub"></a>使用资源提供程序 REST API 创建 IoT 中心
在资源组中使用 [IoT 中心资源提供程序 REST API][lnk-rest-api] 创建 IoT 中心。 还可使用资源提供程序 REST API 更改现有的 IoT 中心。

1. 将以下方法添加到 Program.cs：

    ```csharp
    static void CreateIoTHub(string token)
    {

    }
    ```
2. 将以下代码添加到 **CreateIoTHub** 方法。 该代码创建一个 **HttpClient** 对象，在标头中使用身份验证令牌：

    ```csharp
    HttpClient client = new HttpClient();
    client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", token);
    ```
3. 将以下代码添加到 **CreateIoTHub** 方法。 此代码描述要创建的 IoT 中心，并生成 JSON 表示形式。 有关支持 IoT 中心的位置的最新列表，请参阅 [Azure 状态][lnk-status]：

    ```csharp
    var description = new
    {
      name = iotHubName,
      location = "China East",
      sku = new
      {
        name = "S1",
        tier = "Standard",
        capacity = 1
      }
    };

    var json = JsonConvert.SerializeObject(description, Formatting.Indented);
    ```

4. 将以下代码添加到 **CreateIoTHub** 方法。 使用此代码将 REST 请求提交到 Azure。 然后该代码会检查响应，并检索可用于监视部署任务状态的 URL：

    ```csharp
    var content = new StringContent(JsonConvert.SerializeObject(description), Encoding.UTF8, "application/json");
    var requestUri = string.Format("https://management.chinacloudapi.cn/subscriptions/{0}/resourcegroups/{1}/providers/Microsoft.devices/IotHubs/{2}?api-version=2016-02-03", subscriptionId, rgName, iotHubName);
    var result = client.PutAsync(requestUri, content).Result;

    if (!result.IsSuccessStatusCode)
    {
      Console.WriteLine("Failed {0}", result.Content.ReadAsStringAsync().Result);
      return;
    }

    var asyncStatusUri = result.Headers.GetValues("Azure-AsyncOperation").First();
    ```
5. 将以下代码添加到 **CreateIoTHub** 方法的末尾。 该代码使用在上一步检索的 **asyncStatusUri** 地址等待部署完成：

    ```csharp
    string body;
    do
    {
      Thread.Sleep(10000);
      HttpResponseMessage deploymentstatus = client.GetAsync(asyncStatusUri).Result;
      body = deploymentstatus.Content.ReadAsStringAsync().Result;
    } while (body == "{\"status\":\"Running\"}");
    ```
6. 将以下代码添加到 **CreateIoTHub** 方法的末尾。 此代码检索创建的 IoT 中心的键，并将其打印到控制台：

    ```csharp
    var listKeysUri = string.Format("https://management.azure.cn/subscriptions/{0}/resourceGroups/{1}/providers/Microsoft.Devices/IotHubs/{2}/IoTHubKeys/listkeys?api-version=2016-02-03", subscriptionId, rgName, iotHubName);
    var keysresults = client.PostAsync(listKeysUri, null).Result;

    Console.WriteLine("Keys: {0}", keysresults.Content.ReadAsStringAsync().Result);
    ```

## <a name="complete-and-run-the-application"></a>完成并运行应用程序
现在，可以调用 **CreateIoTHub** 方法来完成应用程序，并生成并运行该应用程序。

1. 将以下代码添加到 **Main** 方法末尾：

    ```csharp
    CreateIoTHub(token.AccessToken);
    Console.ReadLine();
    ```
2. 单击“**生成**”，并单击“**生成解决方案**”。 更正所有错误。
3. 单击“**调试**”，并单击“**开始调试**”以运行应用程序。 运行部署可能需要几分钟时间。

4. 若要验证应用程序是否添加了新的 IoT 中心，请访问 [Azure 门户][lnk-azure-portal]并查看资源列表。 另外，也可以使用 **Get-AzResource** PowerShell cmdlet。

> [!NOTE]
> 本示例应用程序会添加用于对你计费的 S1 标准 IoT 中心。 在完成任务后，可以通过 [Azure 门户][lnk-azure-portal]或者使用 **Remove-AzResource** PowerShell cmdlet 删除该 IoT 中心。
> 
> 

## <a name="next-steps"></a>后续步骤
现已使用资源提供程序 REST API 部署了 IoT 中心，接下来可更进一步探索：

* 阅读了解 [IoT 中心资源提供程序 REST API][lnk-rest-api] 的相关功能。
* 若要详细了解 Azure 资源管理器功能，请阅读 [Azure 资源管理器概述][lnk-azure-rm-overview]。

若要详细了解如何开发 IoT 中心，请参阅以下文章：

- [C SDK 简介][lnk-c-sdk]
- [Azure IoT SDK][lnk-sdks]

若要进一步探索 IoT 中心的功能，请参阅：

* [使用 Azure IoT Edge 将 AI 部署到边缘设备][lnk-iotedge]

<!-- Links -->
[lnk-free-trial]: https://www.azure.cn/pricing/1rmb-trial/
[lnk-azure-portal]: https://portal.azure.cn/
[lnk-status]: https://www.azure.cn/support/service-dashboard/
[lnk-powershell-install]: ../powershell-install-configure.md
[lnk-rest-api]: https://docs.microsoft.com/rest/api/iothub/iothubresource
[lnk-azure-rm-overview]: ../azure-resource-manager/resource-group-overview.md

[lnk-c-sdk]: ./iot-hub-device-sdk-c-intro.md
[lnk-sdks]: ./iot-hub-devguide-sdks.md

[lnk-iotedge]: ./iot-hub-linux-iot-edge-simulated-device.md
