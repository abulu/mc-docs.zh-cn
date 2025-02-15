---
title: Azure 数据工厂中的安全注意事项 | Microsoft Docs
description: 介绍 Azure 数据工厂中的数据移动服务用来帮助保护数据的基本安全基础结构。
services: data-factory
documentationcenter: ''
author: WenJason
manager: digimobile
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.topic: conceptual
origin.date: 06/15/2018
ms.date: 07/08/2019
ms.author: v-jay
ms.openlocfilehash: 4f4654b3bc0ccbce9b3481d7f05ecf57ebd054da
ms.sourcegitcommit: 5191c30e72cbbfc65a27af7b6251f7e076ba9c88
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/05/2019
ms.locfileid: "67569847"
---
#  <a name="security-considerations-for-data-movement-in-azure-data-factory"></a>Azure 数据工厂中数据移动的安全注意事项

本文介绍 Azure 数据工厂中数据移动服务用于帮助保护数据的基本安全基础结构。 数据工厂管理资源建立在 Azure 安全基础结构上，并使用 Azure 提供的所有可能的安全措施。

在数据工厂解决方案中，可以创建一个或多个数据[管道](concepts-pipelines-activities.md)。 “管道”是共同执行一项任务的活动的逻辑分组。 这些管道位于创建数据工厂的区域。 

除使用证书加密的云数据存储的链接服务凭据外，Azure 数据工厂不存储任何其他数据。 使用数据工厂可以创建数据驱动的工作流，协调[受支持数据存储](copy-activity-overview.md#supported-data-stores-and-formats)之间的数据移动，以及使用[计算服务](compute-linked-services.md)在其他区域或本地环境中处理数据。 还可以使用 SDK 与 Azure Monitor 来监视和管理工作流。

数据工厂已获得以下认证：

| **[CSA STAR 认证](https://www.microsoft.com/trustcenter/compliance/csa-star-certification)** |
| :----------------------------------------------------------- |
| **[ISO 20000-1:2011](https://www.microsoft.com/trustcenter/Compliance/ISO-20000-1)** |
| **[ISO 22301:2012](https://www.microsoft.com/trustcenter/compliance/iso-22301)** |
| **[ISO 27001:2013](https://www.microsoft.com/trustcenter/compliance/iso-iec-27001)** |
| **[ISO 27017:2015](https://www.microsoft.com/trustcenter/compliance/iso-iec-27017)** |
| **[ISO 27018:2014](https://www.microsoft.com/trustcenter/compliance/iso-iec-27018)** |
| **[ISO 9001:2015](https://www.microsoft.com/trustcenter/compliance/iso-9001)** |
| **[SOC 1, 2, 3](https://www.microsoft.com/trustcenter/compliance/soc)** |
| **[HIPAA BAA](https://www.microsoft.com/trustcenter/compliance/hipaa)** |

如果对 Azure 合规性以及 Azure 如何保护其专属基础结构感兴趣，请访问 [Azure 信任中心](https://www.trustcenter.cn/zh-cn/default.html)。 有关所有 Azure 合规性产品检查的最新列表 - https://aka.ms/AzureCompliance 。

在本文中，我们将查看以下两个数据移动方案中的安全注意事项： 

- **云场景**：在此场景中，源和目标都可通过 Internet 公开访问。 其中包括托管的云存储服务（如 Azure 存储、Azure SQL 数据仓库、Azure SQL 数据库、Amazon S3 和 Amazon Redshift）、SaaS 服务（如 Salesforce）以及 Web 协议（如 FTP 和 OData）。 可以在[支持的数据存储和格式](copy-activity-overview.md#supported-data-stores-and-formats)中找到受支持数据源的完整列表。
- **混合场景**：在此场景中，源或目标位于防火墙之后或本地公司网络中。 或者，数据存储位于专用网络或虚拟网络（通常是源）中，且不可公开访问。 虚拟机上托管的数据库服务器也属于这种情况。

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="cloud-scenarios"></a>云方案

### <a name="securing-data-store-credentials"></a>保护数据存储凭据

- **在 Azure 数据工厂托管存储中存储加密的凭据**。 数据工厂使用由 Azure 管理的证书对数据存储凭据加密，从而帮助为这些凭据提供保护。 这些证书每两年轮换一次（包括证书续订和凭据迁移）。 这些加密凭据安全地存储在由 Azure 数据工厂管理服务管理的 Azure 存储帐户中。 有关 Azure 存储安全的详细信息，请参阅 [Azure 存储安全概述](../security/security-storage-overview.md)。
- **在 Azure Key Vault 中存储凭据**。 还可以将数据存储的凭据存储在 [Azure Key Vault](/key-vault/) 中。 数据工厂在执行某个活动期间会检索该凭据。 有关详细信息，请参阅[在 Azure Key Vault 中存储凭据](store-credentials-in-key-vault.md)。

### <a name="data-encryption-in-transit"></a>传输中的数据加密
如果云数据存储支持 HTTPS 或 TLS，则数据工厂中数据移动服务与云数据存储之间的所有数据传输均通过安全通道 HTTPS 或 TLS 进行。

> [!NOTE]
> 在与数据库相互传输数据时，与 Azure SQL 数据库和 Azure SQL 数据仓库的所有连接需要经过加密 (SSL/TLS)。 在使用 JSON 创作管道时，请在连接字符串中添加 encryption 属性并将其设置为 **true**。 对于 Azure 存储，可以在连接字符串中使用 **HTTPS**。

> [!NOTE]
> 若要在从 Oracle 移动数据时启用传输加密，请遵循以下选项之一：
> 1. 在 Oracle 服务器中，转到“Oracle 高级安全性(OAS)”并配置加密设置，该设置支持三重 DES 加密 (3DES) 和高级加密标准 (AES)，请参阅[此处](https://docs.oracle.com/cd/E11882_01/network.112/e40393/asointro.htm#i1008759)了解详细信息。 ADF 会自动协商加密方法，以便在与 Oracle 建立连接时使用在 OAS 中配置的加密方法。
> 2. 在 ADF 中，可以在连接字符串中添加 EncryptionMethod=1（在链接服务中）。 这将使用 SSL/TLS 作为加密方法。 若要使用此功能，需要在 Oracle 服务器端的 OAS 中禁用非 SSL 加密设置，以避免加密冲突。

> [!NOTE]
> 使用的 TLS 版本为 1.2。

### <a name="data-encryption-at-rest"></a>静态数据加密
某些数据存储支持静态数据加密。 我们建议为这些数据存储启用数据加密机制。 

#### <a name="azure-sql-data-warehouse"></a>Azure SQL 数据仓库
Azure SQL 数据仓库中的透明数据加密 (TDE) 可帮助对静态数据进行实时加密和解密，避免恶意活动造成的威胁。 此行为对客户端透明。 有关详细信息，请参阅[保护 SQL 数据仓库中的数据库](../sql-data-warehouse/sql-data-warehouse-overview-manage-security.md)。

#### <a name="azure-sql-database"></a>Azure SQL 数据库
Azure SQL 数据库还支持透明数据加密 (TDE)，它无需更改应用程序，即可对数据执行实时加密和解密，从而帮助防止恶意活动的威胁。 此行为对客户端透明。 有关详细信息，请参阅 [SQL 数据库和数据仓库的透明数据加密](https://docs.microsoft.com/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql)。

#### <a name="azure-blob-storage-and-azure-table-storage"></a>Azure Blob 存储和 Azure 表存储
Azure Blob 存储和 Azure 表存储支持存储服务加密 (SSE)，它会在将数据保存到存储中前进行自动加密，在检索前进行自动解密。 有关详细信息，请参阅[静态数据的 Azure 存储服务加密](../storage/common/storage-service-encryption.md)。

#### <a name="amazon-s3"></a>Amazon S3
Amazon S3 支持静态数据的客户端和服务器加密。 有关详细信息，请参阅[使用加密保护数据](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingEncryption.html)。

#### <a name="amazon-redshift"></a>Amazon Redshift
Amazon Redshift 支持静态数据的群集加密。 有关详细信息，请参阅 [Amazon Redshift 数据库加密](https://docs.aws.amazon.com/redshift/latest/mgmt/working-with-db-encryption.html)。 

#### <a name="salesforce"></a>Salesforce
Salesforce 支持防火墙平台加密，它允许加密所有文件、附件和自定义字段。 有关详细信息，请参阅 [Understanding the Web Server OAuth Authentication Flow](https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/intro_understanding_web_server_oauth_flow.htm)（了解 Web 服务器 OAuth 身份验证流）。  

## <a name="hybrid-scenarios"></a>混合场景
混合场景需要在本地网络、虚拟网络 (Azure) 或虚拟私有云 (Amazon) 中安装自承载集成运行时。 自承载集成运行时必须能够访问本地数据存储。 有关自承载集成运行时的详细信息，请参阅[如何创建和配置自承载集成运行时](/data-factory/create-self-hosted-integration-runtime)。 

![自承载集成运行时通道](media/data-movement-security-considerations/data-management-gateway-channels.png)

使用命令通道可在数据工厂中的数据移动服务与自承载集成运行时之间通信。 通信包含与活动相关的信息。 数据信道用于在本地数据存储和云数据存储之间传输数据。    

### <a name="on-premises-data-store-credentials"></a>本地数据存储凭据
本地数据存储的凭据始终经过加密并存储。 凭据可存储在自承载集成运行时计算机本地，或存储在 Azure 数据工厂托管存储中（如同云存储凭据一样）。 

- **在本地存储凭据**。 如果想要在自承载集成运行时本地加密并存储凭据，请遵循[在 Azure 数据工厂中加密本地数据存储的凭据](encrypt-credentials-self-hosted-integration-runtime.md)中的步骤。 所有连接器都支持此选项。 自承载集成运行时使用 Windows [DPAPI](https://msdn.microsoft.com/library/ms995355.aspx) 来加密敏感数据和凭据信息。 

   使用 **New-AzDataFactoryV2LinkedServiceEncryptedCredential** cmdle 可加密链接服务凭据和链接服务中的敏感详细信息。 然后，可以通过 **Set-AzDataFactoryV2LinkedService** cmdlet 使用返回的 JSON（结合连接字符串中的 **EncryptedCredential** 元素）创建链接服务。  

- **在 Azure 数据工厂托管存储中存储**。 如果直接结合 JSON 中内联的连接字符串和凭据使用 **Set-AzDataFactoryV2LinkedService** cmdlet，则链接服务将加密并存储在 Azure 数据工厂托管存储中。 敏感信息仍由证书加密，这些证书由 Azure 管理。



#### <a name="ports-used-when-encrypting-linked-service-on-self-hosted-integration-runtime"></a>在自承载集成运行时中加密链接服务时使用的端口
默认情况下，PowerShell 在装有自承载集成运行时的计算机上使用端口 8050 来确保通信安全。 如有必要，可以更改此端口。  

![网关的 HTTPS 端口](media/data-movement-security-considerations/https-port-for-gateway.png)

 


### <a name="encryption-in-transit"></a>传输中加密
所有数据传输都是通过 HTTPS 和 TLS over TCP 安全通道进行的，可防止与 Azure 服务通信期间发生中间人攻击。

还可以使用 [IPSec VPN](../vpn-gateway/vpn-gateway-about-vpn-devices.md) 或 [Azure ExpressRoute](../expressroute/expressroute-introduction.md) 进一步保护本地网络和 Azure 之间的通信信道。

Azure 虚拟网络是网络在云中的逻辑表示形式。 可以通过设置 IPSec VPN（站点到站点）或 ExpressRoute（专用对等互连）将本地网络连接到虚拟网络。    

下表根据混合数据移动的源和目标位置的不同组合，汇总了有关网络和自承载集成运行时的配置建议。

| Source      | 目标                              | 网络配置                    | 集成运行时安装                |
| ----------- | ---------------------------------------- | ---------------------------------------- | ---------------------------------------- |
| 本地 | 虚拟网络中部署的虚拟机和云服务 | IPSec VPN（点到站点或站点到站点） | 自承载集成运行时应安装在虚拟网络中的 Azure 虚拟机上。  |
| 本地 | 虚拟网络中部署的虚拟机和云服务 | ExpressRoute（专用对等互连）           | 自承载集成运行时应安装在虚拟网络中的 Azure 虚拟机上。  |
| 本地 | 具有公共终结点的基于 Azure 的服务 | ExpressRoute（Azure 对等互连）            | 自承载集成运行时可以在本地安装，也可以安装在 Azure 虚拟机上。 |

下图显示了如何使用自承载集成运行时通过 ExpressRoute 和 IPSec VPN（具有 Azure 虚拟网络）在本地数据库和 Azure 服务之间移动数据：

**ExpressRoute**

![结合使用 ExpressRoute 和网关](media/data-movement-security-considerations/express-route-for-gateway.png) 

**IPSec VPN**

![将 IPSec VPN 与网关配合使用](media/data-movement-security-considerations/ipsec-vpn-for-gateway.png)

### <a name="firewall-configurations-and-whitelisting-ip-address-of-gateway"></a> 防火墙配置及将 IP 地址加入允许列表

#### <a name="firewall-requirements-for-on-premisesprivate-network"></a>本地/专用网络的防火墙要求  
在企业中，企业防火墙在组织的中央路由器上运行。 Windows 防火墙在安装自承载集成运行时的本地计算机上作为守护程序运行。 

下表提供了企业防火墙的出站端口和域要求：

| 域名                  | 出站端口 | 说明                              |
| ----------------------------- | -------------- | ---------------------------------------- |
| `*.servicebus.chinacloudapi.cn`    | 443            | 自承载集成运行时连接到数据工厂中的数据移动服务时需要此端口。 |
| `*.frontend.clouddatahub.net` | 443            | 自承载集成运行时连接到数据工厂服务时需要此端口。 |
| `download.microsoft.com`    | 443            | 自承载集成运行时下载更新时需要此端口。 如果已禁用自动更新，则可以跳过此设置。 |
| `*.core.chinacloudapi.cn`          | 443            | 使用[分阶段复制](copy-activity-performance.md#staged-copy)功能时，由自承载集成运行时用来连接到 Azure 存储帐户。 |
| `*.database.chinacloudapi.cn`      | 1433           | （可选）从/向 Azure SQL 数据库或 Azure SQL 数据仓库复制时需要。 在不打开端口 1433 的情况下，使用暂存复制功能将数据复制到 Azure SQL 数据库或 Azure SQL 数据仓库。 |

> [!NOTE] 
> 可能需要按相应数据源的要求在企业防火墙级别管理端口/允许列表域。 此表仅以 Azure SQL 数据库和 Azure SQL 数据仓库为例。   

下表提供了 Windows 防火墙的入站端口要求：

| 入站端口 | 说明                              |
| ------------- | ---------------------------------------- |
| 8060 (TCP)    | PowerShell 加密 cmdlet（参阅[在 Azure 数据工厂中加密本地数据存储的凭据](encrypt-credentials-self-hosted-integration-runtime.md)）和凭据管理器应用程序需要使用此端口在自承载集成运行时中安全设置本地数据存储的凭据。 |

![网关端口要求](media/data-movement-security-considerations/gateway-port-requirements.png) 

#### <a name="ip-configurations-and-whitelisting-in-data-stores"></a>数据存储中的 IP 配置和允许列表
云中的某些数据存储还需要将访问存储的计算机的 IP 地址加入允许列表。 确保已在防火墙中相应地将自承载集成运行时计算机的 IP 地址加入允许列表或进行配置。

以下云数据存储要求将自承载集成运行时计算机的 IP 地址加入允许列表。 默认情况下，某些这类数据存储可能不需要允许列表。 

- [Azure SQL 数据库](../sql-database/sql-database-firewall-configure.md) 
- [Azure SQL 数据仓库](../sql-data-warehouse/sql-data-warehouse-get-started-provision.md)
- [Azure Cosmos DB](../cosmos-db/firewall-support.md)
- [Amazon Redshift](https://docs.aws.amazon.com/redshift/latest/gsg/rs-gsg-authorize-cluster-access.html) 

## <a name="frequently-asked-questions"></a>常见问题

**是否可在不同的数据工厂之间共享自承载集成运行时？**

是的。 [此处](https://azure.microsoft.com/blog/sharing-a-self-hosted-integration-runtime-infrastructure-with-multiple-data-factories/)提供了更多详细信息。

**需要满足哪些端口要求才能让自承载集成运行时正常工作？**

自承载集成运行时与访问 Internet 建立基于 HTTP 的连接。 必须打开出站端口 443，才能让自承载集成运行时建立此连接。 仅在计算机级别（不是企业防火墙级别）为凭据管理器应用程序打开入站端口 8050。 如果使用 Azure SQL 数据库或 Azure SQL 数据仓库作为源或目标，则还需要打开端口 1433。 有关详细信息，请参阅[防火墙配置和允许列表 IP 地址](#firewall-configurations-and-whitelisting-ip-address-of-gateway)部分。 


## <a name="next-steps"></a>后续步骤
有关 Azure 数据工厂复制活动性能的信息，请参阅[复制活动性能和优化指南](copy-activity-performance.md)。

 
