---
title: 'Combinación de difusión: Azure Explorador de datos'
description: En este artículo se describe la combinación de difusión en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: e64413047ceb83860f7ea47a1540ae99faa7ec55
ms.sourcegitcommit: b4d6c615252e7c7d20fafd99c5501cb0e9e2085b
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2020
ms.locfileid: "83863224"
---
# <a name="broadcast-join"></a>Combinación de difusión

En la actualidad, las combinaciones regulares se ejecutan en un único nodo de clúster.
La combinación de difusión es una estrategia de ejecución de Join, que la distribuirá a través de los nodos del clúster. Esta estrategia es útil cuando el lado izquierdo de la combinación es pequeño (hasta 100 K registros). En este caso, la combinación de difusión tendrá mayor rendimiento que la combinación normal.

Si el lado izquierdo de la combinación es un conjunto de elementos pequeño, puede ejecutar la combinación en el modo de difusión con la sintaxis siguiente (Hint. Strategy = Broadcast):

```kusto
lookupTable 
| join hint.strategy = broadcast (factTable) on key
```

La mejora del rendimiento será más apreciable en escenarios en los que la combinación va seguida de otros operadores como `summarize` . por ejemplo, en esta consulta:

```kusto
lookupTable 
| join hint.strategy = broadcast (factTable) on Key
| summarize dcount(Messages) by Timestamp, Key
```