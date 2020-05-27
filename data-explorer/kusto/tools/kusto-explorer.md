---
title: Introducción a Kusto. Explorer
description: Obtenga información sobre las características de Kusto. Explorer y cómo puede ayudarle a explorar los datos
author: orspod
ms.author: orspodek
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: overview
ms.date: 05/19/2020
ms.openlocfilehash: 15c9ff61067f25f8a0f63ce4078b158277740db3
ms.sourcegitcommit: b4d6c615252e7c7d20fafd99c5501cb0e9e2085b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2020
ms.locfileid: "83863565"
---
# <a name="getting-started-with-kustoexplorer"></a>Introducción a Kusto. Explorer

Kusto. Explorer es una aplicación de escritorio enriquecida que permite explorar los datos mediante el lenguaje de consulta de Kusto en una interfaz de usuario fácil de usar. En esta introducción se explica cómo empezar a configurar Kusto. Explorer y se explica la interfaz de usuario que se va a usar.

Con Kusto. Explorer, puede:
* [Consultar los datos](kusto-explorer-using.md#query-mode).
* [Busque en los datos de](kusto-explorer-using.md#search-mode) las tablas.
* [Visualice los datos](#visualizations-section) en una amplia variedad de gráficos.
* [Compartir consultas y resultados](kusto-explorer-using.md#share-queries-and-results) por correo electrónico o mediante vínculos profundos.

## <a name="installing-kustoexplorer"></a>Instalación de Kusto. Explorer

* Instalar la [herramienta Kusto. Explorer](https://aka.ms/ke)

* En su lugar, acceda al clúster de Kusto con el explorador en: [https://<your_cluster>. kusto.Windows.net](https://your_cluster.kusto.windows.net). Reemplace <your_cluster> por el nombre del clúster de Explorador de datos de Azure.

### <a name="using-chrome-and-kustoexplorer"></a>Usar Chrome y Kusto. Explorer

Si usa Chrome como explorador predeterminado, asegúrese de instalar la extensión ClickOnce para Chrome:

[https://chrome.google.com/webstore/detail/clickonce-for-google-chro/kekahkplibinaibelipdcikofmedafmb/related?hl=en-US](https://chrome.google.com/webstore/detail/clickonce-for-google-chro/kekahkplibinaibelipdcikofmedafmb/related?hl=en-US)

## <a name="overview-of-the-user-interface"></a>Información general de la interfaz de usuario

La interfaz de usuario de Kusto. Explorer está diseñada con un diseño basado en pestañas y paneles, similar al de otros productos de Microsoft: 

1. Desplazarse por las pestañas del [Panel de menús](#menu-panel) para realizar diversas operaciones
2. Administrar las conexiones en el [Panel conexiones](#connections-panel)
3. Crear scripts para ejecutarlos en el panel de scripts
4. Ver los resultados de los scripts en el panel de resultados

:::image type="content" source="images/kusto-explorer/ke-start.png" alt-text="Inicio del explorador de Kusto":::

## <a name="menu-panel"></a>Panel de menús

El panel de menús de Kusto. Explorer incluye las siguientes pestañas:

* [Inicio](#home-tab)
* [Archivo](#file-tab)
* [Conexiones](#connections-tab)
* [Vista](#view-tab)
* [Herramientas](#tools-tab)
* [Supervisión](#monitoring-tab)
* [Administración](#management-tab)
* [Ayuda](#help-tab)

### <a name="home-tab"></a>Pestaña Inicio

:::image type="content" source="images/kusto-explorer/home-tab.png" alt-text="Pestaña Inicio del explorador de Kusto":::

En la pestaña Inicio se muestran las funciones usadas más recientemente, divididas en secciones:

* [Consultar](#query-section)
* [Compartir](#share-section)
* [Visualizaciones](#visualizations-section)
* [Vista](#view-section)
* [Ayuda](#help-tab) 

### <a name="query-section"></a>Sección de consulta

:::image type="content" source="images/kusto-explorer/home-query-menu.png" alt-text="Menú de consulta explorador de Kusto":::

|Menú|    Comportamiento|
|----|----------|
|Lista desplegable modo | <ul><li>Modo de consulta: cambia la ventana de consulta a un [modo de script](kusto-explorer-using.md#query-mode). Los comandos se pueden cargar y guardar como scripts (valor predeterminado)</li> <li> Modo de búsqueda: un único modo de consulta en el que cada comando especificado se procesa inmediatamente y presenta un resultado en la ventana de resultados.</li> <li>Modo buscar + +: permite buscar un término mediante la sintaxis de búsqueda en una o más tablas. Más información sobre el uso del [modo de búsqueda + +](kusto-explorer-using.md#search-mode)</li></ul> |
|Nueva pestaña| Abre una nueva pestaña para consultar Kusto |

### <a name="share-section"></a>Sección compartir

:::image type="content" source="images/kusto-explorer/home-share-menu.png" alt-text="Menú compartir del explorador de Kusto":::

|Menú|    Comportamiento|
|----|----------|
|Datos al portapapeles|    Exporta la consulta y el conjunto de datos a un portapapeles. Si se presenta un gráfico, exporta el gráfico como mapa de bits| 
|Resultado al portapapeles| Exporta el conjunto de datos a un portapapeles. Si se presenta un gráfico, exporta el gráfico como mapa de bits| 
|Consulta al portapapeles| Exporta la consulta a un portapapeles|

### <a name="visualizations-section"></a>Sección visualizaciones

:::image type="content" source="images/kusto-explorer/home-visualizations-menu.png" alt-text="Visualizaciones del menú del explorador de Kusto":::

|Menú         | Comportamiento|
|-------------|---------|
|Gráfico de áreas      | Muestra un gráfico de áreas en el que el eje X es la primera columna (debe ser numérica). Todas las columnas numéricas están asignadas a diferentes series (eje Y) |
|Gráfico de columnas | Muestra un gráfico de columnas en el que todas las columnas numéricas están asignadas a diferentes series (eje Y). La columna de texto antes de Numeric es el eje X (se puede controlar en la interfaz de usuario)|
|Gráfico de barras    | Muestra un gráfico de barras en el que todas las columnas numéricas están asignadas a diferentes series (eje X). La columna de texto antes de Numeric es el eje Y (se puede controlar en la interfaz de usuario)|
|gráfico de áreas apiladas      | Muestra un gráfico de áreas apiladas en el que el eje X es la primera columna (debe ser numérica). Todas las columnas numéricas están asignadas a diferentes series (eje Y) |
|Gráfico de escala de tiempo   | Muestra un gráfico de hora en el que el eje X es la primera columna (debe ser DateTime). Todas las columnas numéricas se asignan a diferentes series (eje Y).|
|Gráfico de líneas   | Muestra un gráfico de líneas en el que el eje X es la primera columna (debe ser numérica). Todas las columnas numéricas se asignan a diferentes series (eje Y).|
|Gráfico de anomalías|    Similar a gráfico, pero encuentra anomalías en los datos de serie temporal mediante el algoritmo de anomalías de machine learning. En la detección de anomalías, Kusto. Explorer usa la función [series_decompose_anomalies](../query/series-decompose-anomaliesfunction.md) . (*) 
|Gráfico circular    |    Muestra un gráfico circular en el que el eje de color es la primera columna. El eje Theta (debe ser una medida, convertida en porcentaje) es la segunda columna.|
|Escala de tiempo |    Muestra un gráfico de escalera en el que el eje X es las dos últimas columnas (debe ser DateTime). El eje Y es un compuesto de las demás columnas.|
|Gráfico de dispersión| Muestra un gráfico de puntos en el que el eje X es la primera columna (debe ser numérica). Todas las columnas numéricas se asignan a diferentes series (eje Y).|
|Gráfico dinámico  | Muestra una tabla dinámica y un gráfico dinámico que proporciona toda la flexibilidad de seleccionar datos, columnas, filas y diversos tipos de gráficos.| 
|Tiempo dinámico   | Navegación interactiva en la línea de tiempo de eventos (dinamización en el eje de tiempo)|

(*) Gráfico de anomalías: el algoritmo espera los datos de los tiempos, que consta de dos columnas:
1. Tiempo en depósitos de intervalo fijo
2. Valor numérico para la detección de anomalías para generar datos de tiempos en Kusto. Explorer, resume por el campo de tiempo y especifica la ubicación del depósito de tiempo.

### <a name="view-section"></a>Sección de vista

:::image type="content" source="images/kusto-explorer/home-view-menu.png" alt-text="Menú de vista del explorador de Kusto":::

|Menú           | Comportamiento|
|---------------|---------|
|Modo de vista completa | Maximiza el espacio de trabajo ocultando el menú de la cinta de opciones y el panel de conexión. Salga del modo de vista completa **Home**seleccionando el  >  **modo de vista completa**de inicio o presionando **F11**.|
|Ocultar columnas vacías| Quita columnas vacías de la cuadrícula de datos|
|Contraer columnas singulares| Contrae las columnas con valores singulares|
|Explorar valores de columna| Muestra la distribución de valores de columna|
|Aumentar fuente  | Aumenta el tamaño de fuente de la pestaña de consulta y de la cuadrícula de datos de resultados.|  
|Reducir fuente  | Reduce el tamaño de fuente de la pestaña de consulta y de la cuadrícula de datos de resultados.|

(*) Configuración de la vista de datos: Kusto. Explorer realiza un seguimiento de los valores que se usan por cada conjunto de columnas único. Así, cuando se reordenan o se quitan columnas, la vista de datos se guarda y se reutilizará cada vez que se recuperen los datos con las mismas columnas. Para restablecer los valores predeterminados de la vista, en la pestaña **vista** , seleccione **restablecer vista**. 

## <a name="file-tab"></a>Pestaña Archivo

:::image type="content" source="images/kusto-explorer/file-tab.png" alt-text="Pestaña archivo del explorador de Kusto":::

|Menú| Comportamiento|
|---------------|---------|
||---------*Script de consulta*---------|
|Nueva pestaña | Abre una nueva ventana de pestaña para consultar Kusto |
|Abrir archivo| Carga datos de un archivo *. kql en el panel de scripts activo.|
|Guardar en el archivo| Guarda el contenido del panel de scripts activo en el archivo *. kql|
|Cerrar pestaña| Cierra la ventana de pestaña actual|
||---------*Guardar datos*---------|
|Datos a CSV       | Exporta los datos a un archivo CSV (valores separados por comas)| 
|Datos a JSON      | Exporta los datos a un archivo con formato JSON|
|Datos a Excel     | Exporta los datos a un archivo XLSX (Excel)|
|Datos a texto      | Exporta los datos a un archivo TXT (texto)| 
|Datos a script de KQL| Exporta la consulta a un archivo de script| 
|Datos a resultados   | Exporta la consulta y los datos a un archivo de resultados (QRES)|
|Ejecutar la consulta en CSV |Ejecuta una consulta y guarda los resultados en un archivo CSV local.|
||---------*Cargar datos*---------|
|De resultados|    Carga la consulta y los datos de un archivo de resultados (QRES)| 
||---------*Portapapeles*---------|
|Consulta y resultados al portapapeles|    Exporta la consulta y el conjunto de datos a un portapapeles. Si se presenta un gráfico, exporta el gráfico como un mapa de bits.| 
|Resultado al portapapeles| Exporta el conjunto de datos a un portapapeles. Si se presenta un gráfico, exporta el gráfico como un mapa de bits.| 
|Consulta al portapapeles| Exporta la consulta a un portapapeles|
||---------*Resultados*---------|
|Borrar caché de resultados| Borra los resultados almacenados en caché de las consultas ejecutadas anteriormente.| 

## <a name="connections-tab"></a>Pestaña Connections (Conexiones)

:::image type="content" source="images/kusto-explorer/connections-tab.png" alt-text="Pestaña conexiones del explorador de Kusto":::

|Menú|Comportamiento|
|----|----------|
||---------*Familias*---------|
|Agregar grupo| Agrega un nuevo grupo de servidores de Kusto|
|Cambiar nombre de grupo| Cambia el nombre del grupo de servidores de Kusto existente|
|Quitar grupo| Quita el grupo de servidores de Kusto existente|
||---------*Clústeres*---------|
|Importar conexiones| Importa conexiones desde un archivo que especifica conexiones|
|Exportar conexiones| Exporta conexiones a un archivo.|
|Agregar conexión| Agrega una nueva conexión de servidor Kusto| 
|Editar conexión| Abre un cuadro de diálogo de propiedades de conexión del servidor de Kusto editando|
|Quitar conexión| Quita la conexión existente con el servidor Kusto|
|Actualizar| Actualiza las propiedades de una conexión de servidor de Kusto|
||---------*Bursátil*---------|
|Inspección de la entidad de seguridad de incorporación| Muestra los detalles actuales del usuario activo|
|Cierre de sesión de AAD| Cierra la sesión del usuario actual de la conexión a AAD.|
||---------*Ámbito de datos*---------|
|Ámbito de almacenamiento en caché|<ul><li>Solicitudes de ejecución de datos activas solo en la [caché de datos activos](../management/cachepolicy.md)</li><li>Todos los datos: ejecutar consultas en todos los datos disponibles (valor predeterminado)</li></ul> |
|Columna DateTime| Nombre de la columna que se puede usar para el filtro previo de hora|
|Filtro de tiempo| Valor de tiempo-filtro previo|

## <a name="view-tab"></a>Pestaña Vista

:::image type="content" source="images/kusto-explorer/view-tab.png" alt-text="Pestaña de vista del explorador de Kusto":::

|Menú|Comportamiento|
|----|----------|
||---------*Aparición*---------|
|Modo de vista completa | Maximiza el espacio de trabajo ocultando el menú de la cinta de opciones y el panel de conexión|
|Aumentar fuente  | Aumenta el tamaño de fuente de la pestaña de consulta y de la cuadrícula de datos de resultados.|  
|Reducir fuente  | Reduce el tamaño de fuente de la pestaña de consulta y de la cuadrícula de datos de resultados.|
|Restablecer diseño|Restablece el diseño de las ventanas y los controles de acoplamiento de la herramienta|
|Cambiar nombre de pestaña de documento |Cambiar el nombre de la pestaña seleccionada |
||---------*Vista de datos*---------|
|Restablecer vista| Restablece la configuración de la vista de datos a sus valores predeterminados (*)|
|Explorar valores de columna|Muestra la distribución de valores de columna|
|Centrarse en las estadísticas de consultas|Cambia el foco a estadísticas de consulta en lugar de a los resultados de la consulta al finalizar la consulta.|
|Ocultar duplicados|Alterna la eliminación de las filas duplicadas de los resultados de la consulta|
|Ocultar columnas vacías|Alterna la eliminación de columnas vacías de los resultados de la consulta|
|Contraer columnas singulares|Alterna la contracción de las columnas con el valor singular|
||---------*Filtrado de datos*---------|
|Filtrar filas en la búsqueda|Alterna la opción para mostrar solo las filas coincidentes en la búsqueda de resultados de la consulta (**Ctrl + F**)|
||---------*Visualizaciones*---------|
|Visualizaciones|Consulte [visualizaciones](#visualizations-section), más arriba. |

(*) Configuración de la vista de datos: Kusto. Explorer realiza un seguimiento de la configuración usada por cada conjunto de columnas único. Cuando se reordenan o quitan columnas, la vista de datos se guarda y se reutilizará cada vez que se recuperen los datos con las mismas columnas. Para restablecer los valores predeterminados de la vista, en la pestaña **vista** , seleccione **restablecer vista**. 

## <a name="tools-tab"></a>Pestaña Herramientas

:::image type="content" source="images/kusto-explorer/tools-tab.png" alt-text="Pestaña Herramientas del explorador de Kusto":::

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
|Opciones| Abre una herramienta para configurar las opciones de la aplicación. Más información sobre [las opciones de Kusto. Explorer](kusto-explorer-options.md).|

## <a name="monitoring-tab"></a>Pestaña supervisión

:::image type="content" source="images/kusto-explorer/monitoring-tab.png" alt-text="Pestaña supervisión del explorador de Kusto":::

|Menú             | Comportamiento|
|-----------------|---------| 
||---------*Control*---------|
|Diagnóstico del clúster | Muestra un resumen de estado del grupo de servidores seleccionado actualmente en el panel conexiones | 
|Datos más recientes: todas las tablas| Muestra un resumen de los datos más recientes de todas las tablas de la base de datos seleccionada actualmente|
|Datos más recientes: tabla seleccionada|Muestra en la barra de estado los datos más recientes de la tabla seleccionada.| 

## <a name="management-tab"></a>Pestaña Administración

:::image type="content" source="images/kusto-explorer/management-tab.png" alt-text="Pestaña Administración del explorador de Kusto":::

|Menú             | Comportamiento|
|-----------------|---------|
||---------*Entidades de seguridad autorizadas*---------|
|Administración de entidades de seguridad autorizadas de clúster |Permite administrar las entidades de seguridad de un clúster para los usuarios autorizados| 
|Administrar entidades de seguridad autorizadas | Habilita la administración de entidades de seguridad de una base de datos para usuarios autorizados| 
|Administrar entidades de seguridad autorizadas de tabla | Habilita la administración de entidades de seguridad de una tabla para usuarios autorizados| 
|Administrar entidades de seguridad autorizadas de función | Habilita la administración de entidades de seguridad de una función para usuarios autorizados| 

## <a name="help-tab"></a>Pestaña ayuda

:::image type="content" source="images/kusto-explorer/help-tab.png" alt-text="Pestaña ayuda del explorador de Kusto":::

|Menú             | Comportamiento|
|-----------------|---------|
||---------*Documentación*---------|
|Ayuda             | Abre un vínculo a la documentación en línea de Kusto  | 
|Novedades       | Abre un documento que enumera todos los cambios de Kusto. Explorer.|
|Informar sobre un problema      |Abre un cuadro de diálogo con dos opciones: <ul><li>Informar de problemas relacionados con el servicio</li><li>Notificar problemas en la aplicación cliente</li></ul> | 
|Sugerir característica  | Abre un vínculo al foro de comentarios de Kusto | 
|Comprobar actualizaciones     | Comprueba si hay actualizaciones de la versión de Kusto. Explorer. | 

## <a name="connections-panel"></a>Panel conexiones

:::image type="content" source="images/kusto-explorer/connections-panel.png" alt-text="Panel de conexiones del explorador de Kusto":::

En el panel conexiones se muestran todas las conexiones de clúster configuradas. Para cada clúster, se muestran las bases de datos, las tablas y los atributos (columnas) que almacenan. Seleccione elementos (que establece un contexto implícito para la búsqueda/consulta en el panel principal) o haga doble clic en los elementos para copiar el nombre en el panel buscar/consulta.

Si el esquema real es grande (por ejemplo, una base de datos con cientos de tablas), puede buscarlo presionando **Ctrl + F** y escribiendo una subcadena (sin distinción de mayúsculas y minúsculas) del nombre de la entidad que está buscando.

Kusto. Explorer admite el control del panel de conexión desde la ventana de consulta, que es útil para los scripts. Por ejemplo, puede iniciar un archivo de script con un comando que indique a Kusto. Explorer que se conecte con el clúster o la base de datos cuyos datos consulta el script mediante la sintaxis siguiente:

<!-- csl -->
```kusto
#connect cluster('help').database('Samples')

StormEvents | count
```

Ejecute cada línea mediante `F5` o similar.

### <a name="control-the-user-identity-connecting-to-kustoexplorer"></a>Controlar la identidad del usuario que se conecta a Kusto. Explorer

El modelo de seguridad predeterminado para las nuevas conexiones es la seguridad federada de AAD. La autenticación se realiza a través del Azure Active Directory mediante la experiencia de usuario de AAD predeterminada.

Si necesita un control más preciso sobre los parámetros de autenticación, puede expandir el cuadro de edición "avanzadas: cadenas de conexión" y proporcionar un valor de [cadena de conexión Kusto](../api/connection-strings/kusto.md) válido.

Por ejemplo, los usuarios con presencia en varios inquilinos de AAD a veces necesitan usar una "proyección" concreta de sus identidades para un inquilino de AAD específico. Para ello, proporcione una cadena de conexión, como la siguiente (Reemplace las palabras en MAYÚSCULAs con valores específicos):

```kusto
Data Source=https://CLUSTER_NAME.kusto.windows.net;Initial Catalog=DATABASE_NAME;AAD Federated Security=True;Authority Id=AAD_TENANT_OF_CLUSTER;User=USER_DOMAIN
```

* `AAD_TENANT_OF_CLUSTER`es un nombre de dominio o un identificador de inquilino de AAD (un GUID) del inquilino de AAD en el que se hospeda el clúster. Suele ser el nombre de dominio de la organización a la que pertenece el clúster como, por ejemplo, `contoso.com` . 
* USER_DOMAIN es la identidad del usuario invitado en ese inquilino (por ejemplo, `user@example.com` ). 

>[!Note]
> El nombre de dominio del usuario no es necesariamente el mismo que el del inquilino que hospeda el clúster.

:::image type="content" source="images/kusto-explorer/advanced-connection-string.png" alt-text="Cadena de conexión avanzada del explorador de Kusto":::

## <a name="keyboard-shortcuts"></a>Métodos abreviados de teclado

Podría encontrar que el uso de métodos abreviados de teclado le permite realizar operaciones más rápido que con el mouse. Eche un vistazo a esta [lista de métodos abreviados de teclado de Kusto. Explorer](kusto-explorer-shortcuts.md) para obtener más información.

## <a name="table-row-colors"></a>Colores de fila de tabla

Kusto. Explorer intenta interpretar la gravedad o el nivel de detalle de cada fila en el panel resultados y colorearlos en consecuencia. Para ello, hace coincidir los valores distintos de cada columna con un conjunto de patrones conocidos ("WARNING", "error", etc.).

Para modificar la combinación de colores de salida o desactivar este comportamiento, en el menú **herramientas** , seleccione **Opciones**  >  **visor de resultados**  >  **detalle esquema de color**.

:::image type="content" source="images/kusto-explorer/ke-color-scheme.png" alt-text="Modificación de la combinación de colores del explorador de Kusto":::

## <a name="next-steps"></a>Pasos siguientes

Más información sobre cómo trabajar con Kusto. Explorer:

* [Usar Kusto. Explorer](kusto-explorer-using.md)
* [Métodos abreviados de teclado de Kusto. Explorer](kusto-explorer-shortcuts.md)
* [Opciones de Kusto. Explorer](kusto-explorer-options.md)
* [Solución de problemas de Kusto. Explorer](kusto-explorer-troubleshooting.md)

Más información sobre las herramientas y utilidades de Kusto. Explorer:
* [Analizador de código Kusto. Explorer](kusto-explorer-code-analyzer.md)
* [Navegación por el código Kusto. Explorer](kusto-explorer-codenav.md)
* [Refactorización de código de Kusto. Explorer](kusto-explorer-refactor.md)
* [Lenguaje de consulta de Kusto (KQL)](https://docs.microsoft.com/azure/kusto/query/)
