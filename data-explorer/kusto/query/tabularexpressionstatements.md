---
title: 'Instrucciones de expresión tabular: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describen las instrucciones de expresión tabular en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 1209f96ed99de79d3fa6ac00e3a115a00a6248e6
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81506620"
---
# <a name="tabular-expression-statements"></a>Instrucciones de expresiones tabulares

La instrucción de expresión tabular es lo que las personas suelen tener en mente cuando hablan de consultas. Esta instrucción suele aparecer en último lugar en la lista de instrucciones, y tanto su entrada como su salida se componen de tablas o conjuntos de datos tabulares.

Kusto utiliza un modelo de flujo de datos para la instrucción de expresión tabular. La estructura típica de una instrucción de expresión tabular es una composición de orígenes de *datos tabulares* (como tablas Kusto), operadores de *datos tabulares* (como filtros y proyecciones) y operadores potencialmente *de representación.* La composición está representada por`|`el carácter de tubería ( ), dando a la instrucción una forma muy regular que representa visualmente el flujo de datos tabulares de izquierda a derecha.
Cada operador acepta un conjunto de datos tabulares "desde la canalización" y entradas adicionales (incluidos otros conjuntos de datos tabulares) del cuerpo del operador y, a continuación, emite un conjunto de datos tabulares al siguiente operador que sigue:   

*source1* `|` *operator1* `|` *operator2* `|` *renderInstruction*

En el ejemplo siguiente, `Logs` el origen es (una referencia a una `where` tabla de la base de datos actual), el primer operador `count` es (que filtra los registros de su entrada según algún predicado por registro) y el segundo operador es (que cuenta el número de registros en su conjunto de datos de entrada):

```kusto
Logs | where Timestamp > ago(1d) | count
```

En el siguiente ejemplo `join` más complejo, el operador se utiliza para combinar registros `Logs` de dos conjuntos de `Events` datos de entrada: uno que es un filtro en la tabla y otro que es un filtro en la tabla.

```kusto
Logs 
| where Timestamp > ago(1d) 
| join 
(
    Events 
    | where continent == 'Europe'
) on RequestId 
```

## <a name="tabular-data-sources"></a>Fuentes de datos tabulares

Un origen de **datos tabular** genera conjuntos de registros, que serán procesados posteriormente por operadores de datos **tabulares.** Kusto soporta varias de estas fuentes:

* Referencias de tabla (que hacen referencia a una tabla Kusto, en la base de datos de contexto o en algún otro clúster o base de datos.)
* El [operador](rangeoperator.md)de rango tabular .
* El [operador de impresión](printoperator.md).
* Una invocación de una función que devuelve una tabla.
* Un literal de [tabla](datatableoperator.md) ("datatable").