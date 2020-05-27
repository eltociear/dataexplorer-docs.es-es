---
title: Usar Kusto. Explorer
description: Aprenda a usar Kusto. Explorer
author: orspod
ms.author: orspodek
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: conceptual
ms.date: 05/19/2020
ms.openlocfilehash: 98a42fd72a28089e6add53aed5346ce2d0e5d993
ms.sourcegitcommit: b4d6c615252e7c7d20fafd99c5501cb0e9e2085b
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2020
ms.locfileid: "83866142"
---
# <a name="using-kustoexplorer"></a>Usar Kusto. Explorer

Kusto. Explorer es una aplicación de escritorio que le permite explorar los datos mediante el lenguaje de consulta de Kusto en una interfaz de usuario fácil de usar. En este artículo se muestra cómo usar los modos de búsqueda y consulta, compartir las consultas y administrar clústeres, bases de datos y tablas.

## <a name="search-mode"></a>Modo buscar + +

El modo buscar + + le permite buscar un término mediante la sintaxis de búsqueda en una o varias tablas.

1. En la pestaña Inicio, en el menú desplegable consulta, seleccione **Buscar + +**.
1. Seleccione **varias tablas**.
1. En **elegir tablas**, defina las tablas que desea buscar.
1. En el cuadro de edición, escriba la frase de búsqueda y seleccione **ir**.
1. Un mapa térmico de la cuadrícula tabla/espacio de tiempo muestra los términos que aparecen y dónde aparecen.

:::image type="content" source="images/kusto-explorer-using/search-plus-plus.png" alt-text="Buscar + + explorador de Kusto":::

1. Seleccione una celda de la cuadrícula y seleccione **Ver detalles** para mostrar las entradas correspondientes en el panel de resultados.

:::image type="content" source="images/kusto-explorer-using/search-plus-plus-results.png" alt-text="Resultados de Search + + del explorador de Kusto":::

## <a name="query-mode"></a>modo de consulta

Kusto. Explorer incluye un eficaz modo de script que le permite escribir, editar y ejecutar consultas ad hoc. El modo de script incluye el resaltado de sintaxis e IntelliSense, por lo que puede aumentar rápidamente su conocimiento del lenguaje de consulta Kusto.

En este documento se describe cómo ejecutar consultas básicas en Kusto. Explorer y cómo agregar parámetros a las consultas.

## <a name="basic-queries"></a>Consultas básicas

Si tiene registros de tabla, puede empezar a explorarlos:

<!-- csl: https://help.kusto.windows.net:443/Samples -->

```kusto
StormEvents | count 
```

Cuando el cursor está en esta línea, se muestra en color gris. Presione **F5** para ejecutar la consulta. 

A continuación se muestran algunas consultas de ejemplo:

<!-- csl: https://help.kusto.windows.net:443/Samples -->

```kusto
// Take 10 lines from the table. Useful to get familiar with the data
StormEvents | limit 10 
```

<!-- csl: https://help.kusto.windows.net:443/Samples -->

```kusto
// Filter by EventType == 'Flood' and State == 'California' (=~ means case insensitive) 
// and take sample of 10 lines
StormEvents 
| where EventType == 'Flood' and State =~ 'California'
| limit 10
```

:::image type="content" source="images/kusto-explorer-using/basic-query.png" alt-text="Consulta básica de Kusto Explorer":::

Más información sobre el [lenguaje de consulta de Kusto](https://docs.microsoft.com/azure/kusto/query/).

## <a name="client-side-query-parameterization"></a>Parametrización de consultas del lado cliente

> [!Note]
> Hay dos tipos de técnicas de parametrización de consulta en Kusto:
> * [Language-Integrated Query parametrización](../query/queryparametersstatement.md) se implementa como parte del motor de consultas y está pensado para que lo usen las aplicaciones que consultan el servicio mediante programación. Este método no se describe en este documento.
>
> * La consulta del lado cliente parametrización, que se describe a continuación, es una característica de la aplicación Kusto. Explorer únicamente. Es equivalente a usar operaciones de reemplazo de cadenas en las consultas antes de enviarlas para que las ejecute el servicio. La sintaxis que se describe a continuación no forma parte del lenguaje de consulta propiamente dicho y no se puede usar cuando se envían consultas al servicio por medio de Kusto. Explorer.

Si usa el mismo valor en varias consultas o en varias pestañas, es muy conveniente cambiar ese valor en cada lugar en el que se use. Este es el motivo por el que Kusto. Explorer admite parámetros de consulta. Los parámetros de consulta se comparten entre las pestañas para que se puedan reutilizar fácilmente. Los parámetros se indican entre {} corchetes. Por ejemplo: `{parameter1}`

El editor de scripts resalta los parámetros de consulta:

:::image type="content" source="images/kusto-explorer-using/parametrized-query-1.png" alt-text="Consulta parametrizadas 1":::

Puede definir y editar fácilmente los parámetros de consulta existentes:


:::image type="content" source="images/kusto-explorer-using/parametrized-query-2.png" alt-text="Editar consulta parametrizadas 2":::


:::image type="content" source="images/kusto-explorer-using/parametrized-query-3.png" alt-text="Editar consulta parametrizadas 3":::

El editor de scripts también tiene IntelliSense para los parámetros de consulta que ya están definidos:

:::image type="content" source="images/kusto-explorer-using/parametrized-query-4.png" alt-text="Paramaterized consulta IntelliSense":::

Puede tener varios conjuntos de parámetros (enumerados en el cuadro combinado de **conjuntos de parámetros** ).
Seleccione **Agregar nuevo** o **eliminar actual** para manipular la lista de conjuntos de parámetros.

:::image type="content" source="images/kusto-explorer-using/parametrized-query-5.png" alt-text="Lista de conjuntos de parámetros":::

## <a name="share-queries-and-results"></a>Compartir consultas y resultados

En Kusto. Explorer, puede compartir consultas y resultados por correo electrónico. También puede crear vínculos profundos que abrirán y ejecutarán una consulta en el explorador.

### <a name="share-queries-and-results-by-email"></a>Compartir consultas y resultados por correo electrónico

Kusto. Explorer proporciona una manera cómoda de compartir consultas y resultados de consultas por correo electrónico.

1. [Ejecute la consulta](#basic-queries) en Kusto. Explorer.
1. En la pestaña Inicio, en la sección compartir, seleccione **exportar al portapapeles** (o presione Ctrl + Mayús + C).

:::image type="content" source="images/kusto-explorer-using/menu-export.png" alt-text="Exportar al portapapeles":::

    Kusto.Explorer pastes the following to the clipboard:
    * Your query
    * The query results (table or chart)
    * The connection details for the Kusto cluster and database
    * A link that will rerun the query automatically

1. Pegue el contenido del portapapeles en un nuevo mensaje de correo electrónico.

:::image type="content" source="images/kusto-explorer-using/share-results-2.png" alt-text="Compartir los resultados por correo electrónico":::

### <a name="deep-linking-queries"></a>Consultas de vínculo profundo

Puede crear un URI que, cuando se abre en un explorador, abre Kusto. Explorer localmente y ejecuta una consulta específica en una base de datos Kusto especificada.

### <a name="limitations"></a>Limitaciones

Las consultas están limitadas a ~ 2000 caracteres debido a las limitaciones del explorador, los proxies HTTP y las herramientas que validan los vínculos, como Microsoft Outlook. La limitación es aproximada porque depende de la longitud del clúster y del nombre de la base de datos. Para más información, consulte [https://support.microsoft.com/kb/208427](https://support.microsoft.com/kb/208427). Para reducir las posibilidades de alcanzar el límite de caracteres, vea [obtener vínculos más cortos](#getting-shorter-links), más abajo.

El formato del URI es:`https://<ClusterCname>.kusto.windows.net/<DatabaseName>web=0?query=<QueryToExecute>`

Por ejemplo:  [https://help.kusto.windows.net/Samples?web=0query=StormEvents+%7c+limit+10](https://help.kusto.windows.net/Samples?web=0query=StormEvents+%7c+limit+10)
 
Este URI abrirá Kusto. Explorer, se conectará al `Help` clúster de Kusto y ejecutará la consulta especificada en la `Samples` base de datos. Si hay una instancia de Kusto. Explorer que ya se está ejecutando, la instancia en ejecución abrirá una nueva pestaña y ejecutará la consulta en ella.

> [!Note] 
> Por motivos de seguridad, la vinculación profunda está deshabilitada para los comandos de control.

### <a name="creating-a-deep-link"></a>Crear un vínculo profundo

La forma más fácil de crear un vínculo profundo es crear la consulta en Kusto. Explorer y, a continuación, usar `Export to Clipboard` para copiar la consulta (incluidos el vínculo profundo y los resultados) en el portapapeles. Después, puede compartirlo por correo electrónico.
        
Cuando se copia en un correo electrónico, el vínculo profundo se muestra en una fuente pequeña. Por ejemplo:

https://help.kusto.windows.net:443/Samples[[Haga clic para ejecutar la consulta](https://help.kusto.windows.net/Samples?web=0&query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSspVuDlqlEoLs3NTSzKrEpVSM4vzSvR0FRIqlRIyszTCC5JLCoJycxN1VEwT9EEKS1KzUtJLVIoAYolZwAlFQCB3oo%2bTAAAAA%3d%3d)] 

El primer vínculo abre Kusto. Explorer y establece el contexto del clúster y de la base de datos de forma adecuada.
El segundo vínculo ( `Click to run query` ) es el vínculo profundo. Si mueve el vínculo a un mensaje de correo electrónico y presiona CTRL + K, puede ver la dirección URL real:

https://help.kusto.windows.net/Samples?web=0&query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSspVuDlqlEoLs3NTSzKrEpVSM4vzSvR0FRIqlRIyszTCC5JLCoJycxN1VEwT9EEKS1KzUtJLVIoAYolZwAlFQCB3oo%2bTAAAAA%3d%3d

### <a name="deep-links-and-parametrized-queries"></a>Vínculos profundos y consultas parametrizadas

Puede usar consultas parametrizadas con una vinculación profunda.

1. Cree una consulta que se forme como una consulta parametrizadas (por ejemplo, `KustoLogs | where Timestamp > ago({Period}) | count` ) 
1. Proporcione un parámetro para cada parámetro de consulta en el URI en este caso:

https://mycluster.kusto.windows.net/MyDatabase?web=0&query=KustoLogs+%7c+where+Timestamp+>+ago({Period})+%7c+count&Period=1h

### <a name="getting-shorter-links"></a>Obtener vínculos más cortos

Las consultas pueden llegar a ser largas. Para reducir la posibilidad de que la consulta supere la longitud máxima, use el `String Kusto.Data.Common.CslCommandGenerator.EncodeQueryAsBase64Url(string query)` método disponible en la biblioteca de cliente de Kusto. Este método genera una versión más compacta de la consulta. Kusto. Explorer también reconoce el formato más corto.

https://help.kusto.windows.net/Samples?web=0&query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSspVuDlqlEoLs3NTSzKrEpVSM4vzSvR0FRIqlRIyszTCC5JLCoJycxN1VEwT9EEKS1KzUtJLVIoAYolZwAlFQCB3oo%2bTAAAAA%3d%3d

La consulta se hace más compacta aplicando la siguiente transformación:

```csharp
 UrlEncode(Base64Encode(GZip(original query)))
```

## <a name="kustoexplorer-command-line-arguments"></a>Argumentos de la línea de comandos de Kusto. Explorer

Los argumentos de la línea de comandos se usan para configurar la herramienta con el fin de realizar funciones adicionales en el inicio. Por ejemplo, cargar un script y conectarse a un clúster. Como tal, los argumentos de la línea de comandos no reemplazan ninguna funcionalidad de Kusto. Explorer.

Los argumentos de la línea de comandos se pasan como parte de la dirección URL que se usa para abrir la aplicación, de forma similar a la [consulta de vínculos profundos](#creating-a-deep-link).

## <a name="command-line-argument-syntax"></a>Sintaxis de los argumentos de línea de comandos

Kusto. Explorer admite varios argumentos de línea de comandos en la siguiente sintaxis (el orden es importante):

[*LocalScriptFile*] [*QueryString*]

* *LocalScriptFile* es el nombre de un archivo de script en el equipo local, que debe tener la extensión `.kql` . Si existe este tipo de archivo, Kusto. Explorer carga automáticamente este archivo cuando se inicia.
* *QueryString* es una cadena que utiliza el formato de cadena de consulta http. Este método proporciona propiedades adicionales, como se describe en la tabla siguiente.

Por ejemplo, para iniciar Kusto. Explorer con un archivo de script denominado `c:\temp\script.kql` y configurado para comunicarse con `help` el clúster, base de datos `Samples` , use el siguiente comando:

```kusto
Kusto.Explorer.exe c:\temp\script.kql uri=https://help.kusto.windows.net/Samples;Fed=true&name=Samples
```

|Argumento  |Descripción                                                               |
|----------|--------------------------------------------------------------------------|
|**Consulta para ejecutar**                                                                 |
|`query`   |Consulta que se va a ejecutar (con codificación Base64). Si está vacío, use `querysrc` .          |
|`querysrc`|La dirección URL de un archivo o BLOB que contiene la consulta que se va a ejecutar (si `query` está vacía).|
|**Conexión al clúster de Kusto**                                                  |
|`uri`     |Cadena de conexión del clúster de Kusto al que se va a conectar.                 |
|`name`    |El nombre para mostrar de la conexión con el clúster de Kusto.                  |
|**Grupo de conexiones**                                                                 |
|`path`    |Dirección URL de un archivo de grupo de conexiones que se va a descargar (con codificación URL).             |
|`group`   |Nombre del grupo de conexión.                                         |
|`filename`|El archivo local que contiene el grupo de conexiones.                              |


## <a name="manage-clusters-databases-tables-or-function-authorized-principals"></a>Administración de clústeres, bases de datos, tablas o entidades de seguridad autorizadas de función

> [!Note]
> Solo los [administradores](../management/access-control/role-based-authorization.md) pueden agregar o quitar entidades de seguridad autorizadas en su propio ámbito.

Haga clic con el botón secundario en la entidad de destino en el [Panel conexiones](kusto-explorer.md#connections-tab)y seleccione **administrar entidades de seguridad autorizadas de clúster**. (También puede seleccionar esta opción en el menú de administración).

:::image type="content" source="images/kusto-explorer-using/right-click-manage-authorized-principals.png" alt-text="Administrar entidades de seguridad autorizadas":::

:::image type="content" source="images/kusto-explorer-using/manage-authorized-principals-window.png" alt-text="Ventana administrar entidades de seguridad autorizadas":::

* Para agregar una nueva entidad de seguridad autorizada, seleccione **Agregar entidad**de seguridad, proporcione los detalles de la entidad de seguridad y confirme la acción.
    
    :::image type="content" source="images/kusto-explorer-using/add-authorized-principals-window.png" alt-text="Agregar entidad de seguridad autorizada":::

    :::image type="content" source="images/kusto-explorer-using/confirm-add-authorized-principals.png" alt-text="Confirmar agregar entidad de seguridad autorizada":::

* Para quitar una entidad de seguridad autorizada existente, seleccione **Drop principal** y confirme la acción.

    :::image type="content" source="images/kusto-explorer-using/confirm-drop-authorized-principals.png" alt-text="Confirmar eliminación de entidad de seguridad autorizada":::


## <a name="next-steps"></a>Pasos siguientes

* [Métodos abreviados de teclado de Kusto. Explorer](kusto-explorer-shortcuts.md)
* [Opciones de Kusto. Explorer](kusto-explorer-options.md)
* [Solución de problemas de Kusto. Explorer](kusto-explorer-troubleshooting.md)

Más información sobre las herramientas y utilidades de Kusto. Explorer:
* [Analizador de código Kusto. Explorer](kusto-explorer-code-analyzer.md)
* [Navegación por el código Kusto. Explorer](kusto-explorer-codenav.md)
* [Refactorización de código de Kusto. Explorer](kusto-explorer-refactor.md)
* [Lenguaje de consulta de Kusto (KQL)](https://docs.microsoft.com/azure/kusto/query/)
