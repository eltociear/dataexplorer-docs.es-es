---
title: 'Valores nulos: Explorador de datos de Azure Microsoft Docs'
description: En este artículo se describen los valores nulos en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: c3a48fdfca855a1ff2f848d4ed97d8162e97b931
ms.sourcegitcommit: 01eb9aaf1df2ebd5002eb7ea7367a9ef85dc4f5d
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2020
ms.locfileid: "81765947"
---
# <a name="null-values"></a>Valores NULL

Todos los tipos de datos escalares de Kusto tienen un valor especial que representa un valor que falta.
Este valor se denomina **valor nulo**o simplemente **null**.

## <a name="null-literals"></a>Literales nulos

El valor nulo de un `T` tipo escalar se representa en `T(null)`el lenguaje de consulta mediante el literal nulo .
Por lo tanto, lo siguiente devuelve una sola fila llena de valores NULL:

```kusto
print bool(null), datetime(null), dynamic(null), guid(null), int(null), long(null), real(null), double(null), time(null)
```

> [!WARNING]
> Tenga en cuenta `string` que actualmente el tipo no admite valores nulos.

## <a name="comparing-null-to-something"></a>Comparando null con algo

El valor nulo no se compara igual a ningún otro valor del tipo de datos, incluido él mismo. (Es decir, `null == null` es falso.) Para determinar si algún valor es el valor nulo, utilice la función [isnull()](../isnullfunction.md) y la función [isnotnull().](../isnotnullfunction.md)

## <a name="binary-operations-on-null"></a>Operaciones binarias en null

En general, null se comporta de una manera "pegajosa" alrededor de los operadores binarios; una operación binaria entre un valor nulo y cualquier otro valor (incluido otro valor nulo) produce un valor nulo.

## <a name="data-ingestion-and-null-values"></a>Ingestión de datos y valores nulos

Para la mayoría de los tipos de datos, un valor que falta en el origen de datos genera un valor nulo en la celda de tabla correspondiente. Una excepción a que `string` son columnas de tipo e ingesta similar a CSV, donde un valor que falta produce una cadena vacía.
Así, por ejemplo, si tenemos: 

```kusto
.create table T [a:string, b:int]

.ingest inline into table T
[,]
[ , ]
[a,1]
```

A continuación:

|a     |b     |isnull(a)|isempty(a)|strlen(a)|isnull(b)|
|------|------|---------|----------|---------|---------|
|&nbsp;|&nbsp;|false    |true      |0        |true     |
|&nbsp;|&nbsp;|false    |false     |1        |true     |
|a     |1     |false    |false     |1        |false    |

::: zone pivot="azuredataexplorer"

* Si ejecuta la consulta anterior en Kusto.Explorer, `true` todos `1`los valores `false` se disiparán `0`como , y todos los valores se mostrarán como .

::: zone-end

* Kusto no ofrece una manera de restringir la columna de una tabla para que no tenga `NOT NULL` valores nulos (en otras palabras, no hay ningún equivalente a la restricción de SQL).
