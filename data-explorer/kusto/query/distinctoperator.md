---
title: 'operador distinto: Explorador de datos de Azure . . . . . . . . . . . . . . . Microsoft Docs'
description: En este artículo se describe el operador distinto en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 4287ca48d3fb5006e67a9266ea05287a7d8a06f6
ms.sourcegitcommit: 29018b3db4ea7d015b1afa65d49ecf918cdff3d6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2020
ms.locfileid: "82030382"
---
# <a name="distinct-operator"></a>Operador distinct

Produce una tabla con la combinación distinta de las columnas proporcionadas de la tabla de entrada. 

```kusto
T | distinct Column1, Column2
```

Produce una tabla con la combinación distinta de todas las columnas de la tabla de entrada.

```kusto
T | distinct *
```

**Ejemplo**

Muestra la combinación distinta de fruta y precio.

```kusto
Table | distinct fruit, price
```

:::image type="content" source="images/distinctoperator/distinct.PNG" alt-text="Distintas":::

**Notas**

* A `summarize by ...`diferencia `distinct` de , el`*`operador admite proporcionar un asterisco ( ) como clave de grupo, lo que facilita su uso en tablas anchas.
* Si el grupo por teclas son `summarize by ...` de alta cardinalidad, usar con la [estrategia de barajado](shufflequery.md) podría ser útil.