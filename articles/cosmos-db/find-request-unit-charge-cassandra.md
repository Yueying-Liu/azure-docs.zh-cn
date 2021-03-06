---
title: 在 Azure Cosmos DB 中查找 Cassandra API 查询的请求单位 (RU) 费用
description: 了解如何 () RU 查找针对 Azure Cosmos 容器执行的 Cassandra 查询的请求单位。 您可以使用 Azure 门户、.NET 和 Java 驱动程序来查找 RU 费用。
author: ThomasWeiss
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: how-to
ms.date: 10/14/2020
ms.author: thweiss
ms.custom: devx-track-js
ms.openlocfilehash: 8f7db5dc48f7fec1bcf2bcd1291b67da7b4412c7
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93094806"
---
# <a name="find-the-request-unit-charge-for-operations-executed-in-azure-cosmos-db-cassandra-api"></a>查找 Azure Cosmos DB 中执行的操作的请求单位费用 Cassandra API
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

Azure Cosmos DB 支持多种 API，例如 SQL、MongoDB、Cassandra、Gremlin 和表。 每个 API 具有自身的数据库操作集。 这些操作包括简单的点读取和写入，以及复杂的查询等等。 每个数据库操作根据其复杂性消耗系统资源。

所有数据库操作的成本将由 Azure Cosmos DB 规范化，并以“请求单位”（简称 RU）表示。 你可以将 RU 视为性能货币，它抽象化了执行 Azure Cosmos DB 支持的数据库操作所需的系统资源，例如 CPU、IOPS 和内存。 不管使用哪个 API 来与 Azure Cosmos 容器和数据库操作交互，都始终以 RU 来计量成本。 无论数据库操作是写入、点读取还是查询，都始终以 RU 来计量成本。 若要了解详细信息，请参阅 " [请求单位" 和 "注意事项](request-units.md) " 一文。

本文介绍了不同的方法，可用于查找对 Azure Cosmos DB Cassandra API 中的容器执行的任何操作的 [请求单位](request-units.md) (RU) 消耗。 如果你使用的是其他 API，请参阅 [适用于 MongoDB 的 api](find-request-unit-charge-mongodb.md)、 [SQL API](find-request-unit-charge.md)、 [Gremlin API](find-request-unit-charge-gremlin.md)和 [表 API](find-request-unit-charge-table.md) 文章来查找 RU/秒的费用。

针对 Azure Cosmos DB Cassandra API 执行操作时，RU 费用将在传入的有效负载中以名为 `RequestCharge` 的字段形式返回。 可以使用多个选项来检索 RU 费用。

## <a name="use-the-net-sdk"></a>使用 .NET SDK

使用 [.NET SDK](https://www.nuget.org/packages/CassandraCSharpDriver/) 时，可以在 `RowSet` 对象的 `Info` 属性下检索传入的有效负载：

```csharp
RowSet rowSet = session.Execute("SELECT table_name FROM system_schema.tables;");
double requestCharge = BitConverter.ToDouble(rowSet.Info.IncomingPayload["RequestCharge"].Reverse().ToArray(), 0);
```

有关详细信息，请参阅[快速入门：使用 .NET SDK 和 Azure Cosmos DB 生成 Cassandra 应用](create-cassandra-dotnet.md)。

## <a name="use-the-java-sdk"></a>使用 Java SDK

使用 [Java SDK](https://mvnrepository.com/artifact/com.datastax.cassandra/cassandra-driver-core) 时，可以通过对 `ResultSet` 对象调用 `getExecutionInfo()` 方法来检索传入的有效负载：

```java
ResultSet resultSet = session.execute("SELECT table_name FROM system_schema.tables;");
Double requestCharge = resultSet.getExecutionInfo().getIncomingPayload().get("RequestCharge").getDouble();
```

有关详细信息，请参阅[快速入门：使用 Java SDK 和 Azure Cosmos DB 生成 Cassandra 应用](create-cassandra-java.md)。

## <a name="next-steps"></a>后续步骤

若要了解如何优化 RU 消耗量，请参阅以下文章：

* [Azure Cosmos DB 中的请求单位和吞吐量](request-units.md)
* [在 Azure Cosmos DB 中优化预配的吞吐量成本](optimize-cost-throughput.md)
* [优化 Azure Cosmos DB 中的查询成本](./optimize-cost-reads-writes.md)