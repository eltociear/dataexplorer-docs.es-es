---
title: Lotes - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describen los lotes en El Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: bd695a1e7ffd9980de2750d38ad9538eeb877538
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81518027"
---
# <a name="batches"></a>Instancias de Batch

Una consulta puede incluir varias instrucciones de expresión tabular, siempre que`;`estén delimitadas por un carácter de punto y coma ( ). A continuación, la consulta devuelve varios resultados tabulares, generados por las instrucciones de expresión tabular y ordenados según el orden de las instrucciones en el texto de la consulta.

Por ejemplo, la siguiente consulta genera dos resultados tabulares. A continuación, las herramientas del agente de usuario`Count of events in Florida` `Count of events in Guam`pueden mostrar esos resultados con el nombre adecuado asociado a cada ( y , respectivamente).

```kusto
StormEvents | where State == "FLORIDA" | count | as ['Count of events in Florida'];
StormEvents | where State == "GUAM" | count | as ['Count of events in Guam']
```

Batch es especialmente útil para escenarios en los que hay un cálculo común que comparten varias subconsultas, como para los paneles. Si el cálculo común es complejo, se recomienda construir la consulta para que se ejecute una sola vez, utilizando la [función materialize():](./materializefunction.md)

```kusto
let m = materialize(StormEvents | summarize n=count() by State);
m | where n > 2000;
m | where n < 10
```

Notas:
* Prefiere el [`materialize`](materializefunction.md) procesamiento por lotes y el uso por lotes del operador de [horquilla](forkoperator.md).