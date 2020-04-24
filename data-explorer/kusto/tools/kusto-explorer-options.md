---
title: 'Opciones del explorador de Kusto: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describen las opciones del explorador de Kusto en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 04/01/2020
ms.openlocfilehash: 5b1bb73778858998f835f1e8718afbfbd6b69443
ms.sourcegitcommit: e1e35431374f2e8b515bbe2a50cd916462741f49
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/23/2020
ms.locfileid: "82108395"
---
# <a name="kusto-explorer-options"></a>Opciones del Kusto Explorer

En las tablas siguientes se describen las opciones para personalizar el comportamiento de Kusto. Explorer desde el cuadro de diálogo**Opciones** de **herramientas** > .

## <a name="general-settings"></a>Configuración general

| Opción | Descripción |
|---------|--------------|
| Modo de herramienta | Habilita las características de la aplicación beta, alfa y experimental. El valor predeterminado es none.|
| Accesibilidad extendida | Cuando está habilitada, la aplicación envía eventos de accesibilidad utilizados por las herramientas de accesibilidad. Puede afectar al rendimiento si está habilitado. De forma predeterminada, está deshabilitada. |
| Permitir el envío de datos de telemetría | Cuando está habilitada, la aplicación envía datos de telemetría cuando se producen errores o la aplicación se bloquea. |
| Mensaje de bienvenida | Cuando está habilitada, la aplicación muestra el mensaje de bienvenida al iniciar. Está habilitada de forma predeterminada.|
| Mostrar tema | La combinación de colores de la interfaz de usuario de la aplicación: claro o oscuro.|
  
## <a name="query-editor"></a>Editor de consultas

| Opción | Descripción |
|---------|--------------|
| Guardado automático de consultas | Cuando está habilitada, la aplicación guarda automáticamente los documentos abiertos. Para cambiar esta configuración, es necesario reiniciar la aplicación para que surta efecto. De forma predeterminada está habilitado.|
| Control de cambios | Cuando está habilitada, la aplicación realiza un seguimiento de los cambios realizados en los scripts de consulta en el disco por otras aplicaciones. Si el script de consulta se cambia externamente, se le notificará y se le pedirá que vuelva a cargarlo. Para cambiar esta configuración, es necesario reiniciar la aplicación. De forma predeterminada está habilitado.|
| Uso de las sesiones de edición | Cuando está habilitado, cada proceso de aplicación posee su propio conjunto de documentos. Deshabilitado de forma predeterminada.|
| Tamaño de fuente | Tamaño de fuente utilizado en el editor de consultas.|
| Seleccionar fondo del comando | Color de fondo que se usa para resaltar el comando seleccionado actualmente.|
| Reemplazar pestañas con espacios | Cuando está habilitada, las pestañas se reemplazan automáticamente por espacios.|
| Deshabilitar la inserción de parámetros de función | Cuando está habilitada, se deshabilita la inserción de parámetros de función desde el portapapeles.|
| Deshabilitar la inyección de parámetros de consulta | Cuando está habilitada, la inserción de parámetros de consulta está deshabilitada.|
| Deshabilitar desencadenadores de ejecución de consulta excepto F5 | Cuando está habilitada, solo la tecla F5 desencadenará consultas para que se ejecuten.|
| Mostrar la ayuda dentro de la aplicación | Cuando está habilitada, los temas de ayuda se muestran dentro de la aplicación. Cuando los temas de ayuda deshabilitados se abren dentro de un explorador.|
| Límite de longitud del parámetro de consulta | Longitud máxima de una cadena que se puede usar como parámetro de consulta. Si se establece este valor en más de 128 KB, pueden producirse problemas de rendimiento. El valor predeterminado es 64 KB.|

## <a name="intellisense"></a>IntelliSense

| Opción | Descripción |
|---------|--------------|
| Versión de IntelliSense | Versión del motor de IntelliSense utilizada. *V1* es el motor clásico. *V2* es el motor moderno. El valor predeterminado es *V2*. |
| IntelliSense: comandos de control | Versión de IntelliSense para los comandos de control. *V1* es el motor clásico. *V2* es el motor moderno. El valor predeterminado es *V2*. | 
| IntelliSense | Habilita o deshabilita IntelliSense. Está habilitada de forma predeterminada.|
| Resaltado de sintaxis | Habilita o deshabilita el resaltado de sintaxis. Está habilitada de forma predeterminada.|
| Sugerencias de herramientas | Habilita o deshabilita la información sobre herramientas que aparece cuando se mantiene el mouse sobre las áreas de la consulta seleccionada. Está habilitada de forma predeterminada.|

## <a name="formatter"></a>Formateador

| Opción | Descripción |
|---------|--------------|
| Versión | Versión del formateador que se usa cuando se aplica la herramienta de consulta se ha mejorado a la consulta actual. *V1* es el formateador clásico. *V2* es el formateador moderno. El valor predeterminado es *V2*.|
| Sangría | Número de espacios para los elementos con sangría.|
| Llaves de función | La posición de las llaves de función. *Vertical* coloca las llaves de apertura y cierre en las nuevas líneas. *Horizontal* deja la llave de apertura en su línea original y coloca la llave de cierre en una nueva línea. *Ninguno* deja todas las llaves tal cual.|
| Operador de canalización | La posición del carácter de canalización (barra) que existe entre los operadores de consulta tabulares. *Nueva línea* coloca todos los caracteres de barra vertical en una nueva línea. *Smart* coloca todos los caracteres de barra vertical en una nueva línea si la consulta ya abarca más de una línea. *Ninguno* deja todos los caracteres de barra vertical tal cual.|
| Punto y coma | La posición de los puntos y comas que finalizan las instrucciones de consulta. La *nueva línea* coloca puntos y comas en una nueva línea, con sangría. *Smart* coloca los puntos y comas en una nueva línea si la propia instrucción abarca más de una línea.  *Ninguno* deja puntos y comas tal cual.
| Insertar sintaxis que falta | Agrega signos de puntuación que faltan como (por ejemplo, comas y puntos y coma) para los que se reciben mensajes de error.|

## <a name="results-viewer"></a>Visor de resultados

| Opción | Descripción |
|---------|--------------|
| Tamaño de fuente | Tamaño de fuente utilizado por la cuadrícula de la tabla de datos, donde se muestran los resultados de la consulta.|
| Combinación de colores de nivel de detalle | Selecciona la combinación de colores para la base de formato de filas en el nivel de detalle detectado automáticamente.|
| Ocultar columnas vacías | Cuando está habilitada, se ocultan las columnas vacías en la vista que muestra los datos.  De forma predeterminada, está deshabilitada.|
| Contraer columnas de un solo valor| Cuando está habilitada, se contrae automáticamente las columnas de un solo valor en la vista que muestra los datos. Las columnas con valores únicos que no estén vacíos se mostrarán como grupos. De forma predeterminada, está deshabilitada.|
| Ordenación automática de los resultados por columnas DATETIME | Cuando está habilitada, las filas se ordenarán automáticamente por columnas de fecha y hora. Está habilitada de forma predeterminada.|
| Rango visible de columna | La cantidad máxima de caracteres que se va a mostrar en una columna. El valor predeterminado es 2048.|
| Visualizar JSON como texto | Cuando está habilitado, los valores JSON se visualizan como texto. De forma predeterminada, está deshabilitada.|
| Tamaño máximo de detalles de texto | Número máximo de caracteres que se mostrarán en el panel de información detallada cuando se haga doble clic en la celda. El valor predeterminado es 32 KB.|

## <a name="connections"></a>Conexiones

| Opción | Descripción |
|---------|--------------|
| Ordenar columnas de tabla alfabéticamente | Cuando está habilitada, las columnas que aparecen en los nodos de tabla del panel conexiones se ordenarán alfabéticamente.|
| Tiempo de espera del servidor de consultas | Tiempo de espera del servidor para la ejecución de la consulta.|
| Avisar al bloquear la conexión | Cuando está habilitada, la aplicación advertirá sobre el bloqueo de conexión cada vez que se intente un cambio de conexión. Está habilitada de forma predeterminada.|
| Exploración diferida de esquemas | Cuando se habilita, el panel conexiones solo captura y muestra el esquema de la base de datos cuando se expande el nodo de la base de datos.|
| Mostrar objetos del sistema ocultos | Cuando está habilitada, se mostrarán los objetos ocultos del sistema si el usuario tiene los permisos adecuados.|
| Consulta de coherencia débil | Cuando está habilitada, se usará la coherencia débil para las consultas.|
| Versión del analizador de Kusto | Versión del analizador que se usa para ejecutar consultas. *V1* es el analizador clásico. *V2* es el analizador moderno. El valor predeterminado es *v1*.|

## <a name="diagnostic-tracing"></a>Seguimiento de diagnósticos

| Opción | Descripción |
|---------|--------------|
| Habilitar el seguimiento | Habilita el seguimiento.|
| Nivel de detalle de seguimiento | Establece el nivel de detalle del seguimiento.|
| Ubicación de seguimientos | La ubicación donde se registran los seguimientos.|
| PlatformTraceVerbosity | Establece el nivel de detalle del seguimiento para la plataforma.| 