---
title: Procedimientos recomendados para usar Power BI para consultar y visualizar datos de Azure Data Explorer
description: En este artículo, aprenderá los procedimientos recomendados para usar Power BI para consultar y visualizar los datos de Azure Data Explorer.
author: orspod
ms.author: orspodek
ms.reviewer: gabil
ms.service: data-explorer
ms.topic: conceptual
ms.date: 09/26/2019
ms.openlocfilehash: 5c5c27afc2c6fa0ee337d336b43222f5ea6583e8
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81494444"
---
# <a name="best-practices-for-using-power-bi-to-query-and-visualize-azure-data-explorer-data"></a>Procedimientos recomendados para usar Power BI para consultar y visualizar datos de Azure Data Explorer

El Explorador de datos de Azure es un servicio de exploración de datos altamente escalable y rápido para datos de telemetría y registro. [Power BI](https://docs.microsoft.com/power-bi/) es una solución de análisis de negocios que le permite visualizar sus datos y compartir los resultados en su organización. Azure Data Explorer ofrece tres opciones para conectarse a los datos en Power BI. Use el [conector integrado](power-bi-connector.md), [importe una consulta de Azure Data Explorer a Power BI](power-bi-imported-query.md) o use una [consulta SQL](power-bi-sql-query.md). En este artículo se proporcionan sugerencias para consultar y visualizar los datos de Azure Data Explorer con Power BI. 

## <a name="best-practices-for-using-power-bi"></a>Procedimientos recomendados para usar Power BI 

Al trabajar con terabytes de datos sin procesar actualizados, siga estas instrucciones para mantener ajustados y actualizados los informes y paneles de Power BI:

* **Viaje ligero**: lleve a Power BI solo los datos que necesita para sus informes. Para hacer un análisis interactivo en profundidad, use la [interfaz de usuario web de Azure Data Explorer](web-query-data.md) que está optimizada para una exploración ad-hoc con el lenguaje de consulta Kusto.

* **Modelo compuesto**: use el [modelo compuesto](https://docs.microsoft.com/power-bi/desktop-composite-models) para combinar datos agregados para paneles de nivel superior con datos operativos filtrados sin procesar. Puede definir claramente cuándo usar datos sin procesar y cuándo usar una vista agregada. 

* **Modo de importación frente al modo DirectQuery**: use el modo de **importación** para la interacción de conjuntos de datos más pequeños. Use el modo **DirectQuery** para conjuntos de datos de gran tamaño que se actualizan con frecuencia. Por ejemplo, cree tablas de dimensiones con el modo de **importación**, ya que son pequeñas y no cambian con frecuencia. Establezca el intervalo de actualización en función de la tasa de actualizaciones de datos esperada. Cree tablas de hechos con el modo **DirectQuery**, ya que estas tablas son de gran tamaño y contienen datos sin procesar. Use estas tablas para presentar datos filtrados con la [obtención de detalles](https://docs.microsoft.com/power-bi/desktop-drillthrough) de Power BI.

* **Paralelismo**: Azure Data Explorer es una plataforma de datos que se puede escalar de manera lineal; por lo tanto, puede mejorar el rendimiento de la representación del panel aumentando el paralelismo del flujo completo, tal y como se muestra a continuación:

   * Aumente el número de [conexiones simultáneas en DirectQuery en Power BI](https://docs.microsoft.com/power-bi/desktop-directquery-about#maximum-number-of-connections-option-for-directquery).

   * Use la [coherencia débil para mejorar el paralelismo](kusto/concepts/queryconsistency.md). Esto puede afectar la actualización de los datos.

* **Segmentaciones eficaces**: puede usar las [segmentaciones de sincronización](https://docs.microsoft.com/power-bi/visuals/power-bi-visualization-slicers#sync-and-use-slicers-on-other-pages) para impedir que los informes carguen datos antes de que todo esté preparado. Una vez que estructure el conjunto de datos, coloque todos los objetos visuales y marque todas las segmentaciones, puede seleccionar la segmentación de sincronización para cargar solo los datos necesarios.

* **Uso de filtros**: use tantos filtros de Power BI como sea posible para centrar la búsqueda de Azure Data Explorer en las particiones de datos pertinentes.

* **Objetos visuales eficaces**: seleccione los objetos visuales con mayor rendimiento para sus datos.

## <a name="tips-for-using-the-azure-data-explorer-connector-for-power-bi-to-query-data"></a>Sugerencias para usar el conector de Azure Data Explorer para Power BI para consultar datos

En la sección siguiente se incluyen sugerencias y trucos para usar el lenguaje de consulta Kusto con Power BI. Uso del [conector de Azure Data Explorer para Power BI](power-bi-connector.md) para visualizar datos

### <a name="complex-queries-in-power-bi"></a>Consultas complejas en Power BI

Las consultas complejas se expresan más fácilmente en Kusto que en Power Query. Deben implementarse como [funciones de Kusto](kusto/query/functions/index.md) y se invocan en Power BI. Este método es necesario cuando **DirectQuery** se usa con instrucciones `let` en la consulta Kusto. Como Power BI combina dos consultas y las instrucciones `let` no se pueden usar con el operador `join`, pueden producirse errores de sintaxis. Por lo tanto, guarde cada parte de la combinación como una función Kusto y permita que Power BI combine estas dos funciones.

### <a name="how-to-simulate-a-relative-date-time-operator"></a>Simulación de un operador relativo de fecha y hora

Power BI no contiene un operador *relativo* de fecha y hora como `ago()`.
Para simuar `ago()`, use una combinación de funciones `DateTime.FixedLocalNow()` y `#duration` de Power BI.

En lugar de esta consulta con el operador `ago()`:

```kusto
    StormEvents | where StartTime > (now()-5d)
    StormEvents | where StartTime > ago(5d)
``` 

Use la consulta equivalente siguiente:

```powerquery-m
let
    Source = Kusto.Contents("help", "Samples", "StormEvents", []),
    #"Filtered Rows" = Table.SelectRows(Source, each [StartTime] > (DateTime.FixedLocalNow()-#duration(5,0,0,0)))
in
    #"Filtered Rows"
```

### <a name="reaching-kusto-query-limits"></a>Alcance de los límites de la consulta Kusto 

De manera predeterminada, las consultas Kusto devuelven hasta 500 000 filas o 64 MB, como se describe en los [límites de consulta](kusto/concepts/querylimits.md). Para reemplazar estos valores predeterminados, use las **opciones avanzadas** en la venta de conexión de **Azure Data Explorer (Kusto)** :

![opciones avanzadas](media/power-bi-best-practices/advanced-options.png)

Estas opciones emiten [instrucciones SET](kusto/query/setstatement.md) con la consulta para cambiar los límites de consulta predeterminados:

  * **Limitar el número de registros de los resultados de la consulta** genera un `set truncationmaxrecords`
  * **Limitar el tamaño de los datos de los resultados de la consulta en Bytes** genera un `set truncationmaxsize`
  * **Deshabilitar el truncamiento del conjunto de resultados** genera un `set notruncation`

### <a name="using-query-parameters"></a>Uso de los parámetros de consulta

Puede usar los [parámetros de consulta](kusto/query/queryparametersstatement.md) para modificar la consulta de manera dinámica. 

#### <a name="using-a-query-parameter-in-the-connection-details"></a>Uso de un parámetro de consulta en los detalles de conexión

Use un parámetro de consulta para filtrar la información de la consulta y optimizar el rendimiento de las consultas.
 
En la ventana **Editar consultas**, **Inicio** > **Editor avanzado**

1. Busque la sección siguiente de la consulta:

    ```powerquery-m
    Source = Kusto.Contents("<Cluster>", "<Database>", "<Query>", [])
    ```
   
   Por ejemplo:

    ```powerquery-m
    Source = Kusto.Contents("Help", "Samples", "StormEvents | where State == 'ALABAMA' | take 100", [])
    ```

1. Reemplace la parte pertinente de la consulta por su parámetro. Divida la consulta en varias partes y vuelva a concatenarlas utilizando el signo & junto con el parámetro.

   Por ejemplo, en la consulta anterior, tomaremos la parte `State == 'ALABAMA'` y la dividiremos en `State == '` y `'` y colocaremos el parámetro `State` entre ellas:
   
    ```kusto
    "StormEvents | where State == '" & State & "' | take 100"
    ```

1. Si la consulta contiene comillas, codifíquelas correctamente. Por ejemplo, la consulta siguiente: 

   ```kusto
   "StormEvents | where State == "ALABAMA" | take 100" 
   ```

   aparecerá en el **Editor avanzado** como se muestra a continuación con dos comillas:

   ```kusto
    "StormEvents | where State == ""ALABAMA"" | take 100"
   ```

   Se debe reemplazar por la consulta siguiente con tres comillas:

   ```kusto
   "StormEvents | where State == """ & State & """ | take 100"
   ```

#### <a name="use-a-query-parameter-in-the-query-steps"></a>Uso de un parámetro de consulta en los pasos de la consulta

Puede usar un parámetro de consulta en cualquier paso de la consulta que lo admita. Por ejemplo, filtre los resultados en función del valor de un parámetro.

![filtrar los resultados con un parámetro](media/power-bi-best-practices/filter-using-parameter.png)

### <a name="dont-use-power-bi-data-refresh-scheduler-to-issue-control-commands-to-kusto"></a>No use el programador de actualización de Power BI para emitir comandos de control a Kusto

Power BI incluye un programador de actualización de datos que puede emitir consultas periódicamente en un origen de datos. Este mecanismo no debe usarse para programar comandos de control en Kusto, ya que Power BI supone que todas las consultas son de solo lectura.

### <a name="power-bi-can-send-only-short-lt2000-characters-queries-to-kusto"></a>Power BI solo puede enviar consultas cortas (&lt;2000 caracteres) a Kusto

Si se ejecuta una consulta en Power BI, se produce el error siguiente: _"DataSource.Error: Web.Contents failed to get contents from..."_ , es probable que la consulta tenga más de 2000 caracteres. Power BI usa **PowerQuery** para consultar a Kusto mediante la emisión de una solicitud HTTP GET que codifica la consulta como parte del URI que se va a recuperar. Por lo tanto, las consultas de Kusto que emite Power BI se limitan a la longitud máxima de un URI de solicitud (2000 caracteres, menos el desplazamiento pequeño). Como solución alternativa, puede definir una [función almacenada](kusto/query/schema-entities/stored-functions.md) en Kusto y haga que Power BI use esa función en la consulta.

## <a name="next-steps"></a>Pasos siguientes

[Visualización de datos mediante el conector de Azure Data Explorer para Power BI](power-bi-connector.md)




