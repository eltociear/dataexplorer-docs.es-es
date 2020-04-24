---
title: Unirse a la difusión- Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe la unión de difusión en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 72c89bf2160f8f5b735fd8c93f9519feae9114d9
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81517313"
---
# <a name="broadcast-join"></a>Combinación de difusión

Hoy en día, las combinaciones regulares se ejecutan en un único nodo de clúster.
La combinación de difusión es una estrategia de ejecución de combinación que la distribuirá a través de nodos de clúster, y es útil cuando el lado izquierdo de la combinación es pequeño (hasta 100K registros), en este caso, la unión será más performant que la unión normal.

Si el lado izquierdo de la unión es un conjunto de datos pequeño, puede ejecutar join en modo broadcast utilizando la siguiente sintaxis (hint.strategy - broadcast):

```kusto
lookupTable 
| join hint.strategy = broadcast (factTable) on key
```

La mejora del rendimiento será más notable en escenarios donde `summarize`la unión es seguida por otros operadores como . por ejemplo en esta consulta:

```kusto
lookupTable 
| join hint.strategy = broadcast (factTable) on Key
| summarize dcount(Messages) by Timestamp, Key
```