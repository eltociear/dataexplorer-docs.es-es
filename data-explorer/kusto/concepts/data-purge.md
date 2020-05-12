---
title: 'Purga de datos: Azure Explorador de datos'
description: En este artículo se describe la purga de datos en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: kedamari
ms.service: data-explorer
ms.topic: reference
ms.date: 05/12/2020
ms.openlocfilehash: 144e56ee89cb35900b8e55cdbcdce597b26f8a68
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83226011"
---
# <a name="data-purge"></a>Purga de datos

[!INCLUDE [gdpr-intro-sentence](../../includes/gdpr-intro-sentence.md)]

Como plataforma de datos, Azure Explorador de datos admite la posibilidad de eliminar registros individuales mediante el uso de Kusto `.purge` y comandos relacionados. También puede [purgar una tabla completa](#purging-an-entire-table).  

> [!WARNING]
> La eliminación de datos a través del `.purge` comando está diseñada para usarse con el fin de proteger los datos personales y no debe usarse en otros escenarios. No se ha diseñado para admitir solicitudes de eliminación frecuentes o la eliminación de grandes cantidades de datos, y puede tener un impacto significativo en el rendimiento del servicio.

## <a name="purge-guidelines"></a>Instrucciones de purga

Diseñe cuidadosamente el esquema de datos e investigue las directivas pertinentes antes de almacenar los datos personales en Azure Explorador de datos.

1. En el mejor de los casos, el período de retención de estos datos es suficientemente corto y los datos se eliminan automáticamente.
1. Si no es posible el uso del período de retención, aísle todos los datos que están sujetos a las reglas de privacidad en un pequeño número de tablas de Azure Explorador de datos. De forma óptima, use solo una tabla y vincúlelo desde todas las demás tablas. Este aislamiento le permite ejecutar el proceso de [purga](#purge-process) de datos en un número reducido de tablas que contienen datos confidenciales y evitar todas las demás tablas.
1. El autor de la llamada debe realizar todos los intentos de procesar por lotes la ejecución de `.purge` comandos en 1-2 comandos por tabla y día. No emita varios comandos con predicados de identidad de usuario únicos. En su lugar, envíe un solo comando cuyo predicado incluya todas las identidades de usuario que requieren purga.

## <a name="purge-process"></a>Purgar proceso

El proceso de purgar datos de forma selectiva de Azure Explorador de datos sucede en los pasos siguientes:

1. Fase 1: proporcione una entrada con un nombre de tabla de Explorador de datos de Azure y un predicado por registro, que indica qué registros se van a eliminar. Kusto examina la tabla que busca identificar las particiones de datos que participarían en la purga de datos. Las particiones identificadas son aquellas que tienen uno o más registros para los que el predicado devuelve true.
1. Fase 2: (eliminación temporal) Reemplace cada partición de datos de la tabla (identificada en el paso (1)) con una versión reincorporada. La versión reingerida no debe tener los registros para los que el predicado devuelve true. Si no se introducen nuevos datos en la tabla, al final de esta fase, las consultas ya no devolverán los datos para los que el predicado devuelve true. La duración de la fase de eliminación temporal de purga depende de los siguientes parámetros: 
     * El número de registros que se deben purgar 
     * Distribución de registros entre las particiones de datos del clúster 
     * El número de nodos del clúster  
     * La capacidad de reserva que tiene para las operaciones de purga
     * Algunos factores más, la duración de la fase 2 puede variar entre unos segundos y muchas horas.
1. Fase 3: (eliminación temporal) vuelva a trabajar con todos los artefactos de almacenamiento que puedan tener los datos "dudosos" y elimínelos del almacenamiento. Esta fase se realiza al menos cinco días después de la finalización de la fase anterior, pero no más de 30 días después del comando inicial. Estas escalas de tiempo se establecen para cumplir los requisitos de privacidad de datos.

La emisión de un `.purge` comando desencadena este proceso, que tarda unos días en completarse. Si la densidad de registros para los que se aplica el predicado es suficientemente grande, el proceso recobrará todos los datos de la tabla de forma efectiva. Esta rerecopilación tiene un impacto significativo en el rendimiento y en COGS.

## <a name="purge-limitations-and-considerations"></a>Limitaciones y consideraciones de purga

* El proceso de purga es final e irreversible. No es posible deshacer este proceso ni recuperar los datos que se han purgado. Los comandos como [Deshacer eliminación de tabla](../management/undo-drop-table-command.md) no pueden recuperar los datos purgados. La reversión de los datos a una versión anterior no puede ir a antes del último comando de purga.

* Antes de ejecutar la purga, compruebe el predicado ejecutando una consulta y comprobando que los resultados coinciden con el resultado esperado. También puede utilizar el proceso de dos pasos que devuelve el número esperado de registros que se van a purgar. 

* El `.purge` comando se ejecuta en el punto de conexión de administración de datos: `https://ingest-[YourClusterName].[region].kusto.windows.net` .
   El comando requiere permisos de [Administrador de base de datos](../management/access-control/role-based-authorization.md) en las bases de datos pertinentes. 
* Debido al impacto en el rendimiento del proceso de purga y para garantizar que se han seguido las [instrucciones de purga](#purge-guidelines) , se espera que el autor de la llamada modifique el esquema de datos de modo que las tablas mínimas incluyan los datos pertinentes, y los comandos por lotes por tabla para reducir el impacto de Cogs significativo del proceso de purga.
* El `predicate` parámetro del comando [. Purge](#purge-table-tablename-records-command) se usa para especificar los registros que se van a purgar.
`Predicate`el tamaño está limitado a 63 KB. Al construir `predicate` :
    * Use el [operador ' in '](../query/inoperator.md), por ejemplo, `where [ColumnName] in ('Id1', 'Id2', .. , 'Id1000')` . 
    * Tenga en cuenta los límites del [operador ' in '](../query/inoperator.md) (la lista puede contener hasta `1,000,000` valores).
    * Si el tamaño de la consulta es grande, use el [ `externaldata` operador](../query/externaldata-operator.md), por ejemplo `where UserId in (externaldata(UserId:string) ["https://...blob.core.windows.net/path/to/file?..."])` . El archivo almacena la lista de identificadores que se van a purgar.
    * El tamaño total de la consulta, después de expandir todos los `externaldata` BLOBs (tamaño total de todos los blobs), no puede superar los 64 MB. 

## <a name="purge-performance"></a>Purgar rendimiento

En un momento dado, solo se puede ejecutar una solicitud de purga en el clúster. Todas las demás solicitudes se ponen en cola en el `Scheduled` Estado. Supervise el tamaño de la cola de solicitudes de purga y manténgase dentro de los límites adecuados para cumplir los requisitos aplicables a los datos.

Para reducir el tiempo de ejecución de purga:
* Siga las [instrucciones de purga](#purge-guidelines) para reducir la cantidad de datos purgados.
* Ajuste la [Directiva de almacenamiento en caché](../management/cachepolicy.md) , ya que la purga tarda más en datos fríos.
* Escalar horizontalmente el clúster

* Aumente la capacidad de purga de clústeres, después de haber tenido cuidado, tal como se detalla en [extensiones purga la capacidad de reconstrucción](../management/capacitypolicy.md#extents-purge-rebuild-capacity). Para cambiar este parámetro es necesario abrir una [incidencia de soporte técnico](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview)

## <a name="trigger-the-purge-process"></a>Desencadenar el proceso de purga

> [!Note]
> La ejecución de purga se invoca mediante la ejecución del comando [Purge TABLE *TableName* Records](#purge-table-tablename-records-command) en el punto de conexión administración de datos https://ingest- [nombredelclúster]. [ Region]. kusto. Windows. net.

### <a name="purge-table-tablename-records-command"></a>Comando Purge Table TableName Records

El comando Purge se puede invocar de dos maneras para distintos escenarios de uso:

* Invocación mediante programación: un único paso destinado a ser invocado por aplicaciones. La llamada a este comando desencadena directamente la secuencia de ejecución de purga.

    **Sintaxis**

     ```kusto
     // Connect to the Data Management service
     #connect "https://ingest-[YourClusterName].[region].kusto.windows.net" 
     
     .purge table [TableName] records in database [DatabaseName] with (noregrets='true') <| [Predicate]
     ```

    > [!NOTE]
    > Genere este comando mediante la API de CslCommandGenerator, disponible como parte del paquete NuGet de la [biblioteca de cliente de Kusto](../api/netfx/about-kusto-data.md) .

* Invocación humana: proceso de dos pasos que requiere una confirmación explícita como paso independiente. La primera invocación del comando devuelve un token de comprobación, que se debe proporcionar para ejecutar la purga real. Esta secuencia reduce el riesgo de eliminar accidentalmente los datos incorrectos. El uso de esta opción puede tardar mucho tiempo en completarse en tablas grandes con datos de caché en frío significativos.
    <!-- If query times-out on DM endpoint (default timeout is 10 minutes), it is recommended to use the [engine `whatif` command](#purge-whatif-command) directly againt the engine endpoint while increasing the [server timeout limit](../concepts/querylimits.md#limit-on-request-execution-time-timeout). Only after you have verified the expected results using the engine whatif command, issue the purge command via the DM endpoint using the 'noregrets' option. -->

     **Sintaxis**

     ```kusto
     // Connect to the Data Management service
     #connect "https://ingest-[YourClusterName].[region].kusto.windows.net" 
     
     // Step #1 - retrieve a verification token (no records will be purged until step #2 is executed)
     .purge table [TableName] records in database [DatabaseName] <| [Predicate]

     // Step #2 - input the verification token to execute purge
     .purge table [TableName] records in database [DatabaseName] with (verificationtoken='<verification token from step #1>') <| [Predicate]
     ```
    
    | Parámetros  | Descripción  |
    |---------|---------|
    | `DatabaseName`   |   Nombre de la base de datos      |
    | `TableName`     |     Nombre de la tabla    |
    | `Predicate`    |    Identifica los registros que se van a purgar. Consulte purga de las limitaciones de predicado a continuación. | 
    | `noregrets`    |     Si se establece, desencadena una activación de un solo paso.    |
    | `verificationtoken`     |  En el escenario de activación en dos pasos ( `noregrets` no establecido), este token se puede usar para ejecutar el segundo paso y confirmar la acción. Si `verificationtoken` no se especifica, se desencadenará el primer paso del comando. La información sobre la purga se devolverá con un token que debe devolverse al comando para realizar el paso #2.   |

    **Purgar limitaciones de predicado**

    * El predicado debe ser una selección simple (por ejemplo, *Where [ColumnName] = = ' X '*  /  *donde [ColumnName] in (' x ', ' Y ', ' Z ') y [OtherColumn] = = ' a '*).
    * Se deben combinar varios filtros con un ' and ', en lugar de `where` cláusulas independientes (por ejemplo, `where [ColumnName] == 'X' and  OtherColumn] == 'Y'` y not `where [ColumnName] == 'X' | where [OtherColumn] == 'Y'` ).
    * El predicado no puede hacer referencia a tablas distintas de la tabla que se va a purgar (*TableName*). El predicado solo puede incluir la instrucción de selección ( `where` ). No puede proyectar columnas específicas de la tabla (esquema de salida al ejecutar '* `table` | El predicado*' debe coincidir con el esquema de tabla].
    * No se admiten las funciones del sistema (como, `ingestion_time()` , `extent_id()` ).

#### <a name="example-two-step-purge"></a>Ejemplo: purga de dos pasos

Para iniciar la purga en un escenario de activación de dos pasos, ejecute el paso #1 del comando:

    ```kusto
    // Connect to the Data Management service
     #connect "https://ingest-[YourClusterName].[region].kusto.windows.net" 
     
    .purge table MyTable records in database MyDatabase <| where CustomerId in ('X', 'Y')
    ```

    **Output**

    | NumRecordsToPurge | EstimatedPurgeExecutionTime| VerificationToken
    |--|--|--
    | 1.596 | 00:00:02 | e43c7184ed22f4f23c7a9d7b124d196be2e570096987e5baadf65057fa65736b

    Then, validate the NumRecordsToPurge before running step #2. 

Para completar una purga en un escenario de activación en dos pasos, use el token de comprobación devuelto en el paso #1 para ejecutar el paso #2:

    ```kusto
    .purge table MyTable records in database MyDatabase
    with (verificationtoken='e43c7184ed22f4f23c7a9d7b124d196be2e570096987e5baadf65057fa65736b')
    <| where CustomerId in ('X', 'Y')
    ```

    **Output**

    | `OperationId` | `DatabaseName` | `TableName`|`ScheduledTime` | `Duration` | `LastUpdatedOn` |`EngineOperationId` | `State` | `StateDetails` |`EngineStartTime` | `EngineDuration` | `Retries` |`ClientRequestId` | `Principal`|
    |--|--|--|--|--|--|--|--|--|--|--|--|--|--|
    | c9651d74-3b80-4183-90bb-bbe9e42eadc4 |MyDatabase |MyTable |2019-01-20 11:41:05.4391686 |00:00:00.1406211 |2019-01-20 11:41:05.4391686 | |Programado | | | |0 |KE. EjecutarComando; 1d0ad28b-f791-4f5a-A60F-0e32318367b7 |ID. de aplicación de AAD =...|

#### <a name="example-single-step-purge"></a>Ejemplo: purga de un solo paso

Para desencadenar una purga en un escenario de activación de un solo paso, ejecute el siguiente comando:

    ```kusto
    // Connect to the Data Management service
     #connect "https://ingest-[YourClusterName].[region].kusto.windows.net" 
     
    .purge table MyTable records in database MyDatabase with (noregrets='true') <| where CustomerId in ('X', 'Y')
    ```

**Salida**

| `OperationId` |`DatabaseName` |`TableName` |`ScheduledTime` |`Duration` |`LastUpdatedOn` |`EngineOperationId` |`State` |`StateDetails` |`EngineStartTime` |`EngineDuration` |`Retries` |`ClientRequestId` |`Principal`|
|--|--|--|--|--|--|--|--|--|--|--|--|--|--|
| c9651d74-3b80-4183-90bb-bbe9e42eadc4 |MyDatabase |MyTable |2019-01-20 11:41:05.4391686 |00:00:00.1406211 |2019-01-20 11:41:05.4391686 | |Programado | | | |0 |KE. EjecutarComando; 1d0ad28b-f791-4f5a-A60F-0e32318367b7 |ID. de aplicación de AAD =...|

### <a name="cancel-purge-operation-command"></a>Comando Cancelar operación de purga

Si es necesario, puede cancelar las solicitudes de purga pendientes.

> [!NOTE]
> Esta operación está pensada para escenarios de recuperación de errores. No se garantiza que se realice correctamente y no debe formar parte de un flujo operativo normal. Solo se puede aplicar a solicitudes en cola (que aún no se envían al nodo del motor para su ejecución). El comando se ejecuta en el punto de conexión de Administración de datos.

**Sintaxis**

```kusto
 .cancel purge <OperationId>
 ```

**Ejemplo**

```kusto
 .cancel purge aa894210-1c60-4657-9d21-adb2887993e1
 ```

**Salida**

La salida de este comando es la misma que la salida del comando ' show purges *OperationId*', que muestra el estado actualizado de la operación de purga que se va a cancelar. Si el intento es correcto, el estado de la operación se actualiza a `Abandoned` . De lo contrario, el estado de la operación no cambia. 

|`OperationId` |`DatabaseName` |`TableName` |`ScheduledTime` |`Duration` |`LastUpdatedOn` |`EngineOperationId` |`State` |`StateDetails` |`EngineStartTime` |`EngineDuration` |`Retries` |`ClientRequestId` |`Principal`
|--|--|--|--|--|--|--|--|--|--|--|--|--|--|
|c9651d74-3b80-4183-90bb-bbe9e42eadc4 |MyDatabase |MyTable |2019-01-20 11:41:05.4391686 |00:00:00.1406211 |2019-01-20 11:41:05.4391686 | |Abandoned | | | |0 |KE. EjecutarComando; 1d0ad28b-f791-4f5a-A60F-0e32318367b7 |ID. de aplicación de AAD =...

## <a name="track-purge-operation-status"></a>Seguimiento del estado de la operación de purga 

> [!Note]
> Se puede realizar un seguimiento de las operaciones de purga con el comando [Show purges](#show-purges-command) , que se ejecuta en el punto de conexión administración de datos https://ingest- [nombredelclúster]. [ Region]. kusto. Windows. net.

Status = ' Completed ' indica la finalización correcta de la primera fase de la operación de purga, es decir, los registros se eliminan temporalmente y ya no están disponibles para realizar consultas. **No** se espera que los clientes realicen el seguimiento y comprueben la finalización de la segunda fase (eliminación temporal). Azure Explorador de datos supervisa internamente esta fase.

### <a name="show-purges-command"></a>Comando Mostrar purgas

`Show purges`el comando muestra el estado de la operación de purga especificando el identificador de la operación en el período de tiempo solicitado. 

```kusto
.show purges <OperationId>
.show purges [in database <DatabaseName>]
.show purges from '<StartDate>' [in database <DatabaseName>]
.show purges from '<StartDate>' to '<EndDate>' [in database <DatabaseName>]
```

| Propiedades  |Descripción  |Obligatorio/opcional|
|---------|------------|-------|
|`OperationId `   |      El identificador de la operación de Administración de datos de salida después de ejecutar la fase única o la segunda fase.   |Mandatory
|`StartDate`    |   Límite de tiempo inferior para las operaciones de filtrado. Si se omite, el valor predeterminado es 24 horas antes de la hora actual.      |Opcional
|`EndDate`    |  Límite de tiempo superior para las operaciones de filtrado. Si se omite, el valor predeterminado es la hora actual.       |Opcional
|`DatabaseName`    |     Nombre de la base de datos para filtrar los resultados.    |Opcional

> [!NOTE]
> Status solo se proporcionará en las bases de datos que el cliente tenga permisos de [Administrador de base de datos](../management/access-control/role-based-authorization.md) .

**Ejemplos**

```kusto
.show purges
.show purges c9651d74-3b80-4183-90bb-bbe9e42eadc4
.show purges from '2018-01-30 12:00'
.show purges from '2018-01-30 12:00' to '2018-02-25 12:00'
.show purges from '2018-01-30 12:00' to '2018-02-25 12:00' in database MyDatabase
```

**Salida** 

|`OperationId` |`DatabaseName` |`TableName` |`ScheduledTime` |`Duration` |`LastUpdatedOn` |`EngineOperationId` |`State` |`StateDetails` |`EngineStartTime` |`EngineDuration` |`Retries` |`ClientRequestId` |`Principal`
|--|--|--|--|--|--|--|--|--|--|--|--|--|--|
|c9651d74-3b80-4183-90bb-bbe9e42eadc4 |MyDatabase |MyTable |2019-01-20 11:41:05.4391686 |00:00:33.6782130 |2019-01-20 11:42:34.6169153 |a0825d4d-6b0f-47f3-a499-54ac5681ab78 |Completed |Purga completada correctamente (artefactos de almacenamiento pendientes de eliminación) |2019-01-20 11:41:34.6486506 |00:00:04.4687310 |0 |KE. EjecutarComando; 1d0ad28b-f791-4f5a-A60F-0e32318367b7 |ID. de aplicación de AAD =...

* `OperationId`: el identificador de operación de DM devuelto al ejecutar la purga. 
* `DatabaseName`* * nombre de la base de datos (distingue mayúsculas de minúsculas). 
* `TableName`-nombre de tabla (distingue mayúsculas de minúsculas). 
* `ScheduledTime`-tiempo de ejecución del comando de purga en el servicio DM. 
* `Duration`-duración total de la operación de purga, incluido el tiempo de espera de la cola de DM de ejecución. 
* `EngineOperationId`: el identificador de operación de la purga real que se ejecuta en el motor. 
* `State`-Estado de purga, puede ser uno de los siguientes valores: 
    * `Scheduled`-la operación de purga está programada para su ejecución. Si el trabajo sigue programado, probablemente haya un trabajo pendiente de operaciones de purga. Consulte [purgar el rendimiento](#purge-performance) para borrar este trabajo pendiente. Si se produce un error transitorio en una operación de purga, el DM volverá a intentarlo y se volverá a programar (por lo que puede ver una transición de operación de programado a en curso y de nuevo a programado).
    * `InProgress`-la operación de purga está en curso en el motor. 
    * `Completed`-purga completada correctamente.
    * `BadInput`-error de purga en la entrada incorrecta y no se volverá a intentar. Este error puede deberse a diversos problemas, como un error de sintaxis en el predicado, un predicado no válido para comandos de purga, una consulta que supere los límites (por ejemplo, más de 1 millón de entidades en un `externaldata` operador o más de 64 MB de tamaño total de consulta expandido) y errores 404 o 403 para los `externaldata` BLOBs.
    * `Failed`-error de purga y no se volverá a intentar. Este error puede producirse si la operación estaba esperando demasiado tiempo en la cola (más de 14 días), debido a un trabajo acumulado de otras operaciones de purga o a una serie de errores que superan el límite de reintentos. Esta última generará una alerta de supervisión interna y la investigará el equipo de Azure Explorador de datos. 
* `StateDetails`: una descripción del estado.
* `EngineStartTime`: la hora a la que se emitió el comando para el motor. Si hay una gran diferencia entre este tiempo y ScheduledTime, suele haber un trabajo acumulado significativo de operaciones de purga y el clúster no se mantiene al día con el ritmo. 
* `EngineDuration`-tiempo de ejecución de purga real en el motor. Si se ha reintentado la purga varias veces, es la suma de todas las duraciones de ejecución. 
* `Retries`-número de veces que el servicio DM ha reintentado la operación debido a un error transitorio.
* `ClientRequestId`-ID. de actividad de cliente de la solicitud de purga de DM. 
* `Principal`-Identity del emisor del comando Purge.

## <a name="purging-an-entire-table"></a>Purga de una tabla completa

La purga de una tabla incluye la eliminación de la tabla y su marca como purgada para que el proceso de eliminación de hardware descrito en el [proceso de purga](#purge-process) se ejecute en ella. Al quitar una tabla sin purgarla, no se eliminan todos sus artefactos de almacenamiento. Estos artefactos se eliminan según la Directiva de retención definida inicialmente establecida en la tabla. El `purge table allrecords` comando es rápido y eficaz y es preferible al proceso de purga de registros, si es aplicable para su escenario. 

> [!Note]
> El comando se invoca mediante la ejecución del comando [Purge TABLE *TableName* allrecords](#purge-table-tablename-allrecords-command) en el punto de conexión administración de datos https://ingest- [nombredelclúster]. [ Region]. kusto. Windows. net.

### <a name="purge-table-tablename-allrecords-command"></a>Comando Purge TABLE *TableName* allrecords

Similar al comando '[. Purgar registros de tabla ](#purge-table-tablename-records-command)', este comando se puede invocar en un modo de programación (de un solo paso) o en un modo manual (de dos pasos).

1. Invocación mediante programación (de un solo paso):

     **Sintaxis**

     ```kusto
     // Connect to the Data Management service
     #connect "https://ingest-[YourClusterName].[Region].kusto.windows.net" 
     
     .purge table [TableName] in database [DatabaseName] allrecords with (noregrets='true')
     ```

1. Invocación humana (dos pasos):

     **Sintaxis**

     ```kusto
   
     // Connect to the Data Management service
     #connect "https://ingest-[YourClusterName].[Region].kusto.windows.net" 
     
     // Step #1 - retrieve a verification token (the table will not be purged until step #2 is executed)

     .purge table [TableName] in database [DatabaseName] allrecords

     // Step #2 - input the verification token to execute purge
     .purge table [TableName] in database [DatabaseName] allrecords with (verificationtoken='<verification token from step #1>')
     ```

    | Parámetros  |Descripción  |
    |---------|---------|
    | `DatabaseName`   |   Nombre de la base de datos.      |
    | `TableName`    |     Nombre de la tabla.    |
    | `noregrets`    |     Si se establece, desencadena una activación de un solo paso.    |
    | `verificationtoken`     |  En el escenario de activación en dos pasos ( `noregrets` no establecido), este token se puede usar para ejecutar el segundo paso y confirmar la acción. Si `verificationtoken` no se especifica, se desencadenará el primer paso del comando. En este paso, se devuelve un token para devolver al comando y realizar el paso #2.|

#### <a name="example-two-step-purge"></a>Ejemplo: purga de dos pasos

1. Para iniciar la purga en un escenario de activación de dos pasos, ejecute el paso #1 del comando: 

    ```kusto
    // Connect to the Data Management service
     #connect "https://ingest-[YourClusterName].[Region].kusto.windows.net" 
     
    .purge table MyTable in database MyDatabase allrecords
    ```

    **Salida**

    | `VerificationToken`|
    |--|
    | e43c7184ed22f4f23c7a9d7b124d196be2e570096987e5baadf65057fa65736b|

1.  Para completar una purga en un escenario de activación en dos pasos, use el token de comprobación devuelto en el paso #1 para ejecutar el paso #2:

    ```kusto
    .purge table MyTable in database MyDatabase allrecords 
    with (verificationtoken='eyJTZXJ2aWNlTmFtZSI6IkVuZ2luZS1pdHNhZ3VpIiwiRGF0YWJhc2VOYW1lIjoiQXp1cmVTdG9yYWdlTG9ncyIsIlRhYmxlTmFtZSI6IkF6dXJlU3RvcmFnZUxvZ3MiLCJQcmVkaWNhdGUiOiIgd2hlcmUgU2VydmVyTGF0ZW5jeSA9PSAyNSJ9')
    ```
    
    La salida es la misma que la salida del comando '. show tables ' (devuelta sin la tabla purgada).

    **Salida**

    |  TableName|DatabaseName|Carpeta|DocString
    |---|---|---|---
    |  OtherTable|MyDatabase|---|---


#### <a name="example-single-step-purge"></a>Ejemplo: purga de un solo paso

Para desencadenar una purga en un escenario de activación de un solo paso, ejecute el siguiente comando:

```kusto
// Connect to the Data Management service
#connect "https://ingest-[YourClusterName].[Region].kusto.windows.net" 

.purge table MyTable in database MyDatabase allrecords with (noregrets='true')
```

La salida es la misma que la salida del comando '. show tables ' (devuelta sin la tabla purgada).

**Salida**

|TableName|DatabaseName|Carpeta|DocString
|---|---|---|---
|OtherTable|MyDatabase|---|---

