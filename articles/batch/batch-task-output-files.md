---
title: 使用 Batch 服务 API 将作业和任务输出持久保存到 Azure 存储 - Azure Batch | Docs
description: 了解如何使用 Batch 服务 API 将 Batch 任务和作业输出持久保存到 Azure 存储。
services: batch
author: lingliw
manager: digimobile
editor: ''
ms.service: batch
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: ''
ms.workload: big-compute
ms.date: 03/05/2019
ms.author: v-lingwu
ms.openlocfilehash: 0666cf89064bf595136674ce0da538ee51977b90
ms.sourcegitcommit: f4351979a313ac7b5700deab684d1153ae51d725
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/12/2019
ms.locfileid: "67845249"
---
# <a name="persist-task-data-to-azure-storage-with-the-batch-service-api"></a>使用 Batch 服务 API 将任务数据持久保存到 Azure 存储

[!INCLUDE [batch-task-output-include](../../includes/batch-task-output-include.md)]

Batch 服务 API 支持将在具有虚拟机配置的池上运行的任务和作业管理器任务的输出数据保存到 Azure 存储。 添加任务时，可以在 Azure 存储中指定一个容器，作为该任务的输出目标。 然后，Batch 服务会在任务完成时，将任何输出数据写入该容器。

使用 Batch 服务 API 来持久保存任务输出时，一大优势是不需修复任务正在运行的应用程序， 只需对客户端应用程序进行几处修改，即可从创建任务的同一代码内部保存任务的输出。

## <a name="when-do-i-use-the-batch-service-api-to-persist-task-output"></a>何时使用 Batch 服务 API 来持久保存任务输出？

Azure Batch 提供多种持久保存任务输出的方式。 使用 Batch 服务 API 是一种便捷的方式，最适合以下情形：

- 需要在不修改任务正在运行的应用程序的情况下，通过编写代码从客户端应用程序持久保存任务输出。
- 需要将 Batch 任务和作业管理器任务的输出持久保存在使用虚拟机配置创建的池中。
- 需要将输出持久保存到使用任意名称的 Azure 存储容器。
- 需要将输出持久保存到根据 [Batch 文件约定标准](https://github.com/Azure/azure-sdk-for-net/tree/psSdkJson6/src/SDKs/Batch/Support/FileConventions#conventions)命名的 Azure 存储容器。 

如果你的情况与上面不同，可能需要考虑不同的方法。 例如，Batch 服务 API 目前不支持在任务正运行时将输出流式传输到 Azure 存储。 若要流式传输输出，请考虑使用适用于 .NET 的 Batch 文件约定库。 对于其他语言，需实现你自己的解决方案。 有关保存任务输出的其他选项的详细信息，请参阅[将作业和任务输出保存到 Azure 存储](batch-task-output.md)。 

## <a name="create-a-container-in-azure-storage"></a>在 Azure 存储中创建容器

若要将任务输出持久保存到 Azure 存储，需创建一个容器，让其充当输出文件的目标。 请在运行任务之前（最好是在提交作业之前）创建该容器。 若要创建容器，请使用适当的 Azure 存储客户端库或 SDK。 有关 Azure 存储 API 的详细信息，请参阅 [Azure 存储文档](/storage/)。

例如，若要通过 C# 来编写应用程序，请使用[适用于 .NET 的 Azure 存储客户端库](https://www.nuget.org/packages/WindowsAzure.Storage/)。 以下示例说明如何创建容器：

```csharp
CloudBlobContainer container = storageAccount.CreateCloudBlobClient().GetContainerReference(containerName);
await container.CreateIfNotExists();
```

## <a name="get-a-shared-access-signature-for-the-container"></a>获取容器的共享访问签名

创建容器以后，请获取对容器具有写访问权限的共享访问签名 (SAS)。 SAS 提供对容器的委托访问权限。 SAS 通过指定一组权限和指定时间间隔授予访问权限。 Batch 服务需要具有写权限的 SAS，以便将任务输出写入容器。 有关 SAS 的详细信息，请参阅[在 Azure 存储中使用共享访问签名 \(SAS\)](../storage/common/storage-dotnet-shared-access-signature-part-1.md)。

使用 Azure 存储 API 获取 SAS 时，该 API 会返回 SAS 令牌字符串。 此令牌字符串包含 SAS 的所有参数，其中包括权限以及 SAS 生效的时间间隔。 若要使用 SAS 来访问 Azure 存储中的容器，需将 SAS 令牌字符串追加到资源 URI。 可以联合使用 URI 与追加的 SAS 令牌对 Azure 存储进行经身份验证的访问。

以下示例演示如何获取容器的只写 SAS 令牌字符串，然后将 SAS 追加到容器 URI：

```csharp
string containerSasToken = container.GetSharedAccessSignature(new SharedAccessBlobPolicy()
{
    SharedAccessExpiryTime = DateTimeOffset.UtcNow.AddDays(1),
    Permissions = SharedAccessBlobPermissions.Write
});

string containerSasUrl = container.Uri.AbsoluteUri + containerSasToken; 
```

## <a name="specify-output-files-for-task-output"></a>指定任务输出的输出文件

若要指定任务的输出文件，请在创建任务时创建 [OutputFile](https://docs.azure.cn/zh-cn/dotnet/api/microsoft.azure.batch.outputfile) 对象的集合，然后将其分配给 [CloudTask.OutputFiles](https://docs.azure.cn/zh-cn/dotnet/api/microsoft.azure.batch.cloudtask.outputfiles#Microsoft_Azure_Batch_CloudTask_OutputFiles) 属性。 

以下 C# 代码示例创建一个将随机数写入名为 `output.txt` 的文件的任务。 该示例创建一个输出文件，以便将 `output.txt` 写入容器。 对于符合文件模式 `std*.txt` 的日志文件（例如 `stdout.txt` 和 `stderr.txt`），该示例也创建输出文件。  容器 URL 需要此前为容器创建的 SAS。 Batch 服务使用 SAS 来验证容器访问权限：

```csharp
new CloudTask(taskId, "cmd /v:ON /c \"echo off && set && (FOR /L %i IN (1,1,100000) DO (ECHO !RANDOM!)) > output.txt\"")
{
    OutputFiles = new List<OutputFile>
    {
        new OutputFile(
            filePattern: @"..\std*.txt",
            destination: new OutputFileDestination(
         new OutputFileBlobContainerDestination(
                    containerUrl: containerSasUrl,
                    path: taskId)),
            uploadOptions: new OutputFileUploadOptions(
            uploadCondition: OutputFileUploadCondition.TaskCompletion)),
        new OutputFile(
            filePattern: @"output.txt",
            destination: 
         new OutputFileDestination(new OutputFileBlobContainerDestination(
                    containerUrl: containerSasUrl,
                    path: taskId + @"\output.txt")),
            uploadOptions: new OutputFileUploadOptions(
            uploadCondition: OutputFileUploadCondition.TaskCompletion)),
}
```

### <a name="specify-a-file-pattern-for-matching"></a>指定要匹配的文件模式

指定输出文件时，可以使用 [OutputFile.FilePattern](https://docs.azure.cn/zh-cn/dotnet/api/microsoft.azure.batch.outputfile.filepattern#Microsoft_Azure_Batch_OutputFile_FilePattern) 属性来指定要匹配的文件模式。 在通过任务创建的文件中，文件模式匹配的可能有零个文件、一个文件或一组文件。

 FilePattern 属性支持文件系统通配符，例如 `*`（适用于非递归匹配）和 `**`（适用于递归匹配）。 例如，上面的代码示例指定以非递归方式与 `std*.txt` 匹配的文件模式： 

`filePattern: @"..\std*.txt"`

若要上传单个文件，请在不使用通配符的情况下指定文件模式。 例如，上面的代码示例指定与 `output.txt` 匹配的文件模式：

`filePattern: @"output.txt"`

### <a name="specify-an-upload-condition"></a>指定上传条件

[OutputFileUploadOptions.UploadCondition](https://docs.azure.cn/zh-cn/dotnet/api/microsoft.azure.batch.outputfileuploadoptions.uploadcondition#Microsoft_Azure_Batch_OutputFileUploadOptions_UploadCondition) 属性允许对输出文件进行条件性上传。 常见方案是在任务成功时上传一组文件，失败时上传另一组文件。 例如，可以只在任务失败且退出时的退出代码非零的情况下，才上传详细的日志文件。 同样，可以只在任务成功的情况下，才上传结果文件，因为这些文件在任务失败时可能会缺失或不完整。

上面的代码示例将 UploadCondition 属性设置为 TaskCompletion。   该设置指定在任务完成后上传文件，不管退出代码的值如何。 

`uploadCondition: OutputFileUploadCondition.TaskCompletion`

有关其他设置，请参阅 [OutputFileUploadCondition](https://docs.azure.cn/zh-cn/dotnet/api/microsoft.azure.batch.common.outputfileuploadcondition) 枚举。

### <a name="disambiguate-files-with-the-same-name"></a>区分名称相同的文件

作业中的任务可能生成名称相同的文件。 例如，系统会为在作业中运行的每个任务创建 `stdout.txt` 和 `stderr.txt`。 由于每个任务在自身上下文中运行，这些文件在节点的文件系统中并不发生冲突。 但是，将多个任务的文件上传到共享容器时，需区分名称相同的文件。

[OutputFileBlobContainerDestination.Path](https://docs.azure.cn/zh-cn/dotnet/api/microsoft.azure.batch.outputfileblobcontainerdestination.path#Microsoft_Azure_Batch_OutputFileBlobContainerDestination_Path) 属性指定输出文件的目标 Blob 或虚拟目录。 可以使用  Path 属性来命名 Blob 或虚拟目录，使名称相同的输出文件在 Azure 存储中具有唯一的名称。 在路径中使用任务 ID 可以很好地确保名称的唯一性，并且可以轻松地标识文件。

如果将  FilePattern 属性设置为通配符表达式，则会将符合模式的所有文件上传到通过 Path 属性指定的虚拟目录。  例如，如果容器为 `mycontainer`，任务 ID 为 `mytask`，文件模式为 `..\std*.txt`，则 Azure 存储中输出文件的绝对 URI 将类似于：

```
https://myaccount.blob.core.chinacloudapi.cn/mycontainer/mytask/stderr.txt
https://myaccount.blob.core.chinacloudapi.cn/mycontainer/mytask/stdout.txt
```

如果将  FilePattern 属性设置为与单个文件名称匹配（即不含任何通配符），则  Path 属性的值指定完全限定的 Blob 名称。 如果预计命名会与多个任务的单个文件冲突，则请包括虚拟目录的名称，将其作为文件名称的一部分，以便区分这些文件。 例如，将  Path 属性设置为包括任务 ID、分隔符（通常为正斜杠）和文件名称：

`path: taskId + @"/output.txt"`

一组任务的输出文件的绝对 URI 将类似于：

```
https://myaccount.blob.core.chinacloudapi.cn/mycontainer/task1/output.txt
https://myaccount.blob.core.chinacloudapi.cn/mycontainer/task2/output.txt
```

有关 Azure 存储中虚拟目录的详细信息，请参阅[列出容器中的 Blob](../storage/blobs/storage-quickstart-blobs-dotnet.md#list-the-blobs-in-a-container)。

## <a name="diagnose-file-upload-errors"></a>诊断文件上传错误

如果上传输出文件到 Azure 存储失败，则任务会转为“已完成”状态，  并会设置 [TaskExecutionInformation.FailureInformation](https://docs.azure.cn/zh-cn/dotnet/api/microsoft.azure.batch.taskexecutioninformation.failureinformation#Microsoft_Azure_Batch_TaskExecutionInformation_FailureInformation) 属性。 通过检查  FailureInformation 属性来确定所发生的具体错误。 例如，下面是在找不到容器的情况下，在文件上传时发生的错误： 

```
Category: UserError
Code: FileUploadContainerNotFound
Message: One of the specified Azure container(s) was not found while attempting to upload an output file
```

每次上传文件时，Batch 都会向计算节点写入两个日志文件，即 `fileuploadout.txt` 和 `fileuploaderr.txt`。 可以检查这些日志文件，详细了解具体的故障。 如果从未尝试过上传文件（例如，因任务本身无法运行而导致这种情况），则这些日志文件不会存在。

## <a name="diagnose-file-upload-performance"></a>诊断文件上传性能

`fileuploadout.txt` 文件记录了上传进度。 可以通过检查此文件来详细了解文件上传花费的时间。 请记住，有许多因素会影响上传性能，其中包括：节点大小、上传时节点上的其他活动、目标容器与 Batch 池是否位于同一区域、同时有多少节点在向存储帐户上传数据，等等。

## <a name="use-the-batch-service-api-with-the-batch-file-conventions-standard"></a>将 Batch 服务 API 与 Batch 文件约定标准配合使用

使用 Batch 服务 API 来持久保存任务输出时，可以随意命名目标容器和 Blob。 也可选择按 [Batch 文件约定标准](https://github.com/Azure/azure-sdk-for-net/tree/psSdkJson6/src/SDKs/Batch/Support/FileConventions#conventions)来命名。 如果给定的输出文件取决于作业和任务的名称，则文件约定标准决定了 Azure 存储中目标容器和 Blob 的名称。 如果使用文件约定标准来命名输出文件，则可在 [Azure 门户](https://portal.azure.cn)中查看输出文件。

如果是在 C# 中进行开发，则可使用[适用于 .NET 的 Batch 文件约定库](https://www.nuget.org/packages/Microsoft.Azure.Batch.Conventions.Files)中内置的方法。 该库为你创建了正确命名的容器和 Blob 路径。 例如，可以通过调用 API 来获取容器的正确名称（基于作业名称）：

```csharp
string containerName = job.OutputStorageContainerName();
```

可以使用 [CloudJobExtensions.GetOutputStorageContainerUrl](https://docs.azure.cn/zh-cn/dotnet/api/microsoft.azure.batch.conventions.files.cloudjobextensions.getoutputstoragecontainerurl) 方法，以便返回用于向容器写入数据的共享访问签名 (SAS) URL。 然后即可将该 SAS 传递给 [OutputFileBlobContainerDestination](https://docs.azure.cn/zh-cn/dotnet/api/microsoft.azure.batch.outputfileblobcontainerdestination) 构造函数。

如果使用 C# 之外的语言进行开发，则需自行实现文件约定标准。

## <a name="code-sample"></a>代码示例

GitHub 上的 [PersistOutputs][github_persistoutputs] sample project is one of the [Azure Batch code samples][github_samples]。 此 Visual Studio 解决方案演示如何使用适用于 .NET 的 Batch 客户端库将任务输出持久保存到持久性存储。 若要运行该示例，请遵循以下步骤：

1. 在 **Visual Studio 2019** 中打开该项目。
2. 将 Batch 和存储**帐户凭据**添加到 Microsoft.Azure.Batch.Samples.Common 项目中的 **AccountSettings.settings**。
3. **生成**（但不要运行）该解决方案。 根据提示还原所有 NuGet 包。
4. 使用 Azure 门户上传 **PersistOutputsTask** 的[应用程序包](batch-application-packages.md)。 在 .zip 包中包含 `PersistOutputsTask.exe` 及其依赖程序集，将应用程序 ID 设置为“PersistOutputsTask”，将应用程序包版本设置为“1.0”。
5. **启动**（运行）**PersistOutputs** 项目。
6. 当系统提示你选择用于运行示例的持久性技术时，请输入  2，以便运行示例，使用 Batch 服务 API 来持久保存任务输出。
7. 根据需要再次运行示例，输入  3，以便通过 Batch 服务 API 来持久保存输出，并根据文件约定标准来命名目标容器和 Blob 路径。

## <a name="next-steps"></a>后续步骤

- 有关使用适用于 .NET 的文件约定库保存任务输出的详细信息，请参阅[使用适用于 .NET 的 Batch 文件约定库将作业和任务数据保存到 Azure 存储](batch-task-output-file-conventions.md)。
- 若要了解在 Azure Batch 中持久保存输出数据的其他方法，请参阅[将作业和任务输出持久保存到 Azure 存储](batch-task-output.md)。

[github_persistoutputs]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/PersistOutputs
[github_samples]: https://github.com/Azure/azure-batch-samples

<!-- Update_Description: link update -->