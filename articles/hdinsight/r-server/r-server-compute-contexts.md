---
title: 适用于 HDInsight 上的 ML Services 的计算上下文选项 - Azure
description: 了解可供 HDInsight 上的 ML Services 用户使用的不同计算上下文选项
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 01/02/2020
ms.openlocfilehash: 21781015aa91c9c953d716b9b3399851f25be9b5
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/26/2020
ms.locfileid: "92536328"
---
# <a name="compute-context-options-for-ml-services-on-hdinsight"></a>适用于 HDInsight 上的 ML Services 的计算上下文选项

Azure HDInsight 上的 ML Services 可设置计算上下文，从而控制执行调用的方式。 本文概述了可用于指定可否以及如何跨边缘节点或 HDInsight 群集的核心并行化执行的相关选项。

群集的边缘节点为连接到群集和运行 R 脚本提供了便捷的位置。 使用边缘节点，可以选择跨边缘节点服务器的各个核心上运行 RevoScaleR 的并行化分布式函数。 还可以通过使用 RevoScaleR 的 Hadoop Map Reduce 或 Apache Spark 计算上下文在群集的各个节点上运行这些函数。

## <a name="ml-services-on-azure-hdinsight"></a>Azure HDInsight 上的 ML Services

[Azure HDInsight 上的 ML Services](r-server-overview.md) 提供最新的基于 R 的分析功能。 它可以使用存储在 [Azure Blob](../../storage/common/storage-introduction.md "Azure Blob 存储") 存储帐户、Data Lake Store 或本地 Linux 文件系统中 Apache Hadoop HDFS 容器中的数据。 由于 ML 服务是在开源 R 上构建的，因此你构建的基于 R 的应用程序可以应用任何 8000 + 开源 R 包。 这些应用程序还可以利用 [RevoScaleR](/machine-learning-server/r-reference/revoscaler/revoscaler)（ML Services 附带的 Microsoft 的大数据分析包）中的例程。  

## <a name="compute-contexts-for-an-edge-node"></a>边缘节点的计算上下文

一般而言，在边缘节点上的 ML Services 群集中运行的 R 脚本会在该节点上的 R 解释器内运行。 但是，调用 RevoScaleR 函数的步骤例外。 RevoScaleR 调用会在计算环境中运行，而计算环境取决于如何设置 RevoScaleR 计算上下文。  从边缘节点运行 R 脚本时，计算上下文的值可能有：

- 本地顺序 (local) 
- 本地并行 (localpar) 
- Map Reduce
- Spark

local 和 localpar 选项的区别只体现在 rxExec 调用的执行方式  。 这两个选项都以并行方式跨所有可用核心执行其他 rx-function 调用，除非使用 RevoScaleR **numCoresToUse** 选项另外指定，例如，`rxOptions(numCoresToUse=6)`。 并行执行选项提供最佳性能。

下表总结了用于设置调用执行方式的各个计算上下文选项：

| 计算上下文  | 设置方式                      | 执行上下文                        |
| ---------------- | ------------------------------- | ---------------------------------------- |
| 本地顺序 | rxSetComputeContext('local')    | 跨边缘节点服务器的核心并行执行，但 rxExec 调用除外（这种调用是串行执行的） |
| 本地并行   | rxSetComputeContext('localpar') | 跨边缘节点服务器的核心并行执行 |
| Spark            | RxSpark()                       | 通过 Spark 跨 HDI 群集的各个节点并行化分布式执行 |
| Map Reduce       | RxHadoopMR()                    | 通过 Map Reduce 跨 HDI 群集的各个节点并行化分布式执行 |

## <a name="guidelines-for-deciding-on-a-compute-context"></a>有关确定计算上下文的指导原则

有三个选项可提供并行执行，根据分析工作的性质、数据大小和位置进行选择。 没有简单的公式可告诉您，使用哪种计算上下文。 但是，有些指导原则可帮助你做出正确的选择，或至少可以帮助你在运行基准测试之前缩小选择范围。 这些指导原则包括：

- 本地 Linux 文件系统比 HDFS 更快。
- 如果数据在本地且采用 XDF，则重复分析的速度更快。
- 最好从文本数据源中流式传输少量数据。 如果数据量较大，请在分析之前将其转换为 XDF。
- 对于极大量的数据，可将数据复制或流式传输到边缘节点进行分析所造成的开销将变得难以控制。
- 在 Hadoop 中进行分析时，Apache Spark 比 Map Reduce 更快。

鉴于这些原则，有一些用于选择计算上下文的常规经验规则，如下面部分所示。

### <a name="local"></a>Local

- 如果要分析的数据量较小，并且不需要重复的分析，请使用 *local* 或 *localpar* 将其直接流式传输到分析例程。
- 如果要分析的数据量较小或者大小适中并且需要重复分析，可将其复制到本地文件系统，导入到 XDF，然后通过 local 或 localpar 进行分析  。

### <a name="apache-spark"></a>Apache Spark

- 如果要分析的数据量较大，可使用 RxHiveData 或 RxParquetData 将它导入到 Spark DataFrame，或导入到 HDFS 中的 XDF（除非存储有问题），然后通过 Spark 计算上下文进行分析  。

### <a name="apache-hadoop-map-reduce"></a>Apache Hadoop Map Reduce

- 仅当遇到 Spark 计算上下文的无法解决问题时，才使用地图减计算上下文，因为它的速度通常较慢。  

## <a name="inline-help-on-rxsetcomputecontext"></a>关于 rxSetComputeContext 的内联帮助
有关 RevoScaleR 计算上下文的详细信息和示例，请参阅 R 中关于 rxSetComputeContext 方法的内联帮助，例如：

```console
> ?rxSetComputeContext
```

也可以参阅 [Machine Learning Server 文档](/machine-learning-server/)中的[分布式计算概述](/machine-learning-server/r/how-to-revoscaler-distributed-computing)。

## <a name="next-steps"></a>后续步骤

本文概述了可跨边缘节点的核心或 HDInsight 群集中指定是否并行化或如何并行化执行的相关选项。 若要详细了解如何通过 HDInsight 群集使用 ML Services，请参阅以下主题：

- [适用于 Apache Hadoop 的 ML Services 概述](r-server-overview.md)
- [适用于 HDInsight 上的 ML Services 的 Azure 存储选项](r-server-storage.md)