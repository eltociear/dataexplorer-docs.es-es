---
title: external_table() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe external_table() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 08/21/2019
ms.openlocfilehash: 9fd03fb3c8452702c3db27a5e0466e8608c04eb9
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81515460"
---
# <a name="external_table"></a>external_table()

Hace referencia a una tabla externa por nombre.

```kusto
external_table('StormEvent')
```

**Sintaxis**

`external_table``(` *TableName* `,` [ *MappingName* ]`)`

**Argumentos**

* *TableName*: el nombre de la tabla externa que se está consultando.
  Debe ser un literal de cadena `blob` que `adl`hace referencia a una tabla externa de tipo o . <!-- TODO: Document data formats supported -->

* *MappingName*: un nombre opcional del objeto de asignación que asigna los campos de las particiones de datos reales (externas) a las columnas de salida de esta función.

**Notas**

Consulte [tablas externas](schema-entities/externaltables.md) para obtener más información sobre tablas externas.

Consulte también [comandos para administrar tablas externas.](../management/externaltables.md)

La `external_table` función tiene restricciones similares a las de la función [de tabla.](tablefunction.md)