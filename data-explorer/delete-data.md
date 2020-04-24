---
title: Eliminación de datos desde el Explorador de datos de Azure
description: En este artículo se describen los escenarios de eliminación de Azure Data Explorer, incluidas la purga, la eliminación de extensiones y las eliminaciones basadas en la retención.
author: orspod
ms.author: orspodek
ms.reviewer: avneraa
ms.service: data-explorer
ms.topic: conceptual
ms.date: 03/12/2020
ms.openlocfilehash: efbea2f9b8502f2521f2376f4aaf57902fa4e302
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81493296"
---
# <a name="delete-data-from-azure-data-explorer"></a>Eliminación de datos desde el Explorador de datos de Azure

Azure Data Explorer admite los distintos escenarios de eliminación que se describen en este artículo. 

## <a name="delete-data-using-the-retention-policy"></a>Eliminación de datos mediante la directiva de retención

Azure Data Explorer elimina automáticamente los datos de acuerdo con la [directiva de retención](kusto/management/retentionpolicy.md). Este método es la forma más eficaz y sencilla de eliminar datos. Establezca la directiva de retención en el nivel de base de datos o de tabla.

Considere una base de datos o una tabla que se establece con 90 días de retención. Si solo se necesitan 60 días de datos, elimine los datos más antiguos de la siguiente manera:

```kusto
.alter-merge database <DatabaseName> policy retention softdelete = 60d

.alter-merge table <TableName> policy retention softdelete = 60d
```

## <a name="delete-data-by-dropping-extents"></a>Eliminación de datos mediante la eliminación de extensiones

La [extensión (partición de datos)](kusto/management/extents-overview.md) es la estructura interna en la que se almacenan los datos. Cada extensión puede contener millones de registros. Las extensiones se pueden eliminar individualmente o en grupo mediante los [comandos de eliminación de extensiones](kusto/management/extents-commands.md#drop-extents). 

### <a name="examples"></a>Ejemplos

Puede eliminar todas las filas de una tabla o solo una extensión específica.

* Eliminación de todas las filas de una tabla:

    ```kusto
    .drop extents from TestTable
    ```

* Eliminación de una extensión específica:

    ```kusto
    .drop extent e9fac0d2-b6d5-4ce3-bdb4-dea052d13b42
    ```

## <a name="delete-individual-rows-using-purge"></a>Eliminación de filas individuales mediante la purga

La [purga de datos](kusto/concepts/data-purge.md) se puede usar para eliminar filas individuales. La eliminación no es inmediata y requiere bastantes recursos del sistema. Por tanto, solo se recomienda para escenarios de cumplimiento.  

