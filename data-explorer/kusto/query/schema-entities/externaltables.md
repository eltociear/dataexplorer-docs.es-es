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
ms.openlocfilehash: a76ef031a3fbcaa26f90fd2c2664920805b823b5
ms.sourcegitcommit: 8e097319ea989661e1958efaa1586459d2b69292
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/15/2020
ms.locfileid: "84780276"
---
# <a name="external-tables"></a>Tablas externas

Una **tabla externa** es una entidad de esquema de Kusto que hace referencia a los datos almacenados fuera de la base de datos de Azure explorador de datos.

De forma similar a [las tablas](tables.md), una tabla externa tiene un esquema bien definido (una lista ordenada de pares de tipo de datos y nombre de columna). A diferencia de las tablas, los datos se almacenan y administran fuera del clúster. Normalmente, los datos se almacenan en un formato estándar, como CSV, parquet, Avro, y no se recopilan en Azure Explorador de datos.

Una **tabla externa** se crea una vez. Vea los siguientes comandos para la creación de tablas externas:
* [Comandos de control general de tabla externa](../../management/externaltables.md)
* [Creación y modificación de tablas SQL externas](../../management/external-sql-tables.md)
* [Crear y modificar tablas en Azure Storage o Azure Data Lake](../../management/external-tables-azurestorage-azuredatalake.md)

Se puede hacer referencia a una **tabla externa** por su nombre mediante la función [external_table ()](../../query/externaltablefunction.md) . 

**Notas**

* Nombres de tablas externas:
   * Distingue mayúsculas de minúsculas.
   * No se puede superponer con los nombres de tabla Kusto.
   * Siga las reglas para [los nombres de entidad](./entity-names.md).
* El límite máximo de tablas externas por base de datos es 1.000.
* Kusto admite [la exportación y](../../management/data-export/export-data-to-an-external-table.md) [exportación continua](../../management/data-export/continuous-data-export.md) a una tabla externa y la [consulta de tablas externas](../../../data-lake-query-data.md).
* La [purga de datos](../../concepts/data-purge.md) no se aplica en las tablas externas. Los registros no se eliminan nunca de las tablas externas.
