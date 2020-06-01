---
title: 'CLI de Kusto: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe la CLI de Kusto en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/24/2020
ms.openlocfilehash: ab82698054e4bcb851f9f05acdd62af569e0704b
ms.sourcegitcommit: 9fe6e34ef3321390ee4e366819ebc9b132b3e03f
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/01/2020
ms.locfileid: "84258120"
---
# <a name="kusto-cli"></a>CLI de Kusto

Kusto. CLI es una utilidad de línea de comandos que se puede usar para enviar solicitudes a Kusto y mostrar los resultados. Kusto. CLI puede ejecutarse en uno de varios modos:

* *Modo de REPL*: el usuario escribe en las consultas y los comandos para que se ejecuten en Kusto, y la herramienta muestra los resultados y, a continuación, espera el siguiente comando o consulta de usuario.
  ("REPL" significa "lectura/evaluación/impresión/bucle").

* *Modo de ejecución*: el usuario proporciona una o más consultas y comandos para que se ejecuten como argumentos de línea de comandos para la herramienta. Se ejecutan en secuencia automáticamente y sus resultados se muestran en la consola. Opcionalmente, al ejecutar todas las consultas de entrada y comandos, la herramienta entra en modo REPL.

* *Modo de script*: similar al modo de ejecución, pero con las consultas y los comandos especificados a través de un archivo (denominado "script").

Kusto. CLI se proporciona principalmente para la automatización de tareas en un servicio de Kusto que normalmente se habría requerido para escribir código (por ejemplo, un programa de C# o un script de PowerShell).

## <a name="getting-the-tool"></a>Obtener la herramienta

Kusto. CLI forma parte del paquete NuGet `Microsoft.Azure.Kusto.Tools` , que se puede descargar [aquí](https://www.nuget.org/packages/Microsoft.Azure.Kusto.Tools/).
Una vez descargado, la carpeta del paquete se `tools` puede extraer en la carpeta de destino; no se requiere ninguna instalación adicional (es decir, se puede instalar con xcopy).



## <a name="running-the-tool"></a>Ejecución de la herramienta

Kusto. CLI requiere al menos un argumento de línea de comandos para ejecutarse. Normalmente, ese argumento es la cadena de conexión al servicio Kusto al que se debe conectar la herramienta; consulte [cadenas de conexión de Kusto](../api/connection-strings/kusto.md). La ejecución de la herramienta sin argumentos de línea de comandos o con un conjunto de argumentos desconocido, o con el `/help` modificador, da lugar a la emisión de un mensaje de ayuda a la consola.

Por ejemplo, use el siguiente comando para ejecutar Kusto. CLI y conectarse al `help` servicio Kusto y establecer el contexto de la base de datos en la `Samples` base de datos:

```
Kusto.Cli.exe "https://help.kusto.windows.net/Samples;Fed=true"
```

> [!NOTE]
> Hemos usado comillas alrededor de la cadena de conexión para evitar que las aplicaciones de Shell, como PowerShell, interpreten el punto y coma ( `;` ) y caracteres similares en la cadena de conexión.

## <a name="command-line-arguments"></a>Argumentos de la línea de comandos

`Kusto.Cli.exe`*ConnectionString* [*Modificadores*]

*ConnectionString*
* La [cadena de conexión de Kusto](../api/connection-strings/kusto.md) que contiene toda la información de conexión de Kusto.
  Tiene como valor predeterminado `net.tcp://localhost/NetDefaultDB`.

`-execute:`*QueryOrCommand*
* Si se especifica, ejecuta Kusto. CLI en modo de ejecución; se ejecuta la consulta o el comando especificado. Este modificador se puede repetir, en cuyo caso las consultas y los comandos se ejecutan secuencialmente en orden de aparición.
  Este modificador no se puede usar junto con `-script` o `-scriptml` .

`-keepRunning:`*EnableKeepRunning*
* Si se especifica (como `true` o `false` ), habilita o deshabilita el cambio al modo REPL una `-script` vez `-execute` procesados todos los valores o.

`-script:`*ScriptFile*
* Si se especifica, ejecuta Kusto. CLI en modo de script. se carga el archivo de script especificado y las consultas o los comandos que hay en él se ejecutan secuencialmente.
  Las líneas nuevas se utilizan para delimitar las consultas y los comandos, excepto cuando las líneas terminan con `&` `&&` combinaciones or, como se explica a continuación.
  Este modificador no se puede usar junto con `-execute` .

`-scriptml:`*ScriptFile*
* Si se especifica, ejecuta Kusto. CLI en modo de script. se carga el archivo de script especificado y las consultas o los comandos que hay en él se ejecutan secuencialmente.
  El archivo de script completo se considera una consulta o un comando únicos.
  Este modificador no se puede usar junto con `-execute` .

`-echo:`*EnableEchoMode*
* Si se especifica (como `true` o `false` ), habilita o deshabilita el modo de eco.
  Cuando el modo de eco está habilitado, todas las consultas o comandos se repiten en la salida.

`-transcript:`*TranscriptFile*  
* Si se especifica, escribe la salida del programa en *TranscriptFile*.

`-logToConsole:`*EnableLogToConsole*
* Si se especifica (como `true` o `false` ), habilita o deshabilita la escritura de la salida del programa en la consola.

`-lineMode:`*EnableLineMode*
* Si se especifica, cambia entre el modo de entrada de línea (el valor predeterminado o cuando se establece en `true` ) y el modo de entrada de bloque (cuando se establece en `false` ). A continuación se muestra una explicación de estos dos modos, que determinan cómo se tratan las líneas nuevas.

**Ejemplo**

```
Kusto.Cli.exe "https://kustolab.kusto.windows.net/;Fed=true" -script:"c:\mycommands.txt"
```

Tenga en cuenta que no debería haber ningún espacio entre los dos puntos y el valor del argumento.

## <a name="directives"></a>Directivas

Kusto. CLI es compatible con una serie de directivas que se ejecutan en la herramienta en lugar de enviarse al servicio para su procesamiento:

|Directiva                      |Descripción|
|-------------------------------|-----------|
|`?`<br>`#h`<br>`#help`         |Obtener un breve mensaje de ayuda.|
|`q`<br>`#quit`<br>`#exit`      |Salga de la herramienta.|
|`#a`<br>`#abort`               |Salga de la herramienta abortively.|
|`#clip`                        |Los resultados de la siguiente consulta o comando se copiarán en el portapapeles.|
|`#cls`                         |Borre la pantalla de la consola.|
|`#connect`*[ConnectionString*]|Se conecta a un servicio Kusto diferente (si se omite *ConnectionString* , se mostrará el actual).|
|`#crp`[*Nombre* [ `=` *valor*]]   |Establece el valor de una propiedad de solicitud de cliente (o simplemente la muestra o muestra todos los valores).|
|`#crp`( `-list` \| `-doc` ) [*Prefijo*]|Muestra las propiedades de la solicitud de cliente (por prefijo o todas).|
|`#dbcontext`[*NombreDeBaseDeDatos*]  |Cambia la base de datos "context" utilizada por las consultas y los comandos a *DatabaseName* (si se omite, se mostrará el contexto actual).|
|`ke`*Texto* de                    |Envía el texto especificado a un proceso Kusto. Explorer en ejecución.|
|`#loop`*Texto* de *recuento*         |Ejecuta el texto un número de veces.|
|`#qp`[*Nombre* [ `=` *valor*]]   |Establece el valor de un parámetro de consulta (o simplemente lo muestra o muestra todos los valores). Se recortarán las comillas simples o dobles desde el principio y el final.|
|`#save`*Nombre de archivo*             |Los resultados de la siguiente consulta o comando se guardarán en el archivo CSV indicado.|
|`#script`*Nombre de archivo*           |Ejecuta el script indicado.|
|`#scriptml`*Nombre de archivo*         |Ejecuta el script de varias líneas indicado.|

## <a name="line-input-mode-and-block-input-mode"></a>Modo de entrada de línea y modo de entrada de bloque

De forma predeterminada, Kusto. CLI se está ejecutando en **modo de entrada de línea**: cada carácter de nueva línea se interpreta como un delimitador entre las consultas y los comandos, y la línea se envía inmediatamente para su ejecución.

En este modo, es posible dividir una consulta o un comando largos en varias líneas. La apariencia del `&` carácter como el último carácter de una línea (antes de la nueva línea) hace que Kusto. CLI siga leyendo la línea siguiente. La apariencia del `&&` carácter como el último carácter de una línea (antes de la nueva línea) hace que Kusto. CLI omita la nueva línea y continúe leyendo la línea siguiente.

Como alternativa, Kusto. CLI también admite la ejecución en **modo de entrada en bloque**: mediante el uso del modificador de línea de comandos o mediante `-lineMode:false` el comando `#blockmode` , uno puede indicar a Kusto. CLI que asuma que cada línea es una continuación de la línea anterior, de modo que las consultas y los comandos se delimiten con una línea de entrada vacía.

## <a name="comments"></a>Comentarios

Kusto. CLI interpreta una `//` cadena que comienza la nueva línea como una línea de comentario. Omite el resto de la línea y continúa leyendo la línea siguiente.

## <a name="tool-only-options"></a>Opciones de solo herramientas

Comandos                        | Efecto                                                                            | Seleccionadas
--------------------------------|-----------------------------------------------------------------------------------|-----------
#timeon|#timeoff                | habilitar o deshabilitar la opción ' Timing ': mostrar las solicitudes de tiempo tomadas                    | VERDADERO
#tableon|#tableoff              | habilitar o deshabilitar la opción ' tableView ': dar formato a los conjuntos de resultados como tablas                  | VERDADERO
#marson|#marsoff                | habilitar o deshabilitar la opción ' marsView ': mostrar los conjuntos de resultados del segundo al último             | FALSO
#resultson|#resultsoff          | habilitar o deshabilitar la opción ' outputResultsSet ': mostrar los conjuntos de resultados                 | VERDADERO
#prettyon|#prettyoff            | habilitar o deshabilitar la opción ' prettyErrors ': quitar Goo innecesarios de errores          | VERDADERO
#markdownon|#markdownoff        | habilitar o deshabilitar la opción ' markdownView ': dar formato a las tablas como MarkDown                   | FALSO
#progressiveon|#progressiveoff  | habilitar o deshabilitar la opción ' progressiveView ': pedir y mostrar resultados progresivos  | FALSO
#linemode|#blockmode            | habilitar o deshabilitar la opción ' lineMode ': modo de entrada de una sola línea                          | VERDADERO

Comandos                              | Efecto                                                                                                         | Default
--------------------------------------|----------------------------------------------------------------------------------------------------------------|-----------
#cridon|#cridoff                      | (habilitar|deshabilitar la opción ' CRID ': mostrar el ClientRequestId antes de enviar la solicitud)                         | FALSO
#csvheaderson|#csvheadersoff          | (habilitar|deshabilitar la opción ' csvHeaders ': incluir encabezados en la salida CSV)                                            | VERDADERO
#focuson|#focusoff                    | (habilitar|deshabilitar la opción ' Focus ': quitar todas las desenfadados adicionales y centrarse en las cosas adecuadas)                       | FALSO
#linemode|#blockmode                  | (habilitar|deshabilitar la opción ' lineMode ': modo de entrada de una sola línea)                                                     | VERDADERO
#markdownon|#markdownoff              | (habilitar|deshabilitar la opción ' markdownView ': dar formato a las tablas como MarkDown)                                              | FALSO
#marson|#marsoff                      | (habilitar|deshabilitar la opción ' marsView ': mostrar los conjuntos de resultados del segundo al último)                                        | FALSO
#prettyon|#prettyoff                  | (habilitar|deshabilitar la opción ' prettyErrors ': quitar Goo innecesarios de errores)                                     | VERDADERO
#querystreamingon|#querystreamingoff  | (habilitar|deshabilitar la opción ' queryStreaming ': usar el punto de conexión queryStreaming (solo equipo Kusto))                    | FALSO
#resultson|#resultsoff                | (habilitar|deshabilitar la opción ' outputResultsSet ': mostrar los conjuntos de resultados)                                            | VERDADERO
#tableon|#tableoff                    | (habilitar|deshabilitar la opción ' tableView ': dar formato a los conjuntos de resultados como tablas)                                             | VERDADERO
#timeon|#timeoff                      | (habilitar|deshabilitar la opción ' Timing ': Mostrar la hora en que se tomaron las solicitudes)                                               | VERDADERO
#typeon|#typeoff                      | (habilitar|deshabilitar la opción ' typeView ': mostrar el tipo de cada columna en la vista de tabla (forzará el streaming = true))  | VERDADERO
#v2protocolon|#v2protocoloff          | (habilitar|deshabilitar la opción ' v2protocol ': usar el protocolo de consulta V2, no V1)                                        | VERDADERO

## <a name="using-kustocli-to-export-results-as-csv"></a>Uso de Kusto. CLI para exportar los resultados como CSV

Kusto. CLI admite un comando especial del lado cliente, `#save` , para exportar los resultados de la consulta **siguiente** a un archivo local en formato CSV. Por ejemplo, la siguiente invocación de Kusto. CLI exportará 10 registros fuera de la `StormEvents` tabla en el `help.kusto.windows.net` clúster ( `Samples` base de datos):

```
Kusto.Cli.exe @help/Samples -execute:"#save c:\temp\test.log" -execute:"StormEvents | take 10"
```

## <a name="using-kustocli-to-control-a-running-instance-of-kustoexplorer"></a>Uso de Kusto. CLI para controlar una instancia en ejecución de Kusto. Explorer

Es posible indicar a Kusto. CLI que se comunique con la instancia "principal" de Kusto. Explorer que se ejecuta en el equipo y envíe consultas de TI para que se ejecuten. Esto puede ser muy útil para los programas que desean ejecutar una serie de consultas de Kusto, pero no desea volver a iniciar el proceso de Kusto. Explorer de nuevo. En el ejemplo siguiente, se usa Kusto. CLI para ejecutar una consulta de nuevo en el clúster de ayuda:

```
#connect cluster('help').database('Samples')

#ke StormEvents | count
```

La sintaxis es muy sencilla: `#ke` , seguida de un espacio en blanco y de la consulta que se va a ejecutar.
A continuación, la consulta se envía a la instancia principal de Kusto. Explorer (si existe) con el clúster o la base de datos actual establecidos en Kusto. CLI.
