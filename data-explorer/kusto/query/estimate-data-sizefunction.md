---
title: 'estimate_data_size (): Explorador de datos de Azure'
description: En este artículo se describe estimate_data_size () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: f0901bddbfa8854e902ab60197164cf830215948
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83224959"
---
# <a name="estimate_data_size"></a>estimate_data_size()

Devuelve un tamaño de datos estimado en bytes de las columnas seleccionadas de la expresión tabular.

```kusto
estimate_data_size(*)
estimate_data_size(Col1, Col2, Col3)
```

**Sintaxis**

`estimate_data_size(*)`

`estimate_data_size(`*col1* `, ` *col2* `, ` ...`)`

**Argumentos**

* *col1*, *col2*: selección de referencias de columna en la expresión tabular de origen que se usan para la estimación del tamaño de los datos. Para incluir todas las columnas, use la `*` sintaxis (asterisco).

**Devuelve**

* Tamaño estimado de los datos en bytes del tamaño del registro. La estimación se basa en los tipos de datos y las longitudes de los valores.

**Ejemplos**

Calcular el tamaño total de los datos mediante `estimated_data_size()` :

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
range x from 1 to 10 step 1                    // x (long) is 8 
| extend Text = '1234567890'                   // Text length is 10  
| summarize Total=sum(estimate_data_size(*))   // (8+10)x10 = 180
```

|Total|
|---|
|180|
