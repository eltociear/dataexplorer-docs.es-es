---
title: 'Herramienta Kusto. Explorer: Azure Explorador de datos'
description: En este artículo se describe la herramienta Kusto. Explorer en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/23/2020
ms.openlocfilehash: 60a414ff871d88de041e8b76671b73d98854fba0
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83374068"
---
# <a name="kustoexplorer-tool"></a>Herramienta Kusto. Explorer

Kusto. Explorer es una aplicación de escritorio enriquecida que permite explorar los datos mediante el lenguaje de consulta de Kusto.

## <a name="getting-the-tool"></a>Obtener la herramienta

* Instalar la [herramienta Kusto. Explorer](https://aka.ms/Kusto.Explorer)

* También puede obtener acceso al clúster de Kusto con el explorador en: `https://<your_cluster>.kusto.windows.net` . Reemplace <your_cluster> por el nombre del clúster de Explorador de datos de Azure.



## <a name="using-chrome-and-kustoexplorer"></a>Usar Chrome y Kusto. Explorer

Si usa Chrome como explorador predeterminado, asegúrese de instalar la extensión ClickOnce para Chrome:

[https://chrome.google.com/webstore/detail/clickonce-for-google-chro/kekahkplibinaibelipdcikofmedafmb/related?hl=en-US](https://chrome.google.com/webstore/detail/clickonce-for-google-chro/kekahkplibinaibelipdcikofmedafmb/related?hl=en-US)



## <a name="overview-of-the-user-experience"></a>Información general de la experiencia del usuario

La ventana Explorador de Kusto tiene varias partes de la interfaz de usuario:

1. [Panel de menús](#menu-panel)
2. [Panel conexiones](#connections-panel)
3. Panel de scripts
4. Panel Resultados

![Inicio de Kusto. Explorer](./Images/KustoTools-KustoExplorer/ke-start.png "iniciar")

### <a name="keyboard-shortcuts"></a>Accesos directos del teclado

Podría encontrar que el uso de métodos abreviados de teclado le permite realizar operaciones más rápido que con el mouse. Eche un vistazo a esta [lista de métodos abreviados de teclado de Kusto. Explorer](kusto-explorer-shortcuts.md).

### <a name="menu-panel"></a>Panel de menús

El panel de menús de Kusto. Explorer incluye las siguientes pestañas:

* [Inicio](#home-tab)
* [Archivo](#file-tab)
* [Conexiones](#connections-tab)
* [Ver](#view-tab)
* [Herramientas](#tools-tab)

* [Administración](#management-tab)
* [Ayuda](#help-tab)

### <a name="home-tab"></a>Pestaña Inicio

![Página principal de Kusto. Explorer](./Images/KustoTools-KustoExplorer/home-tab.png "Pestaña Inicio")

En la pestaña Inicio se muestran las funciones usadas más recientemente, divididas en secciones:

* [Consultar](#query-section)
* [Compartir](#share-section)
* [Visualizaciones](#visualizations-section)
* [Ver](#view-section)
* [Ayuda](#help-tab) 

#### <a name="query-section"></a>Sección de consulta

![Menú de consulta de Kusto. Explorer](./Images/KustoTools-KustoExplorer/home-query-menu.png "menú consulta")

|Menú|    Comportamiento|
|----|----------|
|Lista desplegable modo | <ul><li>Modo de consulta: cambia la ventana de consulta a un [modo de script](#query-mode). Los comandos se pueden cargar y guardar como scripts (valor predeterminado)</li> <li> Modo de búsqueda: un único modo de consulta en el que cada comando especificado se procesa inmediatamente y presenta un resultado en la ventana de resultados.</li> <li>Modo buscar + +: permite buscar un término mediante la sintaxis de búsqueda en una o más tablas. Más información sobre el uso del [modo de búsqueda + +](kusto-explorer.md#search-mode)</li></ul> |
|Nueva pestaña| Abre una nueva pestaña para consultar Kusto |

#### <a name="share-section"></a>Sección compartir

![Sección de recursos compartidos de Kusto. Explorer](./Images/KustoTools-KustoExplorer/home-share-menu.png "menú de recursos compartidos")

|Menú|    Comportamiento|
|----|----------|
|Datos al portapapeles|    Exporta la consulta y el conjunto de datos a un portapapeles. Si se presenta un gráfico, exporta el gráfico como mapa de bits| 
|Resultado al portapapeles| Exporta el conjunto de datos a un portapapeles. Si se presenta un gráfico, exporta el gráfico como mapa de bits| 
|Consulta al portapapeles| Exporta la consulta a un portapapeles|

#### <a name="visualizations-section"></a>Sección visualizaciones

![texto alternativo](./Images/KustoTools-KustoExplorer/home-visualizations-menu.png "menú-Visualizaciones")

|Menú         | Comportamiento|
|-------------|---------|
|Gráfico de áreas      | Muestra un gráfico de áreas en el que el eje X es la primera columna (debe ser numérica) y todas las columnas numéricas están asignadas a diferentes series (eje Y). |
|Gráfico de columnas | Muestra un gráfico de columnas en el que todas las columnas numéricas están asignadas a diferentes series (eje Y) y la columna de texto antes de Numeric es el eje X (se puede controlar en la interfaz de usuario).|
|Gráfico de barras    | Muestra un gráfico de barras en el que todas las columnas numéricas están asignadas a diferentes series (eje X) y la columna de texto antes de Numeric es el eje Y (se puede controlar en la interfaz de usuario).|
|gráfico de áreas apiladas      | Muestra un gráfico de áreas apiladas en el que el eje X es la primera columna (debe ser numérica) y todas las columnas numéricas están asignadas a diferentes series (eje Y) |
|Gráfico de escala de tiempo   | Muestra un gráfico de hora en el que el eje X es la primera columna (debe ser DateTime) y todas las columnas numéricas están asignadas a diferentes series (eje Y).|
|Gráfico de líneas   | Muestra un gráfico de líneas en el que el eje X es la primera columna (debe ser numérica) y todas las columnas numéricas están asignadas a diferentes series (eje Y).|
|Gráfico de anomalías|    Similar a gráfico, pero encuentra anomalías en los datos de serie temporal mediante el algoritmo de anomalías de machine learning. En la detección de anomalías, Kusto. Explorer usa la función [series_decompose_anomalies](../query/series-decompose-anomaliesfunction.md) . (*) 
|Gráfico circular    |    Muestra un gráfico circular en el que el eje de color es la primera columna y el eje Theta (debe ser una medida, convertido en porcentaje) es la segunda columna.|
|Gráfico de escalera |    Muestra un gráfico de escalera en el que el eje X es las dos últimas columnas (debe ser DateTime) y el eje Y es un compuesto de las demás columnas.|
|Gráfico de dispersión| Muestra un gráfico de puntos en el que el eje X es la primera columna (debe ser numérica) y todas las columnas numéricas están asignadas a diferentes series (eje Y).|
|Gráfico dinámico  | Mostrar una tabla dinámica y un gráfico dinámico que proporciona toda la flexibilidad de seleccionar datos, columnas, filas y varios tipos de gráficos.| 
|Tiempo dinámico   | Navegación interactiva en la línea de tiempo de eventos (dinamización en el eje de tiempo)|

(*) Gráfico de anomalías: el algoritmo espera los datos de los tiempos, que consta de dos columnas:
1. Tiempo en depósitos de intervalo fijo
2. Valor numérico para la detección de anomalías para generar esto en Kusto. Explorer, resume el campo Time y especifica la ubicación del depósito de tiempo.

#### <a name="view-section"></a>Sección de vista

![texto alternativo](./Images/KustoTools-KustoExplorer/home-view-menu.png "menú Ver")

|Menú           | Comportamiento|
|---------------|---------|
|Modo de vista completa | Maximiza el espacio de trabajo ocultando el menú de la cinta de opciones y el panel de conexión|
|Ocultar columnas vacías| Quita columnas vacías de la cuadrícula de datos|
|Contraer columnas singulares| Contrae las columnas con valores singulares|
|Explorar valores de columna| Muestra la distribución de valores de columna|
|Aumentar fuente  | Aumenta el tamaño de fuente de la pestaña de consulta y de la cuadrícula de datos de resultados.|  
|Reducir fuente  | Reduce el tamaño de fuente de la pestaña de consulta y de la cuadrícula de datos de resultados.|

(*) Configuración de la vista de datos: Kusto. Explorer realiza un seguimiento de los valores que se usan por cada conjunto de columnas único. Así, cuando se reordenan o se quitan columnas, la vista de datos se guarda y se reutilizará cada vez que se recuperen los datos con las mismas columnas. Para restablecer los valores predeterminados de la vista, en la pestaña **vista** , seleccione **restablecer vista**. 

### <a name="file-tab"></a>Pestaña Archivo

![Archivo Kusto. Explorer](./Images/KustoTools-KustoExplorer/file-tab.png "ficha archivo")

|Menú| Comportamiento|
|---------------|---------|
||---------*Script de consulta*---------|
|Nueva pestaña | Abre una nueva ventana de pestaña para consultar Kusto |
|Abrir archivo| Carga datos de un archivo *. kql en el panel de scripts activo.|
|Guardar en el archivo| Guarda el contenido del panel de scripts activo en el archivo *. kql|
|Cerrar pestaña| Cierra la ventana de pestaña actual|
||---------*Guardar datos*---------|
|A CSV       | Exporta los datos a un archivo CSV (valores separados por comas)| 
|A JSON      | Exporta los datos a un archivo con formato JSON|
|A Excel     | Exporta los datos a un archivo XLSX (Excel)|
|A texto      |    Exporta los datos a un archivo TXT (texto)| 
|A CSL script|    Exporta la consulta a un archivo de script| 
|A resultados   |    Exporta la consulta y los datos a un archivo de resultados (QRES)|
||---------*Cargar datos*---------|
|De resultados|    Carga la consulta y los datos de un archivo de resultados (QRES)| 
||---------*Portapapeles*---------|
|Datos al portapapeles|    Exporta la consulta y el conjunto de datos a un portapapeles. Si se presenta un gráfico, exporta el gráfico como un mapa de bits.| 
|Resultado al portapapeles| Exporta el conjunto de datos a un portapapeles. Si se presenta un gráfico, exporta el gráfico como un mapa de bits.| 
|Consulta al portapapeles| Exporta la consulta al portapapeles|
||---------*Resultados*---------|
|Borrar caché de resultados| Borra los resultados almacenados en caché de las consultas ejecutadas anteriormente.| 

### <a name="connections-tab"></a>Pestaña conexiones

![Pestaña conexiones de Kusto. Explorer](./Images/KustoTools-KustoExplorer/connections-tab.png "conexiones: ficha")

|Menú|Comportamiento|
|----|----------|
||---------*Familias*---------|
|Agregar grupo| Agrega un nuevo grupo de servidores de Kusto|
|Cambiar nombre de grupo| Cambia el nombre del grupo de servidores de Kusto existente|
|Quitar grupo| Quita el grupo de servidores de Kusto existente|
||---------*Clústeres*---------|
|Importación de conexiones| Importa conexiones desde un archivo que especifica conexiones|
|Exportar conexiones| Exporta conexiones a un archivo.|
|Agregar conexión| Agrega una nueva conexión de servidor Kusto| 
|Edición de la conexión| Abre un cuadro de diálogo de propiedades de conexión del servidor de Kusto editando|
|Quitar conexión| Quita la conexión existente con el servidor Kusto|
|Actualizar| Actualiza las propiedades de una conexión de servidor de Kusto|
||---------*Proveedores de identidades*---------|
|Inspección de la entidad de seguridad de conexión| Muestra los detalles actuales del usuario activo|
|Cierre de sesión de AAD| Cierra la sesión del usuario actual de la conexión a AAD.|
||---------*Ámbito de datos*---------|
|Ámbito de almacenamiento en caché|<ul><li>Solicitudes de ejecución de datos activas solo en la [caché de datos activos](../management/cachepolicy.md)</li><li>Todos los datos: ejecutar consultas en todos los datos disponibles (valor predeterminado)</li></ul> |
|Columna DateTime| Nombre de la columna que se puede usar para el filtro previo de hora|
|Filtro de tiempo| Valor de tiempo-filtro previo|

### <a name="view-tab"></a>Pestaña ver

![pestaña ver](./Images/KustoTools-KustoExplorer/view-tab.png "pestaña ver")

|Menú|Comportamiento|
|----|----------|
||---------*Aparición*---------|
|Modo de vista completa | Maximiza el espacio de trabajo ocultando el menú de la cinta de opciones y el panel de conexión|
|Aumentar fuente  | Aumenta el tamaño de fuente de la pestaña de consulta y de la cuadrícula de datos de resultados.|  
|Reducir fuente  | Reduce el tamaño de fuente de la pestaña de consulta y de la cuadrícula de datos de resultados.|
|Restablecer diseño|Restablece el diseño de las ventanas y los controles de acoplamiento de la herramienta|
||---------*Vista de datos*---------|
|Restablecer vista| Restablece la configuración de la vista de datos (*)|
|Explorar valores de columna|Muestra la distribución de valores de columna|
|Centrarse en las estadísticas de consultas|Cambia el foco a estadísticas de consulta en lugar de a los resultados de la consulta al finalizar la consulta.|
|Ocultar duplicados|Alterna la eliminación de las filas duplicadas de los resultados de la consulta|
|Ocultar columnas vacías|Alterna la eliminación de columnas vacías de los resultados de la consulta|
|Contraer columnas singulares|Alterna la contracción de las columnas con el valor singular|
||---------*Filtrado de datos*---------|
|Filtrar filas en la búsqueda|Alterna la opción para mostrar solo las filas coincidentes en la búsqueda de resultados de la consulta (Ctrl + F)|
||---------*Visualizaciones*---------|
|Visualizaciones|Consulte [visualizaciones](#visualizations-section), más arriba. |

(*) Configuración de la vista de datos: Kusto. Explorer realiza un seguimiento de los valores que se usan por cada conjunto de columnas único. Así, cuando se reordenan o se quitan columnas, la vista de datos se guarda y se reutilizará cada vez que se recuperen los datos con las mismas columnas. Para restablecer los valores predeterminados de la vista, en la pestaña **vista** , seleccione **restablecer vista**. 

### <a name="tools-tab"></a>Pestaña Herramientas

![pestaña Herramientas](./Images/KustoTools-KustoExplorer/tools-tab.png "herramientas-pestaña")

|Menú|Comportamiento|
|----|----------|
||---------*IntelliSense*---------|
|Habilitar IntelliSense| Habilita y deshabilita IntelliSense en el panel de scripts|
||---------*Examina*---------|
|Analizador de consultas| Inicia la herramienta analizador de consultas|
|Comprobador de consultas | Analiza la consulta actual y genera un conjunto de recomendaciones de mejora aplicables.|
|Calculadora| Inicia la calculadora|
||---------*Analytics*---------|
|Informes analíticos| Abre un panel con varios informes pregenerados para el análisis de datos|
||---------*Traducir*---------|
|Consulta a Power BI| Traduce una consulta a un formato adecuado para utilizar en Power BI|
||---------*Opciones*---------|
|Opciones de restablecimiento| Establece la configuración de la aplicación en los valores predeterminados|
|Opciones| Abre una herramienta para configurar las opciones de la aplicación. [Detalles](kusto-explorer-options.md)|



### <a name="management-tab"></a>Pestaña Administración

![pestaña Administración](./Images/KustoTools-KustoExplorer/management-tab.png "pestaña Administración")

|Menú             | Comportamiento|
|-----------------|---------|
||---------*Entidades de seguridad autorizadas*---------|
|Administración de entidades de seguridad autorizadas de clúster |Permite administrar las entidades de seguridad de un clúster para los usuarios autorizados| 
|Administrar entidades de seguridad autorizadas | Habilita la administración de entidades de seguridad de una base de datos para usuarios autorizados| 
|Administrar entidades de seguridad autorizadas de tabla | Habilita la administración de entidades de seguridad de una tabla para usuarios autorizados| 
|Administrar entidades de seguridad autorizadas de función | Habilita la administración de entidades de seguridad de una función para usuarios autorizados| 

### <a name="help-tab"></a>Pestaña ayuda

![pestaña ayuda](./Images/KustoTools-KustoExplorer/help-tab.png "pestaña de ayuda")

|Menú             | Comportamiento|
|-----------------|---------|
||---------*Documentación*---------|
|Ayuda             | Abre un vínculo a la documentación en línea de Kusto  | 
|Novedades       | Abre un documento que enumera todos los cambios de Kusto. Explorer.|
|Informar sobre un problema      |Abre un cuadro de diálogo con dos opciones: <ul><li>Informar de problemas relacionados con el servicio</li><li>Notificar problemas en la aplicación cliente</li></ul> | 
|Sugerir característica  | Abre un vínculo al foro de comentarios de Kusto | 
|Comprobar actualizaciones     | Comprueba si hay actualizaciones de la versión de Kusto. Explorer. | 

## <a name="connections-panel"></a>Panel conexiones

![texto alternativo](./Images/KustoTools-KustoExplorer/connectionsPanel.png "conexiones: panel") 

En el panel izquierdo de Kusto. Explorer se muestran todas las conexiones de clúster con las que está configurado el cliente. Para cada clúster, muestra las bases de datos, las tablas y los atributos (columnas) que almacenan. El panel conexiones le permite seleccionar elementos (que establece un contexto implícito para la búsqueda/consulta en el panel principal), o hacer doble clic en los elementos para copiar el nombre en el panel buscar/consulta.

Si el esquema real es grande (por ejemplo, una base de datos con cientos de tablas), es posible buscar el esquema presionando CTRL + F y escribiendo una subcadena (no distingue mayúsculas de minúsculas) del nombre de la entidad que está buscando.

Kusto. Explorer admite el control del panel de conexión desde la ventana de consulta.
Esto es muy útil para los scripts. Por ejemplo, al iniciar un archivo de script con un comando que indica a Kusto. Explorer que se conecte al clúster o la base de datos cuyos datos se están consultando mediante el script mediante la sintaxis siguiente. Como de costumbre, tendrá que ejecutar cada línea mediante `F5` o similar:

```kusto
#connect cluster('help').database('Samples')

StormEvents | count
```

### <a name="controlling-the-user-identity-used-for-connecting-to-kusto"></a>Controlar la identidad del usuario que se usa para conectarse a Kusto

Al agregar una nueva conexión, el modelo de seguridad predeterminado que se usa es la seguridad federada de AAD, en la que la autenticación se realiza a través del Azure Active Directory mediante la experiencia de usuario de AAD predeterminada.

En algunos casos, es posible que necesite un control más preciso sobre los parámetros de autenticación que está disponible en AAD. Si es así, es posible expandir el cuadro de edición "avanzadas: cadenas de conexión" y proporcionar un valor de [cadena de conexión Kusto](../api/connection-strings/kusto.md) válido.

Por ejemplo, los usuarios que tienen presencia en varios inquilinos de AAD a veces necesitan usar una "proyección" concreta de sus identidades para un inquilino de AAD específico. Para ello, proporcione una cadena de conexión como la siguiente (Reemplace las palabras en MAYÚSCULAs con valores específicos):

```
Data Source=https://CLUSTER_NAME.kusto.windows.net;Initial Catalog=DATABASE_NAME;AAD Federated Security=True;Authority Id=AAD_TENANT_OF_CLUSTER;User=USER_DOMAIN
```

Lo único es que `AAD_TENANT_OF_CLUSTER` es un nombre de dominio o un identificador de inquilino de AAD (un GUID) del inquilino de AAD en el que se hospeda el clúster (normalmente, el nombre de dominio de la organización que es el propietario del clúster, como `contoso.com` ) y USER_DOMAIN es la identidad del usuario que ha invitado en ese inquilino (por ejemplo, `joe@fabrikam.com` ). 

>[!Note]
> El nombre de dominio del usuario no es necesariamente el mismo que el del inquilino que hospeda el clúster.

## <a name="table-row-colors"></a>Colores de fila de tabla

Kusto. Explorer intenta "adivinar" la gravedad o el nivel de detalle de cada fila en el panel resultados y colorearla en consecuencia. Para ello, hace coincidir los valores distintos de cada columna con un conjunto de patrones conocidos ("WARNING", "error", etc.).

Para modificar la combinación de colores de salida o desactivar este comportamiento, en el menú **herramientas** , seleccione **Opciones**  >  **visor de resultados**  >  **detalle esquema de color**.

![texto alternativo](./Images/KustoTools-KustoExplorer/ke-color-scheme.png)

## <a name="search-mode"></a>Modo buscar + +

1. En la pestaña Inicio, en el menú desplegable consulta, seleccione "Buscar + +".
2. Seleccione **varias tablas** y, a continuación, en **elegir tablas**, defina las tablas que desea buscar.
3. En el cuadro de edición, escriba la frase de búsqueda y seleccione **ir** .
4. Un mapa térmico de la cuadrícula tabla/espacio de tiempo muestra el término que aparece y dónde aparecen.
5. Seleccione una celda de la cuadrícula y seleccione **Ver detalles** para mostrar las entradas correspondientes.

![texto alternativo](./Images/KustoTools-KustoExplorer/ke-search-beta.jpg "ke-Search-beta") 

## <a name="query-mode"></a>Modo de consulta

Kusto. Explorer tiene un modo de script eficaz que le permite escribir, editar y ejecutar consultas ad hoc. El modo de script incluye el resaltado de sintaxis e IntelliSense, por lo que puede empezar rápidamente al lenguaje Kusto CSL.

### <a name="basic-queries"></a>Consultas básicas

Si tiene registros de tabla, puede empezar a explorarlos escribiendo:

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
StormEvents | count 
```

Cuando el cursor se coloca en esta línea, su color es gris. Al presionar ' F5 ' se ejecuta la consulta. 

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

## <a name="importing-a-local-file-into-a-kusto-table"></a>Importar un archivo local en una tabla Kusto

Kusto. Explorer proporciona una manera cómoda de cargar archivos de la máquina en una tabla Kusto.

1. Asegúrese de que ha creado la tabla con un esquema que coincide con el archivo (por ejemplo, con el comando [. Create Table).](../management/tables.md)

1. Asegúrese de que la extensión de archivo es adecuada para el contenido del archivo. Por ejemplo:
    * Si el archivo contiene valores separados por comas, asegúrese de que el archivo tiene una extensión. csv.
    * Si el archivo contiene valores separados por tabulaciones, asegúrese de que el archivo tiene la extensión. TSV.

1. Haga clic con el botón derecho en la base de datos de destino en el [Panel conexiones](#connections-panel)y seleccione **Actualizar**para que aparezca la tabla.

    ![texto alternativo](./Images/KustoTools-KustoExplorer/right-click-refresh-schema.png "hacer clic con el botón derecho en-actualizar-esquema")

1. Haga clic con el botón secundario en la tabla de destino en el [Panel conexiones](#connections-panel)y seleccione **importar datos de archivos locales**.

    ![texto alternativo](./Images/KustoTools-KustoExplorer/right-click-import-local-file.png "Haga clic con el botón derecho en Import-local-File")

1. Seleccione los archivos que desea cargar y seleccione **abrir**.

    ![texto alternativo](./Images/KustoTools-KustoExplorer/import-local-file-choose-files.png "Import-local-File-Choose-files")

    La barra de progreso muestra el progreso y se muestra un cuadro de diálogo cuando se completa la operación

    ![texto alternativo](./Images/KustoTools-KustoExplorer/import-local-file-progress.png "Import-local-File-Progress")

    ![texto alternativo](./Images/KustoTools-KustoExplorer/import-local-file-complete.png "Import-local-File-complete")

1. Consulte los datos de la tabla (haga doble clic en la tabla en el [Panel conexiones](#connections-panel)).

## <a name="managing-authorized-principals"></a>Administración de entidades de seguridad autorizadas

Kusto. Explorer proporciona una manera cómoda de administrar las entidades de seguridad autorizadas de clúster, base de datos, tabla o función.

> [!Note]
> Solo los [administradores](../management/access-control/role-based-authorization.md) pueden agregar o quitar entidades de seguridad autorizadas en su propio ámbito.

1. Haga clic con el botón secundario en la entidad de destino en el [Panel conexiones](#connections-panel)y seleccione **administrar entidades de seguridad autorizadas**. (También puede hacerlo desde el menú de administración).

    ![texto alternativo](./Images/KustoTools-KustoExplorer/right-click-manage-authorized-principals.png "clic con el botón derecho-administrar-entidades de seguridad autorizadas")

    ![texto alternativo](./Images/KustoTools-KustoExplorer/manage-authorized-principals-window.png "ventana administrar-entidades de seguridad autorizadas")

1. Para agregar una nueva entidad de seguridad autorizada, seleccione **Agregar entidad**de seguridad, proporcione los detalles de la entidad de seguridad y confirme la acción.

    ![texto alternativo](./Images/KustoTools-KustoExplorer/add-authorized-principals-window.png "agregar una ventana de entidades de seguridad autorizadas")

    ![texto alternativo](./Images/KustoTools-KustoExplorer/confirm-add-authorized-principals.png "confirmar-agregar-autorizado-entidades de seguridad")

1. Para quitar una entidad de seguridad autorizada existente, seleccione **Drop principal** y confirme la acción.

    ![texto alternativo](./Images/KustoTools-KustoExplorer/confirm-drop-authorized-principals.png "confirmar y quitar la autorización-entidades de seguridad")

## <a name="sharing-queries-and-results-by-email"></a>Compartir consultas y resultados por correo electrónico

Kusto. Explorer proporciona una manera cómoda de compartir consultas y resultados de consultas por correo electrónico. Seleccione **exportar al portapapeles**y Kusto. Explorer copiará los siguientes elementos en el portapapeles:
1. La consulta
1. Los resultados de la consulta (tabla o gráfico)
1. Los detalles de conexión para el clúster y la base de datos de Kusto
1. Un vínculo que volverá a ejecutar la consulta automáticamente

Funcionamiento:

1. Ejecutar una consulta en Kusto. Explorer
1. Seleccione **exportar al portapapeles** (o presione `Ctrl+Shift+C` )

    ![texto alternativo](./Images/KustoTools-KustoExplorer/menu-export.png "menú-exportar")

1. Abra, por ejemplo, un nuevo mensaje de Outlook.

    ![texto alternativo](./Images/KustoTools-KustoExplorer/share-results.png "compartir: resultados")
    
1. Pegue el contenido del portapapeles en el mensaje de Outlook.

    ![texto alternativo](./Images/KustoTools-KustoExplorer/share-results-2.png "Share-Results-2")

## <a name="client-side-query-parametrization"></a>Parametrización de consulta del lado cliente

> [!WARNING]
> Hay dos tipos de técnicas de parametrización de consulta en Kusto:
> * [Language-Integrated Query parametrización](../query/queryparametersstatement.md) se implementa como parte del motor de consultas y está diseñado para ser utilizado por aplicaciones que consultan el servicio mediante programación.
>
> * La consulta del lado cliente parametrización, que se describe a continuación, es una característica de la aplicación Kusto. Explorer únicamente. Es equivalente a usar operaciones de reemplazo de cadenas en las consultas antes de enviarlas para que las ejecute el servicio. La sintaxis que se describe a continuación no forma parte del propio lenguaje de consulta y no se puede usar cuando se envían consultas al servicio por medio de un Kusto. Explorer.

Si piensa usar el mismo valor en varias consultas o en varias pestañas, será difícil cambiarlo. Sin embargo, Kusto. Explorer admite parámetros de consulta. Los parámetros se indican entre {} corchetes. Por ejemplo: `{parameter1}`

El editor de scripts resalta los parámetros de consulta:

![texto alternativo](./Images/KustoTools-KustoExplorer/parametrized-query-1.png "parametrizadas-consulta-1")

Puede definir y editar fácilmente los parámetros de consulta existentes:

![texto alternativo](./Images/KustoTools-KustoExplorer/parametrized-query-2.png "parametrizadas-consulta-2")

![texto alternativo](./Images/KustoTools-KustoExplorer/parametrized-query-3.png "parametrizadas-consulta-3")

El editor de scripts también tiene IntelliSense para los parámetros de consulta que ya están definidos:

![texto alternativo](./Images/KustoTools-KustoExplorer/parametrized-query-4.png "parametrizadas-consulta-4")

Puede haber varios conjuntos de parámetros (enumerados en el cuadro combinado de **conjuntos de parámetros** ).
Use **Agregar nuevo** y **eliminar actual** para manipular la lista de conjuntos de parámetros.

![texto alternativo](./Images/KustoTools-KustoExplorer/parametrized-query-5.png "parametrizadas-consulta-5")

Los parámetros de consulta se comparten entre las pestañas, por lo que se pueden reutilizar fácilmente.

## <a name="deep-linking-queries"></a>Consultas de vínculo profundo

### <a name="overview"></a>Información general
Puede crear un URI que, cuando se abre en un explorador, Kusto. Explorer se iniciará localmente y ejecutará una consulta específica en una base de datos Kusto especificada.

### <a name="limitations"></a>Limitaciones
Las consultas están limitadas a ~ 2000 caracteres debido a las limitaciones de Internet Explorer (la limitación es aproximada porque depende del clúster y de la longitud del nombre de la base de datos) https://support.microsoft.com/kb/208427 para reducir las posibilidades de que alcance el límite de caracteres, vea [obtener vínculos más cortos](#getting-shorter-links), más abajo.

El formato del URI es: https:// <ClusterCname> . kusto.Windows.net/ <DatabaseName> ? query =<QueryToExecute>

Por ejemplo:  https://help.kusto.windows.net/Samples?query=StormEvents+%7c+limit+10
 
Este URI abrirá Kusto. Explorer, se conectará al `help` clúster de Kusto y ejecutará la consulta especificada en la `Samples` base de datos. Si hay una instancia de Kusto. Explorer que ya se está ejecutando, la instancia en ejecución abrirá una nueva pestaña y ejecutará la consulta en ella.)

**Nota de seguridad**: por razones de seguridad, la vinculación profunda está deshabilitada para los comandos de control.



### <a name="creating-a-deep-link"></a>Crear un vínculo profundo
La forma más fácil de crear un vínculo profundo es crear la consulta en Kusto. Explorer y, a continuación, usar exportar al portapapeles para copiar la consulta (incluidos el vínculo profundo y los resultados) en el portapapeles. Después, puede compartirlo por correo electrónico.
        
Cuando se copia en un correo electrónico, el vínculo profundo se muestra en fuentes pequeñas; por ejemplo:

https://help.kusto.windows.net:443/Samples[[Haga clic para ejecutar la consulta](https://help.kusto.windows.net/Samples?web=0&query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSspVuDlqlEoLs3NTSzKrEpVSM4vzSvR0FRIqlRIyszTCC5JLCoJycxN1VEwT9EEKS1KzUtJLVIoAYolZwAlFQCB3oo%2bTAAAAA%3d%3d)] 

El primer vínculo abre Kusto. Explorer y establece el contexto del clúster y de la base de datos de forma adecuada.
El segundo vínculo (haga clic para ejecutar la consulta) es el vínculo profundo. Si se desplaza al vínculo a un mensaje de correo electrónico y presiona CTRL-K, puede ver la dirección URL real:

https://help.kusto.windows.net/Samples?web=0&query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSspVuDlqlEoLs3NTSzKrEpVSM4vzSvR0FRIqlRIyszTCC5JLCoJycxN1VEwT9EEKS1KzUtJLVIoAYolZwAlFQCB3oo%2bTAAAAA%3d%3d

### <a name="deep-links-and-parametrized-queries"></a>Vínculos profundos y consultas parametrizadas

Puede usar consultas parametrizadas con una vinculación profunda.

1. Cree una consulta que se forme como una consulta parametrizadas (por ejemplo, `KustoLogs | where Timestamp > ago({Period}) | count` ) 
2. Proporcione un parámetro para cada parámetro de consulta en el URI en este caso:

`https://mycluster.kusto.windows.net/MyDatabase?web=0&query=KustoLogs+%7c+where+Timestamp+>+ago({Period})+%7c+count&Period=1h`

### <a name="getting-shorter-links"></a>Obtener vínculos más cortos

Las consultas pueden llegar a ser largas. Para reducir la posibilidad de que la consulta supere la longitud máxima, use el `String Kusto.Data.Common.CslCommandGenerator.EncodeQueryAsBase64Url(string query)` método disponible en la biblioteca de cliente de Kusto. Este método genera una versión más compacta de la consulta. Kusto. Explorer también reconoce el formato más corto.

https://help.kusto.windows.net/Samples?web=0&query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSspVuDlqlEoLs3NTSzKrEpVSM4vzSvR0FRIqlRIyszTCC5JLCoJycxN1VEwT9EEKS1KzUtJLVIoAYolZwAlFQCB3oo%2bTAAAAA%3d%3d

La consulta se hace más compacta aplicando la siguiente transformación:

```csharp
 UrlEncode(Base64Encode(GZip(original query)))
```

## <a name="kustoexplorer-command-line-arguments"></a>Argumentos de la línea de comandos de Kusto. Explorer

Kusto. Explorer admite varios argumentos de línea de comandos en la siguiente sintaxis (el orden es importante):

[*LocalScriptFile*] [*QueryString*]

Donde:
* *LocalScriptFile* es el nombre de un archivo de script en el equipo local que debe tener la extensión `.kql` . Si existe este tipo de archivo, Kusto. Explorer carga automáticamente este archivo cuando se inicia.
* *QueryString* es una cadena con formato que usa el formato de cadena de consulta http. Este método proporciona propiedades adicionales, como se describe en la tabla siguiente.

Por ejemplo, para iniciar Kusto. Explorer con un archivo de script denominado `c:\temp\script.kql` y configurado para comunicarse con `help` el clúster, base de datos `Samples` , use el siguiente comando:

```
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

## <a name="kustoexplorer-connection-files"></a>Archivos de conexión de Kusto. Explorer

Kusto. Explorer mantiene la configuración de las conexiones en la `%LOCALAPPDATA%\Kusto.Explorer` carpeta.
Una lista de grupos de conexión se mantiene dentro `%LOCALAPPDATA%\Kusto.Explorer\UserConnectionGroups.xml` de y cada grupo de conexión se mantiene dentro de un archivo dedicado en `%LOCALAPPDATA%\Kusto.Explorer\Connections\` .

### <a name="format-of-connection-group-files"></a>Formato de archivos de grupo de conexión

La ubicación del archivo es `%LOCALAPPDATA%\Kusto.Explorer\UserConnectionGroups.xml` .  

Se trata de una serialización XML de una matriz de `ServerGroupDescription` objetos con las siguientes propiedades:

```
  <ServerGroupDescription>
    <Name>`Connection Group name`</Name>
    <Details>`Full path to XML file containing the list of connections`</Details>
  </ServerGroupDescription>
```

Ejemplo:

```
<?xml version="1.0"?>
<ArrayOfServerGroupDescription xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema">
  <ServerGroupDescription>
    <Name>Connections</Name>
    <Details>C:\Users\alexans\AppData\Local\Kusto.Explorer\UserConnections.xml</Details>
  </ServerGroupDescription>
</ArrayOfServerGroupDescription>  
```

### <a name="format-of-connection-list-files"></a>Formato de los archivos de lista de conexiones

La ubicación del archivo es: `%LOCALAPPDATA%\Kusto.Explorer\Connections\` .

Se trata de una serialización XML de una matriz de `ServerDescriptionBase` objetos con las siguientes propiedades:

```
   <ServerDescriptionBase xsi:type="ServerDescription">
    <Name>`Connection name`</Name>
    <Details>`Details as shown in UX, usually full URI`</Details>
    <ConnectionString>`Full connection string to the cluster`</ConnectionString>
  </ServerDescriptionBase>
```

Ejemplo:

```
<?xml version="1.0"?>
<ArrayOfServerDescriptionBase xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema">
  <ServerDescriptionBase xsi:type="ServerDescription">
    <Name>Help</Name>
    <Details>https://help.kusto.windows.net</Details>
    <ConnectionString>Data Source=https://help.kusto.windows.net:443;Federated Security=True</ConnectionString>
  </ServerDescriptionBase>
</ArrayOfServerDescriptionBase>
```

## <a name="resetting-kustoexplorer"></a>Restablecer Kusto. Explorer

Si es necesario, puede restablecer completamente Kusto. Explorer. Use el siguiente procedimiento para restablecer progresivamente Kusto. Explorer implementado en el equipo, hasta que se Quite completamente y se deba instalar desde cero.

1. En Windows, Abra **cambiar o quitar programas** (también conocidos como **programas y características**).
1. Seleccione todos los elementos cuyo nombre empiece por `Kusto.Explorer` .
1. Seleccione **Desinstalar**.

   Si se produce un error al desinstalar la aplicación (un problema conocido a veces con aplicaciones ClickOnce), consulte [este artículo de stack Overflow](https://stackoverflow.com/questions/10896223/how-do-i-completely-uninstall-a-clickonce-application-from-my-computer) , en el que se explica cómo hacerlo.

1. Elimine la carpeta `%LOCALAPPDATA%\Kusto.Explorer` . Esto quita todas las conexiones, el historial, etc.

1. Elimine la carpeta `%APPDATA%\Kusto` . Esto quita la caché de tokens de Kusto. Explorer. Tendrá que volver a autenticarse en todos los clústeres.

También es posible revertir a una versión específica de Kusto. Explorer:

1. Ejecute `appwiz.cpl`.
1. Seleccione **Kusto. Explorer** y seleccione **desinstalar o cambiar**.
3. Seleccione **restaurar la aplicación a su estado anterior**.

## <a name="troubleshooting"></a>Solución de problemas

### <a name="kustoexplorer-fails-to-start"></a>No se puede iniciar Kusto. Explorer

#### <a name="kustoexplorer-shows-error-dialog-during-or-after-start-up"></a>Kusto. Explorer muestra un cuadro de diálogo de error durante o después del inicio

**Síntoma:**

Al iniciarse, Kusto. Explorer muestra un `InvalidOperationException` error.

**Posible solución:**

Este error puede sugerir que el sistema operativo se ha dañado o que faltan algunos de los módulos esenciales.
Para comprobar los archivos del sistema que faltan o están dañados, siga los pasos que se describen aquí:   
[https://support.microsoft.com/help/929833/use-the-system-file-checker-tool-to-repair-missing-or-corrupted-system](https://support.microsoft.com/help/929833/use-the-system-file-checker-tool-to-repair-missing-or-corrupted-system)

### <a name="kustoexplorer-always-downloads-even-when-there-are-no-updates"></a>Kusto. Explorer siempre se descarga incluso cuando no hay ninguna actualización

**Síntoma:**

Cada vez que abra Kusto. Explorer, se le pedirá que instale una nueva versión. Kusto. Explorer descarga todo el paquete, sin actualizar realmente la versión ya instalada.

**Posible solución:**

Esto podría ser el resultado de daños en el almacén local de ClickOnce. Para borrar el almacén de ClickOnce local, ejecute el siguiente comando en un símbolo del sistema con privilegios elevados.
> [!Important]
> 1. Si hay otras instancias de aplicaciones ClickOnce o de `dfsvc.exe` , finalice antes de ejecutar este comando.
> 2. Las aplicaciones ClickOnce se reinstalarán automáticamente la próxima vez que se ejecuten, siempre y cuando tenga acceso a la ubicación de instalación original almacenada en el acceso directo de la aplicación. No se eliminarán los accesos directos de la aplicación.

```
rd /q /s %userprofile%\appdata\local\apps\2.0
```

Intente volver a instalar Kusto. Explorer desde uno de los [reflejos de instalación](#getting-the-tool).

#### <a name="clickonce-error-cannot-start-application"></a>Error de ClickOnce: no se puede iniciar la aplicación

**Síntomas**  

* El programa no se inicia y muestra un error que contiene:`External component has thrown an exception`
* El programa no se inicia y muestra un error que contiene:`Value does not fall within the expected range`
* El programa no se inicia y muestra un error que contiene:`The application binding data format is invalid.` 
* El programa no se inicia y muestra un error que contiene:`Exception from HRESULT: 0x800736B2`

Puede explorar los detalles del error haciendo clic `Details` en el siguiente cuadro de diálogo de error:

![texto alternativo](./Images/KustoTools-KustoExplorer/clickonce-err-1.jpg "ClickOnce-Err-1")

```
Following errors were detected during this operation.
    * System.ArgumentException
        - Value does not fall within the expected range.
        - Source: System.Deployment
        - Stack trace:
            at System.Deployment.Application.NativeMethods.CorLaunchApplication(UInt32 hostType, String applicationFullName, Int32 manifestPathsCount, String[] manifestPaths, Int32 activationDataCount, String[] activationData, PROCESS_INFORMATION processInformation)
            at System.Deployment.Application.ComponentStore.ActivateApplication(DefinitionAppId appId, String activationParameter, Boolean useActivationParameter)
            at System.Deployment.Application.SubscriptionStore.ActivateApplication(DefinitionAppId appId, String activationParameter, Boolean useActivationParameter)
            at System.Deployment.Application.ApplicationActivator.Activate(DefinitionAppId appId, AssemblyManifest appManifest, String activationParameter, Boolean useActivationParameter)
            at System.Deployment.Application.ApplicationActivator.ProcessOrFollowShortcut(String shortcutFile, String& errorPageUrl, TempFile& deployFile)
            at System.Deployment.Application.ApplicationActivator.PerformDeploymentActivation(Uri activationUri, Boolean isShortcut, String textualSubId, String deploymentProviderUrlFromExtension, BrowserSettings browserSettings, String& errorPageUrl)
            at System.Deployment.Application.ApplicationActivator.ActivateDeploymentWorker(Object state)
```

**Pasos de solución propuestos:**

1. Desinstale la aplicación Kusto. Explorer mediante `Programs and Features` ( `appwiz.cpl` ).

1. Intente ejecutar `CleanOnlineAppCache` y, a continuación, intente instalar de nuevo Kusto. Explorer. Desde un símbolo del sistema con privilegios elevados: 
    
    ```
    rundll32 %windir%\system32\dfshim.dll CleanOnlineAppCache
    ```

    Vuelva a instalar Kusto. Explorer desde uno de los [reflejos de instalación](#getting-the-tool).

1. Si sigue sin funcionar, elimine el almacén de ClickOnce local. Las aplicaciones ClickOnce se reinstalarán automáticamente la próxima vez que se ejecuten, siempre y cuando tenga acceso a la ubicación de instalación original almacenada en el acceso directo de la aplicación. No se eliminarán los accesos directos de la aplicación.

Desde un símbolo del sistema con privilegios elevados:

    ```
    rd /q /s %userprofile%\appdata\local\apps\2.0
    ```

    Install Kusto.Explorer again from one of the [installation mirrors](#getting-the-tool)

1. Si sigue sin funcionar, quite los archivos de implementación temporales y cambie el nombre de la carpeta local AppData de Kusto. Explorer.

    Desde un símbolo del sistema con privilegios elevados:

    ```
    rd /s/q %userprofile%\AppData\Local\Temp\Deployment
    ren %LOCALAPPDATA%\Kusto.Explorer Kusto.Explorer.bak
    ```

    Vuelva a instalar Kusto. Explorer desde uno de los [reflejos de instalación](#getting-the-tool)

1. Para restaurar las conexiones desde Kusto. Explorer. bak, desde un símbolo del sistema con privilegios elevados:

    ```
    copy %LOCALAPPDATA%\Kusto.Explorer.bak\User*.xml %LOCALAPPDATA%\Kusto.Explorer
    ```

1. Si sigue sin funcionar, habilite el registro detallado de ClickOnce mediante la creación de un valor de cadena de LogVerbosityLevel de 1 en:

`HKEY_CURRENT_USER\Software\Classes\Software\Microsoft\Windows\CurrentVersion\Deployment`, vuelva a realizar la reproducción y envíe el resultado detallado a KEBugReport@microsoft.com . 

#### <a name="clickonce-error-your-administrator-has-blocked-this-application-because-it-potentially-poses-a-security-risk-to-your-computer"></a>Error de ClickOnce: el administrador ha bloqueado esta aplicación porque puede suponer un riesgo de seguridad para el equipo

**Síntoma:**  
El programa no se puede instalar con ninguno de los siguientes errores:
* `Your administrator has blocked this application because it potentially poses a security risk to your computer`.
* `Your security settings do not allow this application to be installed on your computer.`

**Solución:**

1. Esto puede deberse a que otra aplicación invalida el comportamiento predeterminado del mensaje de confianza de ClickOnce. Puede ver los valores de configuración predeterminados, compararlos con los reales del equipo y restablecerlos según sea necesario, tal como se explica [en este artículo de procedimientos](https://docs.microsoft.com/visualstudio/deployment/how-to-configure-the-clickonce-trust-prompt-behavior).

#### <a name="cleanup-application-data"></a>Limpiar datos de la aplicación

A veces, cuando los pasos anteriores de solución de problemas no ayudaron a iniciar Kusto. Explorer, la limpieza de los datos almacenados localmente puede ayudar.

Los datos almacenados por la aplicación Kusto. Explorer se pueden encontrar aquí: `C:\Users\\[your alias]\AppData\Local\Kusto.Explorer` .

> [!NOTE]
> La limpieza de los datos dará lugar a la pérdida de pestañas abiertas (carpeta de recuperación), conexiones guardadas (carpeta conexiones) y configuración de la aplicación (carpeta UserSettings).