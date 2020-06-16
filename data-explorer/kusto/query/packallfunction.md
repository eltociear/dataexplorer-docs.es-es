---
title: pack_all ()-Explorador de datos de Azure | Microsoft Docs
description: En este artículo se describe pack_all () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: f34f2ac316f122034fcf8ee19a8e82e51b6221df
ms.sourcegitcommit: 8e097319ea989661e1958efaa1586459d2b69292
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/15/2020
ms.locfileid: "84780480"
---
# <a name="pack_all"></a>pack_all()

Crea un `dynamic` objeto (contenedor de propiedades) a partir de todas las columnas de la expresión tabular.

**Sintaxis**

`pack_all()`

**Notas**

No se garantiza que la representación del objeto devuelto sea compatible con el nivel de bytes entre ejecuciones. Por ejemplo, las propiedades que aparecen en el contenedor pueden aparecer en un orden diferente.

**Ejemplos**

Dada una tabla SmsMessages 

|SourceNumber |TargetNumber| CharsCount
|---|---|---
|555-555-1234 |555-555-1212 | 46 
|555-555-1234 |555-555-1213 | 50 
|555-555-1212 |555-555-1234 | 32 

La siguiente consulta:

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
datatable(SourceNumber:string,TargetNumber:string,CharsCount:long)
[
'555-555-1234','555-555-1212',46,
'555-555-1234','555-555-1213',50,
'555-555-1212','555-555-1234',32
]
| extend Packed=pack_all()
```
Devuelve:

|TableName |SourceNumber |TargetNumber | Equipado
|---|---|---|---
|SmsMessages|555-555-1234 |555-555-1212 | {"SourceNumber": "555-555-1234", "TargetNumber": "555-555-1212", "CharsCount": 46}
|SmsMessages|555-555-1234 |555-555-1213 | {"SourceNumber": "555-555-1234", "TargetNumber": "555-555-1213", "CharsCount": 50}
|SmsMessages|555-555-1212 |555-555-1234 | {"SourceNumber": "555-555-1212", "TargetNumber": "555-555-1234", "CharsCount": 32}
