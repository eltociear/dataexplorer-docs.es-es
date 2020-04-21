---
title: 'Purga de datos: Explorador de datos de Azure Microsoft Docs'
description: En este artículo se describe la purga de datos en El Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/24/2020
ms.openlocfilehash: 49d024d1deecd8e0c7bf16eda9917cd237fe6319
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81523280"
---
# <a name="data-purge"></a>Purga de datos

>[!Note]
> En este artículo se indican los pasos para eliminar los datos personales del dispositivo o del servicio y puede utilizarse para cumplir con sus obligaciones según el Reglamento general de protección de datos (RGPD). Si está buscando información general sobre el RGPD, consulte la [sección RGPD del portal de confianza](https://servicetrust.microsoft.com/ViewPage/GDPRGetStarted)de servicios .



Como plataforma de datos, Azure Data Explorer (Kusto) admite la `.purge` capacidad de eliminar registros individuales mediante el uso de comandos relacionados y los. También puede [purgar una tabla completa.](#purging-an-entire-table)  

> [!WARNING]
> La eliminación `.purge` de datos a través del comando está diseñada para usarse para proteger los datos personales y no debe utilizarse en otros escenarios. No está diseñado para admitir solicitudes de eliminación frecuentes o eliminación de cantidades masivas de datos, y puede tener un impacto significativo en el rendimiento en el servicio.

## <a name="purge-guidelines"></a>Pautas de purga

Se **recomienda encarecidamente** que los equipos que almacenan datos personales en Azure Data Explorer lo hagan solo después de un diseño cuidadoso del esquema de datos y la investigación de las directivas pertinentes.

1. En el mejor de los casos, el período de retención de estos datos es lo suficientemente corto y los datos se eliminan automáticamente.
2. Si el uso del período de retención no es posible, el enfoque recomendado es aislar todos los datos sujetos a reglas de privacidad en un pequeño número de tablas de Kusto (de manera óptima, solo una tabla) y vincularlo de todas las demás tablas. Esto permite ejecutar el proceso de [purga](#purge-process) de datos en un pequeño número de tablas que contienen datos confidenciales y evitar todas las demás tablas.
3. El autor de la llamada debe `.purge` realizar cada intento de procesar por lotes la ejecución de comandos a 1-2 comandos por tabla por día.
   No emita varios comandos, cada uno con su propio predicado de identidad de usuario; más bien, envíe un único comando cuyo predicado incluya todas las identidades de usuario que requieren purga.

## <a name="purge-process"></a>Proceso de purga

El proceso de purga selectiva de datos de Azure Data Explorer se realiza en los pasos siguientes:

1. **Fase 1:** Dado un nombre de tabla Kusto y un predicado por registro que indica qué registros se van a eliminar, Kusto examina la tabla en busca de identificar particiones de datos que participarían en la purga de datos (tienen uno o varios registros para los que el predicado devuelve true).
2. **Fase 2: (Eliminación suave)** Reemplace cada partición de datos de la tabla (identificada en el paso (1)) por una versión reingerda que no tenga los registros para los que el predicado devuelve true.
   Mientras no se ingien de datos nuevos en la tabla, al final de esta fase las consultas ya no devolverán los datos para los que el predicado devuelve true. 
   La duración de la fase de eliminación temporal de purga depende del número de registros que se deben purgar, su distribución entre las particiones de datos del clúster, el número de nodos del clúster, la capacidad de reserva que tiene para las operaciones de purga y varios otros factores. La duración de la fase 2 puede variar entre unos segundos y muchas horas.
3. **Fase 3: (Eliminación dura)** Vuelva a recuperar todos los artefactos de almacenamiento que puedan tener los datos "venenos" y elimínelos del almacenamiento. Esta fase se realiza al menos 5 días *después* de la finalización de la fase anterior, pero no más de 30 días después del comando inicial, para cumplir con los requisitos de privacidad de datos.

La emisión `.purge` de un comando desencadena este proceso, que tarda unos días en completarse. Tenga en cuenta que si la "densidad" de registros para la que se aplica el predicado es lo suficientemente grande, el proceso reintegrará eficazmente todos los datos de la tabla, por lo tanto, tendrá un impacto significativo en el rendimiento y el costo de los productos vendidos.


## <a name="purge-limitations-and-considerations"></a>Purgar limitaciones y consideraciones

* El proceso de **purga es final e irreversible.** No es posible "deshacer" este proceso o recuperar los datos que se han purgado. Por lo tanto, los comandos como [deshacer la caída](../management/undo-drop-table-command.md) de la tabla no pueden recuperar los datos purgados y la reversión de los datos a una versión anterior no puede ir a "antes" del último comando de purga.

* Para evitar errores, se recomienda comprobar el predicado ejecutando una consulta antes de la purga para asegurarse de que los resultados coinciden con el resultado esperado o usar el proceso de 2 pasos que devuelve el número esperado de registros que se purgarán. 

* Como medida de precaución, el proceso de purga está deshabilitado, de forma predeterminada, en todos los clústeres.
   Habilitar el proceso de purga es una operación de una sola vez que requiere abrir un ticket de [soporte;](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview) por favor, especifique `EnabledForPurge` que desea que la función se encienda.

* El `.purge` comando se ejecuta en `https://ingest-[YourClusterName].kusto.windows.net`el extremo de administración de datos: .
   El comando requiere permisos de administrador de [base](../management/access-control/role-based-authorization.md) de datos en las bases de datos relevantes. 
* Debido al impacto en el rendimiento del proceso de purga y para garantizar que se han seguido [las directrices](#purge-guidelines) de purga, se espera que el autor de la llamada modifique el esquema de datos para que las tablas mínimas incluyan datos relevantes y comandos por lote por tabla para reducir el impacto significativo en el costo de los productos vendidos del proceso de purga.
* El `predicate` parámetro del comando [.purge](#purge-table-tablename-records-command) se utiliza para especificar qué registros purgar.
`Predicate`el tamaño está limitado a 63 KB. Al construir `predicate`el :
    * Utilice el [operador 'in',](../query/inoperator.md) `where [ColumnName] in ('Id1', 'Id2', .. , 'Id1000')`por ejemplo. 
        * Tenga en cuenta los límites del [operador 'in'](../query/inoperator.md) (la lista puede contener hasta `1,000,000` valores).
    * Si el tamaño de la consulta es grande, utilice `where UserId in (externaldata(UserId:string) ["https://...blob.core.windows.net/path/to/file?..."])`el [operador 'externaldata',](../query/externaldata-operator.md)por ejemplo. . El archivo almacena la lista de iDE que se van a purgar.
        * El tamaño total de la consulta, después de expandir todos los blobs de datos externos (tamaño total de todos los blobs) no puede superar los 64 MB. 

## <a name="purge-performance"></a>Rendimiento de purga

Solo se puede ejecutar una solicitud de purga en el clúster, en un momento dado. Todas las demás solicitudes se ponen en cola en estado "Programado". El tamaño de la cola de solicitud de purga debe supervisarse y mantenerse dentro de los límites adecuados para que coincida con los requisitos aplicables a los datos.

Para reducir el tiempo de ejecución de purga:
* Disminuya la cantidad de datos purgados siguiendo [las directrices de purga](#purge-guidelines)
* Ajuste la [directiva de almacenamiento en caché, ya](../management/cachepolicy.md) que la purga tarda más en los datos fríos.
* Escalar horizontalmente el clúster

* Aumente la capacidad de purga del clúster, después de una cuidadosa consideración, como se detalla en Extensión purgar la capacidad de [reconstrucción.](../management/capacitypolicy.md#extents-purge-rebuild-capacity) Cambiar este parámetro requiere abrir un ticket de [soporte](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview)

## <a name="trigger-the-purge-process"></a>Desencadenar el proceso de purga

> [!Note]
> La ejecución de purga se invoca ejecutando el comando [purge table *TableName* records](#purge-table-tablename-records-command) en el punto de conexión de administración de datos (**https://ingest-[YourClusterName].kusto.windows.net**).

### <a name="purge-table-tablename-records-command"></a>Purgar el comando de registros *TableName* de la tabla

El comando Purgar se puede invocar de dos maneras para diferentes escenarios de uso:
1. Invocación mediante programación: un paso único que está destinado a ser invocado por las aplicaciones. Al llamar a este comando desencadena directamente la secuencia de ejecución de purga.

    **Sintaxis**
     ```kusto
     .purge table [TableName] records in database [DatabaseName] with (noregrets='true') <| [Predicate]
     ```

    > [!NOTE]
    > Genere este comando mediante la API CslCommandGenerator, disponible como parte del paquete NuGet de la biblioteca de cliente de [Kusto.](../api/netfx/about-kusto-data.md)

1. Invocación humana: un proceso de dos pasos que requiere una confirmación explícita como paso independiente. La primera invocación del comando devuelve un token de verificación, que se debe proporcionar para ejecutar la purga real. Esta secuencia reduce el riesgo de eliminar inadvertidamente datos incorrectos. El uso de esta opción puede tardar mucho tiempo en completarse en tablas grandes con datos significativos de caché en frío.
    <!-- If query times-out on DM endpoint (default timeout is 10 minutes), it is recommended to use the [engine `whatif` command](#purge-whatif-command) directly againt the engine endpoint while increasing the [server timeout limit](../concepts/querylimits.md#limit-on-request-execution-time-timeout). Only after you have verified the expected results using the engine whatif command, issue the purge command via the DM endpoint using the 'noregrets' option. -->

     **Sintaxis**
     ```kusto
     // Step #1 - retrieve a verification token (no records will be purged until step #2 is executed)
     .purge table [TableName] records in database [DatabaseName] <| [Predicate]

     // Step #2 - input the verification token to execute purge
     .purge table [TableName] records in database [DatabaseName] with (verificationtoken='<verification token from step #1>') <| [Predicate]
     ```
    
    |Parámetros  |Descripción  |
    |---------|---------|
    | DatabaseName   |   Nombre de la base de datos      |
    | TableName     |     Nombre de la tabla    |
    | Predicate    |    Identifica los registros que se van a purgar. Consulte Purgar las limitaciones del predicado a continuación. | 
    | noregrets    |     Si se establece, desencadena una activación de un solo paso.    |
    | verificationtoken     |  En el escenario de activación en dos**pasos (noregrets** no está establecido), este token se puede usar para ejecutar el segundo paso y confirmar la acción. Si no se especifica **verificationtoken,** desencadenará el primer paso del comando, en el que se devuelve información sobre la purga y un token, que se debe devolver al comando para realizar el paso #2.   |

    **Purgar limitaciones de predicados**
    * El predicado debe ser una selección simple (por ejemplo, *donde [ColumnName] á 'X'* / *donde [ColumnName] in ('X', 'Y', 'Z') y [OtherColumn] á 'A'*).
    * Varios filtros deben combinarse con un `where` 'y', en `where [ColumnName] == 'X' and  OtherColumn] == 'Y'` lugar `where [ColumnName] == 'X' | where [OtherColumn] == 'Y'`de cláusulas separadas (por ejemplo, y no).
    * El predicado no puede hacer referencia a tablas que no sean la tabla que se está purgando (*TableName*). El predicado solo puede`where`incluir la instrucción selection ( ). No puede proyectar columnas específicas de la tabla (esquema de salida cuando se ejecuta* `table` ' Predicate*' debe coincidir con el esquema de tabla).
    * Las funciones del `ingestion_time()` `extent_id()`sistema (como, , ) no se admiten como parte del predicado.

#### <a name="example-two-step-purge"></a>Ejemplo: Purga en dos pasos

1. Para iniciar la purga en un escenario de activación en dos pasos, ejecute el paso #1 del comando:

    ```kusto
    .purge table MyTable records in database MyDatabase <| where CustomerId in ('X', 'Y')
    ```

    **Salida**

    |NumRecordsToPurge |EstimatedPurgeExecutionTime| VerificationToken
    |--|--|--
    |1,596 |00:00:02 |e43c7184ed22f4f23c7a9d7b124d196be2e570096987e5baadf65057fa65736b

    Valide NumRecordsToPurge antes de ejecutar el paso #2. 
2. Para completar una purga en un escenario de activación en dos pasos, use el token de verificación devuelto desde el paso #1 ejecutar el paso #2:

    ```kusto
    .purge table MyTable records in database MyDatabase
    with (verificationtoken='e43c7184ed22f4f23c7a9d7b124d196be2e570096987e5baadf65057fa65736b')
    <| where CustomerId in ('X', 'Y')
    ```

    **Salida**

    |OperationId |DatabaseName |TableName |ScheduledTime |Duration |LastUpdatedOn |EngineOperationId |State |StateDetails |EngineStartTime |EngineDuration |Reintentos |ClientRequestId |Principal
    |--|--|--|--|--|--|--|--|--|--|--|--|--|--
    |c9651d74-3b80-4183-90bb-bbe9e42eadc4 |Mydatabase |MyTable |2019-01-20 11:41:05.4391686 |00:00:00.1406211 |2019-01-20 11:41:05.4391686 | |Programado | | | |0 |Ke. RunCommand;1d0ad28b-f791-4f5a-a60f-0e32318367b7 |Id. de la aplicación aAD...

#### <a name="example-single-step-purge"></a>Ejemplo: Purga de un solo paso

Para desencadenar una purga en un escenario de activación de un solo paso, ejecute el siguiente comando:

```kusto
.purge table MyTable records in database MyDatabase with (noregrets='true') <| where CustomerId in ('X', 'Y')
```

**Salida**

|OperationId |DatabaseName |TableName |ScheduledTime |Duration |LastUpdatedOn |EngineOperationId |State |StateDetails |EngineStartTime |EngineDuration |Reintentos |ClientRequestId |Principal
|--|--|--|--|--|--|--|--|--|--|--|--|--|--
|c9651d74-3b80-4183-90bb-bbe9e42eadc4 |Mydatabase |MyTable |2019-01-20 11:41:05.4391686 |00:00:00.1406211 |2019-01-20 11:41:05.4391686 | |Programado | | | |0 |Ke. RunCommand;1d0ad28b-f791-4f5a-a60f-0e32318367b7 |Id. de la aplicación aAD...

### <a name="cancel-purge-operation-command"></a>Cancelar comando de operación de purga

Si es necesario, puede cancelar las solicitudes de purga pendientes.

> [!NOTE]
> Esta operación está pensada para escenarios de recuperación de errores. No se garantiza que tenga éxito y no debe formar parte de un flujo operativo normal. Solo se puede aplicar a las solicitudes en cola (aún no se han enviado al nodo del motor para su ejecución). El comando se ejecuta en el extremo de administración de datos.

**Sintaxis**

```kusto
 .cancel purge <OperationId>
 ```

**Ejemplo**

```kusto
 .cancel purge aa894210-1c60-4657-9d21-adb2887993e1
 ```

**Salida**

La salida de este comando es la misma que la salida del comando 'show purga *OperationId',* mostrando el estado actualizado de la operación de purga que se cancela. Si el intento se realiza correctamente, el estado de la operación se actualiza a 'Abandoned', de lo contrario el estado de la operación no se modifica. 

|OperationId |DatabaseName |TableName |ScheduledTime |Duration |LastUpdatedOn |EngineOperationId |State |StateDetails |EngineStartTime |EngineDuration |Reintentos |ClientRequestId |Principal
|--|--|--|--|--|--|--|--|--|--|--|--|--|--
|c9651d74-3b80-4183-90bb-bbe9e42eadc4 |Mydatabase |MyTable |2019-01-20 11:41:05.4391686 |00:00:00.1406211 |2019-01-20 11:41:05.4391686 | |Abandoned | | | |0 |Ke. RunCommand;1d0ad28b-f791-4f5a-a60f-0e32318367b7 |Id. de la aplicación aAD...

## <a name="track-purge-operation-status"></a>Seguimiento del estado de la operación de purga 

> [!Note]
> Las operaciones de purga se pueden seguir con el comando [show purges,](#show-purges-command) ejecutado en el punto final de administración de datos (**https://ingest-[YourClusterName].kusto.windows.net**).

Estado: 'Completado' indica que se ha completado correctamente la primera fase de la operación de purga, es decir, los registros se eliminan temporalmente y ya no están disponibles para realizar consultas. **No** se espera que los clientes realicen un seguimiento y verifiquen la finalización de la segunda fase (eliminación dura). Esta fase es monitoreada internamente por Kusto.

### <a name="show-purges-command"></a>comando show purges

`Show purges`comando muestra el estado de la operación de purga especificando el identificador de operación dentro del período de tiempo solicitado. 

```kusto
.show purges <OperationId>
.show purges [in database <DatabaseName>]
.show purges from '<StartDate>' [in database <DatabaseName>]
.show purges from '<StartDate>' to '<EndDate>' [in database <DatabaseName>]
```

|Propiedades  |Descripción  |Obligatorio/Opcional
|---------|---------|
|OperationId    |      El identificador de operación de administración de datos se genera después de ejecutar una sola fase o segunda fase.   |Mandatory
|StartDate    |   Límite de tiempo inferior para las operaciones de filtrado. Si se omite, el valor predeterminado es 24 horas antes de la hora actual.      |Opcional
|EndDate    |  Límite de tiempo superior para las operaciones de filtrado. Si se omite, el valor predeterminado es la hora actual.       |Opcional
|DatabaseName    |     Nombre de la base de datos para filtrar los resultados.    |Opcional

> [!NOTE]
> El estado solo se proporcionará en las bases de datos que el cliente tenga permisos de administrador de [base](../management/access-control/role-based-authorization.md) de datos.

**Ejemplos**

```kusto
.show purges
.show purges c9651d74-3b80-4183-90bb-bbe9e42eadc4
.show purges from '2018-01-30 12:00'
.show purges from '2018-01-30 12:00' to '2018-02-25 12:00'
.show purges from '2018-01-30 12:00' to '2018-02-25 12:00' in database MyDatabase
```

**Salida** 

|OperationId |DatabaseName |TableName |ScheduledTime |Duration |LastUpdatedOn |EngineOperationId |State |StateDetails |EngineStartTime |EngineDuration |Reintentos |ClientRequestId |Principal
|--|--|--|--|--|--|--|--|--|--|--|--|--|--
|c9651d74-3b80-4183-90bb-bbe9e42eadc4 |Mydatabase |MyTable |2019-01-20 11:41:05.4391686 |00:00:33.6782130 |2019-01-20 11:42:34.6169153 |a0825d4d-6b0f-47f3-a499-54ac5681ab78 |Completed |Purgar completado correctamente (artefactos de almacenamiento pendientes de eliminación) |2019-01-20 11:41:34.6486506 |00:00:04.4687310 |0 |Ke. RunCommand;1d0ad28b-f791-4f5a-a60f-0e32318367b7 |Id. de la aplicación aAD...

* **OperationId:** el identificador de operación de DM devuelto al ejecutar la purga. 
* **DatabaseName** - nombre de la base de datos (con distinción entre mayúsculas y minúsculas). 
* **TableName** - nombre de tabla (con distinción entre mayúsculas y minúsculas). 
* **ScheduledTime:** tiempo de ejecución del comando purge en el servicio DM. 
* **Duración:** duración total de la operación de purga, incluido el tiempo de espera de la cola de DM de ejecución. 
* **EngineOperationId** - el identificador de operación de la purga real que se ejecuta en el motor. 
* **Estado** - estado de purga, puede ser uno de los siguientes: 
    * Programado: la operación de purga está programada para su ejecución. Si el trabajo sigue siendo **Programado,** es probable que haya una acumulación de operaciones de purga. Consulte [el rendimiento de la purga](#purge-performance) para borrar este trabajo pendiente. Si se produce un error en una operación de purga en un error transitorio, el DM la volverá a intentar y se establecerá en **Programado** de nuevo (por lo que es posible que vea una transición de operación de **Programado** a **InProgress** y de nuevo a **Programado**).
    * InProgress - la operación de purga está en curso en el motor. 
    * Completado - purga completada con éxito.
    * BadInput - purga falló en la entrada incorrecta y no se volverá a intentar. Esto puede deberse a varios problemas, como un error de sintaxis en el predicado, un predicado ilegal para los comandos de purga, una consulta que supera los límites (por ejemplo, más de 1M entidades en un operador de datos externos o más de 64 MB de tamaño total de consulta expandido) y errores 404 o 403 para blobs de datos externos.
    * Error: la purga ha fallado y no se volverá a intentar. Esto puede suceder si la operación estuvo esperando en la cola durante demasiado tiempo (más de 14 días), debido a una acumulación de otras operaciones de purga o a una serie de errores que supera el límite de reintentos. Este último generará una alerta de supervisión interna y será investigado por el equipo de Azure Data Explorer. 
* StateDetails - una descripción del estado.
* EngineStartTime - la hora en que se emitió el comando al motor. Si hay una gran diferencia entre este tiempo y ScheduledTime, normalmente hay una gran acumulación de operaciones de purga y el clúster no está al día con el ritmo. 
* EngineDuration - tiempo de ejecución de purga real en el motor. Si la purga se ha reintentado varias veces, es la suma de todas las duraciones de ejecución. 
* Reintentos: número de veces que el servicio DM ha reintentado la operación debido a un error transitorio.
* ClientRequestId: identificador de actividad de cliente de la solicitud de purga de DM. 
* Principal: identidad del emisor del comando purge.



## <a name="purging-an-entire-table"></a>Purgar una mesa entera
La purga de una tabla incluye quitar la tabla y marcarla como purgada para que el proceso de eliminación de hardware descrito en [El proceso](#purge-process) de purga se ejecute en ella. Al quitar una tabla sin purgarla, no se eliminan todos sus artefactos de almacenamiento (eliminados según la directiva de retención de disco duro establecida inicialmente en la tabla). El `purge table allrecords` comando es rápido y eficiente y es mucho más preferible al proceso de purga de registros, si corresponde para su escenario. 

> [!Note]
> El comando se invoca ejecutando el comando [purge table *TableName* allrecords](#purge-table-tablename-allrecords-command) en el extremo de administración de datos (**https://ingest-[YourClusterName].kusto.windows.net**).

### <a name="purge-table-tablename-allrecords-command"></a>tabla de purga *TableName* allrecords comando

Similar al comando '[.purge table records ](#purge-table-tablename-records-command)', este comando se puede invocar en un modo de programación (un solo paso) o en un modo manual (dos pasos).
1. Invocación programática (paso único):

     **Sintaxis**
     ```kusto
     .purge table [TableName] in database [DatabaseName] allrecords with (noregrets='true')
     ```

2. Invocación humana (dos pasos):

     **Sintaxis**
     ```kusto
     // Step #1 - retrieve a verification token (the table will not be purged until step #2 is executed)
     .purge table [TableName] in database [DatabaseName] allrecords

     // Step #2 - input the verification token to execute purge
     .purge table [TableName] in database [DatabaseName] allrecords with (verificationtoken='<verification token from step #1>')
     ```

    |Parámetros  |Descripción  |
    |---------|---------|
    |**DatabaseName**   |   Nombre de la base de datos.      |
    |**TableName**     |     Nombre de la tabla.    |
    |**noregrets**    |     Si se establece, desencadena una activación de un solo paso.    |
    |**verificationtoken**     |  En el escenario de activación en dos**pasos (noregrets** no está establecido), este token se puede usar para ejecutar el segundo paso y confirmar la acción. Si no se especifica **verificationtoken,** desencadenará el primer paso del comando, en el que se devuelve un token, para volver al comando y realizar el paso del comando #2.|

#### <a name="example-two-step-purge"></a>Ejemplo: Purga en dos pasos

1. Para iniciar la purga en un escenario de activación en dos pasos, ejecute el paso #1 del comando: 

    ```kusto
    .purge table MyTable in database MyDatabase allrecords
    ```

    **Salida**

    | VerificationToken
    |--
    |e43c7184ed22f4f23c7a9d7b124d196be2e570096987e5baadf65057fa65736b

1.  Para completar una purga en un escenario de activación en dos pasos, use el token de verificación devuelto desde el paso #1 ejecutar el paso #2:

    ```kusto
    .purge table MyTable in database MyDatabase allrecords 
    with (verificationtoken='eyJTZXJ2aWNlTmFtZSI6IkVuZ2luZS1pdHNhZ3VpIiwiRGF0YWJhc2VOYW1lIjoiQXp1cmVTdG9yYWdlTG9ncyIsIlRhYmxlTmFtZSI6IkF6dXJlU3RvcmFnZUxvZ3MiLCJQcmVkaWNhdGUiOiIgd2hlcmUgU2VydmVyTGF0ZW5jeSA9PSAyNSJ9')
    ```
    La salida es la misma que la salida del comando '.show tables' (devuelta sin la tabla purgada).

    **Salida**

    |TableName|DatabaseName|Carpeta|DocString
    |---|---|---|---
    |OtherTable|Mydatabase|---|---


#### <a name="example-single-step-purge"></a>Ejemplo: Purga de un solo paso

Para desencadenar una purga en un escenario de activación de un solo paso, ejecute el siguiente comando:

```kusto
.purge table MyTable in database MyDatabase allrecords with (noregrets='true')
```

La salida es la misma que la salida del comando '.show tables' (devuelta sin la tabla purgada).

**Salida**

|TableName|DatabaseName|Carpeta|DocString
|---|---|---|---
|OtherTable|Mydatabase|---|---


