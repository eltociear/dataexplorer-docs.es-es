---
title: Conector de Azure Data Explorer para Power Automate (versión preliminar)
description: Obtenga información sobre el uso del conector de Azure Data Explorer para Power Automate con el fin de crear flujos de tareas programadas o desencadenadas automáticamente.
author: orspod
ms.author: orspodek
ms.reviewer: dorcohen
ms.service: data-explorer
ms.topic: conceptual
ms.date: 03/25/2020
ms.openlocfilehash: 57f3ae587f449739f7e3aaaa0aa817530447097e
ms.sourcegitcommit: 98eabf249b3f2cc7423dade0f386417fb8e36ce7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2020
ms.locfileid: "82868729"
---
# <a name="azure-data-explorer-connector-to-power-automate-preview"></a>Conector de Azure Data Explorer para Power Automate (versión preliminar)

El conector de flujo de Azure Data Explorer permite a Azure Data Explorer usar las funcionalidades de flujo de [Microsoft Power Automate](https://flow.microsoft.com/). Puede ejecutar consultas y comandos de Kusto automáticamente como parte de una tarea programada o desencadenada.

Puede:

* Enviar informes diarios que contienen tablas y gráficos
* Establecer notificaciones basadas en los resultados de la consulta
* Programar comandos de control en clústeres
* Exportar e importar datos entre Azure Data Explorer y otras bases de datos 

Para más información, consulte [Ejemplos de uso del conector de Microsoft Flow (versión preliminar)](flow-usage.md).

##  <a name="sign-in"></a>Iniciar sesión 

1. Al conectarse por primera vez, se le pedirá que inicie sesión.

1. Seleccione **Iniciar sesión** y escriba las credenciales.

![Captura de pantalla con el aviso de inicio de sesión de Azure Data Explorer](./media/flow/flow-signin.png)

## <a name="authentication"></a>Authentication

Puede autenticarse con las credenciales de usuario o con una aplicación Azure Active Directory (Azure AD).

> [!Note]
> Asegúrese de que la aplicación sea una [aplicación de Azure AD](kusto/management/access-control/how-to-provision-aad-app.md) y que esté autorizada para ejecutar consultas en el clúster.

1. En **Ejecutar el comando de control y visualizar los resultados**, seleccione los tres puntos situados en la parte superior derecha del conector de flujo.

   ![Captura de pantalla de la opción Ejecutar el comando de control y visualizar los resultados](./media/flow/flow-addconnection.png)

1. Seleccione **Agregar nueva conexión** > **Conectar con la entidad de servicio**.

   ![Captura de pantalla del inicio de sesión de Azure Data Explorer con la opción Conectar con la entidad de servicio](./media/flow/flow-signin.png)

1. Escriba la información necesaria:
   - Nombre de la conexión: un nombre descriptivo y significativo para la nueva conexión
   - Identificador de cliente: el identificador de la aplicación
   - Secreto de cliente: la clave de aplicación
   - Inquilino: el identificador del directorio de Azure AD en el que se creó la aplicación.

   ![Captura de pantalla del cuadro de diálogo de autenticación de la aplicación de Azure Data Explorer](./media/flow/flow-appauth.png)

Una vez completada la autenticación, verá que el flujo usa la conexión recién agregada.

![Captura de pantalla de la autenticación de la aplicación completada](./media/flow/flow-appauthcomplete.png)

A partir de ahora, este flujo se ejecutará con estas credenciales de aplicación.

## <a name="find-the-azure-kusto-connector"></a>Búsqueda del conector de Azure Kusto

Para usar el conector de flujo, primero debe agregar un desencadenador. Se puede definir un desencadenador en función de un período de tiempo periódico o como respuesta a una acción de flujo anterior.

1. [Cree un flujo](https://flow.microsoft.com/manage/flows/new), o, en la página principal de Microsoft Power Automate, seleccione **Mis flujos** >  **+ Nuevo**.

    ![Captura de pantalla de la página principal de Microsoft Power Automate, con Mis flujos y Nuevo resaltado](./media/flow/flow-newflow.png)

1. Seleccione **Programado: desde cero**.

    ![Captura de pantalla del cuadro de diálogo Nuevo, con Programado: desde cero resaltado](./media/flow/flow-scheduled-from-blank.png)

1. En **Crear un flujo programado**, introduzca la información necesaria.
    
    ![Captura de pantalla de la página Crear un flujo programado con las opciones Nombre de flujo resaltadas](./media/flow/flow-build-scheduled-flow.png)

1. Seleccione **Crear** >  **+ Nuevo paso**.
1. En el cuadro de búsqueda, escriba *Kusto* y seleccione **Azure Data Explorer**.

    ![Captura de pantalla de las opciones de Elegir una acción, con el cuadro de búsqueda y Azure Data Explorer resaltado](./media/flow/flow-actions.png)

## <a name="flow-actions"></a>Acciones de flujo

Al abrir el conector de Azure Data Explorer, hay tres acciones posibles que puede agregar al flujo. En esta sección se describen las funcionalidades y los parámetros de cada acción.

![Captura de pantalla de las acciones del conector de Azure Data Explorer](./media/flow/flow-adx-actions.png)

### <a name="run-control-command-and-visualize-results"></a>Ejecutar el comando de control y visualizar los resultados

Use esta acción para ejecutar un [comando de control](kusto/management/index.md).

1. Especifique la dirección URL del clúster. Por ejemplo, `https://clusterName.eastus.kusto.windows.net`.
1. Escriba el nombre de la base de datos.
1. Especifique el comando de control:
   - Seleccione el contenido dinámico de las aplicaciones y los conectores que se usan en el flujo.
   - Agregue una expresión para acceder a los valores, convertirlos y compararlos.
1. Para enviar los resultados de esta acción por correo electrónico como una tabla o un gráfico, especifique el tipo de gráfico, que puede ser: Puede ser:
   - Una tabla HTML
   - Un gráfico circular
   - Un gráfico de tiempo
   - Un gráfico de barras

![Captura de pantalla de Ejecutar el comando de control y visualizar los resultados](./media/flow/flow-runcontrolcommand.png)

> [!IMPORTANT]
> En el campo **Nombre del clúster**, escriba la dirección URL del clúster.

### <a name="run-query-and-list-results"></a>Ejecutar la consulta y mostrar los resultados

> [!Note]
> Si la consulta comienza con un punto (lo que significa que se trata de un [comando de control](kusto/management/index.md)), utilice [Ejecutar el comando de control y visualizar los resultados](#run-control-command-and-visualize-results).

Esta acción envía una consulta al clúster de Kusto. Las acciones que se agregan después iteran por cada línea de los resultados de la consulta.

En el ejemplo siguiente se desencadena una consulta cada minuto y se envía un correo electrónico en función de los resultados de la consulta. La consulta comprueba el número de líneas en la base de datos y, a continuación, envía un correo electrónico solo si el número de líneas es mayor que 0. 

![Captura de pantalla de Ejecutar la consulta y mostrar los resultados](./media/flow/flow-runquerylistresults-2.png)

> [!Note]
> Si la columna tiene varias líneas, el conector se ejecutará para cada línea de la columna.

### <a name="run-query-and-visualize-results"></a>Ejecutar la consulta y visualizar los resultados
        
> [!Note]
> Si la consulta comienza con un punto (lo que significa que se trata de un [comando de control](kusto/management/index.md)), utilice [Ejecutar el comando de control y visualizar los resultados](#run-control-command-and-visualize-results).
        
Use esta acción para visualizar el resultado de una consulta Kusto como una tabla o un gráfico. Por ejemplo, use este flujo para recibir informes diarios por correo electrónico. 
    
En este ejemplo, los resultados de la consulta se devuelven como una tabla HTML.
            
![Captura de pantalla de Ejecutar la consulta y visualizar los resultados](./media/flow/flow-runquery.png)

> [!IMPORTANT]
> En el campo **Nombre del clúster**, escriba la dirección URL del clúster.

## <a name="email-kusto-query-results"></a>Resultados de la consulta Kusto por correo electrónico

Puede incluir un paso en cualquier flujo para enviar informes por correo electrónico a cualquier dirección de correo electrónico. 

1. Seleccione **+ Nuevo paso** para agregar un nuevo paso al flujo.
1. En el cuadro de búsqueda, escriba *Office 365* y seleccione **Office 365 Outlook**.
1. Seleccione **Enviar correo electrónico (V2)** .
1. Escriba la dirección de correo electrónico a la que desea enviar el informe de correo electrónico.
1. Escriba el asunto del correo electrónico.
1. Seleccione **Vista Código**.
1. Coloque el cursor en el campo **Cuerpo** y seleccione **Agregar contenido dinámico**.
1. Seleccione **BodyHtml**.
    ![Captura de pantalla del cuadro de diálogo Enviar un correo electrónico, con el campo Cuerpo y BodyHtml resaltados](./media/flow/flow-send-email.png)
1. Seleccione **Mostrar opciones avanzadas**.
1. En **Nombre de los datos adjuntos -1**, seleccione **Nombre de datos adjuntos**.
1. En el campo **Contenido de los datos adjuntos**, seleccione **Contenido de los datos adjuntos**.
1. Si es necesario, agregue más datos adjuntos. 
1. Si es necesario, establezca el nivel de importancia.
1. Seleccione **Guardar**.

![Captura de pantalla del cuadro de diálogo Enviar un correo electrónico, con las opciones Nombre de datos adjuntos, Contenido de datos adjuntos y Guardar resaltadas](./media/flow/flow-add-attachments.png)

## <a name="check-if-your-flow-succeeded"></a>Comprobar si el flujo se ejecutó correctamente

Para comprobar si el flujo se ejecutó correctamente, vea el historial de ejecución del flujo:
1. Vaya a la [página principal de Microsoft Power Automate](https://flow.microsoft.com/).
1. En el menú principal, seleccione [Mis flujos](https://flow.microsoft.com/manage/flows).
   
   ![Captura de pantalla del menú principal de Microsoft Power Automate, con Mis flujos resaltados](./media/flow/flow-myflows.png)

1. En la fila del flujo que desea investigar, seleccione el icono Más comandos y, a continuación, **Historial de ejecución**.

    ![Captura de pantalla de la pestaña Mis flujos, con Historial de ejecución resaltado](./media/flow//flow-runhistory.png)

    Todas las ejecuciones de flujo se muestran con información sobre la hora de inicio, la duración y el estado.
    ![Captura de pantalla de la página de resultados de Historial de ejecución](./media/flow/flow-runhistoryresults.png)

    Para obtener detalles completos sobre el flujo, en la página **[Mis flujos](https://flow.microsoft.com/manage/flows)** , seleccione el flujo que desea investigar.

    ![Captura de pantalla de la página de resultados completos de Historial de ejecución](./media/flow/flow-fulldetails.png) 

Para ver por qué se produjo un error en una ejecución, seleccione la hora de inicio de la ejecución. Se muestra el flujo y el paso del flujo en el que se produjo el error se indica mediante un signo de exclamación rojo. Expanda el paso con errores para ver sus detalles. El panel **Detalles** situado a la derecha contiene información sobre el error para que pueda solucionarlo.

![Captura de pantalla de la página de error de flujo](./media/flow/flow-error.png)

## <a name="timeout-exceptions"></a>Excepciones de tiempo de expiración

El flujo puede producir un error y devolver una excepción "RequestTimeout" si se ejecuta durante más de siete minutos.
    
![Captura de pantalla del error de excepción de tiempo de expiración de la solicitud de flujo](./media/flow/flow-requesttimeout.png)

Para corregir un problema de tiempo de expiración, haga que la consulta sea más eficaz para que se ejecute más rápido o sepárela en fragmentos. Cada fragmento se puede ejecutar en una parte diferente de la consulta. Para más información, consulte [Procedimientos recomendados sobre las consultas](kusto/query/best-practices.md).

La misma consulta puede ejecutarse correctamente en Azure Data Explorer donde la hora no está limitada y se puede cambiar.

## <a name="limitations"></a>Limitaciones

* Los resultados devueltos al cliente se limitan a 500 000 registros. La memoria total de esos registros no puede superar los 64 MB y un tiempo de ejecución de siete minutos.
* El conector no admite los operadores [fork](kusto/query/forkoperator.md) y [facet](kusto/query/facetoperator.md).
* El flujo funciona mejor en Microsoft Edge y Chrome.

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre el [conector de aplicaciones lógicas de Microsoft Kusto](kusto/tools/logicapps.md), que es otra forma de ejecutar consultas y comandos Kusto automáticamente como parte de una tarea programada o desencadenada.
