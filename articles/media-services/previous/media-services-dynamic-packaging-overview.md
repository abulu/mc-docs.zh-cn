---
title: Azure 媒体服务动态打包概述 | Microsoft 文档
description: 本主题概述动态打包。
author: WenJason
manager: digimobile
editor: ''
services: media-services
documentationcenter: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
origin.date: 03/21/2019
ms.date: 04/01/2019
ms.author: v-jay
ms.openlocfilehash: 8e0864415b8e1628b05154d4b9b25b36ef1eb3b7
ms.sourcegitcommit: 5fc46672ae90b6598130069f10efeeb634e9a5af
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/19/2019
ms.locfileid: "67236632"
---
# <a name="dynamic-packaging"></a>动态打包

Azure 媒体服务可用于向多种客户端技术（例如，iOS、XBOX、Silverlight、Windows 8）传送多种媒体源文件格式、媒体流格式和内容保护格式。 这些客户端可识别不同的协议，例如，iOS 需要 HTTP Live Streaming (HLS) V4 格式，Silverlight 和 Xbox 需要平滑流式处理。 如果你有一组自适应比特率（多比特率）MP4（ISO 基媒体 14496-12）文件或平滑流式处理文件要提供给了解 MPEG DASH、HLS 或平滑流式处理的客户端，则应利用媒体服务动态打包。

使用动态打包，只需要创建一个包含一组自适应比特率 MP4 文件或自适应比特率平滑流文件的资产。 然后，点播流服务器会确保用户以选定的协议按清单或分段请求中的指定格式接收流。 因此，用户只需以单一存储格式存储文件并为其付费，媒体服务服务就会基于客户端的请求构建并提供相应响应。

下图显示传统编码和静态打包工作流。

![静态编码](./media/media-services-dynamic-packaging-overview/media-services-static-packaging.png)

下图显示了动态打包工作流。

![动态编码](./media/media-services-dynamic-packaging-overview/media-services-dynamic-packaging.png)

## <a name="common-scenario"></a>常见方案

1. 上传一个输入文件（称为夹层文件）。 例如，H.264、MP4 或 WMV（有关受支持格式的列表，请参阅[Media Encoder Standard 支持的格式](media-services-media-encoder-standard-formats.md)）。
2. 将夹层文件编码为 H.264 MP4 自适应比特率集。
3. 通过创建点播定位符来发布包含自适应比特率 MP4 集的资产。
4. 生成用于访问和流式传输内容的流 URL。

## <a name="preparing-assets-for-dynamic-streaming"></a>准备用于动态流式传输的资产

若要准备用于动态流式传输的资产，可以使用以下选项：

- [上传主文件](media-services-dotnet-upload-files.md)。
- [使用 Media Encoder Standard 编码器生成 H.264 MP4 自适应比特率集](media-services-dotnet-encode-with-media-encoder-standard.md)。
- [流式传输内容](media-services-deliver-content-overview.md)。

## <a name="audio-codecs-supported-by-dynamic-packaging"></a>动态打包支持的音频编解码器

动态打包支持 MP4 文件，其中包含使用 [AAC](https://en.wikipedia.org/wiki/Advanced_Audio_Coding)（AAC-LC、HE-AAC v1、HE-AAC v2）、[Dolby Digital Plus](https://en.wikipedia.org/wiki/Dolby_Digital_Plus)（增强版 AC-3 或 E-AC3）、Dolby Atmos 或 [DTS](https://en.wikipedia.org/wiki/DTS_%28sound_system%29)（DTS Express、DTS LBR、DTS HD、DTS HD 无损）编码的音频。 流式传输 Dolby Atmos 内容适用于特定的标准（例如 MPEG-DASH 协议），采用通用流式传输格式 (CSF) 或通用媒体应用程序格式 (CMAF) 分段 MP4，在使用 CMAF 的情况通过 HTTP 实时传送视频流 (HLS) 来进行。

> [!NOTE]
> 动态打包不支持包含 [Dolby Digital](https://en.wikipedia.org/wiki/Dolby_Digital) (AC3) 音频（它是旧编解码器）的文件。

