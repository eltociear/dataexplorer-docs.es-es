---
title: 'Unión entre clústeres: Explorador de datos de Azure Microsoft Docs'
description: En este artículo se describe la unión entre clústeres en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 1199b148fa295ac17417bbf590a73bfc9400a710
ms.sourcegitcommit: 01eb9aaf1df2ebd5002eb7ea7367a9ef85dc4f5d
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2020
ms.locfileid: "81765938"
---
# <a name="cross-cluster-join"></a>Unión entre clústeres

::: zone pivot="azuredataexplorer"

Para una discusión general sobre las consultas entre clústeres, consulte [Consultas entre clústeres o entre bases de datos](cross-cluster-or-database-queries.md)

Es posible realizar operaciones de combinación en conjuntos de datos que residen en clústeres diferentes. Por ejemplo 

```kusto
T | ... | join (cluster("SomeCluster").database("SomeDB").T2 | ...) on Col1 // (1)

cluster("SomeCluster").database("SomeDB").T | ... | join (cluster("SomeCluster2").database("SomeDB2").T2 | ...) on Col1 // (2)
```

En los ejemplos anteriores, la operación de combinación es una combinación entre clústeres que suponiendo que el clúster actual no es ni "SomeCluster" ni "SomeCluster2".

Tenga en cuenta que en el ejemplo siguiente

```kusto
cluster("SomeCluster").database("SomeDB").T | ... | join (cluster("SomeCluster").database("SomeDB2").T2 | ...) on Col1 
```

La operación de combinación no es una combinación entre clústeres porque ambos operandos se originan en el mismo clúster.

Cuando Kusto se encuentre con una unión entre clústeres, decidirá automáticamente dónde ejecutar la propia operación de combinación. Esta decisión puede tener uno de los tres resultados posibles:
* Ejecutar operación de combinación en el clúster del operando izquierdo, el operando derecho será capturado primero por este clúster. (unirse en el ejemplo **(1)** se ejecutará en el clúster local)
* Ejecutar operación de combinación en el clúster del operando derecho, el operando izquierdo será capturado primero por este clúster. (unirse en el ejemplo **(2)** se ejecutará en el "SomeCluster2")
* Ejecutar operación de combinación localmente (es decir, en el clúster que recibió la consulta), ambos operandos serán obtenidos primero por el clúster local

La decisión real depende de la consulta específica, la estrategia de comunicación remota de combinación automática es (versión simplificada): "Si uno de los operandos es local, la combinación se ejecutará localmente. Si ambos operandos son de combinación remota se ejecutará n.o en el clúster del operando derecho".

A veces, el rendimiento de la consulta se puede mejorar significativamente si no se sigue la estrategia de comunicación remota automática. En términos generales, es mejor (desde el punto de vista del rendimiento) ejecutar la operación de combinación en el clúster del operando más grande.

Si en el ejemplo **(1)** el conjunto de datos generado por ```T | ...``` es mucho menor que uno producido por ```cluster("SomeCluster").database("SomeDB").T2 | ...``` él es más eficaz para ejecutar la operación de combinación en "SomeCluster".

Esto se puede lograr dando a Kusto unirse a la pista remota. La sintaxis es:

```kusto
T | ... | join hint.remote=<strategy> (cluster("SomeCluster").database("SomeDB").T2 | ...) on Col1
```

Los siguientes son valores legales para*`strategy`*
* **`left`**- ejecutar la unión en el clúster del operando izquierdo 
* **`right`**- ejecutar join en el clúster del operando derecho
* **`local`**- ejecutar la unión en el clúster del clúster actual
* **`auto`**- (predeterminado) dejar que Kusto tome la decisión automática de remoting

**Nota:** La sugerencia de comunicación remota será ignorada por Kusto si la estrategia insinuada no es aplicable a la operación de combinación.

::: zone-end

::: zone pivot="azuremonitor"

Esto no se admite en Azure Monitor

::: zone-end
