---
title: 'Propiedades de la solicitud y ClientRequestProperties: Azure Explorador de datos'
description: En este artículo se describen las propiedades de solicitud y ClientRequestProperties en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 09/23/2019
ms.openlocfilehash: 137a34ab211b72c2282d242840a46699d5feb3b3
ms.sourcegitcommit: 974d5f2bccabe504583e387904851275567832e7
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/18/2020
ms.locfileid: "83550493"
---
# <a name="request-properties-and-clientrequestproperties"></a>Propiedades de la solicitud y ClientRequestProperties

Cuando se realiza una solicitud desde Kusto a través del SDK de .NET, proporcione:

* Cadena de conexión que indica el extremo de servicio al que se va a conectar, los parámetros de autenticación y la información relacionada con la conexión similar. Mediante programación, la cadena de conexión se representa mediante la `KustoConnectionStringBuilder` clase.

* Nombre de la base de datos que se usa para describir el "ámbito" de la solicitud.

* Texto de la solicitud (consulta o comando).

* Propiedades adicionales que proporciona el cliente al servicio y que se aplican a la solicitud. Mediante programación, estas propiedades están retenidas por una clase denominada `ClientRequestProperties` .

##   <a name="clientrequestproperties"></a>ClientRequestProperties

Las propiedades de la solicitud de cliente tienen muchos usos. 
* Facilita la depuración. Por ejemplo, las propiedades pueden proporcionar cadenas de correlación que se usan para realizar el seguimiento de las interacciones del cliente o servicio. 
* Afecta a qué límites y directivas se aplican a la solicitud. 
* [los parámetros de consulta](../../query/queryparametersstatement.md) permiten a las aplicaciones cliente parametrizar las consultas de Kusto en función de la entrada del usuario.
[lista de propiedades admitidas](#list-of-clientrequestproperties).

La `Kusto.Data.Common.ClientRequestProperties` clase contiene tres tipos de datos.

* Propiedades con nombre.
* Options: asignación de un nombre de opción a un valor de opción.
* Parámetros: asignación de un nombre de parámetro de consulta a un valor de parámetro de consulta.

> [!NOTE]
> Algunas propiedades con nombre están marcadas como "no usar". Dichas propiedades no deben especificarse por los clientes y no tienen ningún efecto en el servicio.

## <a name="the-clientrequestid-x-ms-client-request-id-named-property"></a>Propiedad con nombre ClientRequestId (x-MS-Client-request-ID)

Esta propiedad con nombre tiene la identidad especificada por el cliente de la solicitud. Los clientes deben especificar un valor único por solicitud con cada solicitud que envíen. Este valor hace que los errores de depuración sean más fáciles de hacer y es necesario en algunos escenarios, como la cancelación de consultas.

El nombre de programación de la propiedad es `ClientRequestId` y se convierte en el encabezado HTTP `x-ms-client-request-id` .

El SDK establecerá esta propiedad en un valor (aleatorio) si el cliente no especifica un valor.

El contenido de esta propiedad puede ser cualquier cadena única que se pueda imprimir, como un GUID.
Sin embargo, se recomienda que los clientes usen: *applicationName* `.` *ActivityName* `;` *UniqueID*

* *ApplicationName* identifica la aplicación cliente que realiza la solicitud.
* *ActivityName* identifica el tipo de actividad para el que la aplicación cliente emite la solicitud del cliente.
* *UniqueID* identifica la solicitud concreta.

## <a name="the-application-x-ms-app-named-property"></a>La propiedad denominada Application (x-MS-APP)

La propiedad denominada Application (x-MS-APP) tiene el nombre de la aplicación cliente que realiza la solicitud y se usa para el seguimiento.

El nombre de programación de esta propiedad es `Application` y se convierte en el encabezado HTTP `x-ms-app` . Se puede especificar en la cadena de conexión Kusto como `Application Name for Tracing` .

Esta propiedad se establecerá en el nombre del proceso que hospeda el SDK si el cliente no especifica su propio valor.

## <a name="the-user-x-ms-user-named-property"></a>Propiedad con nombre user (x-ms-User)

La propiedad con nombre user (x-ms-User) tiene la identidad del usuario que realiza la solicitud y se usa para el seguimiento.

El nombre de programación de esta propiedad es `User` y se convierte en el encabezado HTTP `x-ms-user` . Se puede especificar en la cadena de conexión Kusto como `User Name for Tracing` .

## <a name="controlling-request-properties-using-the-rest-api"></a>Control de las propiedades de solicitud mediante la API de REST

Al emitir una solicitud HTTP al servicio Kusto, use la ranura del `properties` documento JSON que es el cuerpo de la solicitud post para proporcionar las propiedades de la solicitud. 

> [!NOTE]
> Algunas de las propiedades (como "ID. de solicitud de cliente", que es el identificador de correlación que proporciona el cliente al servicio para identificar la solicitud) se pueden proporcionar en el encabezado HTTP y también se pueden establecer si se usa HTTP GET.
Para obtener más información, consulte [el objeto de solicitud de la API de REST de Kusto](../rest/request.md).

## <a name="providing-values-for-query-parameterization-as-request-properties"></a>Proporcionar valores para la parametrización de consultas como propiedades de solicitud

Las consultas de Kusto pueden hacer referencia a los parámetros de consulta mediante una instrucción [declare Query-Parameters](../../query/queryparametersstatement.md) especializada en el texto de la consulta. Esta instrucción permite a las aplicaciones cliente parametrizar las consultas de Kusto en función de los datos proporcionados por el usuario, de forma segura y sin miedo a ataques por inyección.

Mediante programación, establezca los valores de las propiedades mediante los `ClearParameter` `SetParameter` métodos, y `HasParameter` .

En la API de REST, los parámetros de consulta aparecen en la misma cadena codificada con JSON que las otras propiedades de la solicitud.

## <a name="sample-client-code-for-using-request-properties"></a>Código de cliente de ejemplo para usar propiedades de solicitud

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

<!-- The following is auto-generated by running  Kusto.Cli.exe -execute:"#crp -doc"           -->
<!-- The following text can be re-produced by running the Kusto.Cli.exe directive '#crp -doc' -->

* `debug_query_externaldata_projection_fusion_disabled`(*OptionDebugQueryDisableExternalDataProjectionFusion*): si se establece, no confunda la proyección en el operador de ExternalData. Booleano
* `debug_query_fanout_threads_percent_external_data`(*OptionDebugQueryFanoutThreadsPercentExternalData*): el porcentaje de subprocesos en los que se va a deshacer la ejecución, para los nodos de datos externos. Inter
* `deferpartialqueryfailures`(*OptionDeferPartialQueryFailures*): si es true, deshabilita los informes de errores de consulta parciales como parte del conjunto de resultados. Booleano
* `max_memory_consumption_per_query_per_node`(*OptionMaxMemoryConsumptionPerQueryPerNode*): invalida la cantidad máxima predeterminada de memoria que puede asignar una consulta completa por nodo. UInt64
* `maxmemoryconsumptionperiterator`(*OptionMaxMemoryConsumptionPerIterator*): invalida la cantidad de memoria máxima predeterminada que un operador de consulta puede asignar. UInt64
* `maxoutputcolumns`(*OptionMaxOutputColumns*): invalida el número máximo predeterminado de columnas que puede producir una consulta. Tal
* `norequesttimeout`(*OptionNoRequestTimeout*): permite establecer el tiempo de espera de la solicitud en su valor máximo. Booleano
* `notruncation`(*OptionNoTruncation*): habilita la supresión del truncamiento de los resultados de la consulta que se devuelven al autor de la llamada. Booleano
* `push_selection_through_aggregation`(*OptionPushSelectionThroughAggregation*): si es true, inserciones en la selección simple a través de la agregación [Boolean]
* `query_admin_super_slacker_mode`(*OptionAdminSuperSlackerMode*): si es true, la ejecución de la consulta se delega en otro nodo [Boolean]
* `query_bin_auto_at`(*QueryBinAutoAt*): al evaluar la función bin_auto (), el valor inicial que se va a usar. [LiteralExpression]
* `query_bin_auto_size`(*QueryBinAutoSize*): al evaluar la función bin_auto (), el valor de tamaño de la ubicación que se va a usar. [LiteralExpression]
* `query_cursor_after_default`(*OptionQueryCursorAfterDefault*): el valor de parámetro predeterminado de la función cursor_after () cuando se llama sin parámetros. [string]
* `query_cursor_before_or_at_default`(*OptionQueryCursorBeforeOrAtDefault*): el valor de parámetro predeterminado de la función cursor_before_or_at () cuando se llama sin parámetros. [string]
* `query_cursor_current`(*OptionQueryCursorCurrent*): reemplaza el valor del cursor devuelto por las funciones cursor_current () o current_cursor (). [string]
* `query_cursor_scoped_tables`(*OptionQueryCursorScopedTables*): lista de nombres de tabla cuyo ámbito se debe cursor_after_default.. cursor_before_or_at_default (el límite superior es opcional). dinámica
* `query_datascope`(*OptionQueryDataScope*): controla el elemento de la consulta, sobre si la consulta se aplica a todos los datos o solo a parte de ella. [' default ', ' All ' o ' hotcache ']
* `query_datetimescope_column`(*OptionQueryDateTimeScopeColumn*): controla el nombre de columna para el ámbito de fecha y hora de la consulta (query_datetimescope_to/query_datetimescope_from). String@
* `query_datetimescope_from`(*OptionQueryDateTimeScopeFrom*): controla el ámbito de fecha y hora de la consulta (la más antigua). El valor se usa como filtro aplicado automáticamente solo en query_datetimescope_column (si se define). [DateTime]
* `query_datetime\scope_to`(*OptionQueryDateTimeScopeTo*): controla el ámbito de fecha y hora de la consulta (la más reciente). El valor se usa como filtro aplicado automáticamente solo en query_datetimescope_column (si se define). [DateTime]
* `query_distribution_nodes_span`(*OptionQueryDistributionNodesSpanSize*): si se establece, controla la manera en que se comporta la combinación de subconsultas: el nodo en ejecución introducirá un nivel adicional en la jerarquía de consultas para cada subgrupo de nodos. Esta opción establece el tamaño del subgrupo. Inter
* `query_fanout_nodes_percent`(*OptionQueryFanoutNodesPercent*): el porcentaje de nodos en los que se va a deshacer la ejecución. Inter
* `query_fanout_threads_percent`(*OptionQueryFanoutThreadsPercent*): el porcentaje de subprocesos en los que se va a deshacer la ejecución. Inter
* `query_language`(*OptionQueryLanguage*): controla cómo se interpreta el texto de la consulta. [' CSL ', ' kql ' o ' SQL ']
* `query_max_entities_in_union`(*OptionMaxEntitiesToUnion*): invalida el número máximo predeterminado de columnas que puede generar una consulta. Tal
* `query_now`(*OptionQueryNow*): invalida el valor de fecha y hora devuelto por la función Now (0s). [DateTime]
* `query_python_debug`(*OptionDebugPython*): si se establece, genera una consulta de depuración de Python para el nodo de Python enumerado (el valor predeterminado es primero). [Boolean o int]
* `query_results_apply_getschema`(*OptionQueryResultsApplyGetSchema*): si se establece, recupera el esquema de cada uno de los datos tabulares de los resultados de la consulta en lugar de los propios datos. Booleano
* `query_results_cache_max_age`(*OptionQueryResultsCacheMaxAge*): si es positivo, controla la antigüedad máxima de los resultados de la consulta almacenada en caché, que Kusto puede devolver [TimeSpan]
* `query_results_progressive_row_count`(*OptionProgressiveQueryMinRowCountPerUpdate*): indica a Kusto el número de registros que se van a enviar en cada actualización. El valor solo surte efecto si OptionResultsProgressiveEnabled está establecido.
* `query_results_progressive_update_period`(*OptionProgressiveProgressReportPeriod*): indica a Kusto con qué frecuencia se envían los fotogramas de progreso. El valor solo surte efecto si OptionResultsProgressiveEnabled está establecido.
* `query_shuffle_broadcast_join`(*ShuffleBroadcastJoin*): habilita orden aleatorio a través de la combinación de difusión.
* `query_take_max_records`(*OptionTakeMaxRecords*): habilita la limitación de los resultados de la consulta a este número de registros. Tal
* `queryconsistency`(*OptionQueryConsistency*): controla la coherencia de las consultas. [' strongconsistency ' o ' normalconsistency ' o ' weakconsistency ']
* `request_callout_disabled`(*OptionRequestCalloutDisabled*): si se especifica, indica que la solicitud no puede llamar a un servicio proporcionado por el usuario. Booleano
* `request_description`(*OptionRequestDescription*): texto arbitrario que el autor de la solicitud quiere incluir como la descripción de la solicitud. String@
* `request_external_table_disabled`(*OptionRequestExternalTableDisabled*): si se especifica, indica que la solicitud no puede invocar código en ExternalTable. Booleano
* `request_readonly`(*OptionRequestReadOnly*): si se especifica, indica que la solicitud no puede escribir nada. Booleano
* `request_remote_entities_disabled`(*OptionRequestRemoteEntitiesDisabled*): si se especifica, indica que la solicitud no puede tener acceso a las bases de datos y clústeres remotos. Booleano
* `request_sandboxed_execution_disabled`(*OptionRequestSandboxedExecutionDisabled*): si se especifica, indica que la solicitud no puede invocar código en el espacio aislado. Booleano
* `results_progressive_enabled`(*OptionResultsProgressiveEnabled*): si se establece, habilita la secuencia de consulta progresiva.
* `servertimeout`(*OptionServerTimeout*): invalida el tiempo de espera de la solicitud predeterminado. TimeSpan
* `truncationmaxrecords`(*OptionTruncationMaxRecords*): invalida el número máximo predeterminado de registros que una consulta puede devolver al autor de la llamada (truncamiento). Tal
* `truncationmaxsize`(*OptionTruncationMaxSize*): invalida el tamaño máximo predeterminado de los datos que una consulta puede devolver al autor de la llamada (truncamiento). Tal
* `validate_permissions`(*OptionValidatePermissions*): valida los permisos del usuario para realizar la consulta y no ejecuta la consulta. Booleano
