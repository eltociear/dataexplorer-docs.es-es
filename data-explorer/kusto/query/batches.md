---
title: 'Lotes: Azure Explorador de datos'
description: En este artículo se describen los lotes de Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: e5a38d53fb9b28fc7da0ddf71132e3a047e8188e
ms.sourcegitcommit: 974d5f2bccabe504583e387904851275567832e7
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/18/2020
ms.locfileid: "83550340"
---
# <a name="batches"></a>Instancias de Batch

Una consulta puede incluir varias instrucciones de expresión tabular, siempre y cuando estén delimitadas por un carácter de punto y coma ( `;` ). A continuación, la consulta devuelve varios resultados tabulares. Los resultados se generan mediante las instrucciones de expresión tabular y se ordenan según el orden de las instrucciones en el texto de la consulta.

Por ejemplo, la siguiente consulta genera dos resultados tabulares. Las herramientas de agente de usuario pueden mostrar los resultados con el nombre adecuado asociado a cada ( `Count of events in Florida` y `Count of events in Guam` , respectivamente).

```kusto
StormEvents | where State == "FLORIDA" | count | as ['Count of events in Florida'];
StormEvents | where State == "GUAM" | count | as ['Count of events in Guam']
```

Batch es útil para escenarios en los que varias subconsultas comparten un cálculo común, como los paneles. Si el cálculo común es complejo, utilice la [función materializar ()](./materializefunction.md) y construya la consulta para que se ejecute solo una vez:

```kusto
let m = materialize(StormEvents | summarize n=count() by State);
m | where n > 2000;
m | where n < 10
```

Notas:
* Prefiere el procesamiento por lotes y [`materialize`](materializefunction.md) el uso del [operador Fork](forkoperator.md).
