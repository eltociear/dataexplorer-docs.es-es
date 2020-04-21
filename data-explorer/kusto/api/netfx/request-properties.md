---
title: Propiedades de solicitud, ClientRequestProperties - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describen las propiedades de solicitud, ClientRequestProperties en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 09/23/2019
ms.openlocfilehash: 0aa496e6011fce98db5a304b9748f27f07590b4f
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81524334"
---
# <a name="request-properties-clientrequestproperties"></a>Propiedades de solicitud, ClientRequestProperties

Al realizar una solicitud desde Kusto a través del SDK de .NET, generalmente se proporcionan los siguientes valores:

1. Cadena de conexión que indica el punto de conexión de servicio al que se va a conectar, los parámetros de autenticación y la información similar relacionada con la conexión. Mediante programación, la cadena de `KustoConnectionStringBuilder` conexión se representa a través de la clase.

2. El nombre de la base de datos que se utiliza para describir el "ámbito" de la solicitud.

3. El texto de la solicitud (consulta o comando) en sí.

4. Propiedades adicionales que el cliente proporciona el servicio y se aplican a la solicitud. Mediante programación, estas propiedades se `ClientRequestProperties`mantienen en una clase denominada .

Las propiedades de solicitud de cliente tienen muchos usos. Algunos de ellos se utilizan para facilitar la depuración (por ejemplo, proporcionando cadenas de correlación que se pueden usar para realizar un seguimiento de las interacciones cliente/servicio), otros se utilizan para afectar a qué límites y directivas se aplican a la solicitud, mientras que una tercera categoría de estas propiedades son parámetros de [consulta.](../../query/queryparametersstatement.md) En esta página aparece una lista completa de las propiedades admitidas.

La `Kusto.Data.Common.ClientRequestProperties` clase contiene tres tipos de datos:

* Propiedades con nombre.

* Opciones (una asignación de un nombre de opción a un valor de opción).

* Parámetros (una asignación de un nombre de parámetro de consulta a un valor de parámetro de consulta).

> [!NOTE]
> Algunas propiedades con nombre se marcan como "no usar". Estas propiedades no deben ser especificadas por los clientes y no tienen ningún efecto en el servicio.

## <a name="the-clientrequestid-x-ms-client-request-id-named-property"></a>La propiedad con nombre ClientRequestId (x-ms-client-request-id)

Esta propiedad con nombre contiene la identidad especificada por el cliente de la solicitud. Se recomienda encarecidamente que los clientes especifiquen un valor único por solicitud con cada solicitud que envíen, ya que facilita los errores de depuración (y es necesario en algunos escenarios, como la cancelación de consultas).

El nombre de programación `ClientRequestId`de esta propiedad es `x-ms-client-request-id`, y se traduce en el encabezado HTTP .

El SDK establecerá esta propiedad en un valor (aleatorio) si el cliente no especifica su propio valor.

El contenido de esta propiedad puede ser cualquier cadena única imprimible, como un GUID.
Sin embargo, se recomienda que los clientes utilicen la siguiente plantilla:

*ApplicationName* `.` *ActivityName* `;` *UniqueId*

Cuando *ApplicationName* identifica la aplicación cliente que realiza la solicitud, *ActivityName* identifica el tipo de actividad para la que la aplicación cliente está emitiendo la solicitud de cliente y *UniqueId* identifica la solicitud específica.

## <a name="the-application-x-ms-app-named-property"></a>La propiedad con nombre Application (x-ms-app)

Esta propiedad con nombre contiene el nombre de la aplicación cliente que realiza la solicitud, con fines de seguimiento.

El nombre de programación `Application`de esta propiedad es `x-ms-app`, y se traduce en el encabezado HTTP . Se puede especificar en la cadena de `Application Name for Tracing`conexión Kusto como .

Esta propiedad se establecerá en el nombre del proceso que hospeda el SDK si el cliente no especifica su propio valor.

## <a name="the-user-x-ms-user-named-property"></a>La propiedad con nombre User (x-ms-user)

Esta propiedad con nombre contiene la identidad del usuario que realiza la solicitud, con fines de seguimiento.

El nombre de programación `User`de esta propiedad es `x-ms-user`, y se traduce en el encabezado HTTP . Se puede especificar en la cadena de `User Name for Tracing`conexión Kusto como .

## <a name="controlling-request-properties-using-the-rest-api"></a>Controlar las propiedades de la solicitud mediante la API de REST

Al emitir una solicitud HTTP al servicio Kusto, utilice la ranura del `properties` documento JSON que es el cuerpo de la solicitud POST para proporcionar propiedades de solicitud. Tenga en cuenta que algunas de las propiedades (como el "id de solicitud de cliente", que es el identificador de correlación que el cliente proporciona al servicio para identificar la solicitud) se pueden proporcionar en el encabezado HTTP, por lo que también se pueden establecer si se utiliza HTTP GET.
Consulte el objeto de solicitud de la API REST de [Kusto](../rest/request.md) para obtener información adicional.

## <a name="providing-values-for-query-parametrization-as-request-properties"></a>Proporcionar valores para la parametrización de consultas como propiedades de solicitud

Las consultas de Kusto pueden hacer referencia a parámetros de consulta mediante una instrucción [declare query-parameters](../../query/queryparametersstatement.md) especializada en el texto de la consulta. Esto permite a las aplicaciones cliente parametrizar las consultas de Kusto en función de la entrada del usuario de forma segura (sin temor a ataques de inyección.)

Mediante programación, se pueden establecer `ClearParameter` `SetParameter`valores `HasParameter` de propiedades mediante los métodos , , y .

En la API resta, los parámetros de consulta aparecen en la misma cadena codificada en JSON que las otras propiedades de solicitud.

## <a name="sample-code-for-using-request-properties"></a>Código de ejemplo para usar propiedades de solicitud

Aquí está un ejemplo para el código de cliente:

```csharp
public static System.Data.IDataReader QueryKusto(
    Kusto.Data.Common.ICslQueryProvider queryProvider,
    string databaseName,
    string query)
{
    var queryParameters = new Dictionary<String, String>()
    {
        { "xIntValue", "111" },
        { "xStrValue", "abc" },
        { "xDoubleValue", "11.1" }
    };

    // Query parameters (and many other properties) are provided
    // by a ClientRequestProperties object handed alongside
    // the query:
    var clientRequestProperties = new Kusto.Data.Common.ClientRequestProperties(
        principalIdentity: null,
        options: null,
        parameters: queryParameters);

    // Having client code provide its own ClientRequestId is
    // highly recommended. It not only allows the caller to
    // cancel the query, but also makes it possible for the Kusto
    // team to investigate query failures end-to-end:
    clientRequestProperties.ClientRequestId
        = "MyApp.MyActivity;"
        + Guid.NewGuid().ToString();

    // This is an example for setting an option
    // ("notruncation", in this case). In most cases this is not
    // needed, but it's included here for completeness:
    clientRequestProperties.SetOption(
        Kusto.Data.Common.ClientRequestProperties.OptionNoTruncation,
        true);
 
    try
    {
        return queryProvider.ExecuteQuery(query, clientRequestProperties);
    }
    catch (Exception ex)
    {
        Console.WriteLine(
            "Failed invoking query '{0}' against Kusto."
            + " To have the Kusto team investigate this failure,"
            + " please open a ticket @ https://aka.ms/kustosupport,"
            + " and provide: ClientRequestId={1}",
            query, clientRequestProperties.ClientRequestId);
        return null;
    }
}
```

## <a name="list-of-clientrequestproperties"></a>Lista de ClientRequestProperties

<!-- The following is auto-generated by running the following command: -->
<!-- Kusto.Cli.exe -execute:"#crp -doc"                                -->

<!-- The following text can be re-produced by running the Kusto.Cli.exe directive '#crp -doc' -->

<!-- The following text can be re-produced by running the Kusto.Cli.exe directive '#crp -doc' -->

* `debug_query_externaldata_projection_fusion_disabled`(*OptionDebugQueryDisableExternalDataProjectionFusion*): Si se establece, no fusione la proyección en el operador ExternalData. [Booleano]
* `debug_query_fanout_threads_percent_external_data`(*OptionDebugQueryFanoutThreadsPercentExternalData*): porcentaje de subprocesos para la ejecución de distribución para nodos de datos externos. [Int]
* `deferpartialqueryfailures`(*OptionDeferPartialQueryFailures*): Si es true, deshabilita los errores de consulta parciales de informes como parte del conjunto de resultados. [Booleano]
* `max_memory_consumption_per_query_per_node`(*OptionMaxMemoryConsumptionPerQueryPerNode*): Reemplaza la cantidad máxima predeterminada de memoria que una consulta completa puede asignar por nodo. [UInt64]
* `maxmemoryconsumptionperiterator`(*OptionMaxMemoryConsumptionPerIterator*): reemplaza la cantidad máxima predeterminada de memoria que un operador de consulta puede asignar. [UInt64]
* `maxoutputcolumns`(*OptionMaxOutputColumns*): reemplaza el número máximo predeterminado de columnas que una consulta puede producir. [Largo]
* `norequesttimeout`(*OptionNoRequestTimeout*): permite establecer el tiempo de espera de la solicitud en su valor máximo. [Booleano]
* `notruncation`(*OptionNoTruncation*): permite suprimir el truncamiento de los resultados de la consulta devueltos al autor de la llamada. [Booleano]
* `push_selection_through_aggregation`(*OptionPushSelectionThroughAggregation*): Si es true, inserte la selección simple a través de la agregación [Boolean]
* `query_admin_super_slacker_mode`(*OptionAdminSuperSlackerMode*): Si es true, delegue la ejecución de la consulta a otro nodo [Boolean]
* `query_bin_auto_at`(*QueryBinAutoAt*): Al evaluar la función bin_auto(), el valor de inicio que se va a utilizar. [LiteralExpression]
* `query_bin_auto_size`(*QueryBinAutoSize*): al evaluar la función bin_auto(), el valor de tamaño de ubicación que se va a utilizar. [LiteralExpression]
* `query_cursor_after_default`(*OptionQueryCursorAfterDefault*): El valor de parámetro predeterminado de la función cursor_after() cuando se llama sin parámetros. [string]
* `query_cursor_before_or_at_default`(*OptionQueryCursorBeforeOrAtDefault*): el valor de parámetro predeterminado de la función cursor_before_or_at() cuando se llama sin parámetros. [string]
* `query_cursor_current`(*OptionQueryCursorCurrent*): reemplaza el valor del cursor devuelto por las funciones cursor_current() o current_cursor(). [string]
* `query_cursor_scoped_tables`(*OptionQueryCursorScopedTables*): lista de nombres de tabla cuyo ámbito cursor_after_default .. cursor_before_or_at_default (el límite superior es opcional). [dinámico]
* `query_datascope`(*OptionQueryDataScope*): controla el ámbito de datos de la consulta, ya sea que la consulta se aplique a todos los datos o solo a una parte de ella. ['default', 'all' o 'hotcache']
* `query_datetimescope_column`(*OptionQueryDateTimeScopeColumn*): controla el nombre de columna para el ámbito datetime de la consulta (query_datetimescope_to / query_datetimescope_from). [String]
* `query_datetimescope_from`(*OptionQueryDateTimeScopeFrom*): controla el ámbito datetime de la consulta (más temprano) -- utilizado como filtro aplicado automáticamente solo en query_datetimescope_column (si está definido). [DateTime]
* `query_datetimescope_to`(*OptionQueryDateTimeScopeTo*): controla el ámbito datetime (más reciente) de la consulta, que se usa como filtro aplicado automáticamente solo en query_datetimescope_column (si está definido). [DateTime]
* `query_distribution_nodes_span`(*OptionQueryDistributionNodesSpanSize*): Si se establece, controla el comportamiento de la combinación de subconsultas: el nodo en ejecución introducirá un nivel adicional en la jerarquía de consultas para cada subgrupo de nodos; el tamaño del subgrupo se establece mediante esta opción. [Int]
* `query_fanout_nodes_percent`(*OptionQueryFanoutNodesPercent*): porcentaje de nodos para la ejecución de fanout. [Int]
* `query_fanout_threads_percent`(*OptionQueryFanoutThreadsPercent*): porcentaje de subprocesos para la ejecución de ventilador. [Int]
* `query_language`(*OptionQueryLanguage*): controla cómo se debe interpretar el texto de la consulta. ['csl','kql' o 'sql']
* `query_max_entities_in_union`(*OptionMaxEntitiesToUnion*): reemplaza el número máximo predeterminado de columnas que una consulta puede producir. [Largo]
* `query_now`(*OptionQueryNow*): reemplaza el valor datetime devuelto por la función now(0s). [DateTime]
* `query_python_debug`(*OptionDebugPython*): Si se establece, genere la consulta de depuración de Python para el nodo Python enumerado (predeterminado primero). [Boolean o Int]
* `query_results_apply_getschema`(*OptionQueryResultsApplyGetSchema*): Si se establece, recupera el esquema de cada dato tabular en los resultados de la consulta en lugar de los propios datos. [Booleano]
* `query_results_cache_max_age`(*OptionQueryResultsCacheMaxAge*): Si es positivo, controla la edad máxima de los resultados de la consulta almacenada en caché que Kusto puede devolver [TimeSpan]
* `query_results_progressive_row_count`(*OptionProgressiveQueryMinRowCountPerUpdate*): Sugerencia para Kusto en cuanto a cuántos registros enviar en cada actualización (solo tiene efecto si OptionResultsProgressiveEnabled está establecido)
* `query_results_progressive_update_period`(*OptionProgressiveProgressReportPeriod*): Sugerencia para Kusto sobre la frecuencia con la que se envían fotogramas de progreso (solo surte efecto si OptionResultsProgressiveEnabled está establecido)
* `query_shuffle_broadcast_join`(*ShuffleBroadcastJoin*): Activa el barajado sobre la unión de difusión.
* `query_take_max_records`(*OptionTakeMaxRecords*): permite limitar los resultados de la consulta a este número de registros. [Largo]
* `queryconsistency`(*OptionQueryConsistency*): controla la coherencia de la consulta. ['consistencia fuerte' o 'normalconsistency' o 'debilidad']
* `request_callout_disabled`(*OptionRequestCalloutDisabled*): Si se especifica, indica que la solicitud no puede llamar a un servicio proporcionado por el usuario. [Booleano]
* `request_description`(*OptionRequestDescription*): texto arbitrario que el autor de la solicitud desea incluir como descripción de la solicitud. [String]
* `request_external_table_disabled`(*OptionRequestExternalTableDisabled*): Si se especifica, indica que la solicitud no puede invocar código en ExternalTable. [Booleano]
* `request_readonly`(*OptionRequestReadOnly*): Si se especifica, indica que la solicitud no debe poder escribir nada. [Booleano]
* `request_remote_entities_disabled`(*OptionRequestRemoteEntitiesDisabled*): Si se especifica, indica que la solicitud no puede tener acceso a bases de datos y clústeres remotos. [Booleano]
* `request_sandboxed_execution_disabled`(*OptionRequestSandboxedExecutionDisabled*): Si se especifica, indica que la solicitud no puede invocar código en el entorno limitado. [Booleano]
* `results_progressive_enabled`(*OptionResultsProgressiveEnabled*): Si se establece, habilita la secuencia de consulta progresiva
* `servertimeout`(*OptionServerTimeout*): reemplaza el tiempo de espera de solicitud predeterminado. [TimeSpan]
* `truncationmaxrecords`(*OptionTruncationMaxRecords*): reemplaza el número máximo predeterminado de registros que una consulta puede devolver al autor de la llamada (truncamiento). [Largo]
* `truncationmaxsize`(*OptionTruncationMaxSize*): reemplaza el tamaño máximo de datos dfefault que una consulta puede devolver al autor de la llamada (truncamiento). [Largo]
* `validate_permissions`(*OptionValidatePermissions*): valida los permisos del usuario para realizar la consulta y no ejecuta la consulta en sí. [Booleano]

