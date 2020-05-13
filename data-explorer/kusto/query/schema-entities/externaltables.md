---
title: 'Tablas externas: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describen las tablas externas de Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 01/22/2020
ms.openlocfilehash: cfcf8a18bac1f6369b75538f2172fe8f25cb9660
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83372946"
---
# <a name="external-tables"></a>Tablas externas

Una **tabla externa** es una entidad de esquema de Kusto que hace referencia a los datos almacenados fuera de Kusto Database.

De forma similar a [las tablas](tables.md), una tabla externa tiene un esquema bien definido (una lista ordenada de pares de tipo de datos y nombre de columna). A diferencia de las tablas, los datos se almacenan y administran fuera del clúster de Kusto. Normalmente, los datos se almacenan en un formato estándar, como CSV, parquet, Avro, y no se ingesta en Kusto.

Una **tabla externa** se crea una vez (vea [comandos de control de tabla externa](../../management/externaltables.md)) y se puede hacer referencia a ella por su nombre mediante la función [external_table ()](../../query/externaltablefunction.md) . 

**Notas**

* Los nombres de las tablas externas distinguen mayúsculas de minúsculas.
* Los nombres de las tablas externas no pueden superponerse con los nombres de tabla Kusto.
* Los nombres de las tablas externas siguen las reglas de [los nombres de entidad](./entity-names.md).
* El límite máximo de tablas externas por base de datos es 1.000.
* Kusto admite la [exportación de datos a una tabla externa](../../management/data-export/export-data-to-an-external-table.md) , así como la [consulta de tablas externas](../../../data-lake-query-data.md).
