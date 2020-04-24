---
title: 'Directiva de actualización: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe la directiva de actualización en El Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2020
ms.openlocfilehash: 812a5af932f69d898bb419631fa6ea3c6bc0795e
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81519404"
---
# <a name="update-policy"></a>Actualización de directiva

La directiva de actualización se establece en una tabla de **destino** que indica a Kusto que le anexe automáticamente datos cada vez que se insertan nuevos datos en la tabla de **origen.** La **consulta** de la directiva de actualización se ejecuta en los datos insertados en la tabla de origen. Esto permite, por ejemplo, la creación de una tabla como vista filtrada de otra tabla, posiblemente con un esquema diferente, directiva de retención, etc.

De forma predeterminada, el error al ejecutar la directiva de actualización no afecta a la ingesta de datos en la tabla de origen. Si la directiva de actualización se define como **transaccional,** el error al ejecutar la directiva de actualización también obliga a producir un error en la ingesta de datos en la tabla de origen. (Se debe tener cuidado cuando esto se hace porque algunos errores de usuario, como la definición de una consulta incorrecta en la directiva de actualización, podrían **impedir** que los datos se ingiere en la tabla de origen.) Los datos ingeridos en el "límite" de las directivas de actualización transaccional están disponibles para una consulta en una sola transacción.

La consulta de la directiva de actualización se ejecuta en un modo especial, en el que "ve" solo los datos recién ingeridos en la tabla de origen. No es posible consultar los datos ya ingeridos de la tabla de origen como parte de esta consulta. Si lo hace, se produce rápidamente una ingestión en el tiempo cuadrático.

Dado que la directiva de actualización se define en la tabla de destino, la ingesta de datos en una tabla de origen puede dar lugar a que se ejecuten varias consultas en esos datos. El orden de ejecución de las directivas de actualización es indefinido.

Por ejemplo, supongamos que la tabla de origen es una tabla de seguimiento de alta velocidad con datos interesantes formateados como una columna de texto libre. Supongamos también que la tabla de destino (en la que se define la directiva de actualización) solo acepta líneas de seguimiento específicas y con un esquema bien estructurado que es una transformación de los datos de texto libre originales mediante el operador de [análisis](../query/parseoperator.md)de Kusto.

La directiva de actualización se comporta de forma similar a la ingesta regular y está sujeta a las mismas restricciones y prácticas recomendadas. Por ejemplo, se escala horizontalmente con el tamaño del clúster y funciona de forma más eficaz si las ingestas se realizan en grandes masas.

## <a name="commands-that-trigger-the-update-policy"></a>Comandos que activan la directiva de actualización

Las directivas de actualización surten efecto cuando los datos se ingingien o se mueven a (se crean extensiones en) una tabla mediante cualquiera de los siguientes comandos:

* [.ingesta (tirar)](../management/data-ingestion/ingest-from-storage.md)
* [.ingesta (en línea)](../management/data-ingestion/ingest-inline.md)
* [.set á .append á .set-or-append .](../management/data-ingestion/ingest-from-query.md)
* [.move extensions](../management/extents-commands.md#move-extents)
* [.reemplazar extensiones](../management/extents-commands.md#replace-extents)

## <a name="the-update-policy-object"></a>El objeto de directiva de actualización

Una tabla puede tener cero, uno o varios objetos de directiva de actualización asociados.
Cada uno de estos objetos se representa como un contenedor de propiedades JSON, con las siguientes propiedades definidas:

|Propiedad |Tipo |Descripción  |
|---------|---------|----------------|
|IsEnabled                     |`bool`  |Estados si la directiva de actualización está habilitada (true) o deshabilitada (false)                                                                                                                               |
|Source                        |`string`|Nombre de la tabla que desencadena la directiva de actualización que se va a invocar                                                                                                                                 |
|Consultar                         |`string`|Una consulta CSL de Kusto que se utiliza para generar los datos de la actualización                                                                                                                           |
|IsTransactional               |`bool`  |Indica si la directiva de actualización es transaccional o no (el valor predeterminado es false). Si no se ejecuta una directiva de actualización transaccional, la tabla de origen no se actualiza con nuevos datos.   |
|PropagateIngestionProperties  |`bool`  |Estados si las propiedades de ingesta (etiquetas de extensión y tiempo de creación) especificadas durante la ingesta en la tabla de origen también deben aplicarse a las de la tabla derivada.                 |

> [!NOTE]
>
> * La tabla de origen y la tabla para la que se define la directiva de actualización deben estar en la misma base de **datos.**
> * Es posible que la consulta **no** incluya consultas entre bases de datos ni entre clústeres.
> * La consulta puede invocar funciones almacenadas.
> * La consulta se limita automáticamente para cubrir los registros recién ingeridos.
> * Se permiten actualizaciones en cascada`[update]`(TableA`[update]`-- -->`[update]`TableB -- --> TableC -- --> ...)
> * En caso de que las directivas de actualización se definan en varias tablas de forma circular, esto se detecta en tiempo de ejecución y se corta la cadena de actualizaciones (lo que significa que los datos se ingerirán solo una vez en cada tabla de la cadena de tablas afectadas).
> * Al hacer `Source` referencia a `Query` la tabla en la parte de la directiva (o en Funciones a las que `TableName` hace referencia este último), asegúrese de **no** usar el nombre completo de la tabla (significado, uso y **no** `database("DatabaseName").TableName` ni). `cluster("ClusterName").database("DatabaseName").TableName`
> * La consulta de la directiva de actualización no puede hacer referencia a ninguna tabla con una directiva de seguridad de [nivel](./rowlevelsecuritypolicy.md) de fila habilitada.
> * Una consulta que se ejecuta como parte de una directiva de actualización **no** tiene acceso de lectura a las tablas que tienen habilitada la [directiva RestrictedViewAccess.](restrictedviewaccesspolicy.md)
> * `PropagateIngestionProperties`sólo tiene efecto en las operaciones de ingesta. Cuando la directiva de actualización `.move extents` se `.replace extents` desencadena como parte de un comando o, esta opción no tiene **ningún** efecto.
> * Cuando se invoca la directiva `.set-or-replace` de actualización como parte de un comando, el comportamiento predeterminado es que los datos de las tablas derivadas también se reemplazan, como en la tabla de origen.

## <a name="retention-policy-on-the-source-table"></a>Política de retención en la tabla de origen

Para no conservar los datos sin procesar en la tabla de origen, puede establecer un período de eliminación temporal de 0 en la directiva de [retención](retentionpolicy.md)de la tabla de origen, al tiempo que establece la directiva de actualización como transaccional.

Esto resultará con:
* Los datos de origen no se pueden consultar desde la tabla de origen.
* Los datos de origen que no se conservan en el almacenamiento duradero como parte de la operación de ingesta.
* Mejora en el rendimiento de la operación.
* Reducción de los recursos utilizados después de la ingestión, para las operaciones de aseo de fondo realizadas en [las extensiones](../management/extents-overview.md) de la tabla de origen.

## <a name="failures"></a>Errores

En algunos casos, la ingesta de datos en la tabla de origen se realiza correctamente, pero se produce un error en la directiva de actualización durante la ingesta a la tabla de destino.

Los errores encontrados mientras se actualizan las directivas se pueden recuperar mediante el [comando .show ingestion failures](../management/ingestionfailures.md), como se indica a continuación:
 
```kusto
.show ingestion failures 
| where FailedOn > ago(1hr) and OriginatesFromUpdatePolicy == true
```

Los errores se tratan de la siguiente manera:

* **Directiva no transaccional:** Kusto omite el error. Cualquier reintento es responsabilidad del propietario de los datos.  
* **Directiva transaccional:** también se producirá un error en la operación de ingesta original que desencadenó la actualización. La tabla de origen y la base de datos no se modificarán con nuevos datos.
  * En caso de que `pull` el método de ingesta sea (el servicio de gestión de datos de Kusto está implicado en el proceso de ingesta), hay un reintento automatizado en toda la operación de ingesta, orquestado por el servicio de gestión de datos de Kusto, de acuerdo con la siguiente lógica:
    * Los reintentos se realizan hasta `DataImporterMaximumRetryPeriod` llegar a la más `DataImporterMaximumRetryAttempts` temprana entre (predeterminado, 2 días) y (predeterminado, 10).
    * Ambos valores anteriores pueden ser alterados en la configuración del servicio de administración de datos, por KustoOps.
    * El período de retroceso comienza a partir de 2 minutos, y crece exponencialmente (2 -> 4 -> 8 -> 16 ... minutos)
  * En cualquier otro caso, cualquier reintento es responsabilidad del propietario de los datos.



## <a name="control-commands"></a>Comandos de control

* Utilice la actualización de directiva TABLE de la [tabla .show](../management/update-policy.md#show-update-policy) para mostrar la directiva de actualización actual de una tabla.
* Utilice la actualización de directiva TABLE de [la tabla .alter](../management/update-policy.md#alter-update-policy) para establecer la directiva de actualización actual de una tabla.
* Utilice la actualización de directivas TABLE de tabla [de combinación de .alter](../management/update-policy.md#alter-merge-table-table-policy-update) para anexar a la directiva de actualización actual de una tabla.
* Utilice la actualización de directiva TABLE de la [tabla .delete](../management/update-policy.md#delete-table-table-policy-update) para anexar a la directiva de actualización actual de una tabla.

## <a name="testing-an-update-policys-performance-impact"></a>Probar el impacto en el rendimiento de una política de actualización

La definición de una directiva de actualización puede afectar al rendimiento de un clúster de Kusto porque afecta a cualquier ingesta en la tabla de origen. Se recomienda encarecidamente que la `Query` parte de la directiva de actualización esté optimizada para funcionar bien.
Puede probar el impacto adicional de rendimiento de una directiva de actualización en una operación de ingesta invocándola en extensiones `Query` específicas y ya existentes, antes de crear o modificar la directiva o una función que utiliza por su parte.

En el ejemplo siguiente se asume lo siguiente:

* El nombre de `Source` la tabla de origen `MySourceTable`(la propiedad de la directiva de actualización) es .
* La `Query` propiedad de la directiva `MyFunction()`de actualización llama a una función denominada .

Mediante [consultas .show](../management/queries.md), puede evaluar el uso de recursos (CPU, memoria, etc.) de la siguiente consulta y/o varias ejecuciones de la misma.

```kusto
.show table MySourceTable extents;
// The following line provides the extent ID for the not-yet-merged extent in the source table which has the most records
let extentId = $command_results | where MaxCreatedOn > ago(1hr) and MinCreatedOn == MaxCreatedOn | top 1 by RowCount desc | project ExtentId;
let MySourceTable = MySourceTable | where extent_id() == toscalar(extentId);
MyFunction()
```