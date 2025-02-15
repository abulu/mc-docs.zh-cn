---
title: 快速入门 - 在 Azure 中的 Linux 虚拟机上安装 Ansible | Azure
description: 本快速入门介绍如何在 Ubuntu、CentOS 和 SLES 上安装和配置 Ansible 以管理 Azure 资源
keywords: ansible, azure, devops, bash, cloudshell, playbook, bash
ms.topic: quickstart
ms.service: ansible
author: rockboyfor
manager: digimobile
ms.author: v-yeche
origin.date: 04/30/2019
ms.date: 07/01/2019
ms.openlocfilehash: 928010f5fd52c33b7c860a23a7d6a038b1595e22
ms.sourcegitcommit: 5191c30e72cbbfc65a27af7b6251f7e076ba9c88
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/05/2019
ms.locfileid: "67570336"
---
<!-- Verify successfully-->
# <a name="quickstart-install-ansible-on-linux-virtual-machines-in-azure"></a>快速入门：在 Azure 中的 Linux 虚拟机上安装 Ansible

使用 Ansible 可以在环境中自动部署和配置资源。 本文介绍如何为某些最常用的 Linux 分发版配置 Ansible。 若要在其他分发版中安装 Ansible，请调整适用于特定平台的安装包。 

## <a name="prerequisites"></a>先决条件

[!INCLUDE [open-source-devops-prereqs-azure-sub.md](../../../includes/open-source-devops-prereqs-azure-subscription.md)]
[!INCLUDE [open-source-devops-prereqs-create-sp.md](../../../includes/open-source-devops-prereqs-create-service-principal.md)]
- **对 Linux 或 Linux 虚拟机的访问权限** - 如果没有 Linux 计算机，请创建 [Linux 虚拟机](/virtual-network/quick-create-cli)。

## <a name="install-ansible-on-an-azure-linux-virtual-machine"></a>在 Azure Linux 虚拟机上安装 Ansible

登录到 Linux 计算机，然后选择下述发行版之一，了解安装 Ansible 的步骤：

- [CentOS 7.4](#centos-74)
- [Ubuntu 16.04 LTS](#ubuntu1604-lts)
- [SLES 12 SP2](#sles-12-sp2)

<a name="centos-74"></a>
### <a name="centos-74"></a>CentOS 7.4

在本部分，我们将 CentOS 配置为使用 Ansible。

1. 打开终端窗口。

1. 输入以下命令，为 Azure Python SDK 模块安装所需的包：

    ```bash
    sudo yum check-update; sudo yum install -y gcc libffi-devel python-devel openssl-devel epel-release
    sudo yum install -y python-pip python-wheel
    ```

1. 输入以下命令安装所需的 Ansible 包：

    ```bash
    sudo pip install ansible[azure]
    ```

1. [创建 Azure 凭据](#create-azure-credentials)。

<a name="ubuntu1604-lts"></a>
### <a name="ubuntu-1604-lts"></a>Ubuntu 16.04 LTS

在本部分，我们将 Ubuntu 配置为使用 Ansible。

1. 打开终端窗口。

1. 输入以下命令，为 Azure Python SDK 模块安装所需的包：

    ```bash
    sudo apt-get update && sudo apt-get install -y libssl-dev libffi-dev python-dev python-pip
    ```

1. 输入以下命令安装所需的 Ansible 包：

    ```bash
    sudo pip install ansible[azure]
    ```

1. [创建 Azure 凭据](#create-azure-credentials)。

<a name="sles-12-sp2"></a>
### <a name="sles-12-sp2"></a>SLES 12 SP2

在本部分，我们将 SLES 配置为使用 Ansible。

1. 打开终端窗口。

1. 输入以下命令，为 Azure Python SDK 模块安装所需的包：

    ```bash
    sudo zypper refresh && sudo zypper --non-interactive install gcc libffi-devel-gcc5 make \
        python-devel libopenssl-devel libtool python-pip python-setuptools
    ```

1. 输入以下命令安装所需的 Ansible 包：

    ```bash
    sudo pip install ansible[azure]
    ```

1. 输入以下命令以删除有冲突的 Python 加密包：

    ```bash
    sudo pip uninstall -y cryptography
    ```

1. [创建 Azure 凭据](#create-azure-credentials)。

## <a name="create-azure-credentials"></a>创建 Azure 凭据

需要以下信息才能配置 Ansible 凭据：

* Azure 订阅 ID 
* 服务主体值

如果使用 Ansible Tower 或 Jenkins，请将服务主体值声明为环境变量。

使用以下方法之一配置 Ansible 凭据：

- [创建 Ansible 凭据文件](#file-credentials)
- [使用 Ansible 环境变量](#env-credentials)

<a name="file-credentials"></a>
### <a name="span-idfile-credentials-create-ansible-credentials-file"></a><span id="file-credentials"/>创建 Ansible 凭据文件

在本部分，我们将创建一个本地凭据文件，以便向 Ansible 提供凭据。 

有关定义 Ansible 凭据的详细信息，请参阅[为 Azure 模块提供凭据](https://docs.ansible.com/ansible/guide_azure.html#providing-credentials-to-azure-modules)。

1. 对于开发环境，请主机虚拟机上创建名为 `credentials` 的文件：

    ```bash
    mkdir ~/.azure
    vi ~/.azure/credentials
    ```

1. 将以下代码行插入到该文件中。 请将占位符替换为服务主体值。

    <!--MOONCAKE CUSTOMIZE ON cloud_environment=AzureChinaCloud -->

    ```bash
    [default]
    subscription_id=<your-subscription_id>
    client_id=<security-principal-appid>
    secret=<security-principal-password>
    tenant=<security-principal-tenant>
    cloud_environment=AzureChinaCloud
    ```

    <!--MOONCAKE CUSTOMIZE ON cloud_environment=AzureChinaCloud -->

1. 保存并关闭该文件。

<a name="env-credentials"></a>
### <a name="span-idenv-credentialsuse-ansible-environment-variables"></a><span id="env-credentials"/>使用 Ansible 环境变量

在本部分，我们将导出服务主体值以配置 Ansible 凭据。

1. 打开终端窗口。

1. 导出服务主体值：

    <!--MOONCAKE CUSTOMIZE ON export AZURE_CLOUD_ENVIRONMENT=AzureChinaCloud-->

    ```bash
    export AZURE_SUBSCRIPTION_ID=<your-subscription_id>
    export AZURE_CLIENT_ID=<security-principal-appid>
    export AZURE_SECRET=<security-principal-password>
    export AZURE_TENANT=<security-principal-tenant>
    export AZURE_CLOUD_ENVIRONMENT=AzureChinaCloud
    ```

    <!--MOONCAKE CUSTOMIZE ON export AZURE_CLOUD_ENVIRONMENT=AzureChinaCloud-->

## <a name="verify-the-configuration"></a>验证配置

若要验证配置是否成功，请使用 Ansible 创建一个 Azure 资源组。

[!INCLUDE [create-resource-group-with-ansible.md](../../../includes/ansible-snippet-create-resource-group.md)]

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"] 
> [快速入门：使用 Ansible 在 Azure 中配置 Linux 虚拟机](./ansible-create-vm.md)

<!-- Update_Description: wording update, update link -->