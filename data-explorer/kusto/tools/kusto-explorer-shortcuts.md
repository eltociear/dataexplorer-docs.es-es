---
title: Métodos abreviados de teclado de Kusto.Explorer (teclas de acceso rápido) - Explorador de azure Data Explorer Microsoft Docs
description: En este artículo se describen los métodos abreviados de teclado de Kusto.Explorer (teclas de acceso rápido) en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/22/2020
ms.openlocfilehash: 81d99d14831905681c2dbbc45aea4b63134a06a8
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81523926"
---
# <a name="kustoexplorer-keyboard-shortcuts-hot-keys"></a>Métodos abreviados de teclado Kusto.Explorer (teclas de acceso rápido)

## <a name="application-level"></a>En el nivel de la aplicación

Los siguientes métodos abreviados de teclado se pueden utilizar desde cualquier contexto:

|Tecla de acceso rápido|Descripción|
|-------|-----------|
|`F1`     | Abre la ayuda|
|`F11`    | Alternar el modo de vista completa|
|`Ctrl`+`F1`| Alternar la apariencia de la cinta |
|`Ctrl`+`+`| Aumenta la fuente de resultados de consultas y datos|
|`Ctrl`+`-`| Disminuye la fuente de resultados de datos y consultas|
|`Ctrl`+`0`| Restablece la fuente de resultados de consultas y datos|
|`Ctrl`+`1` .. `7`| Cambia al panel de documentos de consulta con el número respectivo (1..7)|
|`Ctrl`+`F2`|Cambia el nombre del encabezado del panel Editor de consultas|
|`Ctrl`+`N` |Abre un nuevo editor de consultas|
|`Ctrl`+`O` |Abrir script de editor de consultas en un nuevo editor de consultas|
|`Ctrl`+`W` |Cierra el editor de consultas activo|
|`Ctrl`+`S` |Guarda la consulta en un archivo|
|`Shift`+`F3` | Abre la Galería de Informes Analíticos|
|`Shift`+`F12`| Alterna el tema Luz/Oscuro de la aplicación|
|`Ctrl`+`Shift`+`O`|Abre el cuadro de diálogo de opciones y configuración de Kusto.Explorer|
|`Esc`|Cancelar consulta en ejecución|
|`Shift`+`F5`|Cancelar consulta en ejecución|

## <a name="query-and-results-view"></a>Vista de consultas y resultados

Los siguientes métodos abreviados de teclado se pueden utilizar al editar una consulta en el editor de consultas o cuando el contexto está en la vista de resultados:

|Tecla de acceso rápido|Descripción|
|-------|-----------|
|`Ctrl`+`Shift`+`C`|Copia consultas, enlaces profundos y datos en el portapapeles|
|`Alt`+`Shift`+`C` |Copia la consulta y el enlace profundo del portapapeles en formato HTML|
|`Alt`+`Shift`+`R` |Copia consulta, enlace profundo y datos del portapapeles en formato Markdown|
|`Alt`+`Shift`+`M` |Copia la consulta y el enlace profundo del portapapeles en formato Markdown|
|`Ctrl`+`~` |Copia la consulta y los datos en el portapapeles en formato de reducción |
|`Ctrl`+`Shift`+`D`|Alterna el modo de ocultar filas duplicadas en la vista de datos|
|`Ctrl`+`Shift`+`H`|Alterna el modo de ocultar columnas vacías en la vista de datos|
|`Ctrl`+`Shift`+`J`|Alterna el modo de contraer columnas con un solo valor en la vista de datos|
|`Ctrl`+`Shift`+`A`|Abre una herramienta Analizador de consultas en un nuevo panel de consultas|
|`Alt`+`C`  |Representa el gráfico de columnas sobre los datos existentes|
|`Alt`+`T`  |Representa el gráfico de línea de tiempo sobre los datos existentes|
|`Alt`+`A`  |Representa el gráfico de escala de tiempo de anomalíasobre los datos existentes|
|`Alt`+`P`  |Representa el gráfico circular sobre los datos existentes|
|`Alt`+`L`  |Representa el gráfico de escala de tiempo de escalera sobre los datos existentes|
|`Alt`+`V`  |Representa el gráfico dinámico sobre los datos existentes|
|`Ctrl`+`Shift`+`V`|Muestra el pivote de la línea de tiempo sobre los datos existentes|
|`Ctrl`+`F9`  | Alterna `show only matching rows` / `highlight matching rows` los modos de`Ctrl`+`F`comportamiento de búsqueda de texto de cliente ( ) en la cuadrícula de datos. |
|`Ctrl`+`F10` |Muestra el panel de detalles- donde la fila seleccionada se presenta como cuadrícula de propiedades|
|`Ctrl`+`F`  | Muestra el cuadro de búsqueda del panel que está actualmente en foco. Soportado `Connetions`en `Data Results`, `Query Editor` , y paneles|
|`Ctrl`+`Tab`| Muestra el cuadro de diálogo del selector de documentos del Editor de consultas. Puede mantener `Ctrl` y swithch entre documentos con`Tab` |
|`Ctrl`+`J`|Alterna la apariencia del panel de resultados|
|`Ctrl`+`E`|Alterna la apariencia del editor de consultas y el panel de resultados en el ciclo de:`Query Editor and Results` -> `Query Editor` -> `Query Editor and Results` -> `Results` |
|`Ctrl`+`Shift`+`E`|Alterna la apariencia del editor de consultas y el panel de resultados en el ciclo de:`Query Editor and Results` -> `Results` -> `Query Editor and Results` -> `Query Editor` |
|`Ctrl`+`Shift`+`R` | Se centra en el panel Resultados |
|`Ctrl`+`Shift`+`T` | Se centra en el panel Conexiones |
|`Ctrl`+`Shift`+`Y` | Se centra en el editor de consultas |
|`Ctrl`+`Shift`+`P` | Se centra en el panel Gráfico |
|`Ctrl`+`Shift`+`I` | Se centra en el panel Información de consulta |
|`Ctrl`+`Shift`+`S` | Se centra en el panel Estadísticas de consultas |
|`Ctrl`+`Shift`+`K` | Se centra en el panel Error |
|`Alt`+`Ctrl`+`L`|Bloquea el contexto de conexión actual al Editor de consultas, por lo que cambiar la fila seleccionada en el panel Conneción no tiene ningún efecto en el contexto del Editor de consultas. |

## <a name="results-table-viewer"></a>Visor de tablas de resultados

Los siguientes métodos abreviados de teclado se pueden utilizar cuando la vista de resultados (tabla) está en el foco de teclado activo:

|Tecla de acceso rápido|Descripción|
|-----------|-----------|
|`Ctrl`+`Q` |Mostrar el menú contextual de la columna actual|
|`Ctrl`+`S` |Alternar la clasificación de columnas actual|
|`Ctrl`+`U` |Abre un panel que muestra los valores de columna actuales con filtrado del lado cliente|
|`Ctrl`+`F` | Muestra el cuadro de búsqueda de los resultados|
|`Ctrl`+`F3`| Alterna `show only matching rows` / `highlight matching rows` los modos de`Ctrl`+`F`comportamiento de búsqueda de texto de cliente ( ) en la cuadrícula de datos. |

## <a name="query-editor"></a>Editor de consultas

Los siguientes métodos abreviados de teclado se pueden utilizar al editar una consulta en el editor de consultas:

|Tecla de acceso rápido|Descripción|
|-------|-----------|
|`F1`|Cuando el cursor apunta a un operador o función- abre una ventana de ayuda con información sobre el operador o la función. Si el tema de ayuda no está presente, se abre una URL de ayuda|
|`F5`|Ejecutar consulta seleccionada actualmente|
|`Shift`+`Enter`|Ejecutar consulta seleccionada actualmente|
|`F8`|Obtener resultados de consulta de la memoria caché local. Si los resultados no están presentes - ejecute la consulta seleccionada actualmente|
|`F6`|Ejecutar la consulta `Progressive Results` seleccionada actualmente en modo|
|`Ctrl`+`F5` | Vista previa de los resultados de la consulta seleccionada (muestra pocos resultados y recuento total)|
|`Ctrl`+`Shift`+`Space`| Insertar selecciones de celdas de datos como filtros en la consulta|
|`Ctrl`+`Space`| Forzar la comprobación de reglas de IntelliSense. Las opciones posibles se mostrarán en cualquier regla coincidente |
|`Ctrl`+`Enter`| Añade `pipe` símbolo y se mueve a una nueva línea|
|`Ctrl`+`Z`| Deshacer |
|`Ctrl`+`Y`| Rehacer |
|`Ctrl`+`L`| Elimina la línea actual|
|`Ctrl`+`D`| Elimina la línea actual| 
|`Ctrl`+`F`| Abre `Find and Replace` el diálogo |
|`Ctrl`+`G`| Abre `Go-to line` el diálogo |
|`Ctrl`+`F8` | Mostrar mis consultas pasadas 3 días |
|`Ctrl`+ soporte | Cuando el cursor está `(` `)` entre `[` `]` corchetes, los símbolos, , , , , `{` `}` - mueven el cursor al corchete de apertura o cierre correspondiente |
|`Ctrl`+`Shift`+`Q` | Prettify consulta actual |
|`Ctrl`+`Shift`+`L` | Haga que la consulta actual o la selección sean minúsculas |
|`Ctrl`+`Shift`+`U` |  Hacer consulta actual o selección en mayúsculas |
|`Ctrl`+`Mouse wheel up`| Aumenta la fuente del editor qupery| 
|`Ctrl`+`Mouse wheel down`| Disminuye la fuente del editor de consultas|
|`Alt`+`P` | Abre el cuadro de diálogo de parámetros de consulta |
|`F2`| Abrir línea actual / texto seleccionado en el cuadro de diálogo del editor |
|`Ctrl`+`F6`| Ejecuta el análisis de consultas estáticas de KQL para detectar problemas comunes |
|`F12`| Vaya a la definición del símbolo |
|`Ctrl`+`F12`| Buscar todas las referencias del símbolo actual |
|`Alt`+`Home`| Vaya a la definición del símbolo |
|`Alt`+`Ctrl`+`M`| Extraiga la expresión literal o tabular seleccionada actualmente como instrucción let |
|`Ctrl`+`.`| Extraiga la expresión literal o tabular seleccionada actualmente como instrucción let |
|`Ctrl`+`R`, `Ctrl`+`R` | Cambia el nombre del símbolo actual |
|`Ctrl`+`K`, `Ctrl`+`D` | Inserta la marca de tiempo actual como literal detatime |
|`Ctrl`+`K`, `Ctrl`+`R` | Fragmento `range x from 1 to 1 step 1` de inserción |
|`Ctrl`+`K`, `Ctrl`+`C` | Comentar la línea actual o las líneas seleccionadas |
|`Ctrl`+`K`, `Ctrl`+`F` | Prettify consulta actual |
|`Ctrl`+`K`, `Ctrl`+`V` | Duplicar consulta actual (anexarla al final del documento de consulta actual) |
|`Ctrl`+`K`, `Ctrl`+`U` | Descomiste la línea actual o las líneas seleccionadas |
|`Ctrl`+`K`, `Ctrl`+`S` | Convierta la línea actual o las líneas seleccionadas en literales de cadena de varias líneas |
|`Ctrl`+`K`, `Ctrl`+`M` | Eliminar las marcas literales de `Ctrl` + `K`agitación multilínea (inverso de , `Ctrl` + `S`) |
|`Ctrl`+`M`, `Ctrl`+`M` | Alternar la expansión de esquematización de la consulta actual |
|`Ctrl`+`M`, `Ctrl`+`L` | Alternar la expansión de esquematización de todas las consultas en el documento |

## <a name="json-viewer"></a>Visor JSON

Los siguientes shorcuts de teclado se pueden utilizar desde el visor JSON de resultados (que se muestra cuando se hace doble clic en un valor json en la celda de vista de resultados):

|Tecla de acceso rápido|Descripción|
|-------|-----------|
|`Ctrl`+`Up Arrow`|Navegue a los padres|
|`Ctrl`+`Right Arrow`|Expandir nodo actual (un nivel)|
|`Ctrl`+`Left Arrow`|Contraer nodo actual (un nivel)|
|`Ctrl`+`.`|Alternar la expansión del nodo actual (todos los niveles secundarios expandidos/contraídos)
|`Ctrl`+`Shift`+`.`|Alternar la expansión del nodo primario actual (todos los niveles secundarios expandidos/contraídos)|

## <a name="connection-panel"></a>Panel de conexión

Los siguientes shorcuts de teclado se pueden utilizar desde el visor JSON de resultados (que se muestra cuando se hace doble clic en un valor json en la celda de vista de resultados):

|Tecla de acceso rápido|Descripción|
|-------|-----------|
|`Ctrl`+`Up`Flecha|Navegue a los padres|
|`Ctrl`+`Right`Flecha|Expandir nodo actual (un nivel)|
|`Ctrl`+`Left`Flecha|Contraer nodo actual (un nivel)|
|`Ctrl`+`Shift`+`L`|Colapsar todos los niveles|
|`Ctrl`+`R`|Actualizar la conexión seleccionada actualmente|
|`Insert`|Añadir una nueva conexión|
|`Del`|Eliminar conexión actual|
|`Ctrl`+`E`|Editar la conexión seleccionada actualmente|
|`Ctrl`+`T`|Abra un nuevo editor de consultas utilizando la conexión seleccionada actualmente|

## <a name="diagnostics-and-monitoring"></a>Diagnóstico y supervisión

Los siguientes métodos abreviados de teclado están disponibles en la cinta de `Monitoring` opciones.

|Tecla de acceso rápido|Descripción|
|-----------|-----------|
|`Ctrl`+`Shift`+`F1`|Ejecutar flujo de diagnóstico de clúster|