---
title: Azure 到 Azure 复制问题和错误的 Azure Site Recovery 故障排除 | Azure
description: 排查复制 Azure 虚拟机进行灾难恢复时出现的错误和问题
services: site-recovery
author: rockboyfor
manager: digimobile
ms.service: site-recovery
ms.topic: article
origin.date: 04/08/2019
ms.date: 07/08/2019
ms.author: v-yeche
ms.openlocfilehash: fe4d9810b803dfd9b6534173d250e8b6c4fc3885
ms.sourcegitcommit: e575142416298f4d88e3d12cca58b03c80694a32
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/12/2019
ms.locfileid: "67861677"
---
# <a name="troubleshoot-azure-to-azure-vm-replication-issues"></a>Azure 到 Azure VM 复制问题故障排除

本文介绍将 Azure 虚拟机从一个区域复制和恢复到另一个区域时的 Azure Site Recovery 常见问题，并介绍了如何排查这些问题。 有关支持的配置的详细信息，请参阅 [support matrix for replicating Azure VMs](site-recovery-support-matrix-azure-to-azure.md)（复制 Azure VM 的支持矩阵）。

## <a name="list-of-errors"></a>错误列表
- **[Azure 资源配额问题（错误代码 150097）](#azure-resource-quota-issues-error-code-150097)**
- **[受信任的根证书（错误代码 151066）](#trusted-root-certificates-error-code-151066)**
- **[Site Recovery 的出站连接（错误代码 151195）](#issue-1-failed-to-register-azure-virtual-machine-with-site-recovery-151195-br)**

<a name="azure-resource-quota-issues-error-code-150097"></a>
## <a name="azure-resource-quota-issues-error-code-150097"></a>Azure 资源配额问题（错误代码 150097）
应启用订阅，在计划用作灾难恢复区域的目标区域中创建 Azure VM。 此外，订阅应启用充足的配额以创建特定大小的 VM。 默认情况下，Site Recovery 选取与源 VM 同样大小的目标 VM。 如果无法选择匹配的大小，则会自动选取最接近的大小。 如果没有支持源 VM 配置的匹配大小，将出现下面的错误消息：

错误代码  | **可能的原因** | 建议 
--- | --- | ---
150097<br><br />**消息**：无法为虚拟机 VmName 启用复制。 | - 可能未启用订阅 ID，无法在目标区域位置中创建任何 VM。<br /><br />- 可能未启用订阅 ID 或没有足够的配额，无法在目标区域位置中创建特定大小的 VM。<br /><br />- 对于订阅 ID，在目标区域位置中找不到与源 VM NIC 计数 (2) 匹配的合适的目标 VM 大小。| 联系 [Azure 计费支持](https://support.azure.cn/zh-cn/support/support-azure/)，对订阅启用 VM 创建，以便在目标位置中创建所需大小的 VM。 启用后，重试失败的操作。

### <a name="fix-the-problem"></a>解决问题
可联系 [Azure 计费支持](https://support.azure.cn/zh-cn/support/support-azure/)启用订阅，以便在目标位置中创建所需大小的 VM。

如果目标位置存在容量约束，可禁用复制然后在订阅拥有充足配额的其他位置启用复制，以便创建所需大小的 VM.

<a name="trusted-root-certificates-error-code-151066"></a>
## <a name="trusted-root-certificates-error-code-151066"></a>受信任的根证书（错误代码 151066）

如果 VM 上不存在任何最新的受信任根证书，则“启用复制”作业可能会失败。 没有证书，VM 发出的 Site Recovery 服务调用的身份验证和授权会失败。 将显示失败的“启用复制”Site Recovery 作业的错误消息：

错误代码  | 可能的原因  | **建议**
--- | --- | ---
151066<br><br />**消息**：Site Recovery 配置失败。 | 计算机上不存在用于授权和身份验证的必需受信根证书。 | - 对于运行 Windows 操作系统的 VM，请确保计算机上存在受信任的根证书。 有关信息，请参阅[配置受信任根和不允许的证书](https://technet.microsoft.com/library/dn265983.aspx)。<br><br />- 对于运行 Linux 操作系统的 VM，请按照 Linux 操作系统版本分发商发布的受信任的根证书指南进行操作。

### <a name="fix-the-problem"></a>解决问题
**Windows**

在 VM 上安装所有最新的 Windows 更新，以便所有受信任的根证书均存在于计算机上。 在断开连接的环境中，请按照组织中的标准 Windows 更新过程执行操作并获取证书。 如果 VM 上不存在所需证书，则对 Site Recovery 服务的调用会因安全原因而失败。

按照组织中典型的 Windows 更新管理或证书更新管理过程进行操作，在 VM 上获取所有最新的根证书和更新的证书吊销列表。

要验证问题是否已解决，请在 VM 中使用浏览器转到 login.chinacloudapi.cn。

**Linux**

请按照 Linux 分发商提供的指南进行操作，在 VM 上获取最新的受信任根证书和最新的证书吊销列表。

由于 SuSE Linux 使用符号链接来维护证书列表，因此请按照以下步骤进行操作：

1. 以根用户身份登录。

2. 运行此命令以更改目录。

    ``# cd /etc/ssl/certs``

1. 检查 Symantec 根 CA 证书是否存在。

    ``# ls VeriSign_Class_3_Public_Primary_Certification_Authority_G5.pem``

2. 如果未找到 Symantec 根 CA 证书，则运行以下命令下载文件。 检查是否有任何错误，对于网络故障执行建议的操作。

    ``# wget https://www.symantec.com/content/dam/symantec/docs/other-resources/verisign-class-3-public-primary-certification-authority-g5-en.pem -O VeriSign_Class_3_Public_Primary_Certification_Authority_G5.pem``

3. 检查 Baltimore 根 CA 证书是否存在。

    ``# ls Baltimore_CyberTrust_Root.pem``

4. 如果未找到 Baltimore 根 CA 证书，则下载该证书。  

    ``# wget https://www.digicert.com/CACerts/BaltimoreCyberTrustRoot.crt.pem -O Baltimore_CyberTrust_Root.pem``

5. 检查 DigiCert_Global_Root_CA 证书是否存在。

    ``# ls DigiCert_Global_Root_CA.pem``

6. 如果未找到 DigiCert_Global_Root_CA，则运行以下命令下载该证书。

    ``# wget http://www.digicert.com/CACerts/DigiCertGlobalRootCA.crt``

    ``# openssl x509 -in DigiCertGlobalRootCA.crt -inform der -outform pem -out DigiCert_Global_Root_CA.pem``

7. 运行 rehash 脚本更新新下载的证书的证书使用者哈希。

    ``# c_rehash``

8. 检查是否已为证书创建使用者哈希作为符号链接。

    - 命令

        ``# ls -l | grep Baltimore``

    - 输出

        ``lrwxrwxrwx 1 root root   29 Jan  8 09:48 3ad48a91.0 -> Baltimore_CyberTrust_Root.pem
        -rw-r--r-- 1 root root 1303 Jun  5  2014 Baltimore_CyberTrust_Root.pem``

    - 命令

        ``# ls -l | grep VeriSign_Class_3_Public_Primary_Certification_Authority_G5``

    - 输出

        ``-rw-r--r-- 1 root root 1774 Jun  5  2014 VeriSign_Class_3_Public_Primary_Certification_Authority_G5.pem
        lrwxrwxrwx 1 root root   62 Jan  8 09:48 facacbc6.0 -> VeriSign_Class_3_Public_Primary_Certification_Authority_G5.pem``

    - 命令

        ``# ls -l | grep DigiCert_Global_Root``

    - 输出

        ``lrwxrwxrwx 1 root root   27 Jan  8 09:48 399e7759.0 -> DigiCert_Global_Root_CA.pem
        -rw-r--r-- 1 root root 1380 Jun  5  2014 DigiCert_Global_Root_CA.pem``

9. 使用文件名 b204d74a.0 创建文件 VeriSign_Class_3_Public_Primary_Certification_Authority_G5.pem 的副本

    ``# cp VeriSign_Class_3_Public_Primary_Certification_Authority_G5.pem b204d74a.0``

10. 使用文件名 653b494a.0 创建文件 Baltimore_CyberTrust_Root.pem 的副本

    ``# cp Baltimore_CyberTrust_Root.pem 653b494a.0``

13. 使用文件名 3513523f.0 创建文件 DigiCert_Global_Root_CA.pem 的副本

    ``# cp DigiCert_Global_Root_CA.pem 3513523f.0``  

14. 检查文件是否存在。  

    - 命令

        ``# ls -l 653b494a.0 b204d74a.0 3513523f.0``

    - 输出

        ``-rw-r--r-- 1 root root 1774 Jan  8 09:52 3513523f.0
        -rw-r--r-- 1 root root 1303 Jan  8 09:52 653b494a.0
        -rw-r--r-- 1 root root 1774 Jan  8 09:52 b204d74a.0``

## <a name="outbound-connectivity-for-site-recovery-urls-or-ip-ranges-error-code-151037-or-151072"></a>Site Recovery URL 或 IP范围的出站连接（错误代码 151037 或 151072）

要使 Site Recovery 复制正常运行，必须具有从 VM 到特定 URL 或 IP 范围的出站连接。 如果 VM 位于防火墙后或使用网络安全组 (NSG) 规则来控制出站连接，则可能会遇到以下问题之一。

<a name="issue-1-failed-to-register-azure-virtual-machine-with-site-recovery-151195-br"></a>
### <a name="issue-1-failed-to-register-azure-virtual-machine-with-site-recovery-151195-br-"></a>问题 1：未能向 Site Recovery 注册 Azure 虚拟机 (151195) <br />
- 可能的原因  <br />
    - 由于 DNS 解析失败而无法建立到 Site Recovery 终结点的连接。
    - 在重新保护期间，对虚拟机进行故障转移但无法从 DR 区域访问 DNS 服务器时经常会出现此问题。

- **解决方法**
    - 如果你使用的是自定义 DNS，请确保可以从灾难恢复区域访问 DNS 服务器。 若要检查你是否具有自定义 DNS，请转到“VM”>“灾难恢复网络”>“DNS 服务器”。 尝试从虚拟机访问 DNS 服务器。 如果它无法访问，请通过对 DNS 服务器进行故障转移或创建 DR 网络与 DNS 之间站点的行来使其可访问。

        ![com-error](./media/azure-to-azure-troubleshoot-errors/custom_dns.png)

### <a name="issue-2-site-recovery-configuration-failed-151196"></a>问题 2：Site Recovery 配置失败 (151196)
- 可能的原因  <br />
    - 无法建立到 Office 365 身份验证和标识 IP4 终结点的连接。

- **解决方法**
    - Azure Site Recovery 需要具有对 Office 365 IP 范围的访问权限来进行身份验证。
        如果你使用 Azure 网络安全组 (NSG) 规则/防火墙代理控制 VM 上的出站网络连接，请确保允许到 O365 IP 范围的通信。 创建一个基于 [Azure Active Directory (AAD) 服务标记](../virtual-network/security-overview.md#service-tags)的 NSG 规则以允许访问与 AAD 对应的所有 IP 地址
        - 如果将来要向 Azure Active Directory (AAD) 添加新地址，则需要创建新的 NSG 规则。

> [!NOTE]
> 如果虚拟机位于**标准**内部负载均衡器之后，则默认情况下无法访问 O365 IP，即 login.partner.microsoftonline.cn。 请将其更改为**基本**内部负载均衡器类型或创建[此文](../load-balancer/configure-load-balancer-outbound-cli.md)中提到的出站访问权限。

### <a name="issue-3-site-recovery-configuration-failed-151197"></a>问题 3：Site Recovery 配置失败 (151197)
- 可能的原因  <br />
    - 无法建立到 Azure Site Recovery 服务终结点的连接。

- **解决方法**
    - Azure Site Recovery 需要根据区域访问 [Site Recovery IP 范围](/site-recovery/azure-to-azure-about-networking#outbound-connectivity-for-ip-address-ranges)。 请确保可以从虚拟机访问所需的 IP 范围。

### <a name="issue-4-a2a-replication-failed-when-the-network-traffic-goes-through-on-premises-proxy-server-151072"></a>问题 4：当网络流量通过本地代理服务器时 A2A 复制失败 (151072)
- 可能的原因  <br />
    - 自定义代理设置无效，并且 ASR 移动服务代理未在 IE 中自动检测到代理设置

- **解决方法**
    1. 移动服务代理通过 Windows 上的 IE 和 Linux 上的 /etc/environment 检测代理设置。
    2. 如果只想对 ASR 移动服务设置代理，可在位于以下路径的 ProxyInfo.conf 中提供代理详细信息：<br />
        - ***Linux*** 上的 ``/usr/local/InMage/config/``
        - ***Windows*** 上的 ``C:\ProgramData\Microsoft Azure Site Recovery\Config``
    3. ProxyInfo.conf 应包含采用以下 INI 格式的代理设置。<br />
        *[proxy]*<br />
        *Address=http://1.2.3.4*<br />
        *Port=567*<br />
    4. ASR 移动服务代理仅支持***未经身份验证的代理***。

### <a name="fix-the-problem"></a>解决问题
要将[所需 URL](azure-to-azure-about-networking.md#outbound-connectivity-for-urls) 或[所需 IP 范围](azure-to-azure-about-networking.md#outbound-connectivity-for-ip-address-ranges)列入允许列表，请按照[网络指南文档](site-recovery-azure-to-azure-networking-guidance.md)中的步骤进行操作。

## <a name="disk-not-found-in-the-machine-error-code-150039"></a>计算机中找不到磁盘（错误代码 150039）

必须对附加到 VM 的新磁盘进行初始化。

错误代码  | 可能的原因  | 建议 
--- | --- | ---
150039<br><br />**消息**：逻辑单元号 (LUN) 为 (LUNValue) 的 Azure 数据磁盘 (DiskName) (DiskURI) 未映射到具有相同 LUN 值的 VM 报告的相应磁盘。 | - 新的数据磁盘已附加到 VM，但未初始化。<br /><br />- VM 中的数据磁盘未正确报告附加到 VM 的磁盘的 LUN 值。| 确保数据磁盘已初始化，然后重试该操作。<br /><br />对于 Windows：[附加并初始化新的磁盘](/virtual-machines/windows/attach-managed-disk-portal)。<br /><br />对于 Linux：[在 Linux 中初始化新的数据磁盘](/virtual-machines/linux/add-disk)。

### <a name="fix-the-problem"></a>解决问题
确保数据磁盘已初始化，然后重试该操作：

- 对于 Windows：[附加并初始化新的磁盘](/virtual-machines/windows/attach-managed-disk-portal)。
- 对于 Linux：[在 Linux 中添加新数据磁盘](/virtual-machines/linux/add-disk)。

如果问题仍然存在，请联系支持部门。

## <a name="one-or-more-disks-are-available-for-protectionerror-code-153039"></a>一个或多个磁盘可用于保护（错误代码 153039）
- 可能的原因  <br />
    - 如果最近在保护后将一个或多个磁盘添加到虚拟机。 
    - 如果在保护虚拟机之后初始化了一个或多个磁盘。

### <a name="fix-the-problem"></a>解决问题
可以选择保护磁盘或忽略警告，以使 VM 的复制状态再次恢复正常。<br />
1. 要保护磁盘， 请导航到“复制的项”>“VM”>“磁盘”>单击未受保护的磁盘>“启用复制”。
    ![add_disks](./media/azure-to-azure-troubleshoot-errors/add-disk.png)
2. 要关闭警告， 请转到“复制的项”>“VM”> 在“概述”部分下单击“关闭警报”。
    ![dismiss_warning](./media/azure-to-azure-troubleshoot-errors/dismiss-warning.png)

## <a name="unable-to-see-the-azure-vm-or-resource-group--for-selection-in-enable-replication"></a>无法在“启用复制”中看到要选择的 Azure VM 或资源组

**原因 1：资源组和源虚拟机位于不同的位置** <br />
Azure Site Recovery 当前强制要求源区域资源组和虚拟机应位于同一位置。 如果不是这种情况，那么在保护期间将无法找到虚拟机。 一种解决方法是，可以从 VM 而不是从恢复服务保管库启用复制。 转到“源 VM”>“属性”>“灾难恢复”并启用复制。

**原因 2：资源组不是所选订阅的一部分** <br />
如果资源组不是给定订阅的一部分，则可能无法在保护期间找到该资源组。 确保资源组属于正在使用的订阅。

**原因 3：过时配置** <br />
如果看不到要为其启用复制的虚拟机，可能是因为有过时的 Site Recovery 配置保留在 Azure VM 中。 在以下情况下，Azure VM 中可能留有过时的配置：

- 使用 Site Recovery 启用了 Azure VM 的复制，然后在删除 Site Recovery 保管库时未显式禁用 VM 上的复制。
- 使用 Site Recovery 启用了 Azure VM 的复制，然后在删除包含 Site Recovery 保管库的资源组时未显式禁用 VM 上的复制。

### <a name="fix-the-problem"></a>解决问题

>[!NOTE]
>
>请确保在使用以下脚本之前更新“AzureRM.Resources”模块。

可以使用[删除过时的 ASR 配置脚本](https://github.com/AsrOneSdk/published-scripts/blob/master/Cleanup-Stale-ASR-Config-Azure-VM.ps1)，删除 Azure VM 上过时的 Site Recovery 配置。 删除过时配置后，应能够看到该 VM。

>[!NOTE]
>
> 请确保在下载的[删除过时的 ASR 配置脚本](https://github.com/AsrOneSdk/published-scripts/blob/master/Cleanup-Stale-ASR-Config-Azure-VM.ps1)中将 `Login-AzureRmAccount` 替换为 `Login-AzureRmAccount -Environment AzureChinaCloud`。

## <a name="unable-to-select-virtual-machine-for-protection"></a>无法选择虚拟机进行保护
**原因 1：虚拟机安装的某些扩展处于失败或无响应状态** <br />
转到“虚拟机”>“设置”>“扩展”，并检查是否存在任何失败状态的扩展。 卸载失败的扩展，然后重试保护虚拟机。<br />
**原因 2：[VM 的预配状态无效](#vms-provisioning-state-is-not-valid-error-code-150019)**

## <a name="vms-provisioning-state-is-not-valid-error-code-150019"></a>VM 的预配状态无效（错误代码 150019）

若要在 VM 上启用复制，则预配状态应该是“成功”。  可以通过执行以下步骤来检查 VM 状态。

1. 在 Azure 门户中，从“所有服务”中选择“资源浏览器”。  
2. 展开“订阅”列表并选择你的订阅。 
3. 展开“资源组”并选择 VM 的资源组。 
4. 展开“资源”  列表并选择你的虚拟机
5. 在右侧的实例视图中检查“预配状态”  字段。

### <a name="fix-the-problem"></a>解决问题

- 如果“预配状态”为“失败”，请联系支持人员并提供详细信息以排除故障。  
- 如果“预配状态”为“正在更新”，则无法部署另一扩展。   检查 VM 上是否有任何正在进行的操作，等待它们完成，然后重试失败的 Site Recovery“启用复制”  作业。

## <a name="unable-to-select-target-virtual-network---network-selection-tab-is-grayed-out"></a>无法选择目标虚拟网络 - 网络选择选项卡灰显。

**原因 1：VM 附加到了已映射至“目标网络”的网络。**
- 如果源 VM 在某个虚拟网络中，并且同一虚拟网络中的另一个 VM 已映射到目标资源组中的某个网络，则默认将禁用网络选择下拉列表。

    ![Network_Selection_greyed_out](./media/site-recovery-azure-to-azure-troubleshoot/unabletoselectnw.png)

**原因 2：之前已使用 Azure Site Recovery 保护了 VM，并禁用了复制。**
- 禁用 VM 复制不会删除网络映射。 必须从保护 VM 的恢复服务保管库中删除网络映射。 <br />
    导航到恢复服务保管库并选择“Site Recovery 基础结构”>“网络映射”。 <br />
    ![Delete_NW_Mapping](./media/site-recovery-azure-to-azure-troubleshoot/delete_nw_mapping.png)
- 保护 VM 之后，可以在完成初始设置后更改灾难恢复设置期间配置的目标网络。 <br />
    ![Modify_NW_mapping](./media/site-recovery-azure-to-azure-troubleshoot/modify_nw_mapping.png)
- 请注意，更改网络映射会影响使用该特定网络映射的所有受保护 VM。

## <a name="comvolume-shadow-copy-service-error-error-code-151025"></a>COM+/卷影复制服务错误（错误代码 151025）

错误代码  | 可能的原因  | **建议**
--- | --- | ---
151025<br><br />**消息**：Site Recovery 扩展安装失败 | - 禁用了“COM + 系统应用程序”服务。<br /><br />- 禁用了“卷影复制”服务。| 将“COM + 系统应用程序”和“卷影复制”服务设置为自动或手动启动模式。

### <a name="fix-the-problem"></a>解决问题

可以打开“服务”控制台并确保“COM + 系统应用程序”和“卷影复制”的“启动类型”未设置为“已禁用”。
![com-error](./media/azure-to-azure-troubleshoot-errors/com-error.png)

## <a name="unsupported-managed-disk-size-error-code-150172"></a>不支持的托管磁盘大小（错误代码 150172）

错误代码  | 可能的原因  | **建议**
--- | --- | ---
150172<br><br />**消息**：无法为虚拟机启用保护，因为它的磁盘(DiskName)的大小为(DiskSize)，小于所支持的最小大小 1024 MB。 | - 磁盘小于支持的大小 (1024 MB)| 请确保磁盘大小在支持的大小范围内，然后重试该操作。

<a name="enable-protection-failed-as-device-name-mentioned-in-the-grub-configuration-instead-of-uuid-error-code-151126"></a>
## <a name="enable-protection-failed-as-device-name-mentioned-in-the-grub-configuration-instead-of-uuid-error-code-151126"></a>启用保护失败，因为 GRUB 配置中提到的设备名不是 UUID（错误代码 151126）

**可能的原因：** <br />
GRUB 配置文件（“/boot/grub/menu.lst”、“/boot/grub/grub.cfg”、“/boot/grub2/grub.cfg”或“/etc/default/grub”）可能包含参数“root”和“resume”的值作为实际设备名而非 UUID   。 Site Recovery 要求 UUID 方法，因为设备名可能会在 VM 重启时发生更改，由于故障转移时 VM 可能不会出现相同的名称，从而导致问题。 例如： <br />

- 以下行来自 GRUB 文件 /boot/grub2/grub.cfg  。 <br />
    *linux   /boot/vmlinuz-3.12.49-11-default **root=/dev/sda2**  ${extra_cmdline} **resume=/dev/sda1** splash=silent quiet showopts*

- 以下代码行摘自 GRUB 文件 **/boot/grub/menu.lst** <br />
    *kernel /boot/vmlinuz-3.0.101-63-default **root=/dev/sda2** **resume=/dev/sda1** splash=silent crashkernel=256M-:128M showopts vga=0x314*

如果发现上面的粗体字符串，GRUB 具有参数“root”和“resume”的实际设备名，而不是 UUID。

**如何修复：**<br />
设备名应替换为相应的 UUID。<br />

1. 通过执行命令“blkid \<设备名称>”查找设备的 UUID。 例如：<br />
    
    ```
    $ blkid /dev/sda1
    /dev/sda1: UUID="6f614b44-433b-431b-9ca1-4dd2f6f74f6b" TYPE="swap"   
    $ blkid /dev/sda2
    /dev/sda2: UUID="62927e85-f7ba-40bc-9993-cc1feeb191e4" TYPE="ext3"
    ```

1. 现在请将设备名替换为设备 UUID，格式类似于“root=UUID=\<UUID>”。 例如，对于上述在“/boot/grub2/grub.cfg”、“/boot/grub2/grub.cfg”或“/etc/default/grub”文件中提到的 root 和 resume 参数，如果将设备名称替换为 UUID，则文件中的行将类似于： <br />
    *kernel /boot/vmlinuz-3.0.101-63-default **root=UUID=62927e85-f7ba-40bc-9993-cc1feeb191e4** **resume=UUID=6f614b44-433b-431b-9ca1-4dd2f6f74f6b** splash=silent crashkernel=256M-:128M showopts vga=0x314*
1. 再次重启保护

## <a name="enable-protection-failed-as-device-mentioned-in-the-grub-configuration-doesnt-existerror-code-151124"></a>启用保护失败，因为 GRUB 配置中所述的设备不存在（错误代码 151124）
**可能的原因：** <br />
GRUB 配置文件 ("/boot/grub/menu.lst", "/boot/grub/grub.cfg", "/boot/grub2/grub.cfg" or "/etc/default/grub") 可能包含参数“rd.lvm.lv”或“rd_LVM_LV”，指示在启动时应发现的 LVM 设备。 如果这些 LVM 设备不存在，则受保护的系统本身不会启动，而是停滞在启动过程。 甚至在故障转移 VM 上也会出现相同的问题。 下面是几个示例：

几个示例： <br />

1. 以下代码行摘自 RHEL7 上的 GRUB 文件 **"/boot/grub2/grub.cfg"** 。 <br />
    *linux16 /vmlinuz-3.10.0-957.el7.x86_64 root=/dev/mapper/rhel_mup--rhel7u6-root ro crashkernel=128M\@64M **rd.lvm.lv=rootvg/root rd.lvm.lv=rootvg/swap** rhgb quiet LANG=en_US.UTF-8*<br />
    此处的突出显示部分指明，GRUB 必须在卷组“rootvg”中检测到名为 **“root”** 和 **“swap”** 的两个 LVM 设备。
1. 以下代码行摘自 RHEL7 上的 GRUB 文件 **"/etc/default/grub"** <br />
    *GRUB_CMDLINE_LINUX="crashkernel=auto **rd.lvm.lv=rootvg/root rd.lvm.lv=rootvg/swap** rhgb quiet"*<br />
    此处的突出显示部分指明，GRUB 必须在卷组“rootvg”中检测到名为 **“root”** 和 **“swap”** 的两个 LVM 设备。
1. 以下代码行摘自 RHEL6 上的 GRUB 文件 **"/boot/grub/menu.lst"** <br />
    *kernel /vmlinuz-2.6.32-754.el6.x86_64 ro root=UUID=36dd8b45-e90d-40d6-81ac-ad0d0725d69e rd_NO_LUKS LANG=en_US.UTF-8 rd_NO_MD SYSFONT=latarcyrheb-sun16 crashkernel=auto rd_LVM_LV=rootvg/lv_root  KEYBOARDTYPE=pc KEYTABLE=us rd_LVM_LV=rootvg/lv_swap rd_NO_DM rhgb quiet* <br />
    此处的突出显示部分指明，GRUB 必须在卷组“rootvg”中检测到名为 **“root”** 和 **“swap”** 的两个 LVM 设备。<br />

**如何修复：**<br />

如果 LVM 设备不存在，解决方法是创建该设备，或者从 GRUB 配置文件中删除该设备对应的参数，然后重试启用保护。 <br />

## <a name="site-recovery-mobility-service-update-completed-with-warnings--error-code-151083"></a>Site Recovery 移动服务更新完成但出现警告（错误代码 151083）
Site Recovery 移动服务有多个组件，其中一个称为筛选器驱动程序。 筛选器驱动程序只有在系统重启时才会加载到系统内存中。 每当 Site Recovery 移动服务更新涉及到筛选器驱动程序的更改时，我们都会更新计算机，但仍会发出警告，指出某些修复措施需要重新启动。 这意味着，仅当已加载新的筛选器驱动程序（只有在系统重新启动时才能发生）时，才能实现筛选器驱动程序的修复。<br />
**请注意**这只是一条警告，即使在新代理更新后，现有的复制也能保持正常工作。 可以选择在需要使用新筛选器驱动程序的时候重启，但如果不重启，则旧筛选器驱动程序仍可继续使用。 除了筛选器驱动程序以外，**在更新代理后，无需重新启动，移动服务中也能获得其他任何增强和修复。**  

## <a name="protection-couldnt-be-enabled-as-replica-managed-disk-diskname-replica-already-exists-without-expected-tags-in-the-target-resource-group-error-code-150161"></a>无法启用保护，因为副本托管磁盘“diskname-replica”已存在，但目标资源组中不包含预期的标记（错误代码 150161）

**原因：** 如果虚拟机过去受保护，而在禁止复制期间，某种原因导致副本磁盘未能清理，则可能会出现此错误。<br />
**如何修复：** 删除错误消息中提到的副本磁盘，然后再次重启失败的保护作业。

## <a name="next-steps"></a>后续步骤
[复制 Azure 虚拟机](site-recovery-replicate-azure-to-azure.md)

<!-- Update_Description: update meta properties, wording update, update link -->
