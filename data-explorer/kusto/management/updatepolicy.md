---
title: 'Directiva de actualización de Kusto para los datos agregados a un origen: Azure Explorador de datos'
description: En este artículo se describe la Directiva de actualización en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2020
ms.openlocfilehash: 072c908109fecb695a8961c546deb756caf830ab
ms.sourcegitcommit: 98eabf249b3f2cc7423dade0f386417fb8e36ce7
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2020
ms.locfileid: "82868704"
---
# <a name="update-policy"></a>Actualización de directiva

La Directiva de actualización se establece en una **tabla de destino** que indica a Kusto que anexe automáticamente datos a ella cada vez que se inserten nuevos datos en la **tabla de origen**. La **consulta** de la Directiva de actualización se ejecuta en los datos insertados en la tabla de origen. Esto permite, por ejemplo, la creación de una tabla como la vista filtrada de otra tabla, posiblemente con un esquema diferente, una directiva de retención, etc.

De forma predeterminada, el hecho de no ejecutar la Directiva de actualización no afecta a la ingesta de datos en la tabla de origen. Si la Directiva de actualización se define como **transaccional**, si se produce un error al ejecutar la Directiva de actualización se fuerza también la ingesta de datos en la tabla de origen. (Se debe tener cuidado cuando esto se hace porque algunos errores de usuario, como la definición de una consulta incorrecta en la Directiva de actualización, pueden impedir que se ingesta **ningún** dato en la tabla de origen). Los datos ingeridos en el "límite" de las directivas de actualización transaccional están disponibles para una consulta en una única transacción.

La consulta de la Directiva de actualización se ejecuta en un modo especial, en el que "solo ve" los datos recién incorporados en la tabla de origen. No es posible consultar los datos ya ingestados de la tabla de origen como parte de esta consulta. Si lo hace rápidamente, se generan ingesta de tiempo cuadrático.

Dado que la Directiva de actualización se define en la tabla de destino, la ingesta de datos en una tabla de origen puede dar lugar a que se ejecuten varias consultas en esos datos. El orden de ejecución de las directivas de actualización es indefinido.

Por ejemplo, supongamos que la tabla de origen es una tabla de seguimiento de alta tasa con datos interesantes con formato de columna de texto libre. Supongamos también que la tabla de destino (en la que se define la Directiva de actualización) solo acepta líneas de seguimiento específicas y un esquema bien estructurado que es una transformación de los datos de texto libre originales mediante el [operador Parse](../query/parseoperator.md)de Kusto.

La Directiva de actualización se comporta de forma similar a la ingesta normal y está sujeta a las mismas restricciones y prácticas recomendadas. Por ejemplo, se escala horizontalmente con el tamaño del clúster y funciona de forma más eficaz si se realizan ingesta en grandes cantidades masivas.

## <a name="commands-that-trigger-the-update-policy"></a>Comandos que desencadenan la Directiva de actualización

Las directivas de actualización surten efecto cuando se introducen o se mueven datos a (se crean extensiones en) una tabla con cualquiera de los siguientes comandos:

* [. ingesta (extracción)](../management/data-ingestion/ingest-from-storage.md)
* [. ingesta (insertado)](../management/data-ingestion/ingest-inline.md)
* [. set |. Append |. set-o-Append |. Set-or-Replace](../management/data-ingestion/ingest-from-query.md)
* [. migrar extensiones](../management/extents-commands.md#move-extents)
* [. reemplazar extensiones](../management/extents-commands.md#replace-extents)

## <a name="the-update-policy-object"></a>El objeto de directiva de actualización

Una tabla puede tener cero, uno o más objetos de directiva de actualización asociados.
Cada objeto de este tipo se representa como una bolsa de propiedades JSON, con las siguientes propiedades definidas:

|Propiedad |Tipo |Descripción  |
|---------|---------|----------------|
|IsEnabled                     |`bool`  |Indica si la Directiva de actualización está habilitada (true) o deshabilitada (false)                                                                                                                               |
|Source                        |`string`|Nombre de la tabla que desencadena la Directiva de actualización que se va a invocar                                                                                                                                 |
|Consultar                         |`string`|Una consulta de Kusto CSL que se usa para generar los datos de la actualización                                                                                                                           |
|IsTransactional               |`bool`  |Indica si la Directiva de actualización es transaccional o no (el valor predeterminado es false). Si no se ejecuta una directiva de actualización transaccional, el resultado de la tabla de origen no se actualiza también con los nuevos datos.   |
|PropagateIngestionProperties  |`bool`  |Indica si las propiedades de ingesta (etiquetas de extensión y hora de creación) especificadas durante la ingesta en la tabla de origen se deben aplicar también a las de la tabla derivada.                 |

## <a name="notes"></a>Notas

* La consulta se limita automáticamente para abarcar solo los registros que se acaban de ingesta.
* La consulta puede invocar funciones almacenadas.
* Se permiten las actualizaciones en cascada`TableA` ( `TableB` → `TableC` → →...)
* Cuando la Directiva de actualización se invoca como parte de un `.set-or-replace` comando, el comportamiento predeterminado es que también se reemplazan los datos de las tablas derivadas, tal como se encuentra en la tabla de origen.

## <a name="limitations"></a>Limitaciones

* La tabla de origen y la tabla para la que se define la Directiva de actualización **deben estar en la misma base de datos**.
* La consulta **no** puede incluir consultas entre bases de datos ni entre clústeres.
* En caso de que las directivas de actualización se definan en varias tablas de una manera circular, esto se detecta en tiempo de ejecución y la cadena de actualizaciones se corta (lo que significa que los datos se inscribirán una sola vez en cada tabla de la cadena de tablas afectadas).
* Al hacer referencia `Source` a la tabla `Query` en la parte de la Directiva (o en las funciones a las que hace referencia la última), asegúrese de que **no** usa el nombre completo de la `TableName` tabla (es `cluster("ClusterName").database("DatabaseName").TableName`decir, use, y **no** `database("DatabaseName").TableName` ).
* Una consulta que se ejecuta como parte de una directiva de actualización **no tiene acceso** de lectura a las tablas que tienen habilitada la [Directiva RestrictedViewAccess](restrictedviewaccesspolicy.md) .
* La consulta de la Directiva de actualización no puede hacer referencia a ninguna tabla con una [Directiva de seguridad de nivel de fila](./rowlevelsecuritypolicy.md) que esté habilitada.
* `PropagateIngestionProperties`solo surte efecto en las operaciones de ingesta. Cuando la Directiva de actualización se desencadena como parte de un `.move extents` comando `.replace extents` o, esta opción **no tiene ningún** efecto.

## <a name="retention-policy-on-the-source-table"></a>Directiva de retención en la tabla de origen

Por lo tanto, para no conservar los datos sin procesar en la tabla de origen, puede establecer un período de eliminación temporal de 0 en la [Directiva de retención](retentionpolicy.md)de la tabla de origen, al tiempo que establece la Directiva de actualización como transaccional.

Esto resultará con:
* No se pudieron consultar los datos de origen de la tabla de origen.
* Los datos de origen no se conservan en el almacenamiento duradero como parte de la operación de ingesta.
* Mejora en el rendimiento de la operación.
* Reducción de los recursos utilizados después de la ingesta, para las operaciones de limpieza en segundo plano realizadas en las [extensiones](../management/extents-overview.md) de la tabla de origen.

## <a name="failures"></a>Errores

En algunos casos, la ingesta de datos en la tabla de origen se realiza correctamente, pero se produce un error en la Directiva de actualización durante la ingesta en la tabla de destino.

Los errores encontrados durante la actualización de las directivas se pueden recuperar mediante el [comando. Mostrar errores de ingesta](../management/ingestionfailures.md), como se indica a continuación:
 
```kusto
.show ingestion failures 
| where FailedOn > ago(1hr) and OriginatesFromUpdatePolicy == true
```

Los errores se tratan de la siguiente manera:

* **Directiva no transaccional**: Kusto omite el error. Cualquier reintento es responsabilidad del propietario de los datos.  
* **Directiva transaccional**: también se producirá un error en la operación de ingesta original que desencadenó la actualización. La tabla de origen y la base de datos no se modificarán con datos nuevos.
  * En caso de que el método de `pull` ingesta sea (el servicio administración de datos de Kusto está implicado en el proceso de ingesta), hay un reintento automático en toda la operación de ingesta, organizado por el servicio administración de datos de Kusto, según la lógica siguiente:
    * Los reintentos se realizan hasta alcanzar el más `DataImporterMaximumRetryPeriod` antiguo entre (valor predeterminado = 2 `DataImporterMaximumRetryAttempts` días) y (valor predeterminado = 10).
    * La configuración anterior se puede modificar en la configuración del servicio Administración de datos, por KustoOps.
    * El período de interrupción comienza en 2 minutos y crece exponencialmente (2-> 4-> 8-> 16... tiempo
  * En cualquier otro caso, cualquier reintento es responsabilidad del propietario de los datos.



## <a name="control-commands"></a>Comandos de control

* Use [. show TABLE Table Update](../management/update-policy.md#show-update-policy) de la Directiva para mostrar la Directiva de actualización actual de una tabla.
* Use [. ALTER TABLE Table Update](../management/update-policy.md#alter-update-policy) para establecer la Directiva de actualización actual de una tabla.
* Use [. Alter-Merge TABLE Table Update](../management/update-policy.md#alter-merge-table-table-policy-update) de la Directiva para anexar a la Directiva de actualización actual de una tabla.
* Use la [actualización de directiva](../management/update-policy.md#delete-table-table-policy-update) de tabla de tabla para anexar a la Directiva de actualización actual de una tabla.

## <a name="testing-an-update-policys-performance-impact"></a>Probar el impacto en el rendimiento de una directiva de actualización

La definición de una directiva de actualización puede afectar al rendimiento de un clúster de Kusto porque afecta a cualquier ingesta en la tabla de origen. Se recomienda encarecidamente que la `Query` parte de la Directiva de actualización esté optimizada para funcionar correctamente.
Puede probar el impacto adicional en el rendimiento de una directiva de actualización en una operación de ingesta al invocarlo en extensiones específicas y ya existentes, antes de crear o modificar la Directiva o una función que usa en su `Query` parte.

En el ejemplo siguiente se asume lo siguiente:

* El nombre de la tabla de `Source` origen (la propiedad de la directiva `MySourceTable`de actualización) es.
* La `Query` propiedad de la Directiva de actualización llama a una `MyFunction()`función denominada.

Mediante el uso de [. show queries](../management/queries.md), puede evaluar el uso de recursos (CPU, memoria, etc.) de la consulta siguiente y/o varias ejecuciones de la misma.

```kusto
.show table MySourceTable extents;
// The following line provides the extent ID for the not-yet-merged extent in the source table which has the most records
let extentId = $command_results | where MaxCreatedOn > ago(1hr) and MinCreatedOn == MaxCreatedOn | top 1 by RowCount desc | project ExtentId;
let MySourceTable = MySourceTable | where extent_id() == toscalar(extentId);
MyFunction()
```
