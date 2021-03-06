---
title: Azure 启动诊断
description: Azure 启动诊断和托管启动诊断概述
services: virtual-machines
ms.service: virtual-machines
author: mimckitt
ms.author: mimckitt
ms.topic: conceptual
ms.date: 11/06/2020
ms.openlocfilehash: 408ba76c44d1161a4b91ccc037721796c7b94661
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/02/2020
ms.locfileid: "96500744"
---
# <a name="azure-boot-diagnostics"></a>Azure 启动诊断

启动诊断是 Azure 虚拟机 (VM) 的一项调试功能，可用于诊断 VM 启动故障。 启动诊断使用户能够通过收集串行日志信息和屏幕截图来观察其 VM 在启动时的状态。

## <a name="boot-diagnostics-storage-account"></a>启动诊断存储帐户
在 Azure 门户中创建 VM 时，默认情况下启用启动诊断。 建议的启动诊断体验是使用托管存储帐户，因为它会在创建 Azure VM 时产生显著的性能改进。 这是因为将要使用 Azure 托管存储帐户，从而节省了创建新的用户存储帐户来存储启动诊断数据所需的时间。

另一种启动诊断体验是使用用户管理的存储帐户。 用户可以创建新的存储帐户，也可以使用现有的存储帐户。 

> [!IMPORTANT]
> 启动诊断数据 blob（包括日志和快照映像）存储在托管存储帐户中。 将仅对 blob 使用的 GiB 向客户收费，而不是根据磁盘的预配大小收费。 快照计量将用于托管存储帐户的计费。 由于托管帐户是在标准 LRS 或标准 ZRS 上创建的，因此，仅针对诊断数据 blob 的大小按每月 $0.05/GB 向客户收费。 有关此定价的详细信息，请参阅[托管磁盘定价](https://azure.microsoft.com/pricing/details/managed-disks/)。 客户将看到这一费用与其 VM 资源 URI 相关联。 

## <a name="boot-diagnostics-view"></a>启动诊断视图
虚拟机边栏选项卡中的启动诊断选项位于 Azure 门户的“支持和故障排除”部分。 选择启动诊断会显示屏幕截图和串行日志信息。 串行日志包含内核消息，屏幕快照是 VM 当前状态的快照。 Windows 或 Linux 会根据 VM 是否正在运行来确定预期的屏幕快照会是什么样子。 Windows 用户会看到桌面背景，Linux 用户会看到登录提示。

:::image type="content" source="./media/boot-diagnostics/boot-diagnostics-linux.png" alt-text="Linux 启动诊断的屏幕截图":::
:::image type="content" source="./media/boot-diagnostics/boot-diagnostics-windows.png" alt-text="Windows 启动诊断的屏幕截图":::

## <a name="enable-managed-boot-diagnostics"></a>启用托管启动诊断 
可以通过 Azure 门户、CLI 和 ARM 模板启用托管启动诊断。 尚不支持通过 PowerShell 启用。 

### <a name="enable-managed-boot-diagnostics-using-the-azure-portal"></a>使用 Azure 门户启用托管启动诊断
在 Azure 门户中创建 VM 时，默认设置为使用托管存储帐户启用启动诊断。 若要查看此视图，请在创建 VM 的过程中导航到 " *管理* " 选项卡。 

:::image type="content" source="./media/boot-diagnostics/boot-diagnostics-enable-portal.png" alt-text="在创建 VM 期间启用托管启动诊断的屏幕截图。":::

### <a name="enable-managed-boot-diagnostics-using-cli"></a>使用 CLI 启用托管启动诊断
Azure CLI 2.12.0 和更高版本中支持具有托管存储帐户的启动诊断。 如果不输入存储帐户的名称或 URI，将使用托管帐户。 有关详细信息和代码示例，请参阅 [用于启动诊断的 CLI 文档](/cli/azure/vm/boot-diagnostics?preserve-view=true&view=azure-cli-latest)。

### <a name="enable-managed-boot-diagnostics-using-azure-resource-manager-arm-templates"></a>使用 Azure 资源管理器 (ARM) 模板启用托管启动诊断
API 版本2020-06-01 后的所有内容都支持托管启动诊断。 有关详细信息，请参阅 [启动诊断实例视图](/rest/api/compute/virtualmachines/createorupdate#bootdiagnostics)。

```ARM Template
            "name": "[parameters('virtualMachineName')]",
            "type": "Microsoft.Compute/virtualMachines",
            "apiVersion": "2020-06-01",
            "location": "[parameters('location')]",
            "dependsOn": [
                "[concat('Microsoft.Network/networkInterfaces/', parameters('networkInterfaceName'))]"
            ],
            "properties": {
                "hardwareProfile": {
                    "vmSize": "[parameters('virtualMachineSize')]"
                },
                "storageProfile": {
                    "osDisk": {
                        "createOption": "fromImage",
                        "managedDisk": {
                            "storageAccountType": "[parameters('osDiskType')]"
                        }
                    },
                    "imageReference": {
                        "publisher": "Canonical",
                        "offer": "UbuntuServer",
                        "sku": "18.04-LTS",
                        "version": "latest"
                    }
                },
                "networkProfile": {
                    "networkInterfaces": [
                        {
                            "id": "[resourceId('Microsoft.Network/networkInterfaces', parameters('networkInterfaceName'))]"
                        }
                    ]
                },
                "osProfile": {
                    "computerName": "[parameters('virtualMachineComputerName')]",
                    "adminUsername": "[parameters('adminUsername')]",
                    "linuxConfiguration": {
                        "disablePasswordAuthentication": true
                    }
                },
                "diagnosticsProfile": {
                    "bootDiagnostics": {
                        "enabled": true
                    }
                }
            }
        }
    ],

```

## <a name="limitations"></a>限制
- 启动诊断仅适用于 Azure 资源管理器 VM。
- 托管启动诊断不支持使用非托管 OS 磁盘的虚拟机。
- 启动诊断不支持高级存储帐户。如果将高级存储帐户用于启动诊断，则在启动 VM 时，用户会收到 `StorageAccountTypeNotSupported` 错误。 
- 资源管理器 API 版本“2020-06-01”及更高版本支持托管存储帐户。
- Azure 串行控制台当前与用于启动诊断的托管存储帐户不兼容。 详细了解 [Azure 串行控制台](./troubleshooting/serial-console-overview.md)。
- 门户仅支持将启动诊断用于单个实例 Vm 的托管存储帐户。

## <a name="next-steps"></a>后续步骤

了解有关 [Azure 串行控制台](./troubleshooting/serial-console-overview.md) 的详细信息，以及如何使用启动诊断对 [azure 中的虚拟机进行故障排除](./troubleshooting/boot-diagnostics.md)。