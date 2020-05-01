---
title: '. Alter Column: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe. Alter column en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/11/2020
ms.openlocfilehash: ecf0fa09438f8df5792d8826150d58f06360cace
ms.sourcegitcommit: 1faf502280ebda268cdfbeec2e8ef3d582dfc23e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2020
ms.locfileid: "82617874"
---
# <a name="alter-column"></a>.alter column

Modifica el tipo de datos de una columna de tabla existente.

> [!WARNING]
> Al modificar el tipo de datos de una columna, todos los datos preexistentes de esa columna que no sean del tipo de datos nuevo se omitirán en futuras consultas y se reemplazarán por un [valor null](../query/scalar-data-types/null-values.md). Después de `alter column`usar, no se pueden recuperar los datos, incluso si se usa otro comando para modificar el tipo de columna de nuevo a un valor anterior.
> Consulte [a continuación](#changing-column-type-without-data-loss) el procedimiento recomendado para cambiar el tipo de una columna sin perder datos.

**Sintaxis** 

`.alter``column` [*NombreDeBaseDeDatos* `.`] *TableName* `.` *ColumnName* ColumnName `type` *ColumnNewType* ColumnNewType `=`
 
**Ejemplo** 

```kusto
.alter column ['Table'].['ColumnX'] type=string
```

## <a name="changing-column-type-without-data-loss"></a>Cambiar el tipo de columna sin pérdida de datos

Para cambiar el tipo de columna mientras se conservan los datos históricos, cree una tabla nueva y correctamente escrita.

En cada tabla `T1` en la que desee cambiar un tipo de columna, ejecute los pasos siguientes:

1. Cree una tabla `T1_prime` con el esquema correcto (los tipos de columna adecuados).
1. Intercambie las tablas con el comando [. Rename](rename-table-command.md) tables, que permite intercambiar nombres de tabla.

```kusto
.rename tables T_prime=T1, T1=T_prime
```

Cuando se completa el comando, el nuevo flujo `T1` de datos se escribe correctamente y los datos históricos están disponibles en. `T1_prime`

Hasta `T1_prime` que los datos salen del período de retención, las `T1` consultas que se tocan deben modificarse `T1_prime`para realizar la Unión con.