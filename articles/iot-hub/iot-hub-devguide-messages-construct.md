---
title: 了解 Azure IoT 中心消息格式 | Azure
description: 开发人员指南 - 描述 IoT 中心消息的格式和预期内容。
author: ash2017
manager: briz
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
origin.date: 08/13/2018
ms.author: v-yiso
ms.date: 07/15/2019
ms.openlocfilehash: ffe5eade51126b831ab453fd6004954ff2c11e84
ms.sourcegitcommit: 5191c30e72cbbfc65a27af7b6251f7e076ba9c88
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/05/2019
ms.locfileid: "67570528"
---
# <a name="create-and-read-iot-hub-messages"></a>创建和读取 IoT 中心消息

为了支持无缝的跨协议互操作性，IoT 中心为所有面向设备的协议定义了通用消息格式。 此消息格式用于[设备到云][lnk-d2c]and [cloud-to-device][lnk-c2d]消息。 

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-partial.md)]

IoT 中心使用流式消息传递模式实现设备到云的消息传递。 与[事件中心](/event-hubs/)*事件*和[服务总线](/service-bus-messaging/) *消息*相比，IoT 中心的设备到云消息更类似前者，类似之处在于有大量事件通过可供多个读取器读取的服务。

IoT 中心消息由以下部分组成：

* 一组预先确定的“系统属性”  如下所示。
* 一组 *应用程序属性*。 应用程序可以定义的字符串属性字典，而不需将消息正文反序列化即可进行访问。 IoT 中心永不修改这些属性。
* 不透明的二进制正文。

使用 HTTPS 协议发送设备到云的消息或发送云到设备的消息时，属性名称和值只能包含 ASCII 字母数字字符加上 ``{'!', '#', '$', '%, '&', ''', '*', '+', '-', '.', '^', '_', '`', '|', '~'}``。

使用 IoT 中心进行的设备到云的消息传送具有以下特征：

* 设备到云的消息可持久保留在 IoT 中心的默认 **messages/events** 终结点长达 7 天。

* 设备到云的消息最大可为 256 KB，而且可分成多个批以优化发送。 批最大可为 256 KB。

* IoT 中心不允许任意分区。 设备到云的消息根据其源于的 **deviceId**进行分区。

* 如[控制对 IoT 中心的访问](iot-hub-devguide-security.md)中所述，IoT 中心允许基于设备的身份验证和访问控制。

* 可以使用应用程序属性中的信息对消息进行戳记。 有关详细信息，请参阅[消息扩充](iot-hub-message-enrichments-overview.md)。

有关如何使用不同协议对已发送消息进行编码和解码的详细信息，请参阅 [Azure IoT SDK](iot-hub-devguide-sdks.md)。

下表列出 IoT 中心消息中的系统属性集。

| 属性 | 说明 | 用户可设置吗？ |
| --- | --- | --- |
| message-id |用户可设置的消息标识符，用于请求-答复模式。 格式：ASCII 7 位字母数字字符 + `{'-', ':', '.', '+', '%', '_', '#', '*', '?', '!', '(', ')', ',', '=', '@', ';', '$', '''}` 的区分大小写字符串（最长为 128 个字符）。 | 是 |
| sequence-number |IoT 中心分配给每条云到设备消息的编号（对每个设备队列是唯一的）。 | 对于 C2D 消息为否；对于其他情况则为是。 |
| to |[云到设备](iot-hub-devguide-c2d-guidance.md)消息中指定的目标。 | 对于 C2D 消息为否；对于其他情况则为是。 |
| absolute-expiry-time |消息过期的日期和时间。 | 是 |
| iothub-enqueuedtime |IoT 中心收到[设备到云](iot-hub-devguide-d2c-guidance.md)消息的日期和时间。 | 对于 D2C 消息为否；对于其他情况则为是。 |
| correlation-id |响应消息中的字符串属性，通常包含采用“请求-答复”模式的请求的 MessageId。 | 是 |
| user-id |用于指定消息的源的 ID。 如果消息是由 IoT 中心生成的，则设置为 `{iot hub name}`。 | 否 |
| iothub-ack |反馈消息生成器。 此属性在云到设备的消息中用于请求 IoT 中心因为设备使用消息而生成反馈消息。 可能的值：**none**（默认值）：不生成任何反馈消息；**positive**：如果消息已完成，则接收反馈消息；**negative**：如果消息未由设备完成就过期（或已达到最大传送计数），则收到反馈消息；**full**：positive 和 negative。 <!-- robinsh For more information, see [Message feedback][lnk-feedback].--> | 是 |
| iothub-connection-device-id |IoT 中心对设备到云的消息设置的 ID。 它包含发送消息的设备的 **deviceId** 。 | 对于 D2C 消息为否；对于其他情况则为是。 |
| iothub-connection-auth-generation-id |IoT 中心对设备到云的消息设置的 ID。 它包含发送消息的设备的 **generationId**（根据[设备标识属性](iot-hub-devguide-identity-registry.md#device-identity-properties)）。 | 对于 D2C 消息为否；对于其他情况则为是。 |
| iothub-connection-auth-method |由 IoT 中心对设备到云的消息设置的身份验证方法。 此属性包含用于验证发送消息的设备的身份验证方法的相关信息。 <!-- ROBINSH For more information, see [Device to cloud anti-spoofing][lnk-antispoofing].--> | 对于 D2C 消息为否；对于其他情况则为是。 |
| iothub-creation-time-utc | 在设备上创建消息的日期和时间。 设备必须显式设置此值。 | 是 |

## <a name="message-size"></a>消息大小

IoT 中心用于衡量消息大小的方法与协议无关，仅考虑实际有效负载。 以字节为单位的大小计算以下各项之和：

* 以字节为单位的正文大小。
* 以字节为单位的消息系统属性的所有值的大小。
* 以字节为单位的所有用户属性名称和值的大小。

属性名称和值限制为 ASCII 字符，因此，字符串的长度等于以字节为单位的大小。

## <a name="anti-spoofing-properties"></a>反欺骗属性

为了避免设备到云的消息中出现设备欺骗，IoT 中心使用以下属性在所有消息上加上戳记：

* **iothub-connection-device-id**
* **iothub-connection-auth-generation-id**
* **iothub-connection-auth-method**

根据[设备标识属性](iot-hub-devguide-identity-registry.md#device-identity-properties)，前两个属性包含源设备的 deviceId 和 generationId   。

**iothub-connection-auth-method** 属性包含具有以下属性的 JSON 序列化对象：

```json
{
  "scope": "{ hub | device }",
  "type": "{ symkey | sas | x509 }",
  "issuer": "iothub"
}
```

## <a name="next-steps"></a>后续步骤

有关 IoT 中心消息大小限制的详细信息，请参阅 [IoT 中心配额和限制][lnk-quotas]。

若要了解如何用不同编程语言创建并读取 IoT 中心消息，请参阅[快速入门][lnk-get-started]。

[lnk-messaging]: ./iot-hub-devguide-messaging.md
[lnk-quotas]: ./iot-hub-devguide-quotas-throttling.md
[lnk-get-started]: quickstart-send-telemetry-node.md
[lnk-sdks]: ./iot-hub-devguide-sdks.md
[lnk-c2d]: ./iot-hub-devguide-messages-c2d.md
[lnk-d2c]: ./iot-hub-devguide-messages-d2c.md
[lnk-feedback]: ./iot-hub-devguide-messages-c2d.md#message-feedback
[lnk-device-properties]: ./iot-hub-devguide-identity-registry.md#device-identity-properties
[lnk-antispoofing]: ./iot-hub-devguide-messages-d2c.md#anti-spoofing-properties