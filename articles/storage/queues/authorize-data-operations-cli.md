---
title: 选择如何使用 Azure CLI 向队列数据授予访问权限
titleSuffix: Azure Storage
description: 指定如何对 Azure CLI 的队列数据进行数据操作授权。 可以使用 Azure AD 凭据、帐户访问密钥或共享访问签名 (SAS) 令牌授权数据操作。
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 11/13/2020
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.custom: devx-track-azurecli
ms.openlocfilehash: e753f5b09b6cd03744ba8520c668a8227e56e8a1
ms.sourcegitcommit: 295db318df10f20ae4aa71b5b03f7fb6cba15fc3
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/15/2020
ms.locfileid: "94637337"
---
# <a name="choose-how-to-authorize-access-to-queue-data-with-azure-cli"></a>选择如何使用 Azure CLI 向队列数据授予访问权限

Azure 存储提供用于 Azure CLI 的扩展，使你能够指定要如何对队列数据进行操作授权。 可通过以下方式授权数据操作：

- 使用 Azure Active Directory (Azure AD) 安全主体。 Microsoft 建议使用 Azure AD 凭据以获得高级安全和易用性。
- 使用帐户访问密钥或共享访问签名 (SAS) 令牌。

## <a name="specify-how-data-operations-are-authorized"></a>指定数据操作的授权方式

用于读取和写入队列数据 Azure CLI 命令包括可选 `--auth-mode` 参数。 指定此参数可以指示数据操作的授权方式：

- 将 `--auth-mode` 参数设置为 `login` 可以使用 Azure AD 安全主体登录（建议）。
- 将 `--auth-mode` 参数设置为传统 `key` 值可以尝试检索用于授权的帐户访问密钥。 如果省略 `--auth-mode` 参数，则 Azure CLI 也会尝试检索访问密钥。

若要使用 `--auth-mode` 参数，请确保已安装 Azure CLI 2.0.46 或更高版本。 运行 `az --version` 以查看已安装版本。

> [!IMPORTANT]
> 如果省略 `--auth-mode` 参数或将其设置为 `key`，则 Azure CLI 会尝试使用帐户访问密钥进行授权。 在这种情况下，Microsoft 建议你在命令或 **AZURE_STORAGE_KEY** 环境变量中提供访问密钥。 有关环境变量的详细信息，请参阅标题为[为授权参数设置环境变量](#set-environment-variables-for-authorization-parameters)的部分。
>
> 如果不提供访问密钥，则 Azure CLI 会尝试调用 Azure 存储资源提供程序来检索每个操作的访问密钥。 执行多个需要调用资源提供程序的数据操作可能会导致发生限制。 有关资源提供程序限制的详细信息，请参阅 [Azure 存储资源提供程序的可伸缩性和性能目标](../common/scalability-targets-resource-provider.md)。

## <a name="authorize-with-azure-ad-credentials"></a>使用 Azure AD 凭据授权

使用 Azure AD 凭据登录 Azure CLI 时，会返回 OAuth 2.0 访问令牌。 Azure CLI 自动使用该令牌针对 Blob 或队列存储进行后续数据操作授权。 对于支持的操作，无需再通过命令传递帐户密钥或 SAS 令牌。

可以通过 Azure RBAC)  (Azure 基于角色的访问控制将数据排队到 Azure AD 安全主体。 有关 azure 存储空间中的 Azure 角色的详细信息，请参阅 [使用 AZURE RBAC 管理对 Azure 存储数据的访问权限](../common/storage-auth-aad-rbac-portal.md)。

### <a name="permissions-for-calling-data-operations"></a>用于调用数据操作的权限

队列数据操作支持 Azure 存储扩展。 可调用的操作取决于向 Azure AD 安全主体授予的权限，此安全主体用于登录 Azure CLI。 通过 Azure RBAC 分配对 Azure 存储队列的权限。 例如，如果分配了 **存储队列数据读取器** 角色，则可以运行脚本命令，以便从队列中读取数据。 如果分配了 " **存储队列数据参与者** " 角色，则可以运行脚本命令来读取、写入或删除队列及其包含的数据。

有关队列上的每个 Azure 存储操作所需的权限的详细信息，请参阅 [使用 OAuth 令牌调用存储操作](/rest/api/storageservices/authorize-with-azure-active-directory#call-storage-operations-with-oauth-tokens)。  

### <a name="example-authorize-an-operation-to-create-a-queue-with-azure-ad-credentials"></a>示例：授权操作以使用 Azure AD 凭据创建队列

下面的示例演示如何使用 Azure AD 凭据从 Azure CLI 创建队列。 若要创建队列，需要登录到 Azure CLI，并需要资源组和存储帐户。

1. 在创建队列之前，请将 [存储 Blob 数据参与者](../../role-based-access-control/built-in-roles.md#storage-queue-data-contributor) 角色分配给自己。 即使你是帐户所有者，也需要显式权限才能针对存储帐户执行数据操作。 有关分配 Azure 角色的详细信息，请参阅 [使用 Azure 门户分配 azure 角色以访问 blob 和队列数据](../common/storage-auth-aad-rbac-portal.md)。

    > [!IMPORTANT]
    > 传播 Azure 角色分配可能需要几分钟的时间。

1. 调用 [az storage queue create](/cli/azure/storage/queue#az-storage-queue-create) 命令，并将 `--auth-mode` 参数设置为， `login` 以使用 Azure AD 凭据创建队列。 请务必将尖括号中的占位符值替换为你自己的值：

    ```azurecli
    az storage queue create \
        --account-name <storage-account> \
        --name sample-queue \
        --auth-mode login
    ```

## <a name="authorize-with-the-account-access-key"></a>使用帐户访问密钥授权

如果你拥有帐户密钥，则可以调用任何 Azure 存储数据操作。 一般情况下，使用帐户密钥的安全性较低。 如果帐户密钥已透露，帐户中的所有数据也可能会透露。

下面的示例演示如何使用帐户访问密钥创建队列。 指定帐户密钥，并提供值为 `key` 的 `--auth-mode` 参数：

```azurecli
az storage queue create \
    --account-name <storage-account> \
    --name sample-queue \
    --account-key <key>
    --auth-mode key
```

## <a name="authorize-with-a-sas-token"></a>使用 SAS 令牌授权

如果你拥有 SAS 令牌，则可以调用 SAS 允许的数据操作。 下面的示例演示如何使用 SAS 令牌创建队列：

```azurecli
az storage queue create \
    --account-name <storage-account> \
    --name sample-queue \
    --sas-token <token>
```

## <a name="set-environment-variables-for-authorization-parameters"></a>为授权参数设置环境变量

可以在环境变量中指定授权参数，以免每次调用 Azure 存储数据操作时都要包含这些参数。 下表描述了可用的环境变量。

| 环境变量                  | 说明                                                                                                                                                                                                                                                                                                                                                                     |
|---------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|    AZURE_STORAGE_ACCOUNT              |    存储帐户名称。 此变量应与存储帐户密钥或 SAS 令牌结合使用。 如果该密钥和令牌都不存在，则 Azure CLI 会尝试使用经过身份验证的 Azure AD 帐户检索存储帐户访问密钥。 如果一次性执行大量的命令，可能会达到 Azure 存储资源提供程序的限制。 有关资源提供程序限制的详细信息，请参阅 [Azure 存储资源提供程序的可伸缩性和性能目标](../common/scalability-targets-resource-provider.md)。             |
|    AZURE_STORAGE_KEY                  |    存储帐户密钥。 此变量必须与存储帐户名称结合使用。                                                                                                                                                                                                                                                                          |
|    AZURE_STORAGE_CONNECTION_STRING    |    一个包含存储帐户密钥或 SAS 令牌的连接字符串。 此变量必须与存储帐户名称结合使用。                                                                                                                                                                                                                       |
|    AZURE_STORAGE_SAS_TOKEN            |    共享访问签名 (SAS) 令牌。 此变量必须与存储帐户名称结合使用。                                                                                                                                                                                                                                                            |
|    AZURE_STORAGE_AUTH_MODE            |    用于运行命令的授权模式。 允许的值为 `login`（建议）或 `key`。 如果指定 `login`，Azure CLI 将使用 Azure AD 凭据来授权数据操作。 如果指定传统 `key` 模式，Azure CLI 会尝试查询帐户访问密钥，并使用该密钥为命令授权。    |

## <a name="next-steps"></a>后续步骤

- [使用 Azure CLI 分配一个可以访问 blob 和队列数据的 Azure 角色](../common/storage-auth-aad-rbac-cli.md)
- [使用 Azure 资源托管标识授予对 Blob 和队列数据的访问权限](../common/storage-auth-aad-msi.md)