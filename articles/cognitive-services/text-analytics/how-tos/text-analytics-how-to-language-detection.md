---
title: 通过文本分析 REST API 检测语言 | Microsoft Docs
description: 如何通过 Azure 认知服务使用文本分析 REST API 检测语言。
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: sample
origin.date: 02/26/2019
ms.date: 07/12/2019
ms.author: v-junlch
ms.openlocfilehash: dc30bf113521a6b3deeea28611d882287b9c1415
ms.sourcegitcommit: 8f49da0084910bc97e4590fc1a8fe48dd4028e34
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/12/2019
ms.locfileid: "67844949"
---
# <a name="example-how-to-detect-language-with-text-analytics"></a>示例：如何通过文本分析检测语言

API 的[语言检测](https://dev.cognitive.azure.cn/docs/services/TextAnalytics-v2-1/operations/56f30ceeeda5650db055a3c7)功能评估每个文档的文本输入，并返回带有指示分析强度的分数的语言标识符。

此功能对于用于收集语言未知的任意文本的内容存储非常有用。 可以解析此分析的结果，确定输入文档中使用的语言。 响应还返回一个分数，反映模型的置信度（介于 0 到 1 之间的值）。

我们不会发布此功能的确切语言列表，但它可以检测各种语言、变体、方言和一些区域/文化语言。 

如果内容是用较少使用的语言表示的，则可以尝试“语言检测”来查看它是否返回代码。 无法检测到的语言的响应为 `unknown`。

> [!TIP]
> 文本分析还提供一个基于 Linux 的 Docker 容器映像，用于检测语言，因此可以在靠近数据的位置[安装并运行文本分析容器](text-analytics-how-to-install-containers.md)。

## <a name="preparation"></a>准备工作

必须拥有以下格式的 JSON 文档：ID、文本

每个文档的大小必须少于 5,120 个字符，每个集合最多可包含 1,000 个项目 (ID)。 集合在请求正文中提交。 下面是可能提交用于语言检测的内容示例。

   ```
    {
        "documents": [
            {
                "id": "1",
                "text": "This document is in English."
            },
            {
                "id": "2",
                "text": "Este documento está en inglés."
            },
            {
                "id": "3",
                "text": "Ce document est en anglais."
            },
            {
                "id": "4",
                "text": "本文件为英文"
            },                
            {
                "id": "5",
                "text": "Этот документ на английском языке."
            }
        ]
    }
```

## <a name="step-1-structure-the-request"></a>步骤 1：构造请求

有关请求定义的详细信息，请参阅[如何调用文本分析 API](text-analytics-how-to-call-api.md)。 为方便起见，特重申以下几点：

+ 创建 POST 请求  。 查看此请求的 API 文档：[语言检测 API](https://dev.cognitive.azure.cn/docs/services/TextAnalytics-v2-1/operations/56f30ceeeda5650db055a3c7)

+ 使用 Azure 上的文本分析资源或实例化的[文本分析容器](text-analytics-how-to-install-containers.md)设置 HTTP 终结点，以便检测语言。 它必须包含 `/languages` 资源：`https://chinaeast2.api.cognitive.azure.cn/text/analytics/v2.1/languages`

+ 设置请求头以包含文本分析操作的访问密钥。 有关详细信息，请参阅[如何查找终结点和访问密钥](text-analytics-how-to-access-key.md)。

+ 在请求正文中，提供为此分析准备的 JSON 文档集合

> [!Tip]
> 使用 [Postman](text-analytics-how-to-call-api.md) 或打开[文档](https://dev.cognitive.azure.cn/docs/services/TextAnalytics-v2-1/operations/56f30ceeeda5650db055a3c7)中的“API 测试控制台”来构造请求并将其 POST 到该服务  。

## <a name="step-2-post-the-request"></a>步骤 2：发布请求

在收到请求时执行分析。 有关每分钟和每秒可以发送的请求的大小和数量的信息，请参阅概述中的[数据限制](../overview.md#data-limits)部分。

记住，该服务是无状态服务。 帐户中未存储任何数据。 结果会立即在响应中返回。


## <a name="step-3-view-results"></a>步骤 3：查看结果

所有 POST 请求都将返回 JSON 格式的响应，其中包含 ID 和检测到的属性。

系统会立即返回输出。 可将结果流式传输到接受 JSON 的应用程序，或者将输出保存到本地系统上的文件中，然后将其导入到允许对数据进行排序、搜索和操作的应用程序。

示例请求的结果应类似于以下 JSON。 请注意，它是一个包含多个项目的文档。 输出采用英文。 语言标识符包括友好名称和 [ISO 639-1](https://www.iso.org/standard/22109.html) 格式的语言代码。

正分 1.0 表示分析可能达到的最高可信度。



```
{
    "documents": [
        {
            "id": "1",
            "detectedLanguages": [
                {
                    "name": "English",
                    "iso6391Name": "en",
                    "score": 1
                }
            ]
        },
        {
            "id": "2",
            "detectedLanguages": [
                {
                    "name": "Spanish",
                    "iso6391Name": "es",
                    "score": 1
                }
            ]
        },
        {
            "id": "3",
            "detectedLanguages": [
                {
                    "name": "French",
                    "iso6391Name": "fr",
                    "score": 1
                }
            ]
        },
        {
            "id": "4",
            "detectedLanguages": [
                {
                    "name": "Chinese_Simplified",
                    "iso6391Name": "zh_chs",
                    "score": 1
                }
            ]
        },
        {
            "id": "5",
            "detectedLanguages": [
                {
                    "name": "Russian",
                    "iso6391Name": "ru",
                    "score": 1
                }
            ]
        }
    ],
```

### <a name="ambiguous-content"></a>不明确的内容

如果分析器无法分析输入（例如，假设提交的文本块仅包含阿拉伯数字），将返回 `(Unknown)`。

```
    {
      "id": "5",
      "detectedLanguages": [
        {
          "name": "(Unknown)",
          "iso6391Name": "(Unknown)",
          "score": "NaN"
        }
      ]
```
### <a name="mixed-language-content"></a>混合的语言内容

同一文档中的混合语言内容将返回内容中代表性最强但正评级较低的语言，这反映该评估的边界强度。 在以下示例中，输入是英语、西班牙语和法语的混合。 分析器对每个段中的字符进行计数，确定主要语言。

**输入**

```
{
  "documents": [
    {
      "id": "1",
      "text": "Hello, I would like to take a class at your University. ¿Se ofrecen clases en español? Es mi primera lengua y más fácil para escribir. Que diriez-vous des cours en français?"
    }
  ]
}
```

**输出**

生成的输出包含主要语言，分数低于 1.0，表示可信度较低。

```
{
  "documents": [
    {
      "id": "1",
      "detectedLanguages": [
        {
          "name": "Spanish",
          "iso6391Name": "es",
          "score": 0.9375
        }
      ]
    }
  ],
  "errors": []
}
```

## <a name="summary"></a>摘要

本文介绍了使用认知服务中的文本分析进行语言检测的概念和工作流。 以下是对前面解释和演示的要点的快速提醒：

+ [语言检测](https://dev.cognitive.azure.cn/docs/services/TextAnalytics-v2-1/operations/56f30ceeeda5650db055a3c7)可用于多种语言、变体、方言和某些区域/文化语言。
+ 请求正文中的 JSON 文档包括 ID 和文本。
+ POST 请求的目标是 `/languages` 终结点，方法是使用对订阅有效的个性化[访问密钥和终结点](text-analytics-how-to-access-key.md)。
+ 响应输出包含每个文档 ID 的语言标识符，可以流式传输到接受 JSON 的任何应用，包括 Excel 和 Power BI（仅举几例）。

## <a name="see-also"></a>另请参阅 

 [文本分析概述](../overview.md)  
 [常见问题解答 (FAQ)](../text-analytics-resource-faq.md)</br>
 [文本分析产品页](https://www.azure.cn/zh-cn/home/features/cognitive-services/text-analytics/) 

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [分析情绪](text-analytics-how-to-sentiment-analysis.md)

<!-- Update_Description: wording update -->
