---
title: 'columna de colocación: Explorador de azure Data Explorer . Microsoft Docs'
description: En este artículo se describe la columna de colocación en El Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/11/2020
ms.openlocfilehash: 56ba5499b27b517b3080ee27ac317aa1e0917edd
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81521223"
---
# <a name="drop-column"></a>columna de caída

Quita una columna de una tabla.
Para quitar varias columnas de una tabla, consulte [a continuación](#drop-table-columns).

> [!WARNING]
> Este comando es irreversible. Se eliminarán todos los datos de la columna que se elimine.
> Los comandos futuros para volver a agregar esa columna no podrán restaurar los datos.

**Sintaxis**

`.drop``column` *NombreDeTabla* `.` *ColumnName*

## <a name="drop-table-columns"></a>columnas de tabla desplegable

Quita un número de columnas de una tabla.

> [!WARNING]
> Este comando es irreversible. Se eliminarán todos los datos de las columnas que se eliminen.
> Los comandos futuros para volver a agregar esas columnas no podrán restaurar los datos.

**Sintaxis**

`.drop``table` *NombreTabla* `columns` *Col1* `,` *Col2*Col1 [ Col2 ]... `(``)`