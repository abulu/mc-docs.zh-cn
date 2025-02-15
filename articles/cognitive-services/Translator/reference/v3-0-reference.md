---
title: 文本翻译 API V3.0 参考
titlesuffix: Azure Cognitive Services
description: 文本翻译 API v3.0 参考文档。
services: cognitive-services
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: reference
origin.date: 03/29/2018
ms.date: 07/12/2019
ms.author: v-junlch
ms.openlocfilehash: e698413f97efbcd7887287a57f8ac5d8d4aa8d1f
ms.sourcegitcommit: 8f49da0084910bc97e4590fc1a8fe48dd4028e34
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/12/2019
ms.locfileid: "67844634"
---
# <a name="translator-text-api-v30"></a>文本翻译 API v3.0

## <a name="whats-new"></a>新增功能

文本翻译 API 版本 3 提供了基于 JSON 的新型 Web API。 它通过将现有功能合并到更少的操作中来提高可用性和性能，并提供新功能。

 * 音译可将一种语言的文本从一个脚本转换为另一个脚本。
 * 在一个请求中翻译成多种语言。
 * 在一个请求中进行语言检测、翻译和音译。
 * 字典可查找术语的替代翻译，可查找反向翻译以及显示上下文中使用的术语的示例。
 * 更详细的语言检测结果。

## <a name="base-urls"></a>基 URL

Microsoft Translator 位于多个数据中心位置之外。 目前它们位于 6 个 [Azure 地理区域](https://azure.microsoft.com/global-infrastructure/regions)：

* **中国：** 中国北部和中国东部 2 

在大多数情况下，对 Microsoft 文本翻译 API 的请求由距离请求的来源位置最近的数据中心处理。 如果数据中心出现故障，请求可能会路由到 Azure 地理区域之外。

若要强制由特定 Azure 地理区域处理请求，请将 API 请求中的全球终结点更改为所需的区域终结点：

|说明|Azure 地理区域|基 URL|
|:--|:--|:--|
|Azure|中国北部| chinanorth.api.cognitive.azure.cn|
|Azure|中国东部 2| chinaeast2.api.cognitive.azure.cn|
|Azure|中国| api.translator.azure.cn|

## <a name="authentication"></a>身份验证

订阅文本翻译 API 并使用订阅密钥（Azure 门户中提供）进行身份验证。 

有三个标头可用于对你的订阅进行身份验证。 下表介绍了每个标头的使用方式：

|标头|说明|
|:----|:----|
|Ocp-Apim-Subscription-Key|如果要传递密钥，请与认知服务订阅一起使用  。<br/>该值是文本翻译 API 订阅的 Azure 密钥。|
|授权|如果要传递身份验证令牌，请与认知服务订阅一起使用  。<br/>该值是持有者令牌：`Bearer <token>`。|
|Ocp-Apim-Subscription-Region|在调用 api.translator.azure.cn endpoint 并直接传递机密密钥时所需的请求标头。 指定你的订阅所在的区域（例如 ` Chinanorth` 或 `Chinaeast2`）。|

###  <a name="secret-key"></a>密钥
第一个选项是使用 `Ocp-Apim-Subscription-Key` 标头进行身份验证。 只需将 `Ocp-Apim-Subscription-Key: <YOUR_SECRET_KEY>` 标头添加到你的请求。

### <a name="authorization-token"></a>授权令牌
或者，可以交换访问令牌的密钥。 此令牌作为 `Authorization` 标头包含在每个请求中。 若要获取授权令牌，请向以下 URL 发出 `POST` 请求：

| 环境     | 身份验证服务 URL                                |
|-----------------|-----------------------------------------------------------|
| Azure 中国     | `https://<your region>.api.cognitive.azure.cn/sts/v1.0/issueToken` |

以下是根据给定密钥获取令牌的示例请求：

```
// Pass secret key using header
curl --header 'Ocp-Apim-Subscription-Key: <your-key>' --data "" 'https://<your region>.api.cognitive.azure.cn/sts/v1.0/issueToken'

// Pass secret key using query string parameter
curl --data "" 'https://<your region>.api.cognitive.azure.cn/sts/v1.0/issueToken?Subscription-Key=<your-key>'
```

成功的请求会在响应正文中将编码的访问令牌作为纯文本返回。 有效的令牌在授权中作为持有者令牌传递给翻译服务。

```
Authorization: Bearer <Base64-access_token>
```

身份验证令牌的有效期为 10 分钟。 在对翻译 API 进行多次调用时，应重新使用该令牌。 但是，如果程序在很长一段时间内向翻译 API 发出请求，则程序必须定期（例如每 8 分钟）请求一个新的访问令牌。


## <a name="errors"></a>错误

标准错误响应是具有名为 `error` 的名称/值对的 JSON 对象。 该值也是具有以下属性的 JSON 对象：

  * `code`：服务器定义的错误代码。

  * `message`：一个提供人类可读的错误表示形式的字符串。

例如，拥有试用版订阅的客户会在免费配额用尽后收到以下错误：

```
{
  "error": {
    "code":403001,
    "message":"The operation is not allowed because the subscription has exceeded its free quota."
    }
}
```
错误代码是一个 6 位数字，包括 3 位数的 HTTP 状态代码，后接用于进一步将错误分类的 3 位数。 常见错误代码包括：

| 代码 | 说明 |
|:----|:-----|
| 400000| 某个请求输入无效。|
| 400001| “scope”参数无效。|
| 400002| “category”参数无效。|
| 400003| 语言说明符缺失或无效。|
| 400004| 目标脚本说明符（“To script”）缺失或无效。|
| 400005| 输入文本缺失或无效。|
| 400006| 语言和脚本的组合无效。|
| 400018| 源脚本说明符（“From script”）缺失或无效。|
| 400019| 指定的某个语言不受支持。|
| 400020| 输入文本数组中的某个元素无效。|
| 400021| API 版本参数缺失或无效。|
| 400023| 指定的某个语言对无效。|
| 400035| 源语言（“From”字段）无效。|
| 400036| 目标语言（“To”字段）缺失或无效。|
| 400042| 指定的某个选项（“Options”字段）无效。|
| 400043| 客户端跟踪 ID（ClientTraceId 字段或 X-ClientTranceId 标头）缺失或无效。|
| 400050| 输入文本过长。 查看[请求限制](../request-limits.md)。|
| 400064| “translation”参数缺失或无效。|
| 400070| 目标脚本（ToScript 参数）的数目与目标语言（To 参数）的数目不匹配。|
| 400071| TextType 的值无效。|
| 400072| 输入文本的数组包含过多的元素。|
| 400073| 脚本参数无效。|
| 400074| 请求正文是无效的 JSON。|
| 400075| 语言对和类别组合无效。|
| 400077| 超过了最大请求大小。 查看[请求限制](../request-limits.md)。|
| 400079| 请求用于在源语言与目标语言之间进行翻译的自定义系统不存在。|
| 400080| 语言或脚本不支持音译。|
| 401000| 由于凭据缺失或无效，请求未授权。|
| 401015| “提供的凭据适用于语音 API。 此请求需要文本 API 的凭据。 请使用文本翻译 API 的订阅。”|
| 403000| 不允许该操作。|
| 403001| 由于订阅已超过其免费配额，因此不允许该操作。|
| 405000| 请求的资源不支持该请求方法。|
| 408001| 正在准备所请求的翻译系统。 请在几分钟内重试。|
| 408002| 等待传入流时请求超时。 客户端没有在服务器准备等待的时间内生成请求。 客户端可以在以后的任何时间重复该请求，而不做任何修改。|
| 415000| Content-Type 标头缺失或无效。|
| 429000、429001、429002| 由于客户端已超出请求限制，服务器拒绝了请求。|
| 500000| 发生了意外错误。 如果该错误持续出现，请报告发生错误的日期/时间、响应标头 X-RequestId 中的请求标识符，以及请求标头 X-ClientTraceId 中的客户端标识符。|
| 503000| 服务暂时不可用。 请重试。 如果该错误持续出现，请报告发生错误的日期/时间、响应标头 X-RequestId 中的请求标识符，以及请求标头 X-ClientTraceId 中的客户端标识符。|


<!-- Update_Description: wording update -->