---
title: Consultas - Explorador de Azure Data Explorer ? Microsoft Docs
description: En este artículo se describen las consultas en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: 2ac338600320d3f83a92e22e405f630ee308df2f
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81510785"
---
# <a name="queries"></a>Consultas

Una consulta es una operación de solo lectura en los datos ingeridos de un clúster de Kusto Engine. Las consultas siempre se ejecutan en el contexto de una base de datos determinada del clúster (aunque también pueden hacer referencia a datos de otra base de datos o incluso en otro clúster).

Como la consulta ad hoc de datos es el escenario de máxima prioridad para Kusto, la sintaxis del lenguaje de consulta de Kusto está optimizada para usuarios no expertos que crean y ejecutan consultas sobre sus datos y pueden comprender inequívocamente lo que hace cada consulta (lógicamente).

La sintaxis del lenguaje es la de un flujo de datos, donde "datos" realmente significa "datos tabulares" (datos en una o más filas/columnas de forma rectangular). Como mínimo, una consulta consta de referencias de datos de origen (referencias a tablas Kusto) y uno o`|`varios **operadores** de consulta aplicados en secuencia, indicados visualmente por el uso de un carácter de canalización ( ) para delimitar operadores.

Por ejemplo:

```kusto
StormEvents 
| where State == 'FLORIDA' and StartTime > datetime(2000-01-01)
| count
```
    
Cada filtro precedido por el carácter de barra vertical `|` es una instancia de un *operador*con algunos parámetros. La entrada del operador es la tabla que resulta de la canalización anterior. En la mayoría de los casos, los parámetros son [expresiones escalares](./scalar-data-types/index.md) sobre las columnas de la entrada.
En algunos casos, los parámetros son los nombres de las columnas de entrada y, en otros casos, el parámetro es una segunda tabla. El resultado de una consulta siempre es una tabla, incluso si solo tiene una columna y una fila.

## <a name="reference-query-operators"></a>Referencia: Operadores de consulta

> `T` se utiliza en los siguientes ejemplos de consultas para indicar la tabla de origen o la canalización anterior.