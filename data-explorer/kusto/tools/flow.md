---
title: Conector de Microsoft Flow Azure Kusto (versión preliminar) - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe Microsoft Flow Azure Kusto Connector (versión preliminar) en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 04/15/2019
ms.openlocfilehash: c4e732afb9e6165f803b9dc7e801b4e2be4c2588
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81504104"
---
# <a name="microsoft-flow-azure-kusto-connector-preview"></a>Conector de Microsoft Flow Azure Kusto (versión preliminar)

El conector de Microsoft Flow Azure Kusto permite a los usuarios ejecutar consultas y comandos de Kusto automáticamente como parte de una tarea programada o desencadenada, mediante [Microsoft Flow](https://flow.microsoft.com/).

Los escenarios de uso comunes incluyen:

* Enviar informes diarios que contienen tablas y gráficos
* Establecer notificaciones basadas en los resultados de la consulta
* Programar comandos de control en clústeres
* Exportar e importar datos entre Azure Data Explorer y otras bases de datos 

## <a name="limitations"></a>Limitaciones

1. Los resultados devueltos al cliente están limitados a 500.000 registros. La memoria total de esos registros no puede superar los 64 MB y los 7 minutos de tiempo de ejecución.
2. Actualmente, el conector no admite los operadores de [bifurcación](../query/forkoperator.md) y [faceta.](../query/facetoperator.md)
3. Flow funciona mejor en Internet Explorer y Chrome.

##  <a name="login"></a>Inicio de sesión 

1. Inicie sesión en [Microsoft Flow](https://flow.microsoft.com/).

1. Al conectarse al conector de Azure Kusto por primera vez, se le pedirá que inicie sesión.

1. Haga clic en el botón **Iniciar sesión** e introduzca sus credenciales para empezar a usar Azure Kusto Flow.

![texto alternativo](./Images/KustoTools-Flow/flow-signin.png "flujo-señalin")

## <a name="authentication"></a>Authentication

La autenticación en el conector de Azure Kusto Flow se puede realizar con credenciales de usuario o una aplicación de AAD.



### <a name="aad-application-authentication"></a>Autenticación de aplicaciones de AAD

Puede autenticarse en Azure Kusto Flow con una aplicación de AAD mediante los pasos siguientes:

> Nota: Asegúrese de que la aplicación es una aplicación de [AAD](../management/access-control/how-to-provision-aad-app.md) y está autorizada para ejecutar consultas en el clúster.

1. Haga clic en los tres puntos en la parte superior derecha del conector de Azure Kusto: ![texto alternativo](./Images/KustoTools-Flow/flow-addconnection.png "flujo-addconnection")

1. Seleccione "Agregar nueva conexión" y luego haga clic en "Conectar con la entidad de servicio".
![texto alternativo](./Images/KustoTools-Flow/flow-signin.png "flujo-señalin")

1. Rellene el identificador de la aplicación, la clave de aplicación y el identificador de inquilino.

Por ejemplo, el identificador de inquilino de Microsoft es: 72f988bf-86f1-41af-91ab-2d7cd011db47.
El valor Nombre de conexión es una cadena de su elección destinada a reconocer la nueva conexión agregada.
![texto alternativo](./Images/KustoTools-Flow/flow-appauth.png "flow-appauth")

Una vez completada la autenticación, podrá ver que el flujo está utilizando la nueva conexión agregada.
![texto alternativo](./Images/KustoTools-Flow/flow-appauthcomplete.png "flujo-appauthcompleto")

A partir de ahora, este flujo se ejecutará con las credenciales de la aplicación.

## <a name="find-the-azure-kusto-connector"></a>Busque el conector de Azure Kusto

Para usar el conector de Azure Kusto, primero debe agregar un desencadenador. Un desencadenador se puede definir en función de un período de tiempo periódico o como respuesta a una acción de flujo anterior.

1. [Cree un nuevo flujo.](https://flow.microsoft.com/manage/flows/new)
2. Agregue 'Programación - Recurrencia' como primer paso.
3. Escriba 'Azure Kusto' en el cuadro de búsqueda del segundo paso.

Ahora debería poder ver 'Azure Kusto' como se ve en la imagen de abajo.

![texto alternativo](./Images/KustoTools-Flow/flow-actions.png "flujo-acciones")

## <a name="azure-kusto-flow-actions"></a>Acciones de azure Kusto Flow

Al buscar el conector de Azure Kusto en Flow, verá 3 acciones posibles que puede agregar al flujo.

En la siguiente sección se describen las capacidades y los parámetros de cada acción de Azure Kusto Flow.

![texto alternativo](./Images/KustoTools-Flow/flow-actions.png "flujo-acciones")

### <a name="azure-kusto---run-query-and-visualize-results"></a>Azure Kusto: ejecute la consulta y visualice los resultados

> Nota: Si la consulta comienza con un punto (lo que significa que es un comando de [control),](../management/index.md)utilice el [comando Ejecutar control y visualice](./flow.md#azure-kusto---run-control-command-and-visualize-results) los resultados

Para visualizar el resultado de la consulta de Kusto como una tabla o como un gráfico, puede usar la acción 'Azure Kusto - Ejecutar consulta y visualizar resultados'. Los resultados de esta acción se pueden enviar más tarde por correo electrónico. Usted podría utilizar este flujo, por ejemplo, en caso de que usted quisiera conseguir los informes diarios ICM. 

![texto alternativo](./Images/KustoTools-Flow/flow-runquery.png "flow-runquery")

En este ejemplo, los resultados de la consulta se devuelven como una tabla HTML.

### <a name="azure-kusto---run-control-command-and-visualize-results"></a>Azure Kusto: ejecute el comando de control y visualice los resultados

De forma similar a la acción 'Azure Kusto - Ejecutar consulta y visualizar resultados', también puede ejecutar un comando de [control](../management/index.md) mediante la acción 'Azure Kusto - Ejecutar comando de control y visualizar resultados'.
Los resultados de esta acción se pueden enviar posteriormente por correo electrónico como una tabla o un gráfico.

![texto alternativo](./Images/KustoTools-Flow/flow-runcontrolcommand.png "flow-runcontrolcommand")

En este ejemplo, los resultados del comando de control se representan como un gráfico circular.

### <a name="azure-kusto---run-query-and-list-results"></a>Azure Kusto: ejecute los resultados de la consulta y la lista

> Nota: En caso de que su consulta comience con un punto (lo que significa que es un comando de [control),](../management/index.md)utilice el [comando Ejecutar control y visualice los resultados](./flow.md#azure-kusto---run-control-command-and-visualize-results)

Esta acción envía una consulta al clúster de Kusto. Las acciones que se agregan después recubren en iteración cada línea de los resultados de la consulta.

En el ejemplo siguiente se desencadena una consulta cada minuto y se envía un correo electrónico en función de los resultados de la consulta. La consulta comprueba el número de líneas de la base de datos y, a continuación, envía un correo electrónico solo si el número de líneas es mayor que 0. 

![texto alternativo](./Images/KustoTools-Flow/flow-runquerylistresults.png "flow-runquerylistresults")

Tenga en cuenta que en caso de que la columna tenga varias líneas, el conector siguiente se ejecutaría para cada línea de la columna.

## <a name="email-kusto-query-results"></a>Resultados de la consulta de Email Kusto

Para enviar informes de correo electrónico, siga estos pasos: 

![texto alternativo](./Images/KustoTools-Flow/flow-sendemail.png "flow-sendemail")

1. Haga clic en '+ Nuevo paso' y, a continuación, en 'Agregar una acción'.
2. En el cuadro de búsqueda, escriba 'Office 365 Outlook - Enviar un correo electrónico'.
3. Establezca la 'Para' en su dirección de correo electrónico, la "Asunto" en algún texto, y agregue "Cuerpo" del contenido dinámico al campo "Cuerpo".
4. Haga clic en 'Opciones avanzadas' agregue 'Nombre de archivo adjunto' al campo 'Nombre de los archivos adjuntos', 'Contenido adjunto' al campo 'Contenido de archivos adjuntos' y asegúrese de que 'Es HTML' esté establecido en 'Sí'.
5. En la barra superior, establezca el 'Nombre de flujo' para este flujo.
6. Haga clic en 'Crear flujo' y ya está!

## <a name="how-to-make-sure-flow-succeeded"></a>¿Cómo asegurarse de que el flujo se ha realizado correctamente?

Vaya a [La página](https://flow.microsoft.com/)de inicio de Microsoft Flow , haga clic en Mis [flujos](https://flow.microsoft.com/manage/flows) y, a continuación, haga clic en el botón i.

![texto alternativo](./Images/KustoTools-Flow/flow-myflows.png "flujo-misflujos")

Todas las ejecuciones de flujo se enumeran con el estado, la hora de inicio y la duración.

![texto alternativo](./Images/KustoTools-Flow/flow-runs.png "flujo-corridas")

Al abrir la última ejecución del flujo, si todos los pasos del flujo se marcan con una V verde, el flujo finalizó correctamente.
De lo contrario, expanda el paso marcado con un signo de exclamación rojo para ver los detalles del error.

![texto alternativo](./Images/KustoTools-Flow/flow-failed.png "flujo fallido")

Algunos errores se pueden resolver fácilmente por su cuenta, por ejemplo, errores de sintaxis de consulta:

![texto alternativo](./Images/KustoTools-Flow/flow-syntaxerror.png "error de sintaxis de flujo")



## <a name="having-a-timeout-exception"></a>¿Tiene una excepción de tiempo de espera?

El flujo puede producir un error y devolver la excepción "RequestTimeout" si se ejecuta más de 7 minutos.

Tenga en cuenta que 7 minutos es el tiempo máximo que se pueden ejecutar las consultas de flujo de tiempo antes de que haya una excepción de tiempo de espera.

[Haga clic aquí](#limitations) para ver las limitaciones de Microsoft Flow.

La misma consulta puede ejecutarse correctamente en el Explorador de Kusto, donde el tiempo no es limitado y se puede cambiar.

La excepción "RequestTimeout" se muestra en la imagen abajo:

![texto alternativo](./Images/KustoTools-Flow/flow-requesttimeout.png "flow-requesttimeout")

Para solucionar el problema, puede seguir estos pasos:
1. Obtenga más información sobre [las prácticas recomendadas de consultas](../query/best-practices.md).
2. Intente hacer que la consulta sea más eficaz para que se ejecute más rápido o separarla en fragmentos, cada fragmento puede ejecutarse en una parte diferente de la consulta.

## <a name="usage-examples"></a>Ejemplos de uso

Esta sección contiene varios ejemplos comunes de uso del conector de Azure Kusto Flow.

### <a name="example-1---azure-kusto-flow-and-sql"></a>Ejemplo 1 - Azure Kusto Flow y SQL

Puede usar el flujo de Azure Kusto para consultar los datos y, a continuación, acumularlos en una base de datos SQL. 
> Nota: La inserción SQL se realiza por separación por fila, utilice esto solo para cantidades bajas de datos de salida. 

![texto alternativo](./Images/KustoTools-Flow/flow-sqlexample.png "flow-sqlexample")

### <a name="example-2---push-data-to-power-bi-dataset"></a>Ejemplo 2 - Insertar datos en el conjunto de datos de Power BI

El conector de Azure Kusto Flow se puede usar junto con el conector de Power BI para insertar datos de consultas de Kusto en conjuntos de datos de streaming de Power BI.

Para empezar, cree una nueva acción "Kusto - Ejecutar consulta y resultados de lista".

![texto alternativo](./Images/KustoTools-Flow/flow-listresults.png "resultados de la lista de flujo")

Haga clic en "Nuevo paso", seleccione "Agregar una acción" y busque "Power BI". Haga clic en "Power BI - Add rows to a dataset". 

![texto alternativo](./Images/KustoTools-Flow/flow-powerbiconnector.png "flujo-powerbiconnector")

Rellene el espacio de trabajo, el conjunto de datos y la tabla en los que se insertarán los datos.
Agregue una carga útil que contenga el esquema del conjunto de datos y los resultados de la consulta Kusto relevantes desde la ventana de contenido dinámico. 

![texto alternativo](./Images/KustoTools-Flow/flow-powerbifields.png "flow-powerbifields")

Tenga en cuenta que Flow aplicará automáticamente la acción de Power BI para cada fila de la tabla de resultados de consulta Kusto. 

![texto alternativo](./Images/KustoTools-Flow/flow-powerbiforeach.png "flow-powerbiforeach")

### <a name="example-3---conditional-queries"></a>Ejemplo 3 - Consultas condicionales

Los resultados de las consultas de Kusto se pueden utilizar como entrada o condiciones para las siguientes acciones de Flow.

En el ejemplo siguiente, consultamos kusto para los incidentes ocurridos en el último día. Para cada incidente resuelto, publicamos un mensaje slack al respecto y creamos una notificación push.
Para cada incidente que sigue activo, consultamos Kusto para obtener más información sobre incidentes similares, enviar esa información como un correo electrónico y abrir una tarea de TFS relacionada.

Siga estas instrucciones para crear un flujo similar.

Cree una nueva acción "Kusto - Ejecutar resultados de consulta y lista".
Haga clic en 'Nuevo paso' y seleccione 'Añadir una condición' 

![texto alternativo](./Images/KustoTools-Flow/flow-listresults.png "resultados de la lista de flujo")

Elija en la ventana de contenido dinámico el parámetro que desea utilizar como condición para las siguientes acciones.
Seleccione el tipo de relación y valor para establecer una condición específica en el parámetro especificado.

![texto alternativo](./Images/KustoTools-Flow/flow-condition.png "condición de flujo")

Tenga en cuenta que Flow aplicará esta condición en cada fila de la tabla de resultados de la consulta.

Agregue acciones para cuando la condición sea true y false.

![texto alternativo](./Images/KustoTools-Flow/flow-conditionactions.png "flujo-condicionaciónacciones")

Puede utilizar los valores de resultado de la consulta Kusto como entrada para las siguientes acciones seleccionándolos en la ventana de contenido dinámico.
A continuación agregamos la acción "Slack - Post Message" y la acción "Visual Studio - Create a new work item" que contiene los datos de la consulta Kusto.

![texto alternativo](./Images/KustoTools-Flow/flow-slack.png "flujo flojo")

![texto alternativo](./Images/KustoTools-Flow/flow-visualstudio.png "flujo-visualstudio")

En este ejemplo, si un incidente sigue activo, consultamos Kusto de nuevo para obtener información sobre cómo se resolvieron los incidentes del mismo origen en el pasado.

![texto alternativo](./Images/KustoTools-Flow/flow-conditionquery.png "flow-conditionquery")

Visualizamos esta información como un gráfico circular y la enceramos por correo electrónico a nuestro equipo.

![texto alternativo](./Images/KustoTools-Flow/flow-conditionemail.png "flujo-condicióncorreo electrónico")

### <a name="example-4---email-multiple-azure-kusto-flow-charts"></a>Ejemplo 4 - Envíe por correo electrónico varios gráficos de Azure Kusto Flow

Cree un nuevo flujo con el desencadenador "Recurrence" y defina el intervalo del flujo y la frecuencia. 

Agregue un nuevo paso, con una o más acciones de 'Kusto - Ejecutar consulta y visualizar resultados'. 

![texto alternativo](./Images/KustoTools-Flow/flow-severalqueries.png "flujo-variasconsultas")

Para cada 'Kusto - Ejecutar consulta y visualizar resultado' defina los siguientes campos:

Nombre del clúster, nombre de la base de datos, consulta y tipo de gráfico (tabla Html/ gráfico circular/gráfico de tiempo/ gráfico de barras/ escriba valor personalizado).

![texto alternativo](./Images/KustoTools-Flow/flow-visualizeresultsmultipleattachments.png "flujo-visualizeresultsmultipleattachments")

Después de terminar con las acciones "Kusto - Ejecutar consulta y visualizar el resultado", agregue la acción "Enviar un correo electrónico". 

Asegúrese de insertar en el campo "Cuerpo" el cuerpo requerido, con el fin de adjuntar el resultado de visualización de la consulta al cuerpo del correo electrónico.
Además, para añadir un archivo adjunto al correo electrónico, agregue 'Nombre del archivo adjunto' y 'Contenido de archivo adjunto'.

Asegúrese de seleccionar "Sí" en el campo "es HTML".

![texto alternativo](./Images/KustoTools-Flow/flow-emailmultipleattachments.png "flujo-emailmultipleattachments")

Resultados:

![texto alternativo](./Images/KustoTools-Flow/flow-resultsmultipleattachments.png "flujo-resultadosmultipleadjuntos")

![texto alternativo](./Images/KustoTools-Flow/flow-resultsmultipleattachments2.png "flujo-resultadosmultipleattachments2")

### <a name="example-5---send-a-different-email-to-different-contacts"></a>Ejemplo 5 - Enviar un correo electrónico diferente a diferentes contactos

Puede aprovechar Azure Kusto Flow para enviar diferentes correos electrónicos personalizados a diferentes contactos. Las direcciones de correo electrónico, así como el contenido del correo electrónico son el resultado de una consulta de Kusto.

Vea el ejemplo siguiente:

![texto alternativo](./Images/KustoTools-Flow/flow-dynamicemailkusto.png "flujo dinámicoemailkusto")

![texto alternativo](./Images/KustoTools-Flow/flow-dynamicemail.png "flujo-dynamicemail")

### <a name="example-6---create-custom-html-table"></a>Ejemplo 6 - Crear tabla HTML personalizada

Puede aprovechar Azure Kusto Flow para crear y usar elementos HTML personalizados, como una tabla HTML personalizada.

En el ejemplo siguiente se muestra cómo crear una tabla HTML personalizada. La tabla HTML tendrá sus filas coloreadas por nivel de registro (el mismo que en el Explorador de Kusto).

Siga estas instrucciones para crear un flujo similar:

Cree una nueva acción "Kusto - Ejecutar resultados de consulta y lista".

![texto alternativo](./Images/KustoTools-Flow/flow-listresultforhtmltable.png "flow-listresultforhtmltable")

Recorro los resultados de la consulta y cree el cuerpo de la tabla HTML. En primer lugar, cree una variable que contendrá la cadena HTML - haga clic en 'Nuevo paso', seleccione 'Agregar una acción' y busque 'Variables'. Haga clic en 'Variables - Inicializar variable'. Inicialice una variable de cadena de la siguiente manera:

![texto alternativo](./Images/KustoTools-Flow/flow-initializevariable.png "flow-initializevariable")

Bucle sobre los resultados - haga clic en 'Nuevo paso' y elija 'Añadir una acción'. Busque 'Variables'. Seleccione 'Variables - Anexar a la variable de cadena'. Elija el nombre de variable que inicializó antes y cree las filas de la tabla HTML con los resultados de la consulta. Al elegir los resultados de la consulta, 'Aplicar a cada uno' se agrega automáticamente.

En el ejemplo siguiente, se utiliza la siguiente expresión if para definir el estilo de cada fila:

if(equals(items('Apply_to_each')?[' Level'], 'Warning'), 'Yellow', if(equals(items('Apply_to_each')?[' Nivel'], 'Error'), 'rojo', 'blanco'))

![texto alternativo](./Images/KustoTools-Flow/flow-createhtmltableloopcontent.png "flow-createhtmltableloopcontent")

Por último, cree el contenido HTML completo. Agregue una nueva acción fuera de 'Aplicar a cada uno'. En el ejemplo siguiente, la acción utilizada es 'Enviar un correo electrónico'. Defina la tabla HTML utilizando la variable de los pasos anteriores. Si va a enviar un correo electrónico, haga clic en "Mostrar opciones avanzadas" y elija "Sí" en "Es HTML"

![texto alternativo](./Images/KustoTools-Flow/flow-customhtmltablemail.png "flow-customhtmltablemail")

y aquí está el resultado:

![texto alternativo](./Images/KustoTools-Flow/flow-customhtmltableresult.png "flow-customhtmltableresult")






