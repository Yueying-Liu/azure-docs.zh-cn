---
title: 必应自定义搜索 Java 客户端库快速入门
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 02/27/2020
ms.custom: devx-track-java
ms.author: aahi
ms.openlocfilehash: d9c5fe2653cff0d83a145964a3ad9eed166d0688
ms.sourcegitcommit: 22da82c32accf97a82919bf50b9901668dc55c97
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/08/2020
ms.locfileid: "94371542"
---
开始使用适用于 Java 的必应自定义搜索客户端库。 请按照以下步骤安装程序包并试用基本任务的示例代码。 借助必应自定义搜索 API，可为关注的主题创建定制的无广告搜索体验。 可以在 [GitHub](https://github.com/Azure-Samples/cognitive-services-java-sdk-samples/tree/master/Search/BingCustomSearch) 上找到此示例的源代码

使用适用于 Java 的必应自定义搜索客户端库，可以执行以下操作：

* 使用必应自定义搜索实例在网上查找搜索结果。

[参考文档](/java/api/overview/azure/cognitiveservices/client/bingcustomsearch?view=azure-java-stable) | [库源代码](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/cognitiveservices/Search.BingCustomSearch) | [项目 (Maven)](https://search.maven.org/artifact/com.microsoft.azure.cognitiveservices/azure-cognitiveservices-customsearch/) | [示例](https://github.com/Azure-Samples/cognitive-services-java-sdk-samples)

## <a name="prerequisites"></a>先决条件

* Azure 订阅 - [免费创建订阅](https://azure.microsoft.com/free/cognitive-services/)。
* 最新版 [Java 开发工具包 (JDK)](https://www.oracle.com/technetwork/java/javase/downloads/index.html)。
* [Gradle 生成工具](https://gradle.org/install/)，或其他依赖项管理器。
* 必应自定义搜索实例。 请参阅[快速入门：创建第一个必应自定义搜索实例](../../quick-start.md)，了解详细信息。

[!INCLUDE [cognitive-services-bing-custom-search-prerequisites](~/includes/cognitive-services-bing-custom-search-signup-requirements.md)]

从资源获取密钥后，为密钥[创建一个环境变量](../../../cognitive-services-apis-create-account.md#configure-an-environment-variable-for-authentication)（名为 `AZURE_BING_CUSTOM_SEARCH_API_KEY`）。

### <a name="create-a-new-gradle-project"></a>创建新的 Gradle 项目

> [!TIP]
> 如果使用 Gradle，则可在 [Maven 中央存储库](https://search.maven.org/artifact/com.microsoft.azure.cognitiveservices/azure-cognitiveservices-textanalytics/)中找到客户端库以及其他依赖项管理器的详细信息。

在控制台窗口（例如 cmd、PowerShell 或 Bash）中，为应用创建一个新目录并导航到该目录。

```console
mkdir myapp && cd myapp
```

从工作目录运行 `gradle init` 命令。 此命令创建 Gradle 的基本生成文件，包括 *build.gradle.kts* 文件，在运行时使用该文件配置应用程序。

```console
gradle init --type basic
```

当提示你选择一个 **DSL** 时，选择 **Kotlin** 。

## <a name="install-the-client-library"></a>安装客户端库

找到 *build.gradle.kts* ，并使用喜好的 IDE 或文本编辑器将其打开。 然后将以下生成配置复制到其中。 确保 `dependencies` 下包含客户端库：

```kotlin
plugins {
    java
    application
}
application {
    mainClassName = "main.java.BingCustomSearchSample"
}
repositories {
    mavenCentral()
}
dependencies {
    compile("org.slf4j:slf4j-simple:1.7.25")
    compile("com.microsoft.azure.cognitiveservices:azure-cognitiveservices-customsearch:1.0.2")
}
```

为示例应用创建一个文件夹。 在工作目录中运行以下命令：

```console
mkdir src/main/java
```

导航到新文件夹，创建名为 *BingCustomSearchSample.java* 的文件。 打开它，并添加以下 `import` 语句：


[!code-java[import statements](~/cognitive-services-java-sdk-samples/Search/BingCustomSearch/src/main/java/BingCustomSearchSample.java?name=imports)]

创建名为 `BingCustomSearchSample` 的类

```java
public class BingCustomSearchSample {
}
```

在该类中创建一个 `main` 方法，并为资源的密钥创建变量。 如果在启动应用程序后创建了环境变量，请关闭再重新打开运行该应用程序的编辑器、IDE 或 shell，然后才能访问该变量。 稍后将定义方法。

[!code-java[main method](~/cognitive-services-java-sdk-samples/Search/BingCustomSearch/src/main/java/BingCustomSearchSample.java?name=main)]

## <a name="object-model"></a>对象模型

必应自定义搜索客户端是一个 [BingCustomSearchAPI](/java/api/com.microsoft.azure.cognitiveservices.search.customsearch.bingcustomsearchapi?view=azure-java-stable) 对象，该对象通过 [BingCustomSearchManager](/java/api/com.microsoft.azure.cognitiveservices.search.customsearch.bingcustomsearchmanager?view=azure-java-stable) 对象的 [authenticate()](/java/api/com.microsoft.azure.cognitiveservices.search.customsearch.bingcustomsearchmanager.authenticate) 方法创建。 可以使用客户端的 [BingCustomInstances.search()](/java/api/com.microsoft.azure.cognitiveservices.search.customsearch.bingcustominstances.search?view=azure-java-stable#com_microsoft_azure_cognitiveservices_search_customsearch_BingCustomInstances_search__) 方法发送搜索请求。

API 响应是一个 [SearchResponse](/java/api/com.microsoft.azure.cognitiveservices.search.customsearch.models.searchresponse?view=azure-java-stable) 对象，该对象包含有关搜索查询的信息以及搜索结果。

## <a name="code-examples"></a>代码示例

这些代码片段演示如何使用适用于 Java 的必应自定义搜索客户端库执行以下任务：

* [对客户端进行身份验证](#authenticate-the-client)
* [从自定义搜索实例获取搜索结果](#get-search-results-from-your-custom-search-instance)

## <a name="authenticate-the-client"></a>验证客户端

main 方法应该包含一个使用密钥的 [BingCustomSearchManager](/java/api/com.microsoft.azure.cognitiveservices.search.customsearch.bingcustomsearchapi?view=azure-java-stable) 对象，并调用其 `authenticate()`。

```java
BingCustomSearchAPI client = BingCustomSearchManager.authenticate(subscriptionKey);
```

## <a name="get-search-results-from-your-custom-search-instance"></a>从自定义搜索实例获取搜索结果

使用客户端的 [BingCustomInstances.search()](/java/api/com.microsoft.azure.cognitiveservices.search.customsearch.bingcustominstances.search?view=azure-java-stable#com_microsoft_azure_cognitiveservices_search_customsearch_BingCustomInstances_search__) 函数，向自定义实例发送搜索查询。 将 `withCustomConfig` 设置为自定义配置 ID，或默认设置为 `1`。 从 API 获得响应以后，检查是否发现了任何搜索结果。 如果发现了搜索结果，请通过调用响应的 `webPages().value().get()` 函数来获取第一个搜索结果，并输出结果的名称和 URL。

[!code-java[call the custom search API](~/cognitive-services-java-sdk-samples/Search/BingCustomSearch/src/main/java/BingCustomSearchSample.java?name=runSample)]

## <a name="run-the-application"></a>运行应用程序

在项目的主目录中使用以下命令来生成应用：

```console
gradle build
```

使用 `run` 目标运行应用程序：

```console
gradle run
```

## <a name="clean-up-resources"></a>清理资源

如果想要清理并删除认知服务订阅，可以删除资源或资源组。 删除资源组同时也会删除与之相关联的任何其他资源。

* [门户](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [Azure CLI](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [生成自定义搜索 Web 应用](../../tutorials/custom-search-web-page.md)