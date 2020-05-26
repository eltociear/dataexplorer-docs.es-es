---
title: Supervisión del rendimiento, el estado y el uso de Azure Data Explorer con métricas
description: Aprenda a usar métricas de Azure Data Explorer para supervisar el rendimiento, el mantenimiento y el uso del clúster.
author: orspod
ms.author: orspodek
ms.reviewer: gabil
ms.service: data-explorer
ms.topic: conceptual
ms.date: 01/19/2020
ms.openlocfilehash: ffe641c6e96db1378be7a8ce845685fb2b66715f
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83374307"
---
# <a name="monitor-azure-data-explorer-performance-health-and-usage-with-metrics"></a>Supervisión del rendimiento, el mantenimiento y el uso de Azure Data Explorer con métricas

Azure Data Explorer es un servicio de análisis de datos rápido y totalmente administrado para analizar en tiempo real grandes volúmenes de datos de que se transmiten desde aplicaciones, sitios web, dispositivos IoT, etc. Para usar Azure Data Explorer, cree primero un clúster y una o varias bases de datos en ese clúster. A continuación, ingerirá (cargará) los datos en una base de datos para que pueda ejecutar consultas en ella. Las métricas de Azure Data Explorer proporcionan indicadores clave sobre el mantenimiento y el rendimiento de los recursos del clúster. Use las métricas que se detallan en este artículo para supervisar el mantenimiento y el rendimiento del clúster de Azure Data Explorer en su escenario específico como métricas independientes. También puede usar como métricas como base operacional para los [paneles de Azure](/azure/azure-portal/azure-portal-dashboards) y las [alertas de Azure](/azure/azure-monitor/platform/alerts-metric-overview).

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción a Azure. En caso de no tener ninguna, puede crear una [cuenta gratuita de Azure](https://azure.microsoft.com/free/).
* Un [clúster y una base de datos](create-cluster-database-portal.md).

## <a name="using-metrics"></a>Uso de métricas

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
1. En el clúster de Azure Data Explorer, seleccione **Métricas** para abrir el panel de métricas y comenzar el análisis del clúster.
    ![Seleccione Métricas](media/using-metrics/select-metrics.png).
1. En el panel de métricas:  ![Panel de métricas](media/using-metrics/metrics-pane.png)
    1. Para crear un gráfico de métricas, seleccione el nombre de la **métrica** y la **agregación** de interés por métrica. Los selectores **Recurso** y **Espacio de nombres de métricas** están seleccionados previamente para el clúster de Azure Data Explorer. Para más información sobre las distintas métricas, consulte [Métricas que admite Azure Data Explorer](#supported-azure-data-explorer-metrics).
    1. Seleccione **Agregar métrica** para ver varias métricas trazadas en el mismo gráfico.
    1. Seleccione **+ Nuevo gráfico** para ver varios gráficos en una vista.
    1. Use el selector de tiempo para cambiar el intervalo de tiempo (valor predeterminado: últimas 24 horas).
    1. Use [**Agregar filtro** y **Apply splitting**](/azure/azure-monitor/platform/metrics-getting-started#apply-dimension-filters-and-splitting) (Aplicar separación) con métricas que tengan dimensiones.
    1. Seleccione **Anclar al panel** para agregar la configuración del gráfico a los paneles para que pueda verla de nuevo.
    1. Establezca **Nueva regla de alertas** para visualizar las métricas mediante los criterios establecidos. La nueva regla de alerta incluirá su recurso de destino, métrica, división y dimensiones de filtro del gráfico. Modifique esta configuración en el [panel de creación de reglas de alerta](/azure/azure-monitor/platform/metrics-charts#create-alert-rules).

Información adicional sobre el uso del [Explorador de métricas](/azure/azure-monitor/platform/metrics-getting-started).

## <a name="supported-azure-data-explorer-metrics"></a>Métricas que admite Azure Data Explorer

Las métricas que admite Azure Data Explorer se dividen en varias categorías en función del uso. 

### <a name="cluster-health-metrics"></a>Métricas de mantenimiento de clúster

Las métricas de mantenimiento del clúster realizan el seguimiento del estado general del clúster. Esto incluye la capacidad de respuesta, y el uso de recursos y de la ingesta.

**Métrica** | **Unidad** | **Agregación** | **Descripción de la métrica** | **Dimensiones** |
|---|---|---|---|---|
| Uso de la caché | Percent | Media, máxima, mínima | Porcentaje de recursos de caché asignados actualmente en uso por el clúster. La memoria caché tiene el tamaño de SSD asignado para la actividad del usuario según la directiva de caché definida. Una utilización media de la caché del 80 % o menos es un estado sostenible para un clúster. Si el porcentaje es superior, el clúster se debe [escalar verticalmente](manage-cluster-vertical-scaling.md) hasta un plan de tarifa optimizado para almacenamiento, u [horizontalmente](manage-cluster-horizontal-scaling.md) a más instancias. Como alternativa, adapte la directiva de caché (menos días en caché). Si la utilización de la caché es superior al 100 %, el tamaño de los datos para almacenarse en caché, según la directiva de almacenamiento en caché, es superior al tamaño total de la caché del clúster. | None |
| CPU | Percent | Media, máxima, mínima | Porcentaje de recursos de proceso asignados actualmente en uso por las máquinas del clúster. Una CPU media del 80 % o menos es sostenible para un clúster. El valor máximo de CPU es 100 %, lo que significa que no hay otros recursos de proceso para procesar los datos. Cuando un clúster funciona bien, compruebe el valor máximo de la CPU para determinar si existen CPU específicas que están bloqueadas. | None |
| Uso de la ingesta | Percent | Media, máxima, mínima | Porcentaje de recursos reales usados para ingerir datos de los recursos totales asignados en la directiva de capacidad para realizar la ingesta. La directiva de capacidad predeterminada es no más de 512 operaciones simultáneas de ingesta o el 75 % de los recursos del clúster invertidos en la ingesta. La utilización media de la ingesta de un 80 % o menos es un estado sostenible para un clúster. El valor máximo de utilización de la ingesta es 100 %, lo que significa que se usa toda la capacidad de ingesta del clúster y, como resultado, se puede generar una cola de ingesta. | None |
| Mantener conexión | Count | Avg | Realiza un seguimiento de la capacidad de respuesta del clúster. Un clúster que responde totalmente devuelve el valor 1 y un clúster desconectado o bloqueado devuelve 0. |
| Número total de comandos limitados | Count | Media, máxima, mínima | Número de comandos limitados (rechazados) en el clúster, ya que se alcanzó el número máximo permitido de comandos simultáneos (en paralelo). | None |
| Número total de extensiones | Count | Media, máxima, mínima | Número total de extensiones de datos del clúster. Los cambios en esta métrica pueden implicar cambios masivos en la estructura de datos y una carga elevada en el clúster, ya que la combinación de extensiones de datos es una actividad que consume gran cantidad de CPU. | None |
| | | | |

### <a name="export-health-and-performance-metrics"></a>Métricas de mantenimiento y rendimiento de la exportación

Las métricas de mantenimiento y rendimiento de la exportación realizan un seguimiento del estado general y del rendimiento de las operaciones de exportación, como la demora, los resultados, el número de registros y el uso.

**Métrica** | **Unidad** | **Agregación** | **Descripción de la métrica** | **Dimensiones** |
|---|---|---|---|---|
Continuous export number of exported records (Exportación continua: número de registros exportados)    | Count | Sum | Número de registros exportados en todos los trabajos de exportación continua. | None |
Continuous export max lateness (Máx. de demora en la exportación continua) |    Count   | Max   | Demora (en minutos) indicada por los trabajos de exportación continua del clúster. | None |
Continuous export pending count (Recuento pendiente de exportaciones continuas) | Count | Max   | Número de trabajos de exportación continua pendientes. Estos trabajos están listos para ejecutarse pero esperan en una cola, posiblemente debido a la capacidad insuficiente. 
Continuous export result (Resultado de la exportación continua)    | Count |   Count   | Resultado de error o correcto de cada ejecución de exportación continua. | ContinuousExportName |
Utilización de la exportación |    Percent | Max   | Capacidad de exportación usada, de la capacidad de exportación total del clúster (entre 0 y 100). | None |
| | | | |

### <a name="ingestion-health-and-performance-metrics"></a>Métricas de mantenimiento y rendimiento de la ingesta

Las métricas de mantenimiento y rendimiento de la ingesta realizan un seguimiento del estado general y del rendimiento de las operaciones de ingesta, como la demora, los resultados y el volumen.

**Métrica** | **Unidad** | **Agregación** | **Descripción de la métrica** | **Dimensiones** |
|---|---|---|---|---|
| Events processed (for Event/IoT Hubs) [Eventos procesados (para instancias de Event Hubs o IoT Hub)] | Count | Máximo, mínimo, suma | Número total de eventos leídos de los centros de eventos y procesados por el clúster. Los eventos se dividen en eventos rechazados y aceptados por el motor del clúster. | EventStatus |
| Latencia de la ingesta | Segundos | Media, máxima, mínima | La latencia de datos ingeridos desde el momento en que se recibieron los datos en el clúster hasta que está preparado para la consulta. El periodo de latencia de ingesta depende del escenario de ingesta. | None |
| Resultado de la ingesta | Count | Count | Número total de operaciones de ingesta correctas e incorrectas. Use **Apply splitting** (Aplicar separación) crear depósitos de resultados correctos e incorrectos y analizar las dimensiones (**Valor** > **Estado**).| IngestionResultDetails |
| Volumen de ingesta (en MB) | Count | Máxima, suma | El tamaño total de datos ingeridos en el clúster (en MB) antes de la compresión. | Base de datos |
| | | | |  

### <a name="query-performance"></a>Rendimiento de las consultas

Las métricas de rendimiento de las consultas realizan un seguimiento de la duración de la consulta y del total de consultas simultáneas o limitadas.

**Métrica** | **Unidad** | **Agregación** | **Descripción de la métrica** | **Dimensiones** |
|---|---|---|---|---|
| Duración de la consulta | Milisegundos | Media, mínima, máxima, suma | Tiempo total hasta que se reciben los resultados de la consulta (no incluye la latencia de red). | QueryStatus |
| Número total de consultas simultáneas | Count | Media, máxima, mínima | Número de consultas que se ejecutan en paralelo en el clúster. Esta métrica es una buena manera de calcular la carga en el clúster. | None |
| Número total de consultas limitadas | Count | Media, máxima, mínima | Número de consultas limitadas (rechazadas) en el clúster. El número máximo permitido de consultas simultáneas (en paralelo) se define en la directiva de consulta simultánea. | None |
| | | | |

### <a name="streaming-ingest-metrics"></a>Métricas de la ingesta de streaming

Las métricas de la ingesta de streaming realizan un seguimiento de los datos de ingesta de streaming y de la tasa de solicitudes, la duración y los resultados.

**Métrica** | **Unidad** | **Agregación** | **Descripción de la métrica** | **Dimensiones** |
|---|---|---|---|---|
Streaming Ingest Data Rate (Velocidad de datos de la ingesta de streaming) |    Count   | RateRequestsPerSecond | Volumen total de datos ingeridos en el clúster. | None |
Streaming Ingest Duration (Duración de la ingesta de streaming)   | Milisegundos  | Media, máxima, mínima | Duración total de las solicitudes de ingesta de streaming. | None |
Streaming Ingest Request Rate [Velocidad de solicitudes de ingesta de streaming]   | Count | Recuento, media, máxima, mínima, suma | Total de solicitudes de ingesta de streaming. | None |
Streaming Ingest Result (Resultado de la ingesta de streaming) | Count | Avg   | Total de solicitudes de ingesta de streaming por tipo de resultado. | Resultado |
| | | | |

Información adicional sobre las [métricas del clúster de Azure Data Explorer admitidas](/azure/azure-monitor/platform/metrics-supported#microsoftkustoclusters).


## <a name="next-steps"></a>Pasos siguientes

* [Tutorial: Ingesta y consulta de datos de supervisión en Azure Data Explorer](ingest-data-no-code.md)
* [Supervisión de las operaciones de ingesta de Azure Data Explorer mediante registros de diagnóstico](using-diagnostic-logs.md)
* [Inicio rápido: Consulta de datos en Azure Data Explorer](web-query-data.md)
