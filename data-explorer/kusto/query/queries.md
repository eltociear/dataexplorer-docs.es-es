---
title: 'Consultas: Azure Explorador de datos'
description: En este artículo se describen las consultas en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: fb842bcda70c2986bd754f55184413eec594d412
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83373124"
---
# <a name="queries"></a>Consultas

Una consulta es una operación de solo lectura en los datos ingeridos del clúster de Kusto Engine. Las consultas siempre se ejecutan en el contexto de una base de datos determinada en el clúster (aunque también pueden hacer referencia a los datos de otra base de datos o incluso en otro clúster).

Como la consulta ad hoc de los datos es el escenario de prioridad superior de Kusto, la sintaxis del lenguaje de consulta de Kusto está optimizada para los usuarios no expertos que crean y ejecutan consultas sobre sus datos y pueden comprender de manera inequívoca qué hace cada consulta (lógicamente).

La sintaxis del lenguaje es la de un flujo de datos, donde "datos" realmente significa "datos tabulares" (datos de una o varias formas rectangulares de filas o columnas). Como mínimo, una consulta consta de referencias de datos de origen (referencias a tablas de Kusto) y uno o varios **operadores de consulta** aplicados en secuencia, que se indican visualmente mediante el uso de un carácter de barra vertical ( `|` ) para delimitar los operadores.

Por ejemplo:

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
StormEvents 
| where State == 'FLORIDA' and StartTime > datetime(2000-01-01)
| count
```
    
Cada filtro precedido por el carácter de barra vertical `|` es una instancia de un *operador*con algunos parámetros. La entrada del operador es la tabla que resulta de la canalización anterior. En la mayoría de los casos, los parámetros son [expresiones escalares](./scalar-data-types/index.md) sobre las columnas de la entrada.
En algunos casos, los parámetros son los nombres de las columnas de entrada y, en otros casos, el parámetro es una segunda tabla. El resultado de una consulta siempre es una tabla, incluso si solo tiene una columna y una fila.

## <a name="reference-query-operators"></a>Referencia: operadores de consulta

> `T` se utiliza en los siguientes ejemplos de consultas para indicar la tabla de origen o la canalización anterior.
