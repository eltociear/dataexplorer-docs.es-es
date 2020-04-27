---
title: Conector de flujos de Microsoft Azure Data Explorer (versión preliminar)
description: Obtenga información sobre el uso del conector de Microsoft Flow para crear flujos de tareas programadas o desencadenadas automáticamente.
author: orspod
ms.author: orspodek
ms.reviewer: dorcohen
ms.service: data-explorer
ms.topic: conceptual
ms.date: 03/25/2020
ms.openlocfilehash: 95b5e60c0993d3e364dc54ca7c06e06eaae876cb
ms.sourcegitcommit: e1e35431374f2e8b515bbe2a50cd916462741f49
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/23/2020
ms.locfileid: "82108293"
---
# <a name="microsoft-flow-connector-preview"></a>Conector de Microsoft Flow (versión preliminar)

El conector de flujos de Azure Data Explorer permite a Azure Data Explorer usar las [funcionalidades de flujo de Microsoft Power Automate](https://flow.microsoft.com/) para ejecutar consultas y comandos Kusto automáticamente como parte de una tarea programada o desencadenada.

Los escenarios de uso comunes incluyen:

* Enviar informes diarios que contienen tablas y gráficos
* Establecer notificaciones basadas en los resultados de la consulta
* Programar comandos de control en clústeres
* Exportar e importar datos entre Azure Data Explorer y otras bases de datos 

Para más información, vea [Ejemplos de uso del conector de Microsoft Flow](flow-usage.md).

##  <a name="log-in"></a>Registro 

1. Inicie sesión en [Microsoft Power Automate](https://flow.microsoft.com/).

1. Al conectarse por primera vez, se le pedirá que inicie sesión.

1. Seleccione **Iniciar sesión** y escriba las credenciales.

![Cuadro de diálogo de inicio de sesión](./media/flow/flow-signin.png)

## <a name="authentication"></a>Authentication

Puede autenticarse con las credenciales de usuario o una aplicación de AAD.

> [!Note]
> Asegúrese de que la aplicación es una [aplicación de AAD](kusto/management/access-control/how-to-provision-aad-app.md) y que está autorizada para ejecutar consultas en el clúster.

1. Seleccione los tres puntos situados en la parte superior derecha del conector de Microsoft Flow: ![Agregar una conexión](./media/flow/flow-addconnection.png)

1. Seleccione **Agregar nueva conexión** y, después, **Conectar con la entidad de servicio**.
![Cuadro de diálogo de inicio de sesión](./media/flow/flow-signin.png)

1. Escriba la información necesaria:
    * Nombre de la conexión: un nombre descriptivo y significativo para la nueva conexión
    * Identificador de cliente: el identificador de la aplicación
    * Secreto de cliente: La clave de aplicación.
    * Inquilino: el identificador del directorio de AAD en el que se creó la aplicación. Por ejemplo, el identificador de inquilino de Microsoft es: 72f988bf-86f1-41af-91ab-2d7cd011db47

![Autenticación de la aplicación](./media/flow/flow-appauth.png)

Una vez completada la autenticación, verá que el flujo usa la conexión recién agregada.

![Autenticación de la aplicación completada](./media/flow/flow-appauthcomplete.png)

A partir de ahora, este flujo se ejecutará con estas credenciales de aplicación.

## <a name="find-the-azure-kusto-connector"></a>Búsqueda del conector de Azure Kusto

Para usar el conector de Microsoft Flow, primero debe agregar un desencadenador. Un desencadenador se puede definir en función de un período de tiempo periódico o como respuesta a una acción de flujo anterior.

1. [Cree un flujo](https://flow.microsoft.com/manage/flows/new) o, en la página principal, seleccione la acción **Mis flujos** y, a continuación, seleccione **+ Nuevo**.

    ![Creación de un flujo](./media/flow/flow-newflow.png)

1. Agregue programado desde cero.

    ![Nuevo flujo programado](./media/flow/flow-scheduled-from-blank.png)

1. Escriba la información necesaria en la página Crear un flujo programado.
    ![Crear un flujo programado](./media/flow/flow-build-scheduled-flow.png)
1. Seleccione **Crear**.
1. Seleccione **+ New step**(+ Nuevo paso).
1. En el cuadro de búsqueda, escriba "Kusto".

    ![Seleccionar acciones de flujo](./media/flow/flow-actions.png)

1. Seleccione **Azure Data Explorer**.

## <a name="flow-actions"></a>Acciones de flujo

Al abrir el conector de Azure Data Explorer, hay tres acciones posibles que puede agregar al flujo.

En esta sección se describen las funcionalidades y los parámetros de cada acción de Microsoft Flow.

![Acciones de flujo de Azure Data Explorer](./media/flow/flow-adx-actions.png)

### <a name="run-control-command-and-visualize-results"></a>Ejecutar el comando de control y visualizar los resultados

Use el comando Ejecutar control y la acción Visualizar los resultados para ejecutar un [comando de control](kusto/management/index.md).

1. Especifique la dirección URL del clúster. Por ejemplo: `https://clusterName.eastus.kusto.windows.net`
1. Escriba el nombre de la base de datos.
1. Especifique el comando de control:
    * Seleccione el contenido dinámico de las aplicaciones y los conectores que se usan en el flujo.
    * Agregue una expresión para acceder a valores, convertirlos y compararlos.
1. Para enviar los resultados de esta acción por correo electrónico como una tabla o un gráfico, especifique el tipo de gráfico, que puede ser:
    * Una tabla HTML
    * Un gráfico circular
    * Un gráfico de tiempo
    * Un gráfico de barras

![Comando Ejecutar control de flujo](./media/flow/flow-runcontrolcommand.png)

> [!IMPORTANT]
> En el campo *Nombre del clúster*, escriba la dirección URL del clúster.

### <a name="run-query-and-list-results"></a>Ejecutar la consulta y mostrar los resultados

> [!Note]
> Si la consulta comienza con un punto (lo que significa que se trata de un [comando de control](kusto/management/index.md)), use [Ejecutar el comando de control y visualizar los resultados](#run-control-command-and-visualize-results).

Esta acción envía una consulta al clúster de Kusto. Las acciones que se agregan después iteran por cada línea de los resultados de la consulta.

En el ejemplo siguiente se desencadena una consulta cada minuto y se envía un correo electrónico en función de los resultados de la consulta. La consulta comprueba el número de líneas en la base de datos y, a continuación, envía un correo electrónico solo si el número de líneas es mayor que 0. 

![Ejecutar consulta y mostrar resultados](./media/flow/flow-runquerylistresults-2.png)

> [!Note]
> Si la columna tiene varias líneas, el conector se ejecutará para cada línea de la columna.

### <a name="run-query-and-visualize-results"></a>Ejecutar la consulta y visualizar los resultados
        
> [!Note]
> Si la consulta comienza con un punto (lo que significa que se trata de un [comando de control](kusto/management/index.md)), use [Ejecutar el comando de control y visualizar los resultados](#run-control-command-and-visualize-results).
        
Use la acción Ejecutar la consulta y visualizar los resultados para visualizar el resultado de la consulta Kusto como una tabla o un gráfico. Por ejemplo, use este flujo para recibir informes ICM diarios por correo electrónico. 
    
En este ejemplo, los resultados de la consulta se devuelven como una tabla HTML.
            
![Ejecutar la consulta y visualizar los resultados](./media/flow/flow-runquery.png)

> [!IMPORTANT]
> En el campo *Nombre del clúster*, escriba la dirección URL del clúster.

## <a name="email-kusto-query-results"></a>Resultados de la consulta Kusto por correo electrónico

Puede incluir un paso en cualquier flujo para enviar informes por correo electrónico a cualquier dirección de correo electrónico. 

1. Seleccione **+ Nuevo paso** para agregar un nuevo paso al flujo.
1. En el campo de búsqueda, escriba Office 365 y seleccione **Office 365 Outlook**.
1. Seleccione **Enviar correo electrónico (V2)** .
1. Escriba la dirección de correo electrónico a la que desea enviar el informe de correo electrónico.
1. Escriba el asunto del correo electrónico.
1. Seleccione **Vista Código**.
1. Coloque el cursor en el campo *Cuerpo* y seleccione **Agregar contenido dinámico**.
1. Seleccione **BodyHtml**.
    ![Enviar correo electrónico](./media/flow/flow-send-email.png)
1. Seleccione **Mostrar opciones avanzadas**.
1. En el campo *Nombre de los datos adjuntos -1*, seleccione **Nombre de archivo adjunto**.
1. En el campo *Contenido de los datos adjuntos*, seleccione **Contenido de los datos adjuntos**.
1. Si es necesario, agregue más datos adjuntos. 
1. Si es necesario, establezca el nivel de importancia.
1. Seleccione **Guardar**.

![Enviar correo electrónico](./media/flow/flow-add-attachments.png)

## <a name="check-if-your-flow-succeeded"></a>Comprobar si el flujo se ejecutó correctamente

Para comprobar si el flujo se ejecutó correctamente, vea el historial de ejecución del flujo:
1. Vaya a la [página principal de Microsoft Flow](https://flow.microsoft.com/).
1. En el menú principal, seleccione [Mis flujos](https://flow.microsoft.com/manage/flows).
    ![Página Mis flujos](./media/flow/flow-myflows.png)
1. En la fila del flujo que desea investigar, seleccione el icono Más comandos y, a continuación, **Historial de ejecución**.

    ![Menú Historial de ejecución](./media/flow//flow-runhistory.png)

    Todas las ejecuciones de flujo se muestran con la hora de inicio, la duración y el estado.
    ![Página de resultados del Historial de ejecución](./media/flow/flow-runhistoryresults.png)

    Para obtener detalles completos sobre el flujo, en la página [Mis flujos](https://flow.microsoft.com/manage/flows), seleccione el flujo que desea investigar.

    ![Página de resultados completos del Historial de ejecución](./media/flow/flow-fulldetails.png) 

Para ver por qué se produjo un error en una ejecución, seleccione la hora de inicio de la ejecución. El flujo aparece y el paso del flujo en el que se produjo el error se indica mediante un signo de exclamación rojo. Expanda el paso con errores para ver sus detalles. El panel derecho contiene información sobre el error para que pueda solucionarlo.

![Error de flujo](./media/flow/flow-error.png)

## <a name="timeout-exceptions"></a>Excepciones de tiempo de expiración

El flujo puede producir un error y devolver una excepción "RequestTimeout" si se ejecuta durante más de siete minutos.

Más información sobre las [limitaciones de Microsoft Flow](#limitations)
    
La misma consulta puede ejecutarse correctamente en Azure Data Explorer donde la hora no está limitada y se puede cambiar.
            
La excepción "RequestTimeout" se muestra en la imagen siguiente:
    
![Error de excepción de tiempo de expiración de la solicitud de flujo](./media/flow/flow-requesttimeout.png)
    
Para corregir un problema de tiempo de expiración, intente hacer que la consulta sea más eficaz para que se ejecute más rápido o sepárela en fragmentos. Cada fragmento se puede ejecutar en una parte diferente de la consulta.

Para más información, lea sobre los [Procedimientos recomendados de las consultas](kusto/query/best-practices.md).

## <a name="limitations"></a>Limitaciones

* Los resultados devueltos al cliente se limitan a 500 000 registros. La memoria total de esos registros no puede superar los 64 MB y el tiempo de ejecución de siete minutos.
* El conector no admite los operadores [bifurcación](kusto/query/forkoperator.md) y [faceta](kusto/query/facetoperator.md).
* Flow funciona mejor en Microsoft Edge y Chrome.

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre el [conector de aplicaciones lógicas de Microsoft Azure Explorer](kusto/tools/logicapps.md), que es otra forma de ejecutar consultas y comandos Kusto automáticamente como parte de una tarea programada o desencadenada.
