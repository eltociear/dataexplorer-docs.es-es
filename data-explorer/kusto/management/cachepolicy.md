---
title: 'Directiva de caché (caché en caliente y en frío): Azure Explorador de datos'
description: En este artículo se describe la Directiva de caché (caché en caliente y en frío) en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2020
ms.openlocfilehash: 130526b41030ac3936236f8fd8bba81f20b4bb0e
ms.sourcegitcommit: 188f89553b9d0230a8e7152fa1fce56c09ebb6d6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/08/2020
ms.locfileid: "84512527"
---
# <a name="cache-policy-hot-and-cold-cache"></a>Directiva de caché (caché en caliente y en frío) 

Azure Explorador de datos almacena sus datos incurridos en un almacenamiento confiable (normalmente Azure Blob Storage), fuera de sus nodos de procesamiento real (como Azure Compute). Para acelerar las consultas en esos datos, Azure Explorador de datos almacena en caché, o parte de ellos, en sus nodos de procesamiento, SSD o incluso en la RAM. Azure Explorador de datos incluye un sofisticado mecanismo de caché diseñado para decidir de forma inteligente Qué objetos de datos se deben almacenar en caché. La memoria caché permite a Azure Explorador de datos describir los artefactos de datos que usa, por lo que los datos más importantes pueden tener prioridad. Por ejemplo, los índices de columna y las particiones de datos de columna,

El mejor rendimiento de las consultas se consigue cuando todos los datos introducidos se almacenan en caché. A veces, ciertos datos no justifican el costo de mantenerlos "semiactivos" en el almacenamiento de SSD local.
Por ejemplo, muchos equipos tienen en cuenta que los registros más antiguos a los que se accede rara vez tienen menos importancia.
Prefieren tener un rendimiento reducido al consultar estos datos, en lugar de pagar para mantenerlos en caliente todo el tiempo.

Caché de Explorador de datos de Azure proporciona una **Directiva de caché** granular que los clientes pueden usar para diferenciar entre: caché de **datos activa** y caché de **datos en frío**. La memoria caché de Explorador de datos de Azure intenta mantener todos los datos que se encuentran en la categoría de caché de datos activa, en SSD (o RAM) local, hasta el tamaño definido de la caché de datos activa. El espacio de SSD local restante se usará para almacenar los datos que no están clasificados como activos. Una implicación útil de este diseño es que las consultas que cargan muchos datos inactivos de un almacenamiento confiable no extraerán datos de la memoria caché de datos activa. Como resultado, no habrá un impacto importante en las consultas que impliquen los datos en la memoria caché de datos activa.

Las principales implicaciones de establecer la Directiva de caché activa son las siguientes:
* **Costo**: el costo del almacenamiento confiable puede ser mucho menor que el de SSD local. Actualmente, es aproximadamente 45 veces más baratas en Azure.
* **Rendimiento**: los datos se consultan más rápido cuando se encuentra en un SSD local, especialmente en el caso de consultas por rangos que examinan grandes cantidades de datos.  

Use el [comando Directiva de caché](cache-policy.md) para administrar la Directiva de caché.

> [!TIP]
>Azure Explorador de datos está diseñado para consultas ad hoc con conjuntos de resultados intermedios que ajustan la RAM total del clúster.
>En el caso de trabajos grandes, como Map-reduce, donde desea almacenar los resultados intermedios en un almacenamiento persistente, como una SSD, use la característica de exportación continua. Esta característica permite realizar consultas por lotes de ejecución prolongada mediante servicios como HDInsight o Azure Databricks.
 
## <a name="how-cache-policy-is-applied"></a>Cómo se aplica la Directiva de caché

Cuando los datos se introducen en Azure Explorador de datos, el sistema realiza un seguimiento de la fecha y la hora de la ingesta y del grado de creación. El valor de fecha y hora de ingesta de la extensión (o el valor máximo, si se compiló una extensión a partir de varias extensiones preexistentes), se usa para evaluar la Directiva de caché.

> [!Note]
> Puede especificar un valor para la fecha y la hora de la ingesta mediante la propiedad ingesta `creationTime` .

De forma predeterminada, la Directiva efectiva es `null` , lo que significa que todos los datos se consideran **activos**.
Una directiva que no es de `null` tabla invalida una directiva de nivel de base de datos.

## <a name="scoping-queries-to-hot-cache"></a>Ámbito de consultas en caché activa

Kusto admite las consultas que se limitan a los datos de caché de acceso frecuente.
Hay varias posibilidades de consulta.

- Agregue una propiedad de solicitud de cliente denominada `query_datascope` a la consulta.
   Valores posibles: `default` , `all` y `hotcache` .
- Use una `set` instrucción en el texto de la consulta: `set query_datascope='...'` .
   Los valores posibles son los mismos que para la propiedad de solicitud de cliente.
- Agregue un `datascope=...` texto inmediatamente después de una referencia de tabla en el cuerpo de la consulta. 
   Los valores posibles son `all` y `hotcache`.

El `default` valor indica el uso de la configuración predeterminada del clúster, que determina que la consulta debe cubrir todos los datos.

Si hay una discrepancia entre los distintos métodos, `set` tiene prioridad sobre la propiedad de solicitud de cliente. La especificación de un valor para una referencia de tabla tiene prioridad sobre ambos.

Por ejemplo, en la siguiente consulta, todas las referencias de tabla usarán solo los datos de la memoria caché activa, salvo la segunda referencia a "T", que se limita a todos los datos:

```kusto
set query_datascope="hotcache";
T | union U | join (T datascope=all | where Timestamp < ago(365d) on X
```

## <a name="cache-policy-vs-retention-policy"></a>Directiva de caché frente a Directiva de retención

La Directiva de caché es independiente de la [Directiva de retención](./retentionpolicy.md): 
- La Directiva de caché define cómo priorizar los recursos. Las consultas de datos importantes serán más rápidas y resistentes al impacto de las consultas en datos menos importantes.
- La Directiva de retención define el alcance de los datos consultables en una tabla o base de datos (concretamente, `SoftDeletePeriod` ).

Configure esta directiva para lograr el equilibrio óptimo entre costo y rendimiento, en función del patrón de consulta esperado.

Ejemplo:
* `SoftDeletePeriod`= 56D
* `hot cache policy`= 28D

En el ejemplo, los últimos 28 días de datos se encontrarán en el SSD del clúster y los 28 días adicionales de los datos se almacenarán en el almacenamiento de blobs de Azure.
Puede ejecutar consultas en los datos completos de 56 días.
 
