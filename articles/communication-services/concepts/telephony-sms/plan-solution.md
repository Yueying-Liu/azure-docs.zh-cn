---
title: 规划 Azure 通信服务的电话服务和短信解决方案
titleSuffix: An Azure Communication Services concept document
description: 了解如何有效规划电话号码和电话服务的使用。
author: prakulka
manager: nmurav
services: azure-communication-services
ms.author: prakulka
ms.date: 10/05/2020
ms.topic: overview
ms.custom: references_regions
ms.service: azure-communication-services
ms.openlocfilehash: 0420360b70485e49dc6cd06dbeb19400c0f73ef5
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/15/2020
ms.locfileid: "92070344"
---
# <a name="plan-your-telephony-and-sms-solution"></a>规划电话服务和短信解决方案

[!INCLUDE [Public Preview Notice](../../includes/public-preview-include.md)]


借助 Azure 通信服务，可以使用电话号码通过公用电话交换网 (PSTN) 进行语音呼叫和发送短信。 在本文档中，我们将使用通信服务查看电话号码类型、配置选项和区域可用性，以规划电话服务和短信解决方案。

[!INCLUDE [Emergency Calling Notice](../../includes/emergency-calling-notice-include.md)]


## <a name="phone-number-types-in-azure-communication-services"></a>Azure 通信服务中的电话号码类型
 
通信服务提供两种电话号码：本地和免费。 

### <a name="local-numbers"></a>本地号码
本地（地理）号码是由美国本地区号组成的 10 位电话号码。 例如，`+1 (206) XXX-XXXX` 是区号为 `206` 的本地号码。 该区号已分配给西雅图市。 这些电话号码通常由个人和本地企业使用。 Azure 通信服务在美国提供本地号码。 这些号码可用于拨打电话，但不能发送短信。 

### <a name="toll-free-numbers"></a>免费号码
免费号码是具有不同区号的 10 位电话号码，可以从任何电话号码免费拨打。 例如，`+1 (800) XXX-XXXX` 是北美地区的免费号码。 这些电话号码通常用于客户服务。 Azure 通信服务在美国提供免费号码。 这些号码可用于拨打电话和发送短信。 免费号码不能由个人使用，只能分配给应用程序。

#### <a name="choosing-a-phone-number-type"></a>选择电话号码类型

如果你的电话号码将由应用程序使用（例如，代表你的服务拨打电话或发送短信），则可以选择免费或本地（地理）号码。 如果应用程序要发送短信和/或拨打电话，则可以选择免费号码。

如果你的电话号码将由某人（例如，你的呼叫应用程序的用户）使用，则必须使用本地（地理）电话号码。 

下表总结了这些电话号码类型： 

| 电话号码类型 | 示例                              | 上市国家/地区    | 电话号码功能 |常见用例                                                                                                     |
| ----------------- | ------------------------------------ | ----------------------- | ------------------------|------------------------------------------------------------------------------------------------------------------- |
| 本地（地理）        | +1（本地区号）XXX XX XX  | US                      | 通话（拨出） | 将电话号码分配给应用程序中的用户  |
| 免费电话         | +1（免费电话 *区号*）XXX XX XX | US                      | 通话（拨出）、短信（传入/传出）| 将电话号码分配给互动语音响应 (IVR) 系统/机器人、短信应用程序                                        |


## <a name="phone-number-features-in-azure-communication-services"></a>Azure 通信服务中的电话号码功能 

对于大多数电话号码，我们允许配置“按菜单点菜”功能。 你可以在 Azure 通信服务中租赁电话号码时选择这些功能。

你可以使用的功能取决于你进行操作时所在的国家/地区、你的用例以及所选电话号码类型。 出于法规要求，这些功能因国家/地区而异。 Azure 通信服务提供以下电话号码功能：

- **单向传出短信**：此选项允许你向用户发送短信。 这在通知和双因素身份验证方案中很有用。 
- **双向传入和传出短信**：此选项允许你使用电话号码向用户发送短信以及接收来自用户的短信。 这在客户服务方案中很有用。
- **单向外拨电话**：此选项允许你向用户拨打电话，并为服务拨出的电话配置呼叫方 ID。 这在客户服务和语音通知方案中很有用。

## <a name="countryregion-availability"></a>上市国家/地区

下表显示了你可以在何处获取不同类型的电话号码，以及与这些电话号码类型关联的拨入和拨出通话以及短信功能。

|号码类型| 号码获取地 | 呼叫目的地                                        | 呼叫源                                    |短信发送目的地       | 短信发送源 |
|-----------| ------------------ | ---------------------------------------------------  |-------------------------------------------------------|-----------------------|--------|
| 本地（地理）  | US                 | 美国、加拿大、英国、德国、法国 等等*| 美国、加拿大、英国、德国、法国 等等* |不可用| 不可用 |
| 免费电话 | US                 | US                                                   | US                                                    |US                | US |

\* 有关呼叫目的地和定价的更多详细信息，请参阅[定价页](../pricing.md)。

## <a name="azure-subscriptions-eligibility"></a>Azure 订阅资格

若要获取电话号码，需要使用 Azure 付费订阅。 无法在试用帐户上或通过 Azure 免费额度获取电话号码。 

目前，电话号码的可用性仅限于有美国帐单邮寄地址的 Azure 订阅。

## <a name="next-steps"></a>后续步骤

### <a name="quickstarts"></a>快速入门

- [获取电话号码](../../quickstarts/telephony-sms/get-phone-number.md)
- [拨打电话](../../quickstarts/voice-video-calling/calling-client-samples.md)
- [发送短信](../../quickstarts/telephony-sms/send.md)

### <a name="conceptual-documentation"></a>概念文档

- [语音和视频概念](../voice-video-calling/about-call-types.md)
- [呼叫流](../call-flows.md)
- [定价](../pricing.md)
