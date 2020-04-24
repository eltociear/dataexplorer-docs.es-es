---
title: 'Directiva de caché (caché en caliente y en frío): Explorador de azure Data Explorer ( Cache) Microsoft Docs'
description: En este artículo se describe la directiva de caché (caché activa y fría) en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2020
ms.openlocfilehash: 591763ac5d94a8361a4b78c1b199bb05299cc004
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81522141"
---
# <a name="cache-policy-hot-and-cold-cache"></a>Directiva de caché (caché activa y fría)

Azure Data Explorer almacena sus datos ingeridos en almacenamiento confiable (normalmente Azure Blob Storage), lejos de sus nodos de procesamiento real (por ejemplo, Azure Compute). Para acelerar las consultas de esos datos, Azure Data Explorer almacena en caché estos datos (o partes de ellos) en sus nodos de procesamiento, SSD o incluso en RAM. Azure Data Explorer incluye un sofisticado mecanismo de caché diseñado para decidir de forma inteligente qué objetos de datos almacenar en caché. La memoria caché permite a Azure Data Explorer describir los artefactos de datos que usa (como índices de columna y particiones de datos de columna) para que los datos más "importantes" puedan tener prioridad.

Aunque se logra el mejor rendimiento de las consultas cuando se almacenan en caché todos los datos ingeridos, a menudo ciertos datos no justifican el costo de mantenerlos "calientes" en el almacenamiento SSD local.
Por ejemplo, muchos equipos consideran que los registros más antiguos a los que se accede con frecuencia son de menor importancia.
Preferirían tener un rendimiento reducido al consultar estos datos, en lugar de pagar para mantenerlos calientes todo el tiempo.

La caché del Explorador de datos de Azure proporciona una directiva de **caché** granular que los clientes pueden usar para diferenciar entre dos directivas de caché de datos: caché de **datos en caliente** y caché de datos en **frío.** La caché del Explorador de datos de Azure intenta mantener todos los datos que caen en la caché de datos en caliente en SSD (o RAM) local, hasta el tamaño definido de la caché de datos en caliente. El espacio SSD local restante se utilizará para contener datos que no se clasifican como calientes. Una implicación útil de este diseño es que las consultas que cargan una gran cantidad de datos fríos desde un almacenamiento confiable no expulsarán los datos de la caché de datos en caliente. Por lo tanto, no habrá un impacto importante en las consultas que impliquen los datos en la caché de datos en caliente.

Las principales implicaciones de establecer la directiva de caché en caliente son:
* **Costo** El costo del almacenamiento confiable puede ser dramáticamente menor que el SSD local (por ejemplo, en Azure actualmente es unas 45 veces más barato).
* **Rendimiento** Los datos se pueden consultar más rápido cuando están en SSD local. Esto es particularmente cierto para las consultas de rango, es decir, las consultas que analizan grandes cantidades de datos.  

[Los comandos](cache-policy.md) de control permiten a los administradores administrar la directiva de caché.

## <a name="how-the-cache-policy-gets-applied"></a>Cómo se aplica la directiva de caché

Cuando se ingenian datos en Azure Data Explorer, el sistema realiza un seguimiento de la fecha y hora en la que se realizó la ingesta y se creó la extensión. El valor de fecha y hora de ingesta de la extensión (o valor máximo, si se creó una extensión a partir de varias extensiones preexistentes) se utiliza para evaluar la directiva de caché.

De forma predeterminada, `null`la política efectiva es , lo que significa que todos los datos se consideran **calientes.**
Una directiva`null` que no es de nivel de tabla reemplaza una directiva de nivel de base de datos.

> [!Note] 
> Puede especificar un valor para la fecha y hora `creationTime`de ingesta mediante la propiedad ingestion . 

## <a name="scoping-queries-to-hot-cache"></a>Scoping consultas a la caché en caliente

Kusto admite consultas cuyo ámbito es solo para los datos de caché en caliente. Existen varias formas de hacerlo:

- Agregue una propiedad `query_datascope` de solicitud de `default`cliente `all`llamada `hotcache`a la consulta Valores posibles: , , y .
- Utilice `set` una instrucción en `set query_datascope='...'`el texto de la consulta: , Los valores posibles son los mismos que para la propiedad de solicitud de cliente.
- Agregue `datascope=...` un texto inmediatamente después de una referencia de tabla en el cuerpo de la consulta. Los valores posibles son `all` y `hotcache`.

El `default` valor indica el uso de la configuración predeterminada del clúster, que determina que la consulta debe cubrir todos los datos.



Si hay una discrepancia entre `set` los diferentes métodos: tiene prioridad sobre la propiedad de solicitud de cliente y especificar un valor para una referencia de tabla tiene prioridad sobre ambos.

Por ejemplo, en la siguiente consulta todas las referencias de tabla `T` solo usarán datos hotcache, excepto la segunda referencia a la que se limita a todos los datos:

```kusto
set query_datascope="hotcache";
T | union U | join (T datascope=all | where Timestamp < ago(365d) on X
```

## <a name="cache-policy-vs-retention-policy"></a>Directiva de caché frente a directiva de retención

La directiva de caché es independiente de la directiva de [retención:](./retentionpolicy.md) 
- La directiva de caché define cómo priorizar los recursos para que las consultas sobre datos importantes sean más rápidas y resistentes al impacto de las consultas sobre datos menos importantes. 
- La directiva de retención define la extensión de los `SoftDeletePeriod`datos consultables en una tabla o base de datos (específicamente, ).

Se recomienda configurar esta directiva para lograr el equilibrio óptimo entre el costo y el rendimiento en función del patrón de consulta esperado.

Ejemplo:
* `SoftDeletePeriod`56d
* `hot cache policy`28d

En el ejemplo, los últimos 28 días de datos estarán en el SSD del clúster y los 28 días **adicionales** se almacenarán en Azure Blob Storage. Puede ejecutar consultas en los 56 días completos de datos. 

## <a name="cache-policy-does-not-make-kusto-a-cold-storage-technology"></a>La directiva de caché no convierte a Kusto en una tecnología de almacenamiento en frío

Azure Data Explorer está diseñado para realizar consultas ad hoc, con conjuntos de resultados intermedios que se ajustan a la ram total del clúster. Para trabajos grandes, como map-reduce, donde desearía almacenar resultados intermedios en almacenamiento persistente (como un SSD) utilice la característica de exportación continua. Esto le permite realizar consultas por lotes de larga ejecución mediante servicios como HDInsight o Azure Databricks.