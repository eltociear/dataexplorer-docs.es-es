---
title: 'Exportación continua de datos: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe la exportación continua de datos en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: yifats
ms.service: data-explorer
ms.topic: reference
ms.date: 03/27/2020
ms.openlocfilehash: 4ea4532d8547011b2b281988ff1534cd1d49da86
ms.sourcegitcommit: 9fe6e34ef3321390ee4e366819ebc9b132b3e03f
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/01/2020
ms.locfileid: "84258086"
---
# <a name="continuous-data-export"></a>Exportación de datos continua

Exportar datos continuamente de Kusto a una [tabla externa](../externaltables.md). La tabla externa define el destino (por ejemplo, Azure Blob Storage) y el esquema de los datos exportados. Los datos exportados se definen mediante una consulta de ejecución periódica. Los resultados se almacenan en la tabla externa. El proceso garantiza que todos los registros se exporten "exactamente una vez" (excluidas las tablas de dimensiones, en las que todos los registros se evalúan en todas las ejecuciones). 

La exportación continua de datos requiere que [cree una tabla externa](../external-tables-azurestorage-azuredatalake.md#create-or-alter-external-table) y, a continuación, [cree una definición de exportación continua](#create-or-alter-continuous-export) que apunte a la tabla externa. 

> [!NOTE] 
> * Kusto no admite la exportación de registros históricos ingeridos antes de la creación de la exportación continua (como parte de la exportación continua). Los registros históricos se pueden exportar por separado mediante el [comando de exportación](export-data-to-an-external-table.md)(no continuo). Para obtener más información, consulte [exportar datos históricos](#exporting-historical-data).
> * La exportación continua no funciona para los datos ingeridos mediante la ingesta de streaming. 
> * Actualmente, no se puede configurar la exportación continua en una tabla en la que está habilitada una [Directiva de seguridad de nivel de fila](../../management/rowlevelsecuritypolicy.md) .
> * La exportación continua no se admite para las tablas externas con `impersonate` en sus [cadenas de conexión](../../api/connection-strings/storage.md).
> * Si los artefactos usados por la exportación continua están diseñados para desencadenar notificaciones de Event Grid, consulte la [sección problemas conocidos de la documentación de Event Grid](../data-ingestion/eventgrid.md#known-issues).

## <a name="notes"></a>Notas

* La garantía de la exportación "exactamente una vez" solo es para los archivos que se muestran en el [comando Mostrar artefactos exportados](#show-continuous-export-artifacts). 
La exportación continua no garantiza que cada registro se escribirá una sola vez en la tabla externa. Si se produce un error una vez iniciada la exportación y ya se han escrito algunos artefactos en la tabla externa, la tabla externa puede contener Duplicados (o incluso archivos dañados, en caso de _que_ se haya anulado la operación de escritura antes de la finalización). En tales casos, los artefactos no se eliminan de la tabla externa, pero *no* se mostrarán en el [comando Mostrar artefactos exportados](#show-continuous-export-artifacts). Consumo de los archivos exportados mediante `show exported artifacts command` . 
 no garantiza ningún duplicado (y no daños).
* Para garantizar la exportación "exactamente una vez", la exportación continua utiliza [cursores de base de datos](../databasecursor.md). 
La [Directiva IngestionTime](../ingestiontime-policy.md) debe estar habilitada en todas las tablas a las que se hace referencia en la consulta que se deben procesar "exactamente una vez" en la exportación. La Directiva está habilitada de forma predeterminada en todas las tablas recién creadas.
* El esquema de salida de la consulta de exportación *debe* coincidir con el esquema de la tabla externa a la que se exporta. 
* La exportación continua no admite llamadas entre bases de datos o clústeres.
* La exportación continua se ejecuta según el período de tiempo configurado para ella. El valor recomendado para este intervalo es al menos varios minutos, en función de las latencias que desee aceptar. La exportación continua *no está* diseñada para los datos de streaming constantemente fuera de Kusto. Se ejecuta en un modo distribuido, donde todos los nodos se exportan simultáneamente. Si el intervalo de datos consultado por cada ejecución es pequeño, la salida de la exportación continua serían muchos artefactos pequeños (el número depende del número de nodos del clúster). 
* El número de operaciones de exportación que se pueden ejecutar simultáneamente está limitado por la capacidad de exportación de datos del clúster (consulte [limitación](../../management/capacitypolicy.md#throttling)). Si el clúster no tiene suficiente capacidad para administrar todas las exportaciones continuas, algunas comenzarán atrás. 
 
* De forma predeterminada, se supone que todas las tablas a las que se hace referencia en la consulta Export son [tablas de hechos](../../concepts/fact-and-dimension-tables.md). 
Por lo tanto, tienen como *ámbito* el cursor de base de datos. La consulta de exportación solo incluye los registros que se han unido desde la ejecución de exportación anterior. 
La consulta de exportación puede contener [tablas de dimensiones](../../concepts/fact-and-dimension-tables.md) en las que *todos* los registros de la tabla de dimensiones se incluyen en *todas* las consultas de exportación. 
    * Al usar combinaciones entre las tablas de hechos y de dimensiones en la exportación continua, tenga en cuenta que los registros de la tabla de hechos solo se procesan una vez: Si la exportación se ejecuta mientras faltan los registros de las tablas de dimensiones en algunas claves, los registros de las claves respectivas se omitirán o incluirán valores NULL para las columnas de dimensión de los archivos exportados (dependiendo de si la consulta usa la combinación interna o externa). La propiedad forcedLatency de la definición de exportación continua puede ser útil para estos casos, donde las tablas de hechos y dimensiones se ingestan al mismo tiempo (para los registros coincidentes).
    * Continuous: solo se admite la exportación de tablas de dimensiones. La consulta de exportación debe incluir al menos una sola tabla de hechos.
    * La sintaxis declara explícitamente las tablas cuyo ámbito es (FACT) y que no tienen ámbito (dimensión). Vea el `over` parámetro en el [comando CREATE](#create-or-alter-continuous-export) para obtener más información.

* El número de archivos exportados en cada iteración de exportación continua depende de cómo se cree la partición de la tabla externa. Para obtener más información, vea la sección notas en el [comando exportar a tabla externa](export-data-to-an-external-table.md). Cada iteración continua de exportación siempre escribe en *nuevos* archivos y nunca se anexa a los existentes. Como resultado, el número de archivos exportados depende también de la frecuencia con la que se ejecuta la exportación continua ( `intervalBetweenRuns` parámetro).

Todos los comandos de exportación continua requieren [permisos de administrador de base de datos](../access-control/role-based-authorization.md).

## <a name="create-or-alter-continuous-export"></a>Crear o modificar exportación continua

**Sintaxis:**

`.create-or-alter``continuous-export` *ContinuousExportName* <br>
[ `over` `(` *T1*, *T2* `)` ] <br>
`to``table` *ExternalTableName* <br> [ `with` `(` *PropertyName* `=` *PropertyValue* `,` ... `)` ]<br>
\<| *Misma*

**Propiedades**:

| Propiedad.             | Tipo     | Descripción   |
|----------------------|----------|---------------------------------------|
| ContinuousExportName | String   | Nombre de la exportación continua. El nombre debe ser único en la base de datos y se usa para ejecutar periódicamente la exportación continua.      |
| ExternalTableName    | String   | Nombre de la [tabla externa](../externaltables.md) a la que se va a exportar.  |
| Consulta                | String   | Consulta que se va a exportar.  |
| Over (T1, T2)        | String   | Lista opcional separada por comas de las tablas de hechos de la consulta. Si no se especifica, se supone que todas las tablas a las que se hace referencia en la consulta son tablas de hechos. Si se especifica, las tablas que *no* están en esta lista se tratan como tablas de dimensiones y no se limitarán (todos los registros participarán en todas las exportaciones). Vea la [sección Notas](#notes) para obtener más información. |
| intervalBetweenRuns  | TimeSpan | El intervalo de tiempo entre ejecuciones de exportación continuas. Debe ser mayor que 1 minuto.   |
| forcedLatency        | TimeSpan | Un período de tiempo opcional para limitar la consulta a los registros que se ingeriron solo antes de este período (en relación con la hora actual). Esta propiedad es útil si, por ejemplo, la consulta realiza algunas agregaciones o combinaciones y desea asegurarse de que todos los registros relevantes ya se han ingerido antes de ejecutar la exportación.

Además de lo anterior, todas las propiedades que se admiten en el [comando Export to external Table](export-data-to-an-external-table.md) se admiten en el comando Continuous Export Create. 

**Ejemplo**:

```kusto
.create-or-alter continuous-export MyExport
over (T)
to table ExternalBlob
with
(intervalBetweenRuns=1h, 
 forcedLatency=10m, 
 sizeLimit=104857600)
<| T
```

| Nombre     | ExternalTableName | Consulta | ForcedLatency | IntervalBetweenRuns | CursorScopedTables         | ExportProperties                   |
|----------|-------------------|-------|---------------|---------------------|----------------------------|------------------------------------|
| Exportar | ExternalBlob      | S     | 00:10:00      | 01:00:00            | [<br>  "[' DB ']. ['] '<br>] | {<br>  "SizeLimit": 104857600<br>} |

## <a name="show-continuous-export"></a>Mostrar exportación continua

**Sintaxis:**

`.show``continuous-export` *ContinuousExportName*

Devuelve las propiedades de exportación continua de *ContinuousExportName*. 

**Propiedades**

| Propiedad.             | Tipo   | Descripción                |
|----------------------|--------|----------------------------|
| ContinuousExportName | String | Nombre de la exportación continua. |


`.show` `continuous-exports`

Devuelve todas las exportaciones continuas de la base de datos. 

**Salida:**

| Parámetro de salida    | Tipo     | Descripción                                                             |
|---------------------|----------|-------------------------------------------------------------------------|
| CursorScopedTables  | String   | Lista de tablas de ámbito explícito (FACT) (JSON serializado)               |
| ExportProperties    | String   | Exportar propiedades (JSON serializado)                                     |
| Exportado          | DateTime | Última fecha y hora de la ingesta que se exportó correctamente       |
| ExternalTableName   | String   | Nombre de la tabla externa                                              |
| ForcedLatency       | TimeSpan | Latencia forzada (NULL si no se proporciona)                                   |
| IntervalBetweenRuns | TimeSpan | Intervalo entre ejecuciones                                                   |
| IsDisabled          | Boolean  | True si la exportación continua está deshabilitada                               |
| IsRunning (           | Boolean  | True si la exportación continua se está ejecutando actualmente                      |
| LastRunResult       | String   | Los resultados de la última ejecución de la exportación continua ( `Completed` o `Failed` ) |
| LastRunTime         | DateTime | La última vez que se ejecutó la exportación continua (hora de inicio)           |
| Nombre                | String   | Nombre de la exportación continua                                           |
| Consulta               | String   | Exportación de consultas                                                            |
| StartCursor         | String   | Punto de inicio de la primera ejecución de esta exportación continua         |

## <a name="show-continuous-export-artifacts"></a>Mostrar artefactos de exportación continua

**Sintaxis:**

`.show``continuous-export` *ContinuousExportName*`exported-artifacts`

Devuelve todos los artefactos exportados por la exportación continua en todas las ejecuciones. Filtre los resultados por la columna TIMESTAMP del comando para ver solo los registros de interés. El historial de los artefactos exportados se conserva durante 14 días. 

**Propiedades**

| Propiedad.             | Tipo   | Descripción                |
|----------------------|--------|----------------------------|
| ContinuousExportName | String | Nombre de la exportación continua. |

**Salida:**

| Parámetro de salida  | Tipo     | Descripción                            |
|-------------------|----------|----------------------------------------|
| Timestamp         | Datetime | Marca de tiempo de ejecución de la exportación continua |
| ExternalTableName | String   | Nombre de la tabla externa             |
| Ruta de acceso              | String   | Ruta de acceso de resultados                            |
| NumRecords        | long     | Número de registros exportados a la ruta de acceso     |

**Ejemplo**: 

```kusto
.show continuous-export MyExport exported-artifacts | where Timestamp > ago(1h)
```

| Timestamp                   | ExternalTableName | Ruta de acceso             | NumRecords | SizeInBytes |
|-----------------------------|-------------------|------------------|------------|-------------|
| 2018-12-20 07:31:30.2634216 | ExternalBlob      | `http://storageaccount.blob.core.windows.net/container1/1_6ca073fd4c8740ec9a2f574eaa98f579.csv` | 10                          | 1024              |

## <a name="show-continuous-export-failures"></a>Mostrar errores de exportación continua

**Sintaxis:**

`.show``continuous-export` *ContinuousExportName*`failures`

Devuelve todos los errores registrados como parte de la exportación continua. Filtre los resultados por la columna TIMESTAMP del comando para ver solo el intervalo de tiempo de interés. 

**Propiedades**

| Propiedad.             | Tipo   | Descripción                |
|----------------------|--------|----------------------------|
| ContinuousExportName | String | Nombre de la exportación continua  |

**Salida:**

| Parámetro de salida | Tipo      | Descripción                                         |
|------------------|-----------|-----------------------------------------------------|
| Timestamp        | Datetime  | Marca de tiempo del error.                           |
| OperationId      | String    | IDENTIFICADOR de la operación del error.                    |
| Nombre             | String    | Nombre de la exportación continua.                             |
| LastSuccessRun   | Timestamp | Última ejecución correcta de la exportación continua.   |
| FailureKind      | String    | Error/PartialFailure. PartialFailure indica que algunos artefactos se exportaron correctamente antes de que se produjera el error. |
| Detalles          | String    | Detalles del error.                              |

**Ejemplo**: 

```kusto
.show continuous-export MyExport failures 
```

| Timestamp                   | OperationId                          | Nombre     | LastSuccessRun              | FailureKind | Detalles    |
|-----------------------------|--------------------------------------|----------|-----------------------------|-------------|------------|
| 2019-01-01 11:07:41.1887304 | ec641435-2505-4532-ba19-d6ab88c96a9d | Exportar | 2019-01-01 11:06:35.6308140 | Error     | Detalles... |

## <a name="drop-continuous-export"></a>Quitar exportación continua

**Sintaxis:**

`.drop``continuous-export` *ContinuousExportName*

**Propiedades**

| Propiedad.             | Tipo   | Descripción                |
|----------------------|--------|----------------------------|
| ContinuousExportName | String | Nombre de la exportación continua |

**Salida:**

Las exportaciones continuas restantes en la base de datos (posterior a la eliminación). Esquema de salida como en el [comando Mostrar exportación continua](#show-continuous-export).

## <a name="disable-or-enable-continuous-export"></a>Deshabilitar o habilitar la exportación continua

**Sintaxis:**

`.enable``continuous-export` *ContinuousExportName* 

`.disable``continuous-export` *ContinuousExportName* 

Puede deshabilitar o habilitar el trabajo de exportación continua. Una exportación continua deshabilitada no se ejecutará, pero su estado actual es persistente y se puede reanudar cuando la exportación continua está habilitada. Al habilitar una exportación continua que se ha deshabilitado durante mucho tiempo, la exportación continuará desde donde se detuvo por última vez cuando se deshabilitó la exportación. Esta continuación puede dar lugar a una exportación de larga ejecución, lo que impide que se ejecuten otras exportaciones, si no hay suficiente capacidad de clúster para atender todos los procesos. Las exportaciones continuas se ejecutan en el último tiempo de ejecución en orden ascendente (la exportación más antigua se ejecutará primero, hasta que se complete la puesta al día). 

**Propiedades**

| Propiedad.             | Tipo   | Descripción                |
|----------------------|--------|----------------------------|
| ContinuousExportName | String | Nombre de la exportación continua |

**Salida:**

Resultado del [comando Mostrar exportación continua](#show-continuous-export) de la exportación continua modificada. 




## <a name="exporting-historical-data"></a>Exportar datos históricos

La exportación continua comienza a exportar datos solo desde el punto de su creación. Los registros ingeridos antes de ese tiempo se deben exportar por separado mediante el [comando de exportación](export-data-to-an-external-table.md)(no continuo). Para evitar duplicados con los datos exportados por la exportación continua, use el StartCursor devuelto por el [comando Mostrar exportación continua](#show-continuous-export) y exporte únicamente los registros donde cursor_before_or_at el valor del cursor. Observe el ejemplo siguiente. Es posible que los datos históricos sean demasiado grandes para exportarlos en un solo comando de exportación. Por lo tanto, divida la consulta en varios lotes más pequeños. 

```kusto
.show continuous-export MyExport | project StartCursor
```

| StartCursor        |
|--------------------|
| 636751928823156645 |

Seguido de: 

```kusto
.export async to table ExternalBlob
<| T | where cursor_before_or_at("636751928823156645")
```
