---
title: estimate_data_size() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe estimate_data_size() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 9664a7c9caf0506cdb7274eed5e143f89c82cebb
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81515732"
---
# <a name="estimate_data_size"></a>estimate_data_size()

Devuelve un tamaño de datos estimado en bytes de las columnas seleccionadas de la expresión tabular.

```kusto
estimate_data_size(*)
estimate_data_size(Col1, Col2, Col3)
```

**Sintaxis**

`estimate_data_size(*)`

`estimate_data_size(`*col1*`, `*col2*`, `...`)`

**Argumentos**

* *col1*, *col2*: Selección de referencias de columna en la expresión tabular de origen que se utilizan para la estimación del tamaño de datos. Para incluir todas `*` las columnas, utilice la sintaxis (asterisco).

**Devuelve**

* El tamaño de datos estimado en bytes del tamaño del registro. La estimación se basa en tipos de datos y longitudes de valores.

**Ejemplos**

Cálculo del tamaño `estimated_data_size()`total de los datos mediante:

```kusto
range x from 1 to 10 step 1                    // x (long) is 8 
| extend Text = '1234567890'                   // Text length is 10  
| summarize Total=sum(estimate_data_size(*))   // (8+10)x10 = 180
```

|Total|
|---|
|180|