---
title: 'Tablas externas: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describen las tablas externas en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 01/22/2020
ms.openlocfilehash: f4a059ba4533ed91353e75b499e564e6dd06d591
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81509391"
---
# <a name="external-tables"></a>Tablas externas

Una **tabla externa** es una entidad de esquema Kusto que hace referencia a datos almacenados fuera de la base de datos de Kusto.

Al igual que [las tablas,](tables.md)una tabla externa tiene un esquema bien definido (una lista ordenada de pares de nombre de columna y tipo de datos). A diferencia de las tablas, los datos se almacenan y administran fuera del clúster de Kusto. Más comúnmente los datos se almacenan en algún formato estándar como CSV, Parquet, Avro, y no es ingerido por Kusto.

Una vez se crea una **tabla externa** (consulte [Comandos](../../management/externaltables.md)de control de tabla externa ) y su nombre puede hacer referencia mediante la función [external_table().](../../query/externaltablefunction.md) 

**Notas**

* Los nombres de tabla externos distinguen mayúsculas de minúsculas.
* Los nombres de tabla externos no se pueden superponer con los nombres de tabla de Kusto.
* Los nombres de tabla externos siguen las reglas para los nombres de [entidad.](./entity-names.md)
* El límite máximo de tablas externas por base de datos es de 1.000.
* Kusto admite [la exportación de datos a una tabla externa,](../../management/data-export/export-data-to-an-external-table.md) así como la consulta de [tablas externas.](https://docs.microsoft.com/azure/data-explorer/data-lake-query-data)
