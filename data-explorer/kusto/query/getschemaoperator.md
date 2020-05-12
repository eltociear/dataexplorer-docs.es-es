---
title: 'operador GetSchema: Explorador de datos de Azure'
description: En este artículo se describe el operador GetSchema en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 333c4d59a7ed62fd031ab52019c10abd821fd858
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83226761"
---
# <a name="getschema-operator"></a>Operador getschema 

Generar una tabla que represente un esquema tabular de la entrada.

```kusto
T | summarize MyCount=count() by Country | getschema 
```

**Sintaxis**

*T* `| ``getschema`

**Ejemplo**

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
StormEvents
| top 10 by Timestamp
| getschema
```

|ColumnName|ColumnOrdinal|DataType|ColumnType|
|---|---|---|---|
|Timestamp|0|System.DateTime|datetime|
|Idioma|1|System.String|string|
|Página|2|System.String|string|
|Vistas|3|System.Int64|long
|BytesDelivered|4|System.Int64|long
