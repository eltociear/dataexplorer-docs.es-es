---
title: CLI de Kusto - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe la CLI de Kusto en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/24/2020
ms.openlocfilehash: 7443ad32b78a48f6b35be4f4b680ac6c728aedd2
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81524249"
---
# <a name="kusto-cli"></a>Kusto CLI

Kusto.Cli es una utilidad de línea de comandos que se puede utilizar para enviar solicitudes a Kusto y mostrar sus resultados. Kusto.Cli puede ejecutarse en uno de varios modos:

* *Modo REPL:* el usuario escribe en las consultas y comandos que se van a ejecutar en Kusto, y la herramienta muestra los resultados y, a continuación, espera la siguiente consulta/comando de usuario.
  ("REPL" significa "read/eval/print/loop".)

* *Modo de ejecución:* el usuario proporciona una o más consultas y comandos para ejecutar como argumentos de línea de comandos para la herramienta. Estos se ejecutan en secuencia automáticamente y sus resultados se envían a la consola. Opcionalmente, después de ejecutar todas las consultas y comandos de entrada, la herramienta pasa al modo REPL.

* *Modo de script*: Similar al modo de ejecución, pero con las consultas y comandos especificados a través de un archivo (llamado "script").

Kusto.Cli se proporciona principalmente para automatizar tareas en un servicio Kusto que normalmente habría requerido escribir código (por ejemplo, un programa de C o un script de PowerShell.)

## <a name="getting-the-tool"></a>Obtener la herramienta

Kusto.Cli forma parte del `Microsoft.Azure.Kusto.Tools`paquete NuGet, que se puede descargar [aquí.](https://www.nuget.org/packages/Microsoft.Azure.Kusto.Tools/)
Una vez descargado, la `tools` carpeta del paquete se puede extraer a la carpeta de destino; no se requiere ninguna instalación adicional (es decir, es instalable en xcopy).



## <a name="running-the-tool"></a>Ejecución de la herramienta

Kusto.Cli requiere al menos un argumento de línea de comandos para ejecutarse. Normalmente, ese argumento es la cadena de conexión al servicio Kusto al que la herramienta debe conectarse; consulte Cadenas de [conexión Kusto](../api/connection-strings/kusto.md). La ejecución de la herramienta sin argumentos de línea de comandos, o con un conjunto desconocido de argumentos, o con el `/help` modificador, da como resultado un mensaje de ayuda que se emite a la consola.

Por ejemplo, utilice el siguiente comando para ejecutar Kusto.Cli y conectarse al `help` `Samples` servicio Kusto y establecer el contexto de la base de datos en la base de datos:

```
Kusto.Cli.exe "https://help.kusto.windows.net/Samples;Fed=true"
```

> [!NOTE]
> Hemos utilizado comillas alrededor de la cadena de conexión para evitar que las`;`aplicaciones de shell, como PowerShell, interpreten el punto y coma ( ) y caracteres similares en la cadena de conexión.

## <a name="command-line-arguments"></a>Argumentos de la línea de comandos

`Kusto.Cli.exe`*ConnectionString* [*Interruptores*]

*Connectionstring*
* La cadena de [conexión de Kusto](../api/connection-strings/kusto.md) que contiene toda la información de conexión de Kusto.
  Su valor predeterminado es `net.tcp://localhost/NetDefaultDB`.

`-execute:`*QueryOrCommand*
* Si se especifica, ejecuta Kusto.Cli en modo de ejecución; se ejecuta la consulta o el comando especificado. Este modificador puede repetirse, en cuyo caso las consultas/comandos se ejecutan secuencialmente en orden de aparición.
  Este conmutador no se `-script` `-scriptml`puede utilizar junto con o .

`-keepRunning:`*EnableKeepRunning*
* Si se especifica `true` (como o `false`), habilita o deshabilita el `-script` `-execute` cambio al modo REPL después de que se hayan procesado todos o los valores.

`-script:`*ScriptFile*
* Si se especifica, ejecuta Kusto.Cli en modo de script; se carga el archivo de script especificado y las consultas o comandos que contiene se ejecutan secuencialmente.
  Las líneas nuevas se utilizan para delimitar consultas/comandos, excepto cuando las líneas terminan con `&` o `&&` combinaciones, como se explica a continuación.
  Este conmutador no se `-execute`puede utilizar junto con .

`-scriptml:`*ScriptFile*
* Si se especifica, ejecuta Kusto.Cli en modo de script; se carga el archivo de script especificado y las consultas o comandos que contiene se ejecutan secuencialmente.
  Todo el archivo de script se considera una sola consulta o comando.
  Este conmutador no se `-execute`puede utilizar junto con .

`-echo:`*EnableEchoMode*
* Si se especifica `true` (como o ), `false`habilita o deshabilita el modo de eco.
  Cuando se habilita el modo de eco, cada consulta o comando se repite en la salida.

`-transcript:`*TranscriptFile*  
* Si se especifica, escribe la salida del programa en *TranscriptFile*.

`-logToConsole:`*EnableLogToConsole*
* Si se especifica `true` (como o ), `false`habilita o deshabilita la escritura de la salida del programa en la consola.

`-lineMode:`*EnableLineMode*
* Si se especifica, cambia entre el modo de entrada `true`de línea (el valor `false`predeterminado o cuando se establece en ) y el modo de entrada de bloque (cuando se establece en ). Vea a continuación una explicación de estos dos modos, que determinan cómo se tratan las nuevas líneas.

**Ejemplo**

```
Kusto.Cli.exe "https://kustolab.kusto.windows.net/;Fed=true" -script:"c:\mycommands.txt"
```

Tenga en cuenta que no debe haber espacio entre los dos puntos y el valor del argumento.

## <a name="directives"></a>Directivas

Kusto.Cli admite una serie de directivas que se ejecuta en la herramienta en lugar de enviarse al servicio para su procesamiento:

|Directiva                      |Descripción|
|-------------------------------|-----------|
|`?`<br>`#h`<br>`#help`         |Recibe un breve mensaje de ayuda.|
|`q`<br>`#quit`<br>`#exit`      |Salga de la herramienta.|
|`#a`<br>`#abort`               |Salga de la herramienta de forma abortiva.|
|`#clip`                        |Los resultados de la siguiente consulta o comando se copiarán en el portapapeles.|
|`#cls`                         |Borre la pantalla de la consola.|
|`#connect`*[ConnectionString*]|Se conecta a un servicio Kusto diferente (si se omite *ConnectionString,* se mostrará el actual.)|
|`#crp`[*Name* Nombre`=` [ *Valor*]]   |Establece el valor de una propiedad de solicitud de cliente (o simplemente la muestra o muestra todos los valores).|
|`#crp` (`-list` | `-doc`) [*Prefijo*]|Enumera las propiedades de solicitud de cliente (por prefijo o todas).|
|`#dbcontext`[*DatabaseName*]  |Cambia la base de datos "contexto" utilizada por las consultas y comandos a *DatabaseName* (si se omite, se mostrará el contexto actual.)|
|`ke`*Texto*                    |Envía el texto especificado a un proceso Kusto.Explorer en ejecución.|
|`#loop`*Contar* *texto*         |Ejecuta el texto varias veces.|
|`#qp`[*Name* Nombre`=` [ *Valor*]]   |Establece el valor de un parámetro de consulta (o simplemente lo muestra o muestra todos los valores). Las comillas simples/dobles del principio/fin se recortarán.|
|`#save`*Nombre de archivo*             |Los resultados de la siguiente consulta o comando se guardarán en el archivo CSV indicado.|
|`#script`*Nombre de archivo*           |Ejecuta el script indicado.|
|`#scriptml`*Nombre de archivo*         |Ejecuta el script multilínea indicado.|

## <a name="line-input-mode-and-block-input-mode"></a>Modo de entrada de línea y modo de entrada de bloque

De forma predeterminada, Kusto.Cli se ejecuta en modo de entrada de **línea:** cada carácter de nueva línea se interpreta como un delimitador entre consultas/comandos y la línea se envía inmediatamente para su ejecución.

En este modo, es posible dividir una consulta o comando largo en varias líneas. La apariencia `&` del carácter como el último carácter de una línea (antes de la nueva línea) hace que Kusto.Cli continúe leyendo la siguiente línea. La apariencia `&&` del carácter como el último carácter de una línea (antes de la nueva línea) hace que Kusto.Cli ignore la nueva línea y continúe leyendo la siguiente línea.

Como alternativa, Kusto.Cli también admite la ejecución en modo de `-lineMode:false` entrada de `#blockmode` **bloque:** mediante el modificador de línea de comandos o mediante el comando , se puede indicar a Kusto.Cli que suponga que cada línea es una continuación de la línea anterior, de modo que las consultas y los comandos están delimitados únicamente por una línea de entrada vacía.

## <a name="comments"></a>Comentarios

Kusto.Cli interpreta `//` una cadena que comienza una nueva línea como una línea de comentario. Ignora el resto de la línea y continúa leyendo la siguiente línea.

## <a name="tool-only-options"></a>Opciones de solo herramienta

Comandos:                        | Efecto                                                                            | Actualmente
--------------------------------|-----------------------------------------------------------------------------------|-----------
#timeon|#timeoff                | opción de activación/desactivación 'timing': mostrar el tiempo que tardaron las solicitudes                    | TRUE
#tableon|#tableoff              | opción de activar/desactivar 'tableView': dar formato a los conjuntos de resultados como tablas                  | TRUE
#marson|#marsoff                | activar/desactivar la opción 'marsView': muestra los conjuntos de resultados de 2o a último             | FALSE
#resultson|#resultsoff          | opción de activar/desactivar 'outputResultsSet': mostrar los conjuntos de resultados                 | TRUE
#prettyon|#prettyoff            | activar/desactivar la opción 'prettyErrors': eliminar goo innecesario de los errores          | TRUE
#markdownon|#markdownoff        | opción de activar/desactivar 'markdownView': formatear tablas como MarkDown                   | FALSE
#progressiveon|#progressiveoff  | activar/desactivar la opción 'progressiveView': solicitar y mostrar resultados progresivos  | FALSE
#linemode|#blockmode            | opción de activación/desactivación 'lineMode': modo de entrada de una sola línea                          | TRUE

Comandos:                              | Efecto                                                                                                         | Default
--------------------------------------|----------------------------------------------------------------------------------------------------------------|-----------
#cridon|#cridoff                      | (habilitar|opción de deshabilitar 'crid': mostrar el ClientRequestId antes de enviar la solicitud)                         | FALSE
#csvheaderson|#csvheadersoff          | (habilitar|desactivar la opción 'csvHeaders': incluir encabezados en la salida CSV)                                            | TRUE
#focuson|#focusoff                    | (habilitar|desactivar la opción 'enfoque': eliminar toda la pelusa adicional y centrarse en las cosas correctas)                       | FALSE
#linemode|#blockmode                  | (habilitar|desactivar la opción 'lineMode': modo de entrada de una sola línea)                                                     | TRUE
#markdownon|#markdownoff              | (habilitar|desactivar la opción 'markdownView': formatear tablas como MarkDown)                                              | FALSE
#marson|#marsoff                      | (habilitar|desactivar la opción 'marsView': mostrar los conjuntos de resultados de 2o a último)                                        | FALSE
#prettyon|#prettyoff                  | (habilitar|desactivar la opción 'prettyErrors': eliminar goo innecesario de los errores)                                     | TRUE
#querystreamingon|#querystreamingoff  | (habilitar|deshabilitar la opción 'queryStreaming': use el punto de conexión queryStreaming (solo equipo de Kusto))                    | FALSE
#resultson|#resultsoff                | (habilitar|desactivar la opción 'outputResultsSet': mostrar los conjuntos de resultados)                                            | TRUE
#tableon|#tableoff                    | (habilitar|desactivar la opción 'tableView': dar formato a los conjuntos de resultados como tablas)                                             | TRUE
#timeon|#timeoff                      | (habilitar|desactivar la opción 'timing': mostrar las solicitudes de tiempo que se tardan)                                               | TRUE
#typeon|#typeoff                      | (habilitar|desactivar la opción 'typeView': mostrar el tipo de cada columna en la vista de tabla (forzará streaming-true))  | TRUE
#v2protocolon|#v2protocoloff          | (habilitar|desactivar la opción 'v2protocol': utilice el protocolo de consulta v2, no v1)                                        | TRUE

## <a name="using-kustocli-to-export-results-as-csv"></a>Uso de Kusto.Cli para exportar resultados como CSV

Kusto.Cli admite un comando especial `#save`del lado cliente, para exportar los resultados de la **siguiente** consulta a un archivo local en formato CSV. Por ejemplo, la siguiente invocación de Kusto.Cli exportará 10 registros de la `StormEvents` tabla en el `help.kusto.windows.net` clúster (`Samples` base de datos):

```
Kusto.Cli.exe @help/Samples -execute:"#save c:\temp\test.log" -execute:"StormEvents | take 10"
```

## <a name="using-kustocli-to-control-a-running-instance-of-kustoexplorer"></a>Uso de Kusto.Cli para controlar una instancia en ejecución de Kusto.Explorer

Es posible indicar a Kusto.Cli que se comunique con la instancia "primaria" de Kusto.Explorer que se ejecuta en el equipo y enviarle consultas para ejecutarlas. Esto puede ser muy útil para los programas que desean ejecutar una serie de consultas de Kusto, pero no quieren iniciar el proceso Kusto.Explorer una y otra vez. En el ejemplo siguiente, Kusto.Cli se utiliza para ejecutar una consulta de nuevo en el clúster de ayuda:

```
#connect cluster('help').database('Samples')

#ke StormEvents | count
```

La sintaxis es `#ke`muy simple: , seguido de espacios en blanco y la consulta que se va a ejecutar.
A continuación, la consulta se envía a la instancia principal de Kusto.Explorer (si existe) con el clúster/base de datos actual establecido en Kusto.Cli.