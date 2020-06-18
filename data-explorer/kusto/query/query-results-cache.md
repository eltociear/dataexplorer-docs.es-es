---
title: 'Caché de resultados de la consulta: Azure Explorador de datos'
description: En este artículo se describe la funcionalidad de caché de resultados de consulta en Azure Explorador de datos.
services: data-explorer
author: amitof
ms.author: amitof
ms.reviewer: orspodek
ms.service: data-explorer
ms.topic: reference
ms.date: 06/16/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 3c1e1eec2bb0ad4d856ca690039dea9aedd78e8f
ms.sourcegitcommit: a8575e80c65eab2a2118842e59f62aee0ff0e416
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/17/2020
ms.locfileid: "84943129"
---
# <a name="query-results-cache"></a>Caché de resultados de consulta

Kusto incluye una caché de resultados de la consulta. Puede indicar la disposición para obtener resultados almacenados en caché al emitir una consulta. Si la memoria caché puede devolver los resultados de la consulta, tendrá un mejor rendimiento de las consultas y un menor consumo de recursos a costa de una "obsolescencia" en los resultados.

## <a name="indicating-willingness-to-use-the-cache"></a>Indicar que el uso de la memoria caché está dispuesto

Establezca la `query_results_cache_max_age` opción como parte de la consulta para indicar la disposición de usar la memoria caché de resultados de la consulta. Puede establecer esta opción en el texto de la consulta o como una propiedad de solicitud de cliente. Por ejemplo:

```kusto
set query_results_cache_max_age = time(5m);
GithubEvent
| where CreatedAt > ago(180d)
| summarize arg_max(CreatedAt, Type) by Id
```

El valor de la opción es un `timespan` que indica la "antigüedad" máxima de la memoria caché de resultados, medida a partir de la hora de inicio de la consulta. Más allá del intervalo de tiempo establecido, la entrada de caché está obsoleta y no se volverá a usar. Establecer un valor de 0 equivale a no establecer la opción.

## <a name="how-compatibility-between-queries-is-determined"></a>Cómo se determina la compatibilidad entre consultas

El query_results_cache devuelve los resultados solo para las consultas que se consideran "idénticas" a una consulta almacenada en caché anterior. Dos consultas se consideran idénticas si se cumplen todas las condiciones siguientes:

* Las dos consultas tienen la misma representación (como cadenas UTF-8).

* Las dos consultas se realizan en la misma base de datos.

* Las dos consultas comparten las mismas [propiedades de solicitud de cliente](../api/netfx/request-properties.md). Las siguientes propiedades se omiten para el almacenamiento en caché:
   * [ClientRequestId](../api/netfx/request-properties.md#the-clientrequestid-x-ms-client-request-id-named-property)
   * [Aplicación](../api/netfx/request-properties.md#the-application-x-ms-app-named-property)
   * [User](../api/netfx/request-properties.md#the-user-x-ms-user-named-property)

## <a name="if-the-query-results-cache-cant-find-a-valid-cache-entry"></a>Si la caché de resultados de la consulta no puede encontrar una entrada válida de caché

Si un resultado almacenado en caché que cumple las restricciones de tiempo no se pudo encontrar o no hay un resultado almacenado en caché de una consulta "idéntica" en la memoria caché, la consulta se ejecutará y sus resultados se almacenarán en caché, siempre y cuando: 

* La ejecución de la consulta se completa correctamente y
* El tamaño de los resultados de la consulta no supera los 16 MB.

## <a name="how-the-service-indicates-that-the-query-results-are-being-served-from-the-cache"></a>Cómo indica el servicio que los resultados de la consulta se atienden desde la memoria caché

Al responder a una consulta, Kusto envía una tabla de respuesta [ExtendedProperties](../api/rest/response.md) adicional que incluye una `Key` columna y una `Value` columna.
Los resultados de la consulta en caché tendrán una fila adicional anexada a esa tabla:
* La columna de la fila contendrá `Key` la cadena`ServerCache`
* La columna de la fila contendrá `Value` un contenedor de propiedades con dos campos:
   * `OriginalClientRequestId`: Especifica el [ClientRequestId](../api/netfx/request-properties.md#the-clientrequestid-x-ms-client-request-id-named-property)de la solicitud original.
   * `OriginalStartedOn`: Especifica la hora de inicio de la ejecución de la solicitud original.

## <a name="distribution"></a>Distribución

Los nodos de clúster no comparten la memoria caché. Cada nodo tiene una memoria caché dedicada en su propio almacenamiento privado. Si dos consultas idénticas se encuentran en nodos diferentes, la consulta se ejecutará y se almacenará en caché en ambos nodos. Este proceso puede producirse si se usa la [coherencia débil](../concepts/queryconsistency.md) .

## <a name="management"></a>Administración

Se admiten los siguientes comandos de administración y observación:

* [Mostrar caché](../management/show-query-results-cache-command.md).
* [Borrar caché](../management/clear-query-results-cache-command.md).

## <a name="capacity"></a>Capacity

La capacidad de la memoria caché está fija actualmente en 1 GB por nodo de clúster.
La Directiva de expulsión es LRU.
