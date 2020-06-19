---
title: 'operador de búsqueda: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe el operador de búsqueda en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: edd35e5e259666e8ce4360c072aaac6717e6f8c3
ms.sourcegitcommit: f9d3f54114fb8fab5c487b6aea9230260b85c41d
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/18/2020
ms.locfileid: "85071868"
---
# <a name="search-operator"></a>Operador search

El operador de búsqueda proporciona una experiencia de búsqueda de varias tablas o varias columnas.

## <a name="syntax"></a>Sintaxis

* [*TabularSource* `|` ] `search`[ `kind=` *CaseSensitivity*] [ `in` `(` *TableSources* `)` ] *SearchPredicate*

## <a name="arguments"></a>Argumentos

* *TabularSource*: expresión tabular opcional que actúa como origen de datos en el que se va a buscar, como un nombre de tabla, un [operador de Unión](unionoperator.md), los resultados de una consulta tabular, etc. No puede aparecer junto con la frase opcional que incluye *TableSources*.

* *CaseSensitivity*: una marca opcional que controla el comportamiento de todos los `string` operadores escalares con respecto a la distinción de mayúsculas y minúsculas. Los valores válidos son los dos sinónimos `default` y `case_insensitive` (que es el valor predeterminado para operadores como, por lo `has` que no se distingue entre mayúsculas y minúsculas) y `case_sensitive` (que fuerza a todos los operadores de este tipo en el modo de coincidencia que distingue entre mayúsculas y minúsculas).

* *TableSources*: lista opcional separada por comas de nombres de tabla "con comodín" para formar parte de la búsqueda.
  La lista tiene la misma sintaxis que la lista del [operador Union](unionoperator.md).
  No puede aparecer junto con el *TabularSource*opcional.

* *SearchPredicate*: predicado obligatorio que define lo que se va a buscar (es decir, una expresión booleana que se evalúa para cada registro de la entrada y que, si se devuelve `true` , se genera el registro). La sintaxis de *SearchPredicate* extiende y modifica la sintaxis de Kusto normal de las Expresiones booleanas:

  **Extensiones de coincidencia de cadenas**: los literales de cadena que aparecen como términos en *SearchPredicate* indican una coincidencia de términos entre todas las columnas y el literal mediante `has` ,, `hasprefix` `hassuffix` y las versiones invertido ( `!` ) o con distinción de mayúsculas y minúsculas ( `sc` ) de estos operadores. La decisión de si se debe aplicar `has` , `hasprefix` o `hassuffix` depende de si el literal comienza o finaliza (o ambos) mediante un asterisco ( `*` ). No se permiten asteriscos dentro del literal.

    |Literal   |Operador   |
    |----------|-----------|
    |`billg`   |`has`      |
    |`*billg`  |`hassuffix`|
    |`billg*`  |`hasprefix`|
    |`*billg*` |`contains` |
    |`bi*lg`   |`matches regex`|

  **Restricción de columna**: de forma predeterminada, las extensiones de coincidencia de cadenas intentan coincidir con todas las columnas del conjunto de datos. Es posible restringir esta coincidencia a una columna determinada mediante la sintaxis siguiente: *columnName* `:` *StringLiteral*.

  **Igualdad de cadena**: las coincidencias exactas de una columna con un valor de cadena (en lugar de una coincidencia de términos) se pueden realizar mediante la sintaxis *columnName* `==` *StringLiteral*.

  **Otras expresiones booleanas**: la sintaxis admite todas las Expresiones booleanas Kusto normales.
    Por ejemplo, `"error" and x==123` significa: buscar registros que tengan el término `error` en cualquiera de sus columnas y tener el valor `123` en la `x` columna ".

  **Coincidencia regex**: la coincidencia de la expresión regular se indica mediante la sintaxis de la *columna* `matches regex` *StringLiteral* , donde *StringLiteral* es el patrón Regex.

Tenga en cuenta que si se omiten *TabularSource* y *TableSources* , la búsqueda se realiza en todas las tablas y vistas sin restricciones de la base de datos en el ámbito.

## <a name="summary-of-string-matching-extensions"></a>Resumen de extensiones coincidentes de cadena

  |# |Sintaxis                                 |Significado (equivalente `where` )           |Comentarios|
  |--|---------------------------------------|---------------------------------------|--------|
  | 1|`search "err"`                         |`where * has "err"`                    ||
  | 2|`search in (T1,T2,A*) and "err"`       |<code>union T1,T2,A* &#124; where * has "err"<code>   ||
  | 3|`search col:"err"`                     |`where col has "err"`                  ||
  | 4|`search col=="err"`                    |`where col=="err"`                     ||
  | 5|`search "err*"`                        |`where * hasprefix "err"`              ||
  | 6|`search "*err"`                        |`where * hassuffix "err"`              ||
  | 7|`search "*err*"`                       |`where * contains "err"`               ||
  | 8|`search "Lab*PC"`                      |`where * matches regex @"\bLab.*PC\b"`||
  | 9|`search *`                             |`where 0==0`                           ||
  |10|`search col matches regex "..."`       |`where col matches regex "..."`        ||
  |11|`search kind=case_sensitive`           |                                       |Todas las comparaciones de cadenas distinguen mayúsculas de minúsculas|
  |12|`search "abc" and ("def" or "hij")`    |`where * has "abc" and (* has "def" or * has hij")`||
  |13|`search "err" or (A>a and A<b)`        |`where * has "err" or (A>a and A<b)`   ||

## <a name="remarks"></a>Comentarios

**A diferencia** del [operador Find](findoperator.md), el `search` operador no admite lo siguiente:

1. `withsource=`: La salida siempre incluirá una columna denominada `$table` de tipo `string` cuyo valor es el nombre de la tabla desde la que se recuperó cada registro (o algún nombre generado por el sistema si el origen no es una tabla sino una expresión compuesta).
2. `project=`, `project-smart` : El esquema de salida es equivalente al `project-smart` esquema de salida.

## <a name="examples"></a>Ejemplos

```kusto
// 1. Simple term search over all unrestricted tables and views of the database in scope
search "billg"

// 2. Like (1), but looking only for records that match both terms
search "billg" and ("steveb" or "satyan")

// 3. Like (1), but looking only in the TraceEvent table
search in (TraceEvent) and "billg"

// 4. Like (2), but performing a case-sensitive match of all terms
search "BillB" and ("SteveB" or "SatyaN")

// 5. Like (1), but restricting the match to some columns
search CEO:"billg" or CSA:"billg"

// 6. Like (1), but only for some specific time limit
search "billg" and Timestamp >= datetime(1981-01-01)

// 7. Searches over all the higher-ups
search in (C*, TF) "billg" or "davec" or "steveb"

// 8. A different way to say (7). Prefer to use (7) when possible
union C*, TF | search "billg" or "davec" or "steveb"
```

## <a name="performance-tips"></a>Sugerencias para mejorar el rendimiento

  |# |Sugerencia                                                                                  |Prefer                                        |Over                                                                    |
  |--|-------------------------------------------------------------------------------------|----------------------------------------------|------------------------------------------------------------------------|
  | 1| Prefiere usar un solo `search` operador en varios operadores consecutivos `search`|`search "billg" and ("steveb" or "satyan")`   |<code>search "billg" &#124; search "steveb" or "satyan"<code>           ||
  | 2| Prefiere filtrar dentro del `search` operador                                       |`search "billg" and "steveb"`                 |<code>search * &#124; where * has "billg" and * has "steveb"<code>      ||
