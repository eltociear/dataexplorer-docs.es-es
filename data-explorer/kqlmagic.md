---
title: Uso de Jupyter Notebook para analizar datos en Azure Data Explorer
description: En este tema se muestra cómo analizar datos en Azure Data Explorer mediante una instancia de Jupyter Notebook y la extensión Kqlmagic.
author: orspod
ms.author: orspodek
ms.reviewer: mblythe
ms.service: data-explorer
ms.topic: conceptual
ms.date: 07/10/2019
ms.openlocfilehash: 33fe6750c355c8dc79bbbd9223166786f844a502
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81492832"
---
# <a name="use-a-jupyter-notebook-and-kqlmagic-extension-to-analyze-data-in-azure-data-explorer"></a>Uso de Jupyter Notebook y una extensión de Kqlmagic para analizar datos en Azure Data Explorer

Jupyter Notebook es una aplicación web de código abierto que le permite crear y compartir documentos que contengan código de producción, ecuaciones, visualizaciones y texto narrativo. En el uso, se incluye la transformación y limpieza de datos, simulación numérica, modelado estadístico, visualización de datos y aprendizaje automático.
[Jupyter Notebook](https://jupyter.org/) admite varias funciones mágicas que amplían las funciones del kernel al admitir comandos adicionales. KQL magic es un comando que amplía las funciones del kernel de Python en Jupyter Notebook para que pueda ejecutar consultas del lenguaje de consulta Kusto de forma nativa. Puede combinar fácilmente el lenguaje de consulta Kusto con Python para consultar y visualizar datos mediante la biblioteca de Plot.ly enriquecida integrada con comandos de `render`. Se admiten orígenes de datos para la ejecución de consultas. Entre estos orígenes de datos, se incluye Azure Data Explorer, un servicio de exploración de datos rápido y altamente escalable para datos de telemetría y registro, así como registros de Azure Monitor y Application Insights. KQL magic también funciona con Azure Notebooks, Jupyter Lab y la extensión Jupyter de Visual Studio Code.

## <a name="prerequisites"></a>Prerrequisitos

- Una cuenta de correo electrónico organizativa que pertenezca a Azure Active Directory (AAD).
- Jupyter Notebook instalado en el equipo local o usar Azure Notebooks y clonar la instancia de ejemplo de [Azure Notebooks](https://kustomagicsamples-manojraheja.notebooks.azure.com/j/notebooks/Getting%20Started%20with%20kqlmagic%20on%20Azure%20Data%20Explorer.ipynb).

## <a name="install-kql-magic-library"></a>Instalación de la biblioteca KQL magic

1. Instale KQL magic:

    ```python
    !pip install Kqlmagic --no-cache-dir  --upgrade
    ```
    > [!NOTE]
    > Al usar Azure Notebooks, este paso no es necesario.

1. Cargue KQL magic:

    ```python
    %reload_ext Kqlmagic
    ```
    > [!NOTE]
    > Cambie la versión del kernel a Python 3.6; para ello, haga clic en Kernel > Change Kernel > Python 3.6 (Kernel > Cambiar kernel > Python 3.6).
    
## <a name="connect-to-the-azure-data-explorer-help-cluster"></a>Conectarse al clúster de ayuda de Azure Data Explorer

Use el comando siguiente para conectarse a la base de datos *Samples* (Ejemplos) hospedada en el clúster *Help* (Ayuda). Para usuarios de un entorno de AAD distinto de Microsoft, reemplace el nombre del inquilino `Microsoft.com` por su inquilino de AAD.

```python
%kql AzureDataExplorer://tenant="Microsoft.com";code;cluster='help';database='Samples'
```

## <a name="query-and-visualize"></a>Consultar y visualizar

Consulte datos mediante el [operador “render”](kusto/query/renderoperator.md) y visualice datos mediante la biblioteca de ploy.ly. Esta consulta y visualización proporciona una experiencia integrada que usa KQL de forma nativa. Kqlmagic admite la mayoría de los gráficos, excepto `timepivot`, `pivotchart` y `ladderchart`. La representación se admite con todos los atributos, excepto `kind`, `ysplit` y `accumulate`. 

### <a name="query-and-render-piechart"></a>Consultar y representar un gráfico circular

```python
%%kql
StormEvents
| summarize statecount=count() by State
| sort by statecount 
| limit 10
| render piechart title="My Pie Chart by State"
```

### <a name="query-and-render-timechart"></a>Consultar y representar un gráfico de tiempo

```python
%%kql
StormEvents
| summarize count() by bin(StartTime,7d)
| render timechart
```

> [!NOTE]
> Los gráficos son interactivos. Seleccione un intervalo de tiempo para explorar un momento específico.

### <a name="customize-the-chart-colors"></a>Personalizar los colores del gráfico

Si no le gusta la paleta de colores predeterminada, puede personalizar los gráficos con opciones de paleta. Aquí encontrará las paletas disponibles: [Selección de la paleta de colores para el resultado del gráfico de la consulta de KQL magic](https://mybinder.org/v2/gh/Microsoft/jupyter-Kqlmagic/master?filepath=notebooks%2FColorYourCharts.ipynb)

1. Para obtener una lista de paleta:

    ```python
    %kql --palettes -popup_window
    ```

1. Seleccione la paleta de colores `cool` y vuelva a representar la consulta:

    ```python
    %%kql -palette_name "cool"
    StormEvents
    | summarize statecount=count() by State
    | sort by statecount
    | limit 10
    | render piechart title="My Pie Chart by State"
    ```

## <a name="parameterize-a-query-with-python"></a>Parametrizar una consulta con Python

KQL magic permite un intercambio sencillo entre Python y el lenguaje de consulta Kusto. Para obtener más información: [Parametrización de una consulta de KQL magic con Python](https://mybinder.org/v2/gh/Microsoft/jupyter-Kqlmagic/master?filepath=notebooks%2FParametrizeYourQuery.ipynb)

### <a name="use-a-python-variable-in-your-kql-query"></a>Usar una variable de Python en una consulta KQL

Puede usar el valor de una variable de Python en una consulta para filtrar los datos:

```python
statefilter = ["TEXAS", "KANSAS"]
```

```python
%%kql
let _state = statefilter;
StormEvents 
| where State in (_state) 
| summarize statecount=count() by bin(StartTime,1d), State
| render timechart title = "Trend"
```

### <a name="convert-query-results-to-pandas-dataframe"></a>Convertir resultados de la consulta en Pandas DataFrame

Puede acceder a los resultados de una consulta KQL en Pandas DataFrame. Accederá a los resultados de la última consulta ejecutada mediante la variable `_kql_raw_result_` y podrá convertir fácilmente los resultados en Pandas DataFrame, como se indica a continuación:

```python
df = _kql_raw_result_.to_dataframe()
df.head(10)
```

### <a name="example"></a>Ejemplo

En muchos escenarios de análisis, puede que quiera crear blocs de notas reutilizables que contengan un gran número de consultas y pasar los resultados de una consulta a consultas posteriores. En el ejemplo siguiente, se usa la variable de Python `statefilter` para filtrar los datos.

1. Ejecute la consulta para ver los 10 estados principales con un máximo de `DamageProperty`:

    ```python
    %%kql
    StormEvents
    | summarize max(DamageProperty) by State
    | order by max_DamageProperty desc
    | limit 10
    ```

1. Ejecute una consulta para extraer el estado principal y establecerlo en una variable de Python:

    ```python
    df = _kql_raw_result_.to_dataframe()
    statefilter =df.loc[0].State
    statefilter
    ```

1. Ejecute una consulta con la instrucción `let` y la variable de Python:

    ```python
    %%kql
    let _state = statefilter;
    StormEvents 
    | where State in (_state)
    | summarize statecount=count() by bin(StartTime,1d), State
    | render timechart title = "Trend"
    ```

1. Ejecute el comando de ayuda:

    ```python
    %kql --help "help"
    ```

> [!TIP]
> Para recibir información sobre todas las configuraciones disponibles, use `%config Kqlmagic`. Para solucionar problemas y capturar errores de Kusto, como problemas de conexión y consultas incorrectas, use `%config Kqlmagic.short_errors=False`.

## <a name="next-steps"></a>Pasos siguientes

Ejecute el comando de ayuda para explorar los siguientes blocs de notas de ejemplo, que contienen todas las características admitidas:
- [Introducción a KQL magic para Azure Data Explorer](https://mybinder.org/v2/gh/Microsoft/jupyter-Kqlmagic/master?filepath=notebooks%2FQuickStart.ipynb) 
- [Introducción a KQL magic para Application Insights](https://mybinder.org/v2/gh/Microsoft/jupyter-Kqlmagic/master?filepath=notebooks%2FQuickStartAI.ipynb) 
- [Get started with KQL magic for Azure Monitor logs](https://mybinder.org/v2/gh/Microsoft/jupyter-Kqlmagic/master?filepath=notebooks%2FQuickStartLA.ipynb) (Introducción a KQL magic para registros de Azure Monitor) 
- [Parametrizar una consulta de KQL magic con Python](https://mybinder.org/v2/gh/Microsoft/jupyter-Kqlmagic/master?filepath=notebooks%2FParametrizeYourQuery.ipynb) 
- [Selección de la paleta de colores para el resultado del gráfico de la consulta de KQL magic](https://mybinder.org/v2/gh/Microsoft/jupyter-Kqlmagic/master?filepath=notebooks%2FColorYourCharts.ipynb)
