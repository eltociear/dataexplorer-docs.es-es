---
title: Opciones de Kusto Explorer- Explorador de Azure Data Explorer Microsoft Docs
description: En este artículo se describen las opciones de Kusto Explorer en El Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 04/01/2020
ms.openlocfilehash: dee96dd937b4258525673ced10f206836f6e51c2
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81524011"
---
# <a name="kusto-explorer-options"></a>Opciones de Kusto Explorer

En las tablas siguientes se describen las opciones para personalizar el comportamiento de Kusto.Explorer desde **el** > cuadro de diálogo**Opciones** de herramientas.

![**Herramientas** > **Opciones**](\images\Kusto-Tools-Explorer-Options\tools-options.png)

## <a name="general-settings"></a>Configuración general

| Opción | Descripción |
|---------|--------------|
| Modo De herramientas | Permite características de aplicaciones beta, alfa y experimentales. El valor predeterminado es none.|
| Accesibilidad ampliada | Cuando está habilitada, la aplicación envía eventos de accesibilidad utilizados por las herramientas de accesibilidad. Puede afectar al rendimiento si está habilitado. El valor predeterminado está deshabilitado. |
| Permitir el envío de datos de telemetría | Cuando está habilitada, la aplicación envía datos de telemetría cuando se producen errores o la aplicación se bloquea. |
| Mensaje de bienvenida | Cuando está habilitada, la aplicación muestra el mensaje de bienvenida al iniciarse. El valor predeterminado está habilitado.|
| Tema de visualización | Esquema de color de la interfaz de usuario de la aplicación: claro u oscuro.|
  
## <a name="query-editor"></a>Editor de consultas

| Opción | Descripción |
|---------|--------------|
| Consultas de guardado automático | Cuando está habilitada, la aplicación guarda automáticamente los documentos abiertos. Para cambiar esta configuración es necesario reiniciar la aplicación para que surta efecto. De forma predeterminada está habilitado.|
| Control de cambios | Cuando está habilitada, la aplicación realiza un seguimiento de los cambios realizados en los scripts de consulta en el disco por otras aplicaciones. Si el script de consulta se cambia externamente, se le notificará y se le pedirá que lo vuelva a cargar. Para cambiar esta configuración es necesario reiniciar la aplicación. De forma predeterminada está habilitado.|
| Usar editar sesiones | Cuando está habilitado, cada proceso de aplicación posee su propio conjunto de documentos. Deshabilitado de forma predeterminada.|
| Tamaño de fuente | El tamaño de fuente utilizado en el editor de consultas.|
| Seleccionar fondo de comando | El color de fondo utilizado para resaltar el comando seleccionado actualmente.|
| Sustituya las pestañas por espacios | Cuando está habilitada, las pestañas se reemplazan automáticamente por espacios.|
| Desactivar la inyección de parámetros de función | Cuando está habilitada, la inserción de parámetros de función desde el portapapeles está deshabilitada.|
| Deshabilitar la inserción de parámetros de consulta | Cuando está habilitada, la inserción de parámetros de consulta está deshabilitada.|
| Deshabilitar desencadenadores de ejecución de consultas excepto F5 | Cuando está habilitada, solo la tecla F5 desencadenará las consultas que se ejecutarán.|
| Mostrar ayuda dentro de la aplicación | Cuando está habilitado, los temas de Ayuda se muestran dentro de la aplicación. Cuando se abren temas de Ayuda deshabilitados dentro de un explorador.|
| Límite de longitud del parámetro de consulta | La longitud máxima de una cadena que se puede utilizar como parámetro de consulta. Establecer este valor en más de 128K puede causar problemas de rendimiento. El valor predeterminado es 64K.|

## <a name="intellisense"></a>IntelliSense

| Opción | Descripción |
|---------|--------------|
| Versión de IntelliSense | Se utiliza la versión del motor IntelliSense. *V1* es el motor clásico. *V2* es el motor moderno. El valor predeterminado es *V2*. |
| IntelliSense: Comandos de control | La versión de IntelliSense para comandos de control. *V1* es el motor clásico. *V2* es el motor moderno. El valor predeterminado es *V2*. | 
| IntelliSense | Habilita o deshabilita IntelliSense. El valor predeterminado está habilitado.|
| Resaltado de sintaxis | Habilita o deshabilita el resaltado de sintaxis. El valor predeterminado está habilitado.|
| Consejos de herramientas | Habilita o deshabilita las sugerencias de herramientas que aparecen cuando el mouse pasa el ratón sobre las áreas de la consulta seleccionada. El valor predeterminado está habilitado.|

## <a name="formatter"></a>Formateador

| Opción | Descripción |
|---------|--------------|
| Versión | La versión del formateador que se utiliza cuando se aplica la herramienta Consulta de Prettify a la consulta actual. *V1* es el formateador clásico. *V2* es la formatea moderna. El valor predeterminado es *V2*.|
| Sangría | El número de espacios para los elementos con sangría.|
| Frenos de función | La colocación de llaves de función. *Vertical* coloca llaves abiertas y cercanas en nuevas líneas. *Horizontal* deja la llave abierta en su línea original y coloca la llave de cierre en una nueva línea. *Ninguno* deja todos los frenos tal como están.|
| Operador de tuberías | La colocación del carácter de canalización (barra) que existe entre los operadores de consulta tabulares. *Nueva línea* coloca todos los caracteres de tubería en una nueva línea. *Smart* coloca todos los caracteres de canalización en una nueva línea si la consulta ya abarca más de una línea. *Ninguno* deja todos los caracteres de tubería tal como están.|
| Punto y coma | La colocación de punto y coma que finalizan las instrucciones de consulta. *Nueva línea* coloca punto y coma en una nueva línea, con sangría. *Smart* coloca punto y coma en una nueva línea si la propia instrucción abarca más de una línea.  *Ninguno* deja punto y coma tal cual.
| Insertar sintaxis faltante | Agrega signos de puntuación que faltan (como comas y punto y coma) para los que está recibiendo mensajes de error.|

## <a name="results-viewer"></a>Visor de resultados

| Opción | Descripción |
|---------|--------------|
| Tamaño de fuente | El tamaño de fuente utilizado por la cuadrícula de la tabla de datos, donde se muestran los resultados de la consulta.|
| Esquema de color de verbosidad | Selecciona el esquema de color para la base de formato de fila en el nivel de detalle detectado automáticamente.|
| Ocultar columnas vacías | Cuando está habilitada, las columnas vacías en la vista que muestran los datos se ocultarán.  El valor predeterminado está deshabilitado.|
| Contraer columnas de un solo valor| Cuando está habilitado, se contraerá automáticamente las columnas de un solo valor en la vista que muestra los datos. Las columnas con valores únicos no vacíos se mostrarán como grupos. El valor predeterminado está deshabilitado.|
| Ordenar automáticamente los resultados por columnas de fecha y hora | Cuando está habilitada, las filas se ordenarán automáticamente por columnas de fecha y hora. El valor predeterminado está habilitado.|
| Rango visible de columna | La cantidad máxima de caracteres que se mostrarán en una columna. El valor predeterminado es 2048.|
| Visualizar JSON como texto | Cuando está habilitado, los valores JSON se visualizan como texto. El valor predeterminado está deshabilitado.|
| Tamaño máximo de los detalles del texto | El número máximo de caracteres que se mostrarán en el panel de información detallada cuando se haga doble clic en la celda. El valor predeterminado es 32KB.|

## <a name="connections"></a>Conexiones

| Opción | Descripción |
|---------|--------------|
| Ordenar columnas de tabla alfabéticamente | Cuando está habilitada, las columnas que aparecen debajo de los nodos de tabla en el panel de conexiones se ordenarán alfabéticamente.|
| Tiempo de espera del servidor de consultas | El tiempo de espera del servidor para la ejecución de consultas.|
| Advertir en el bloqueo de conexión | Cuando está habilitada, la aplicación advertirá sobre el bloqueo de conexión cada vez que se intente realizar un cambio de conexión. El valor predeterminado está habilitado.|
| Exploración de esquemas diferidos | Cuando está habilitado, el panel de conexiones solo capturará y mostrará el esquema de base de datos cuando se expanda el nodo de base de datos.|
| Mostrar objetos ocultos del sistema | Cuando está habilitado, se mostrarán objetos ocultos del sistema si el usuario tiene los permisos adecuados.|
| Consultar consistencia débil | Cuando está habilitada, se usará una coherencia débil para las consultas.|
| Versión del analizador Kusto | La versión del analizador utilizada para ejecutar consultas. *V1* es el analizador clásico. *V2* es el analizador moderno. El valor predeterminado es *V1*.|

## <a name="diagnostic-tracing"></a>Seguimiento diagnóstico

| Opción | Descripción |
|---------|--------------|
| Habilitar el seguimiento | Habilita el seguimiento.|
| Detalle de trazas | Establece la verbosidad del seguimiento.|
| Ubicación de los rastros | La ubicación donde se registran los seguimientos.|
| PlatformTraceVerbosity | Establece la verbosidad del seguimiento de la plataforma.| 