---
title: '注册并扫描 Power BI 租户 (预览版) '
description: 了解如何使用 Azure 监控范围门户注册和扫描 Power BI 租户。
author: viseshag
ms.author: viseshag
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: how-to
ms.date: 11/19/2020
ms.openlocfilehash: af394b68a943f4c89358a719c155606c264b9dc4
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/03/2020
ms.locfileid: "96551780"
---
# <a name="register-and-scan-a-power-bi-tenant-preview"></a>注册并扫描 Power BI 租户 (预览版) 

本文介绍如何使用 Azure 监控范围门户注册和扫描 Power BI 租户。

> [!Note]
> 如果监控范围实例和 Power BI 租户位于同一 Azure 租户中，则只能使用托管标识 (MSI) 身份验证来设置 Power BI 租户的扫描。 如果监控范围实例和 Power BI 租户位于不同的 Azure 租户中，则必须使用委派的身份验证进行身份验证，并且必须使用 PowerShell 来设置扫描。 请参阅 [使用 PowerShell 注册和扫描 Power BI](powershell-register-scan-power-bi.md)。

## <a name="create-a-security-group-for-permissions"></a>创建权限的安全组

若要设置身份验证，请创建一个安全组，并向其添加目录的托管标识。

1. 在 [Azure 门户](https://portal.azure.com)中，搜索 **Azure Active Directory**。
1. 在 Azure Active Directory 中创建一个新的安全组，方法是 [使用 Azure Active Directory 创建一个基本组并添加成员](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-groups-create-azure-portal)。

    > [!Tip]
    > 如果你已有要使用的安全组，则可以跳过此步骤。

1. 选择 " **安全性** " 作为 " **组类型**"。

    :::image type="content" source="./media/setup-power-bi-scan-PowerShell/security-group.png" alt-text="安全组类型":::

1. 将目录的托管标识添加到此安全组。 选择 " **成员**"，然后选择 " **+ 添加成员**"。

    :::image type="content" source="./media/setup-power-bi-scan-PowerShell/add-group-member.png" alt-text="将目录的托管实例添加到组。":::

1. 搜索你的目录并将其选中。

    :::image type="content" source="./media/setup-power-bi-scan-PowerShell/add-catalog-to-group-by-search.png" alt-text="通过搜索来添加目录":::

    你应该会看到一条成功通知，显示它已添加。

    :::image type="content" source="./media/setup-power-bi-scan-PowerShell/success-add-catalog-msi.png" alt-text="添加目录 MSI 成功":::

## <a name="associate-the-security-group-with-the-tenant"></a>将安全组与租户关联

1. 登录到 [Power BI 管理门户](https://app.powerbi.com/admin-portal/tenantSettings?allowServicePrincipalsUseReadAdminAPIsUI=1)。 将此功能标志追加到 URI：  `allowServicePrincipalsUseReadAdminAPIsUI=1` 。 此标志启用允许你关联安全组的功能。 例如，

    ```http
    https://app.powerbi.com/admin-portal/tenantSettings?allowServicePrincipalsUseReadAdminAPIsUI=1
    ```

    > [!Important]
    > 你需要成为 Power BI 管理员才能看到 "租户设置" 页。

1. 选择 "**开发人员设置**" "  >  **允许服务主体使用只读 Power BI api (预览版)**"。
1. 选择 " **特定安全组**"。

    :::image type="content" source="./media/setup-power-bi-scan-PowerShell/allow-service-principals-power-bi-admin.png" alt-text="显示如何允许服务主体获得只读 Power BI 管理 API 权限的图像":::

    > [!Caution]
    > 如果允许创建的安全组 (将数据目录托管标识作为成员) 使用只读 Power BI 管理 Api，则还允许该安全组访问此租户中所有 ) 项目的元数据 (例如仪表板和报表名称、所有者、说明等。 将元数据提取到 Azure 监控范围中后，监控范围的权限（而不是 Power BI 权限）确定可以查看该元数据的用户。

    > [!Note]
    > 你可以从开发人员设置中删除安全组，但不会从监控范围帐户中删除之前提取的元数据。 如果需要，可以单独删除它。

## <a name="register-your-power-bi-and-set-up-a-scan"></a>注册 Power BI 并设置扫描

现在，你已将目录权限指定为连接到 Power BI 租户的管理员 API，你可以从目录门户设置扫描。

首先，将一个特殊的功能标志添加到监控范围 URL 

1. 将以下字符串添加到监控范围实例的 uri 的末尾： `?feature.ext.catalog={"pbi":"true"}` 。 这会在目录中启用 Power BI 注册选项。

1. 选择 " **管理中心** " 图标。

    :::image type="content" source="media/setup-power-bi-scan-catalog-portal/management-center.png" alt-text="管理中心图标。":::

1. 然后，在 "**数据源**" 中选择 " **+ 新建**"。

    :::image type="content" source="media/setup-power-bi-scan-catalog-portal/data-sources.png" alt-text="&quot;新建数据源&quot; 按钮的图像":::

    选择 " **Power BI** " 作为数据源。

    :::image type="content" source="media/setup-power-bi-scan-catalog-portal/select-power-bi-data-source.png" alt-text="显示可供选择的数据源列表的图像":::

1. 为 Power BI 实例指定友好名称。

    :::image type="content" source="media/setup-power-bi-scan-catalog-portal/power-bi-friendly-name.png" alt-text="显示 Power BI 数据源-友好名称的图像":::

    名称长度必须介于3-63 个字符之间，并且必须仅包含字母、数字、下划线和连字符。  不允许使用空格。

    默认情况下，系统将查找同一 Azure 订阅中存在的 Power BI 租户。

    :::image type="content" source="media/setup-power-bi-scan-catalog-portal/power-bi-datasource-registered.png" alt-text="已注册 Power BI 数据源":::

1. 为扫描指定一个名称。 请注意，唯一支持的身份验证方法是 **托管标识**。

    :::image type="content" source="media/setup-power-bi-scan-catalog-portal/power-bi-scan-setup.png" alt-text="显示 Power BI 扫描设置的图像":::

    扫描名称长度必须介于3-63 个字符之间，并且必须仅包含字母、数字、下划线和连字符。  不允许使用空格。

1. 设置扫描触发器。 你的选项为 **每7天****一次**，**每隔30天**。

    :::image type="content" source="media/setup-power-bi-scan-catalog-portal/scan-trigger.png" alt-text="扫描触发器图像":::

1. **查看新扫描** 时，选择 "**保存并运行**" 以启动扫描。

    :::image type="content" source="media/setup-power-bi-scan-catalog-portal/save-run-power-bi-scan.png" alt-text="保存并运行 Power BI 屏幕图像":::

## <a name="next-steps"></a>后续步骤

若要了解如何使用 PowerShell cmdlet 注册和扫描 Power BI 租户，请参阅：
  
- [使用 PowerShell 注册和扫描 Power BI](powershell-register-scan-power-bi.md)
