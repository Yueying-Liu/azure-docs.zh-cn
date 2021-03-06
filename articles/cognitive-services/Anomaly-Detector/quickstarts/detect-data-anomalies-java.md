---
title: 快速入门：使用异常检测器 REST API 和 Java 检测时序数据的异常
titleSuffix: Azure Cognitive Services
description: 了解如何使用异常检测器 API 以批或流数据的形式检测数据系列的异常。
services: cognitive-services
author: mrbullwinkle
manager: nitinme
ms.service: cognitive-services
ms.subservice: anomaly-detector
ms.topic: quickstart
ms.date: 09/03/2020
ms.custom: devx-track-java
ms.author: mbullwin
ms.openlocfilehash: e23e9016b53a1d9f99809d792d710f865918a098
ms.sourcegitcommit: 2c586a0fbec6968205f3dc2af20e89e01f1b74b5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/14/2020
ms.locfileid: "92019675"
---
# <a name="quickstart-detect-anomalies-in-your-time-series-data-using-the-anomaly-detector-rest-api-and-java"></a>快速入门：使用异常检测器 REST API 和 Java 检测时序数据的异常

参考本快速入门可以开始使用异常检测器 API 的两种检测模式来检测时序数据的异常。 此 Java 应用程序发送包含 JSON 格式的时序数据的 API 请求，并获取响应。

| API 请求                                        | 应用程序输出                                                                                                                         |
|----------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------|
| 以批的形式检测异常                        | JSON 响应包含时序数据中每个数据点的异常状态（和其他数据），以及检测到的任何异常所在的位置。 |
| 检测最新数据点的异常状态 | JSON 响应包含时序数据中最新数据点的异常状态（和其他数据）。   |
| 检测标记新数据趋势的更改点 | 包含时序数据中检测到的更改点的 JSON 响应。 |

虽然此应用程序是使用 Java 编写的，但 API 是一种 RESTful Web 服务，与大多数编程语言兼容。 可在 [GitHub](https://github.com/Azure-Samples/AnomalyDetector/blob/master/quickstarts/java-detect-anomalies.java) 上找到本快速入门的源代码。

## <a name="prerequisites"></a>先决条件

- Azure 订阅 - [免费创建订阅](https://azure.microsoft.com/free/cognitive-services)
- 拥有 Azure 订阅后，可在 Azure 门户中<a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesAnomalyDetector"  title="创建异常检测器资源"  target="_blank">创建异常检测器资源<span class="docon docon-navigate-external x-hidden-focus"></span></a>来获取密钥和终结点。 等待其部署并单击“转到资源”按钮。
    - 需要从创建的资源获取密钥和终结点，以便将应用程序连接到异常检测器 API。 你稍后会在快速入门中将密钥和终结点粘贴到下方的代码中。
    可以使用免费定价层 (`F0`) 试用该服务，然后再升级到付费层进行生产。
- [Java&trade; 开发工具包 (JDK) 7](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) 或更高版本。
- 将这些库从 Maven 存储库导入
    - [以 Java 编写的 JSON](https://mvnrepository.com/artifact/org.json/json) 包
    - [Apache HttpClient](https://mvnrepository.com/artifact/org.apache.httpcomponents/httpclient) 包

- 包含时序数据点的 JSON 文件。 可在 [GitHub](https://github.com/Azure-Samples/anomalydetector/blob/master/example-data/request-data.json) 上找到本快速入门的示例数据。

[!INCLUDE [anomaly-detector-environment-variables](../includes/environment-variables.md)]

## <a name="create-a-new-application"></a>创建新应用程序

1. 创建新的 Java 项目并导入以下库。

    [!code-java[Import statements](~/samples-anomaly-detector/quickstarts/java-detect-anomalies.java?name=imports)]

2. 为订阅密钥和终结点创建变量。 下面是可用于异常检测的 URI。 稍后会将这些 URL 追加到服务终结点，以创建 API 请求 URL。

    |检测方法  |URI  |
    |---------|---------|
    |批量检测    | `/anomalydetector/v1.0/timeseries/entire/detect`        |
    |对最新数据点进行检测     | `/anomalydetector/v1.0/timeseries/last/detect`        |
    | 更改点检测 | `/anomalydetector/v1.0/timeseries/changepoint/detect`   |

    [!code-java[Initial key and endpoint variables](~/samples-anomaly-detector/quickstarts/java-detect-anomalies.java?name=vars)]

## <a name="create-a-function-to-send-requests"></a>创建用于发送请求的函数

1. 创建名为 `sendRequest()` 的新函数，该函数使用上面创建的变量。 然后，执行以下步骤。

2. 创建可以向 API 发送请求的 `CloseableHttpClient` 对象。 通过将终结点和异常检测器 URL 结合使用，将请求发送到 `HttpPost` 请求对象。

3. 使用请求的 `setHeader()` 函数将 `Content-Type` 标头设置为 `application/json`，然后将订阅密钥添加到 `Ocp-Apim-Subscription-Key` 标头。

4. 使用请求的 `setEntity()` 函数来设置要发送的数据。

5. 使用客户端的 `execute()` 函数来发送请求，并将其保存到 `CloseableHttpResponse` 对象。

6. 创建 `HttpEntity` 对象，用于存储响应内容。 使用 `getEntity()` 获取内容。 如果响应不为空，请将其返回。

    [!code-java[API request method](~/samples-anomaly-detector/quickstarts/java-detect-anomalies.java?name=request)]

## <a name="detect-anomalies-as-a-batch"></a>以批的形式检测异常

1. 创建名为 `detectAnomaliesBatch()` 的方法，以批的形式检测整个数据中的异常。 调用在上面使用终结点、URL、订阅密钥和 JSON 数据创建的 `sendRequest()` 方法。 获取结果，并将其输出到控制台。

2. 如果响应包含 `code` 字段，请输出错误代码和错误消息。

3. 否则，请查找异常在数据集中的位置。 响应的 `isAnomaly` 字段包含一个布尔值，该值与给定的数据点是否为异常相关。 获取 JSON 数组，循环访问该数组，并输出任何 `true` 值的索引。 如果找到任何此类值，这些值对应于异常数据点的索引。

    [!code-java[Method for batch detection](~/samples-anomaly-detector/quickstarts/java-detect-anomalies.java?name=detectBatch)]

## <a name="detect-the-anomaly-status-of-the-latest-data-point"></a>检测最新数据点的异常状态

创建名为 `detectAnomaliesLatest()` 的方法，以检测数据集中最后一个数据点的异常状态。 调用在上面使用终结点、URL、订阅密钥和 JSON 数据创建的 `sendRequest()` 方法。 获取结果，并将其输出到控制台。

[!code-java[Latest point detection method](~/samples-anomaly-detector/quickstarts/java-detect-anomalies.java?name=detectLatest)]


## <a name="detect-change-points-in-the-data"></a>检测数据中的更改点

1. 创建名为 `detectChangePoints()` 的方法，以批的形式检测整个数据中的异常。 调用在上面使用终结点、URL、订阅密钥和 JSON 数据创建的 `sendRequest()` 方法。 获取结果，并将其输出到控制台。

2. 如果响应包含 `code` 字段，请输出错误代码和错误消息。

3. 否则，请查找更改点在数据集中的位置。 响应的 `isChangePoint` 字段包含一个布尔值，指示给定的数据点是否为趋势更改点。 获取 JSON 数组，循环访问该数组，并输出任何 `true` 值的索引。 如果找到任何此类值，这些值对应于趋势更改点的索引。

    [!code-java[detect change points](~/samples-anomaly-detector/quickstarts/java-detect-anomalies.java?name=detectChangePoint)]

## <a name="load-your-time-series-data-and-send-the-request"></a>加载时序数据并发送请求

1. 在应用程序的 main 方法中读入 JSON 文件，其中包含将要添加到请求中的数据。

2. 调用上面创建的两个异常检测函数。

    [!code-java[Main method](~/samples-anomaly-detector/quickstarts/java-detect-anomalies.java?name=main)]

### <a name="example-response"></a>示例响应

成功的响应以 JSON 格式返回。 单击以下链接在 GitHub 上查看 JSON 响应：
* [批量检测响应示例](https://github.com/Azure-Samples/anomalydetector/blob/master/example-data/batch-response.json)
* [最新数据点检测响应示例](https://github.com/Azure-Samples/anomalydetector/blob/master/example-data/latest-point-response.json)
* [更改点检测响应示例](https://github.com/Azure-Samples/anomalydetector/blob/master/example-data/change-point-sample.json)

[!INCLUDE [anomaly-detector-next-steps](../includes/quickstart-cleanup-next-steps.md)]
