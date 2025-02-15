---
title: Azure 存储资源管理器故障排除指南 | Microsoft Docs
description: Azure 存储资源管理器调试方法概述
services: virtual-machines
author: WenJason
ms.service: virtual-machines
ms.topic: troubleshooting
origin.date: 06/15/2018
ms.date: 05/25/2019
ms.author: v-jay
ms.openlocfilehash: d24558e814cc8e3ffc476107d857e9539f3d900a
ms.sourcegitcommit: 5fc46672ae90b6598130069f10efeeb634e9a5af
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/19/2019
ms.locfileid: "67236580"
---
# <a name="azure-storage-explorer-troubleshooting-guide"></a>Azure 存储资源管理器故障排除指南

Azure 存储资源管理器是一款独立应用，可用于在 Windows、macOS 和 Linux 上轻松处理 Azure 存储数据。 应用可连接到托管在 Azure、National Clouds 和 Azure Stack 上的存储帐户。

本指南汇总了存储资源管理器中常见问题的解决方案。

## <a name="role-based-access-control-permission-issues"></a>基于角色的访问控制权限问题

[基于角色的访问控制 (RBAC)](/role-based-access-control/overview) 是通过将权限集组合成“角色”，来提供对 Azure 资源的精细访问管理。  可遵循以下建议在存储资源管理器中正常运行 RBAC。

### <a name="what-do-i-need-to-see-my-resources-in-storage-explorer"></a>需要满足哪些条件才能在存储资源管理器中查看我的资源？

如果在使用 RBAC 访问存储资源时遇到问题，原因可能是你尚未获得相应的角色。 以下部分介绍了存储资源管理器目前要求提供哪些权限来访问存储资源。

如果你不确定自己是否拥有相应的角色或权限，请联系 Azure 帐户管理员。

#### <a name="read-listget-storage-accounts"></a>读取：列出/获取存储帐户

必须有权列出存储帐户。 可以请求分配“读取者”角色来获取此权限。

#### <a name="list-storage-account-keys"></a>列出存储帐户密钥

存储资源管理器还可以使用帐户密钥对请求进行身份验证。 可以使用权限更高的角色（例如“参与者”角色）来获取密钥的访问权限。

> [!NOTE]
> 访问密钥向其任何持有者授予不受限制的权限。 因此，通常不建议将其分发给帐户用户。 如果需要撤消访问密钥，可以通过 [Azure 门户](https://portal.azure.cn/)重新生成访问密钥。

#### <a name="data-roles"></a>数据角色

必须至少拥有一个可以授予对资源中数据的读取访问权限的角色。 例如，如果需要列出或下载 Blob，则至少需要拥有“存储 Blob 数据读取者”角色。

### <a name="why-do-i-need-a-management-layer-role-to-see-my-resources-in-storage-explorer"></a>为何需要管理层角色才能在存储资源管理器中查看我的资源？

Azure 存储提供两个访问层：“管理”和“数据”。   订阅和存储帐户是通过管理层访问的。 容器、Blob 和其他数据资源是通过数据层访问的。 例如，若要从 Azure 获取存储帐户的列表，应向管理终结点发送请求。 若要列出帐户中的 Blob 容器，应向相应的服务终结点发送请求。

RBAC 角色可以包含对管理或数据访问层的权限。 例如，“读取者”角色授予对管理层资源的只读访问权限。

严格地讲，“读取者”角色不提供数据层的权限，并非一定要有该角色才能访问数据层。

在存储资源管理器中，可以通过收集连接到 Azure 资源所需的信息，来轻松访问资源。 例如，若要显示 Blob 容器，存储资源管理器会向 Blob 服务终结点发送“列出容器”请求。 若要获取该终结点，存储资源管理器会搜索你有权访问的订阅和存储帐户列表。 但是，若要查找订阅和存储帐户，存储资源管理器还需要有权访问管理层。

如果你没有一个可以授予任何管理层权限的角色，则存储资源管理器无法获取连接到数据层所需的信息。

### <a name="what-if-i-cant-get-the-management-layer-permissions-i-need-from-my-administrator"></a>如果我无法从管理员获取管理层权限，该怎么办？

目前，我们尚未制定 RBAC 相关的解决方法。 一种解决方法是请求一个 SAS URI 并将其[附加到资源](/vs-azure-tools-storage-manage-with-storage-explorer?tabs=linux#attach-a-service-by-using-a-shared-access-signature-sas)。

## <a name="error-self-signed-certificate-in-certificate-chain-and-similar-errors"></a>错误：证书链中的自签名证书（和类似错误）

证书错误是由以下两种情况之一导致的：

1. 应用通过“透明代理”进行连接，这意味着一台服务器（例如公司的服务器）正在截取 HTTPS 流量，对其进行解密，然后使用自签名证书对其进行加密。
2. 正在运行的应用程序正在向收到的 HTTPS 消息注入自签名 SSL 证书。 注入证书的应用程序示例包括防病毒软件和网络流量检查软件。

当存储资源管理器看到自签名或不受信任的证书时，无法再判断收到的 HTTPS 消息是否被更改。 如果拥有自签名证书的副本，可通过执行以下步骤，让存储资源管理器信任它：

1. 获取证书的 Base-64 编码 X.509 (.cer) 副本
2. 单击“编辑” > “SSL 证书” > “导入证书”，然后使用文件选取器查找、选择和打开 .cer 文件   

此问题还有可能是由于存在多个证书（根证书和中间证书）造成的。 必须添加这两个证书才能解决错误。

如果你不确定该证书来源于何处，可以尝试执行以下步骤来找到它：

1. 安装 Open SSL
    * [Windows](https://slproweb.com/products/Win32OpenSSL.html)（任何精简版本均可）
    * Mac 和 Linux：应包含在操作系统中
2. 运行 Open SSL
    * Windows：打开安装目录，单击“/bin/”，然后双击“openssl.exe”   。
    * Mac 和 Linux：从终端运行 penssl  。
3. 执行 `s_client -showcerts -connect microsoft.com:443`
4. 查找自签名证书。 如果不确定哪些证书是自签名证书，请查看使用者 `("s:")` 和证书颁发者 `("i:")` 相同的任意位置。
5. 找到任何自签名证书时，对于每个证书，将从“-----BEGIN CERTIFICATE-----”到“-----END CERTIFICATE-----”在内的所有内容复制并粘贴到新的 .cer 文件   。
6. 打开存储资源管理器，单击“编辑” > “SSL 证书” > “导入证书”，然后使用文件选取器查找、选择并打开已创建的 .cer 文件    。

如果通过上述步骤无法找到任何自签名证书，请通过反馈工具联系我们以获取更多帮助。 也可选择通过命令行使用 `--ignore-certificate-errors` 标志启动存储资源管理器。 使用此标志启动后，存储资源管理器将忽略证书错误。

## <a name="sign-in-issues"></a>登录问题

### <a name="blank-sign-in-dialog"></a>空白登录对话框

出现空白登录对话框的原因往往是 ADFS 要求存储资源管理器执行 Electron 不支持的重定向。 若要解决此问题，可以尝试使用设备代码流进行登录。 若要执行此操作，请完成以下步骤：

1. 菜单：“预览”->“使用设备代码登录”。
2. 打开“连接”对话框（通过左侧垂直栏上的插头图标或“帐户”面板上的“添加帐户”）。
3. 选择要登录到的环境。
4. 单击“登录”按钮。
5. 按照下一个面板上的说明进行操作。

如果在登录到要使用的帐户时，由于默认浏览器已登录到其他帐户而遇到问题，可通过以下方法解决：

1. 手动将链接和代码复制到浏览器的私有会话中。
2. 手动将链接和代码复制到其他浏览器中。

### <a name="reauthentication-loop-or-upn-change"></a>重新验证循环或 UPN 更改

如果你处于重新验证循环中，或者已更改其中一个帐户的 UPN，请尝试以下步骤：

1. 删除所有帐户，然后关闭存储资源管理器
2. 从计算机中删除 .IdentityService 文件夹。 在 Windows 中，该文件夹位于 `C:\users\<username>\AppData\Local`。 对于 Mac 和 Linux，可以在用户目录的根目录中找到该文件夹。
3. 如果使用的是 Mac 或 Linux，则还需要从 OS 的密钥存储中删除 Microsoft.Developer.IdentityService 条目。 在 Mac 上，密钥存储是“Gnome Keychain”应用程序。 对于 Linux，该应用程序通常称为“Keyring”，但名称可能会有所不同，具体取决于分发版。

## <a name="mac-keychain-errors"></a>Mac 密钥链错误

有时，macOS 密钥链可能会进入导致存储资源管理器的身份验证库出现问题的状态。 若要使密钥链摆脱此状态，请尝试以下步骤：

1. 关闭存储资源管理器。
2. 打开密钥链（**cmd+space**、键入密钥链、按 Enter）。
3. 选择“登录”密钥链。
4. 单击挂锁图标以锁定密钥链（挂锁在完成后将对锁定位置进行动画处理，这可能需要几秒钟的时间，具体要取决于你打开的应用）。

    ![图像](./media/storage-explorer-troubleshooting/unlockingkeychain.png)

5. 启动存储资源管理器。
6. 应出现一个弹出窗口，其中显示“服务中心想要访问密钥链”。 如果是这样，请输入 Mac 管理员帐户密码，然后单击“始终允许”  （如果“始终允许”  不可用，则单击“允许”  ）。
7. 请尝试登录。

### <a name="general-sign-in-troubleshooting-steps"></a>常规登录故障排除步骤

* 在 macOS 上操作时，如果登录窗口永远不会出现，而是一直显示“正在等待身份验证...”对话框，请尝试[这些步骤](#mac-keychain-errors)
* 重启存储资源管理器
* 如果身份验证窗口为空，请等待至少一分钟，然后关闭身份验证对话框。
* 确保为计算机和存储资源管理器正确配置了代理和证书设置。
* 如果在 Windows 上操作，并且有权访问同一台计算机上的 Visual Studio 2019 且可以登录，请尝试登录到 Visual Studio 2019。 成功登录 Visual Studio 2019 后，则可打开存储资源管理器并在帐户面板中查看帐户。

如果这些方法均不起作用，请[在 GitHub 上提出问题](https://github.com/Microsoft/AzureStorageExplorer/issues)。

### <a name="missing-subscriptions-and-broken-tenants"></a>缺少订阅和中断的租户

如果成功登录后无法检索订阅，请尝试以下故障排除方法：

* 验证你的帐户是否有权访问所需的订阅。 可以通过登录到尝试使用的 Azure 环境的门户，来验证是否能够访问这些订阅。
* 确保已使用正确的 Azure 环境（Azure 中国世纪互联）登录。
* 如果使用代理，请确保已正确配置存储资源管理器代理。
* 尝试删除并重新添加帐户。
* 如果有“更多信息”链接，请查看针对失败的租户报告的错误消息。 如果不确定如何处理看到的错误消息，请随时[在 GitHub 上提出问题](https://github.com/Microsoft/AzureStorageExplorer/issues)。

## <a name="cant-remove-attached-account-or-storage-resource"></a>无法删除附加的帐户或存储资源

如果无法通过 UI 删除附加的帐户或存储资源，可以通过删除以下文件夹来手动删除所有附加的资源：

* Windows： `%AppData%/StorageExplorer`
* macOS：`/Users/<your_name>/Library/Application Support/StorageExplorer`
* Linux：`~/.config/StorageExplorer`

> [!NOTE]
> 在删除上述文件夹之前关闭存储资源管理器。

> [!NOTE]
> 如果曾经导入了任何 SSL 证书，请备份 `certs` 目录的内容。 以后，可以使用备份来重新导入 SSL 证书。

## <a name="proxy-issues"></a>代理问题

首先，请确保输入的以下信息都是正确的：

* 代理 URL 和端口号
* 用户名和密码（如果代理需要）

> [!NOTE]
> 存储资源管理器不支持使用代理 auto-config 文件来配置代理设置。

### <a name="common-solutions"></a>常见解决方法

如果仍遇到问题，请尝试以下故障排除方法：

* 如果在不使用代理时可以连接到 Internet，请确认未启用代理设置时存储资源管理器可以正常工作。 如果出现这种情况，可能是代理设置有问题。 与代理服务器管理员合作，找到问题。
* 确认使用代理服务器的其他应用程序按预期运行。
* 验证是否能够连接到尝试使用的 Azure 环境的门户
* 确认可以收到来自服务终结点的响应。 在浏览器中输入其中一个终结点 URL。 如果可以连接，应收到 InvalidQueryParameterValue 或类似的 XML 响应。
* 如果其他人也在通过该代理服务器使用存储资源管理器，请确认他们可以连接。 如果他们可以连接，请与代理服务器管理员联系。

### <a name="tools-for-diagnosing-issues"></a>诊断问题的工具

如果有 Fiddler for Windows 等网络服务工具，则能够按以下方式诊断问题：

* 如果必须通过代理工作，则必须将网络服务工具配置为通过代理进行连接。
* 检查网络服务工具使用的端口号。
* 输入存储资源管理器中代理设置的本地主机 URL 和网络服务工具的端口号。 如果操作无误，网络工具将开始记录存储资源管理器向管理和服务终结点发出的网络请求。 例如，如果在浏览器中输入 https://cawablobgrs.blob.core.chinacloudapi.cn/ 作为 Blob 终结点，将收到类似以下的响应，表示资源存在，尽管无法访问。

![代码示例](./media/storage-explorer-troubleshooting/4022502_en_2.png)

### <a name="contact-proxy-server-admin"></a>与代理服务器管理员联系

如果代理设置正确，则必须与代理服务器管理员联系，并

* 确保代理不会阻止到 Azure 管理或资源终结点的流量。
* 验证代理服务器使用的身份验证协议。 存储资源管理器当前不支持 NTLM 代理。

## <a name="unable-to-retrieve-children-error-message"></a>“无法检索子级”错误消息

如果通过代理连接到 Azure，请确认代理设置正确无误。 如果已获取以订阅或帐户所有者身份访问资源的权限，请验证是否对该资源拥有读取或列出权限。

## <a name="connection-string-doesnt-have-complete-configuration-settings"></a>连接字符串没有完整的配置设置

如果收到此错误消息，则表示你可能没有所需的权限来获取你的存储帐户的密钥。 若要确认是否如此，请转到门户并找到你的存储帐户。 可以通过右键单击存储帐户的节点并单击“在门户中打开”来快速执行此操作。 执行此操作后，转到“访问密钥”边栏选项卡。 如果你无权查看密钥，则会看到其中显示了消息“你没有访问权限”的页面。 若要解决此问题，可以从其他某人获取帐户密钥并结合名称和密钥附加存储帐户，或者，向某人索要存储帐户的 SAS 并使用它来附加存储帐户。

如果看到了帐户密钥，请在 GitHub 上提出问题，使我们能够帮助你解决问题。

## <a name="issues-with-sas-url"></a>SAS URL 的问题

如果使用 SAS URL 连接到服务并遇到以下错误：

* 请确认 URL 提供了读取或列出资源所需的权限。
* 请确认 URL 未过期。
* 如果 SAS URL 基于访问策略，请确认访问策略尚未吊销。

如果意外附加了无效的 SAS URL，并且无法分离，请执行以下步骤：

1. 运行存储资源管理器时，按 F12 打开开发人员工具窗口。
2. 单击“应用程序”选项卡，然后单击左侧树中的“本地存储”> file://。
3. 查找与有问题的 SAS URI 服务类型关联的键。 例如，如果用于 blob 容器的 SAS URI 错误，请查找名为 `StorageExplorer_AddStorageServiceSAS_v1_blob` 的键。
4. 键的值应为 JSON 数组。 查找与错误 URI 关联的对象，并将其删除。
5. 按 Ctrl+R 重新加载存储资源管理器。

## <a name="linux-dependencies"></a>Linux 依赖项

一般情况下，需要安装以下包才能在 Linux 上运行存储资源管理器：

* [.NET Core 2.0 运行时](https://docs.microsoft.com/dotnet/core/linux-prerequisites?tabs=netcore2x) 注意：存储资源管理器 1.7.0 及更低版本需要 .NET Core 2.0。 如果安装了更高版本的 .NET Core，则需修补存储资源管理器（见下）。 如果运行存储资源管理器 1.8.0 或更高版本，则最高应该能使用 .NET Core 2.2。 高于 2.2 的版本目前尚未验证其使用情况。
* `libgnome-keyring-common` 和 `libgnome-keyring-dev`
* `libgconf-2-4`

根据所用的分发版，可能还需要安装不同的包或其他包。

Ubuntu 18.04、16.04 和 14.04 正式支持存储资源管理器。 全新计算机上的安装步骤如下：

# <a name="ubuntu-1804tab1804"></a>[Ubuntu 18.04](#tab/1804)

1. 下载存储资源管理器
2. 安装 .NET Core 运行时，已验证的最新版本为：[2.0.8](https://dotnet.microsoft.com/download/linux-package-manager/ubuntu18-04/runtime-2.0.8)（如果已安装更高的版本，可能需要修补存储资源管理器，请参阅下文）
3. 运行 `sudo apt-get install libgconf-2-4`
4. 运行 `sudo apt install libgnome-keyring-common libgnome-keyring-dev`

# <a name="ubuntu-1604tab1604"></a>[Ubuntu 16.04](#tab/1604)

1. 下载存储资源管理器
2. 安装 .NET Core 运行时，已验证的最新版本为：[2.0.8](https://dotnet.microsoft.com/download/linux-package-manager/ubuntu16-04/runtime-2.0.8)（如果已安装更高的版本，可能需要修补存储资源管理器，请参阅下文）
3. 运行 `sudo apt install libgnome-keyring-dev`

# <a name="ubuntu-1404tab1404"></a>Ubuntu 14.04[](#tab/1404)

1. 下载存储资源管理器
2. 安装 .NET Core 运行时，已验证的最新版本为：[2.0.8](https://dotnet.microsoft.com/download/linux-package-manager/ubuntu14-04/runtime-2.0.8)（如果已安装更高的版本，可能需要修补存储资源管理器，请参阅下文）
3. 运行 `sudo apt install libgnome-keyring-dev`

---

### <a name="patching-storage-explorer-for-newer-versions-of-net-core"></a>修补适用于 .NET Core 更高版本的存储资源管理器 
如果安装的 .NET Core 版本高于 2.0，而运行的存储资源管理器版本为 1.7.0 或更低，则很有可能需要通过完成以下步骤来修补存储资源管理器：
1. [从 Nuget](https://www.nuget.org/packages/StreamJsonRpc/1.5.43) 中下载 StreamJsonRpc 版本 1.5.43。 在页面的右侧找到“下载包”链接。
2. 下载该包后，将其文件扩展名从 `.nupkg` 更改为 `.zip`
3. 将包解压缩
4. 转到 `streamjsonrpc.1.5.43/lib/netstandard1.1/`
5. 将 `StreamJsonRpc.dll` 复制到存储资源管理器文件夹中的以下位置：
    1. `StorageExplorer/resources/app/ServiceHub/Services/Microsoft.Developer.IdentityService/`
    2. `StorageExplorer/resources/app/ServiceHub/Hosts/ServiceHub.Host.Core.CLR.x64/`

## <a name="open-in-explorer-from-azure-portal-doesnt-work"></a>Azure 门户中的“在资源管理器中打开”不起作用

如果在 Azure 门户上单击“在资源管理器中打开”按钮不起作用，请确保使用兼容的浏览器。 以下浏览器已通过兼容性测试。
* Microsoft Edge
* Mozilla Firefox
* Google Chrome
* Microsoft Internet Explorer

## <a name="next-steps"></a>后续步骤

如果上述解决方法均不起作用，请[在 GitHub 上提出问题](https://github.com/Microsoft/AzureStorageExplorer/issues)。 也可以使用左下角的“向 GitHub 报告问题”按钮快速转到 GitHub。

![反馈](./media/storage-explorer-troubleshooting/feedback-button.PNG)
