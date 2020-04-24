---
title: 'Columna .alter: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe la columna .alter en El Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/11/2020
ms.openlocfilehash: d41b4f452125fbfebc319112db244deaca79f37a
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81522617"
---
# <a name="alter-column"></a>Columna .alter

Altera el tipo de datos de una columna de tabla existente.

> [!WARNING]
> Al modificar el tipo de datos de una columna, los datos preexistentes de esa columna que no sean del nuevo tipo de datos se omitirán en consultas futuras y se reemplazarán por un [valor nulo.](../query/scalar-data-types/null-values.md) Después `alter column`de usar , esos datos no se pueden recuperar, incluso mediante el uso de otro comando para modificar el tipo de columna a un valor anterior.
> Consulte [a continuación](#changing-column-type-without-data-loss) el procedimiento recomendado para cambiar el tipo de columna sin perder datos.

**Sintaxis** 

`.alter``column` [*DatabaseName* `.`] *NombreDeTabla* `.` *ColumnNewType* *ColumnName* `type` `=`
 
**Ejemplo** 

```
.alter column ['Table'].['ColumnX'] type=string
```

## <a name="changing-column-type-without-data-loss"></a>Cambio del tipo de columna sin pérdida de datos

Para cambiar el tipo de columna mientras conserva los datos históricos, cree una nueva tabla con el tipo correcto.

Para cada `T1` tabla en la que desee cambiar un tipo de columna, ejecute los pasos siguientes:

1. Cree una `T1_prime` tabla con el esquema correcto (los tipos de columna correctos).
1. Intercambie las tablas mediante el comando [.rename tables,](rename-table-command.md) que permite intercambiar nombres de tabla.

```
.rename tables T_prime=T1, T1=T_prime
```

Cuando se completa el comando, `T1` los nuevos flujos de datos a `T1_prime`los que ahora se escriben correctamente y los datos históricos están disponibles en .

Hasta `T1_prime` que los datos salgan de `T1` la ventana de retención, las consultas que se tocan deben modificarse para realizar la unión con `T1_prime`.