---
title: 'CLI de Kusto: Azure Explorador de datos'
description: En este artículo se describe la CLI de Kusto en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/24/2020
ms.openlocfilehash: adc606c787ab20f228a9fbd132d1b82660aa57c6
ms.sourcegitcommit: 188f89553b9d0230a8e7152fa1fce56c09ebb6d6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/08/2020
ms.locfileid: "84512493"
---
# <a name="kusto-cli"></a>CLI de Kusto

Kusto. CLI es una utilidad de línea de comandos que se usa para enviar solicitudes a Kusto y mostrar los resultados. Puede ejecutarse en uno de varios modos:

* *Modo REPL*: el usuario escribe consultas y comandos, y la herramienta muestra los resultados y, a continuación, espera el siguiente comando o consulta de usuario.
  ("REPL" significa "lectura/evaluación/impresión/bucle").

* *Modo de ejecución*: el usuario escribe una o más consultas y comandos que se ejecutan como argumentos de la línea de comandos. Los argumentos se ejecutan automáticamente en secuencia y sus resultados se muestran en la consola. Opcionalmente, después de que se hayan ejecutado todas las consultas de entrada y comandos, la herramienta pasa al modo REPL.

* *Modo de script*: similar al modo de ejecución, pero con las consultas y los comandos especificados a través de un archivo "script".

Kusto. CLI se proporciona principalmente para la automatización de tareas en un servicio de Kusto que normalmente requiere la escritura de código. Por ejemplo, un programa de C# o un script de PowerShell.

## <a name="get-the-tool"></a>Obtener la herramienta

Kusto. CLI forma parte del paquete de NuGet `Microsoft.Azure.Kusto.Tools` que [puede descargar](https://www.nuget.org/packages/Microsoft.Azure.Kusto.Tools/).
Una vez descargado, extraiga la carpeta del paquete `tools` en la carpeta de destino. No se requiere ninguna instalación adicional, ya que se instala con XCopy.

## <a name="run-the-tool"></a>Ejecución de la herramienta

Kusto. CLI requiere al menos un argumento de línea de comandos para ejecutarse. Normalmente, ese argumento es la cadena de conexión al servicio Kusto al que se debe conectar la herramienta.
Para obtener más información, vea [cadenas de conexión de Kusto](../api/connection-strings/kusto.md). Si ejecuta la herramienta sin argumentos de línea de comandos, con un conjunto de argumentos desconocido, o con el `/help` modificador, se mostrará un mensaje de ayuda en la consola.

Por ejemplo, use el siguiente comando para ejecutar Kusto. CLI. El comando se conectará al `help` servicio Kusto y establecerá el contexto de la base de datos en la `Samples` base de datos:

```
Kusto.Cli.exe "https://help.kusto.windows.net/Samples;Fed=true"
```

> [!NOTE]
> Use comillas dobles alrededor de la cadena de conexión para evitar que las aplicaciones de Shell, como PowerShell, no interpreten el punto y coma ( `;` ) y caracteres similares.

## <a name="command-line-arguments"></a>Argumentos de la línea de comandos

`Kusto.Cli.exe`*ConnectionString* [*Modificadores*]

*ConnectionString*
* La [cadena de conexión de Kusto](../api/connection-strings/kusto.md) que contiene toda la información de conexión de Kusto.
  Su valor predeterminado es `net.tcp://localhost/NetDefaultDB`.

`-execute:`*QueryOrCommand*
* Si se especifica, ejecuta Kusto. CLI en modo de ejecución y se ejecuta la consulta o el comando especificados. Este modificador se puede repetir y las consultas y los comandos se ejecutan secuencialmente en orden de aparición.
  Este modificador no se puede usar junto con `-script` o `-scriptml` .

`-keepRunning:`*EnableKeepRunning*
* Si se especifica, como `true` o `false` , habilita o deshabilita el modo REPL una vez `-script` `-execute` procesados todos los valores o.

`-script:`*ScriptFile*
* Si se especifica, ejecuta Kusto. CLI en modo de script. Se carga el archivo de script especificado y las consultas o los comandos que hay en él se ejecutan secuencialmente.
  Las líneas nuevas se utilizan para delimitar las consultas y los comandos, excepto cuando las líneas terminan con una `&` combinación de o `&&` , como se explica a continuación.
  Este modificador no se puede usar junto con `-execute` .

`-scriptml:`*ScriptFile*
* Si se especifica, ejecuta Kusto. CLI en modo de script. Se carga el archivo de script especificado y las consultas o los comandos que hay en él se ejecutan secuencialmente.
  El archivo de script completo se considera una consulta o un comando únicos.
  Este modificador no se puede usar junto con `-execute` .

`-echo:`*EnableEchoMode*
* Si se especifica, como `true` o `false` , habilita o deshabilita el modo de eco.
  Cuando el modo de eco está habilitado, todas las consultas o comandos se repiten en la salida.

`-transcript:`*TranscriptFile*  
* Si se especifica, escribe la salida del programa en *TranscriptFile*.

`-logToConsole:`*EnableLogToConsole*
* Si se especifica, como `true` o `false` , habilita o deshabilita la presentación de la salida del programa en la consola.

`-lineMode:`*EnableLineMode*
* Si se especifica, cambia entre el modo de entrada de línea predeterminado, cuando se establece en `true` y el modo de entrada de bloque, cuando se establece en `false` . A continuación se muestra una explicación de estos dos modos, que determinan cómo se tratan las líneas nuevas.

**Ejemplo**

```
Kusto.Cli.exe "https://kustolab.kusto.windows.net/;Fed=true" -script:"c:\mycommands.txt"
```

> [!NOTE] 
> No debe haber ningún espacio entre los dos puntos y el valor del argumento

## <a name="directives"></a>Directivas

Kusto. CLI ejecuta varias directivas en la herramienta en lugar de enviarlas al servicio para su procesamiento.

|Directiva                      |Descripción|
|-------------------------------|-----------|
|`?`<br>`#h`<br>`#help`         |Obtener un breve mensaje de ayuda|
|`q`<br>`#quit`<br>`#exit`      |Salir de la herramienta|
|`#a`<br>`#abort`               |Salga de la herramienta abortively|
|`#clip`                        |Los resultados de la siguiente consulta o comando se copiarán en el portapapeles.|
|`#cls`                         |Borrar la pantalla de la consola|
|`#connect`*[ConnectionString*]|Se conecta a un servicio Kusto diferente (si se omite *ConnectionString* , se mostrará el actual)|
|`#crp`[*Nombre* [ `=` *valor*]]   |Establece el valor de una propiedad de solicitud de cliente, o simplemente la muestra, o muestra todos los valores.|
|`#crp`( `-list` \| `-doc` ) [*Prefijo*]|Muestra las propiedades de la solicitud de cliente, por prefijo o todas|
|`#dbcontext`[*NombreDeBaseDeDatos*]  |Cambia la base de datos "context" utilizada por las consultas y los comandos a *DatabaseName*. Si se omite, se muestra el contexto actual|
|`ke` *Text*                    |Envía el texto especificado a un proceso Kusto. Explorer en ejecución.|
|`#loop`*Texto* de *recuento*         |Ejecuta el texto un número de veces.|
|`#qp`[*Nombre* [ `=` *valor*]]   |Establece el valor de un parámetro de consulta, o simplemente lo muestra, o muestra todos los valores. Se recortarán las comillas simples o dobles al principio o al final|
|`#save`*Nombre de archivo*             |Los resultados de la siguiente consulta o comando se guardarán en el archivo CSV indicado|
|`#script`*Nombre de archivo*           |Ejecuta el script indicado|
|`#scriptml`*Nombre de archivo*         |Ejecuta el script de varias líneas indicado|

## <a name="line-input-mode-and-block-input-mode"></a>Modo de entrada de línea y modo de entrada de bloque

De forma predeterminada, Kusto. CLI se ejecuta en **modo de entrada de línea**. Cada carácter de nueva línea se interpreta como un delimitador entre las consultas y los comandos, y la línea se envía inmediatamente para su ejecución.

En este modo, puede dividir una consulta o un comando largos en varias líneas. El `&` carácter como el último carácter de una línea, antes de la nueva línea, hace que Kusto. CLI continúe leyendo la línea siguiente. El `&&` carácter como el último carácter de una línea, antes de la nueva línea, hace que Kusto. CLI omita la nueva línea y continúe leyendo la línea siguiente.

Kusto. CLI también admite la ejecución en **modo de entrada de bloque**. Mediante el uso del modificador de la línea de comandos `-lineMode:false` o mediante la directiva `#blockmode` , puede indicar a Kusto. CLI que asuma que cada línea es una continuación de la línea anterior, de modo que las consultas y los comandos se delimiten solo con una línea de entrada vacía.

## <a name="comments"></a>Comentarios

Kusto. CLI interpreta una `//` cadena que comienza la nueva línea como una línea de comentario. Omite el resto de la línea y continúa leyendo la línea siguiente.

## <a name="tool-only-options"></a>Opciones de solo herramientas

Comandos                        | Efecto                                                                            | Seleccionadas
--------------------------------|-----------------------------------------------------------------------------------|-----------
#timeon|#timeoff                | opción Habilitar/deshabilitar `timing` : mostrar las solicitudes de tiempo tomadas                    | VERDADERO
#tableon|#tableoff              | opción Habilitar/deshabilitar `tableView` : dar formato a los conjuntos de resultados como tablas                  | VERDADERO
#marson|#marsoff                | opción Habilitar/deshabilitar `marsView` : mostrar los conjuntos de resultados de segundo a último          | FALSO
#resultson|#resultsoff          | opción Habilitar/deshabilitar `outputResultsSet` : mostrar los conjuntos de resultados                 | VERDADERO
#prettyon|#prettyoff            | opción Habilitar/deshabilitar `prettyErrors` : limpieza de errores                             | VERDADERO
#markdownon|#markdownoff        | opción Habilitar/deshabilitar `markdownView` : dar formato a las tablas como MarkDown                   | FALSO
#progressiveon|#progressiveoff  | opción Habilitar/deshabilitar `progressiveView` : pedir y mostrar resultados progresivos  | FALSO
#linemode|#blockmode            | opción Enable/Disable `lineMode` : modo de entrada de una sola línea                          | VERDADERO

Comandos                              | Efecto                                                                                                         | Valor predeterminado
--------------------------------------|----------------------------------------------------------------------------------------------------------------|-----------
#cridon|#cridoff                      | (habilitar|opción disable `crid` : mostrar el ClientRequestId antes de enviar la solicitud)                          | FALSO
#csvheaderson|#csvheadersoff          | (habilitar|opción disable `csvHeaders` : incluir encabezados en la salida CSV)                                            | VERDADERO
#focuson|#focusoff                    | (habilitar|opción de deshabilitación `focus` : Quite todas las desenfadados adicionales y céntrese en el material adecuado)                        | FALSO
#linemode|#blockmode                  | (habilitar|opción disable `lineMode` : modo de entrada de una sola línea)                                                      | VERDADERO
#markdownon|#markdownoff              | (habilitar|opción disable `markdownView` : dar formato a las tablas como MarkDown                                              | FALSO
#marson|#marsoff                      | (habilitar|opción disable `marsView` : mostrar los conjuntos de resultados de segundo a último.                                      | FALSO
#prettyon|#prettyoff                  | (habilitar|opción de deshabilitación `prettyErrors` : limpieza de errores)                                                        | VERDADERO
#querystreamingon|#querystreamingoff  | (habilitar|opción disable `queryStreaming` : usar el punto de conexión queryStreaming (solo en el equipo Kusto))                    | FALSO
#resultson|#resultsoff                | (habilitar|opción disable `outputResultsSet` : mostrar los conjuntos de resultados)                                            | VERDADERO
#tableon|#tableoff                    | (habilitar|opción disable `tableView` : dar formato a los conjuntos de resultados como tablas)                                              | VERDADERO
#timeon|#timeoff                      | (habilitar|opción disable `timing` : muestra la cantidad de tiempo que Tardón las solicitudes.                               | VERDADERO
#typeon|#typeoff                      | (habilitar|opción disable `typeView` : muestra el tipo de cada columna en la vista de tabla. Fuerzas de streaming = true)| VERDADERO
#v2protocolon|#v2protocoloff          | (habilitar|opción disable `v2protocol` : usar el protocolo de consulta V2, no V1)                                        | VERDADERO

## <a name="use-kustocli-to-export-results-as-csv"></a>Use Kusto. CLI para exportar los resultados como CSV.

Kusto. CLI tiene un comando especial del lado cliente `#save` que exporta los resultados de la consulta **siguiente** a un archivo local en formato CSV. Por ejemplo, en la línea siguiente se exportarán 10 registros de la `StormEvents` tabla al `help.kusto.windows.net` clúster, `Samples` base de datos:

```
Kusto.Cli.exe @help/Samples -execute:"#save c:\temp\test.log" -execute:"StormEvents | take 10"
```

## <a name="use-kustocli-to-control-a-running-instance-of-kustoexplorer"></a>Use Kusto. CLI para controlar una instancia en ejecución de Kusto. Explorer

Puede indicar a Kusto. CLI que se comunique con la instancia "principal" de Kusto. Explorer que se ejecuta en el equipo y envíe consultas de ti. Este mecanismo puede ser útil para los programas que desean ejecutar un número de consultas, pero no desea iniciar el proceso Kusto. Explorer varias veces. En el ejemplo siguiente, se usa Kusto. CLI para ejecutar una consulta en el clúster de ayuda:

```
#connect cluster('help').database('Samples')

#ke StormEvents | count
```

La sintaxis es sencilla: `#ke` , seguida de un espacio en blanco y la consulta que se va a ejecutar.
A continuación, la consulta se envía a la instancia principal de Kusto. Explorer, si existe, con la base de datos o el clúster actual establecidos en Kusto. CLI.
 
