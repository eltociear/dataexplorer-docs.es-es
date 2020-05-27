---
title: 'Unión entre clústeres: Azure Explorador de datos'
description: En este artículo se describe la combinación entre clústeres en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 10c31a249ddd0ad8151f8a6e4a76fde7f87c2400
ms.sourcegitcommit: b4d6c615252e7c7d20fafd99c5501cb0e9e2085b
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2020
ms.locfileid: "83863173"
---
# <a name="cross-cluster-join"></a>Combinación entre clústeres

::: zone pivot="azuredataexplorer"

Para obtener información general sobre las consultas entre clústeres, consulte [consultas entre clústeres o entre bases de datos](cross-cluster-or-database-queries.md) .

Es posible realizar una operación de Unión en conjuntos de valores que residen en clústeres diferentes. Por ejemplo:

```kusto
T | ... | join (cluster("SomeCluster").database("SomeDB").T2 | ...) on Col1 // (1)

cluster("SomeCluster").database("SomeDB").T | ... | join (cluster("SomeCluster2").database("SomeDB2").T2 | ...) on Col1 // (2)
```

En el ejemplo anterior, la operación de combinación es una combinación entre clústeres, suponiendo que el clúster actual no sea "SomeCluster" o "SomeCluster2".

En el ejemplo siguiente:

```kusto
cluster("SomeCluster").database("SomeDB").T | ... | join (cluster("SomeCluster").database("SomeDB2").T2 | ...) on Col1 
```

la operación de combinación no es una combinación entre clústeres porque sus operandos se originan en el mismo clúster.

Cuando Kusto encuentra una combinación entre clústeres, decidirá automáticamente dónde ejecutar la operación de combinación. Esta decisión puede tener uno de los tres resultados posibles:

* Ejecutar operación de combinación en el clúster del operando izquierdo, el operando derecho se capturará primero mediante este clúster. (la combinación en el ejemplo **(1)** se ejecutará en el clúster local)
* Ejecutar operación de combinación en el clúster del operando derecho, este clúster capturará primero el operando izquierdo. (la combinación en el ejemplo **(2)** se ejecutará en "SomeCluster2")
* Ejecutar la operación de Unión localmente (es decir, en el clúster que recibió la consulta), el clúster local capturará primero ambos operandos.

La decisión real depende de la consulta específica. La estrategia de comunicación remota de combinación automática es (versión simplificada): "si uno de los operandos es local, la combinación se ejecutará localmente. Si ambos operandos son remotos, se ejecutará JOIN en el clúster del operando derecho ".

A veces, el rendimiento de la consulta se puede mejorar si no se sigue la estrategia automática de comunicación remota. En este caso, ejecute la operación de combinación en el clúster del operando más grande.

Si, por ejemplo **(1)** , el conjunto de información generado por `T | ...` es mucho menor que uno generado por `cluster("SomeCluster").database("SomeDB").T2 | ...` , es más eficaz ejecutar la operación de combinación en "SomeCluster".

Esta operación se puede realizar proporcionando la sugerencia de comunicación remota de Kusto join. La sintaxis es:

```kusto
T | ... | join hint.remote=<strategy> (cluster("SomeCluster").database("SomeDB").T2 | ...) on Col1
```

Los valores siguientes son válidos para`strategy`
* `left`-ejecutar la combinación en el clúster del operando izquierdo 
* `right`-ejecutar la combinación en el clúster del operando derecho
* `local`-ejecutar la combinación en el clúster del clúster actual
* `auto`-(valor predeterminado) permitir que Kusto tome la decisión de comunicación remota automática

> [!Note]
> Kusto pasará por alto la sugerencia join Remoting si la estrategia sugerida no es aplicable a la operación de combinación.

::: zone-end

::: zone pivot="azuremonitor"

Esta funcionalidad no se admite en Azure Monitor

::: zone-end
