---
title: operador getschema - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe el operador getschema en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 2faeee575f1af72cfad808253853ae96aba7a28f
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81514372"
---
# <a name="getschema-operator"></a>Operador getschema 

Genere una tabla que represente un esquema tabular de la entrada.

```kusto
T | summarize MyCount=count() by Country | getschema 
```

**Sintaxis**

*T* `| ``getschema`

**Ejemplo**

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