---
title: Administración del escalado horizontal del clúster para satisfacer la demanda en Azure Data Explorer
description: En este artículo se describen los pasos para escalar y reducir horizontalmente un clúster de Azure Data Explorer en función de los cambios en la demanda.
author: orspod
ms.author: orspodek
ms.reviewer: gabil
ms.service: data-explorer
ms.topic: conceptual
ms.date: 12/09/2019
ms.openlocfilehash: 18bd73797070ba569d1a43a8c66ab92526682f36
ms.sourcegitcommit: 1faf502280ebda268cdfbeec2e8ef3d582dfc23e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2020
ms.locfileid: "82619166"
---
# <a name="manage-cluster-horizontal-scaling-scale-out-in-azure-data-explorer-to-accommodate-changing-demand"></a>Administración del escalado horizontal de clústeres en Azure Data Explorer para ajustarse a los cambios en la demanda

Ajustar el tamaño de un clúster de forma adecuada es fundamental para el rendimiento del Explorador de datos de Azure. El tamaño de un clúster estático puede provocar una infrautilización o sobreutilización, y ninguna de estas situaciones es la ideal. Dado que la demanda en un clúster no se puede predecir con precisión absoluta, es mejor *escalar* el clúster mediante la adición y la eliminación de recursos de CPU y capacidad según los cambios en la demanda. 

Existen dos flujos de trabajo de escalado para un clúster de Azure Data Explorer: 
* Escalado horizontal, también denominado reducción horizontal.
* [Escalado vertical](manage-cluster-vertical-scaling.md), también denominado reducción vertical.
En este artículo se explica el flujo de trabajo del escalado horizontal.

## <a name="configure-horizontal-scaling"></a>Configuración del escalado horizontal

El escalado horizontal permite escalar el recuento de instancias automáticamente en función de programaciones y reglas predefinidas. Para especificar la configuración de escalabilidad automática del clúster:

1. En Azure Portal, vaya al recurso de clúster de Azure Data Explorer. En **Configuración**, seleccione **Escalar horizontalmente**. 

2. En la ventana **Escalar horizontalmente**, seleccione el método de escalado automático que quiera: **Escala manual**, **Optimized autoscale** (Escalabilidad automática optimizada) o **Custom autoscale** (Escalabilidad automática personalizada).

### <a name="manual-scale"></a>Escala manual

La escala manual es la configuración predeterminada durante la creación del clúster. El clúster tiene una capacidad estática que no cambia automáticamente. Para seleccionar la capacidad estática, use la barra de **Recuento de instancias**. El escalado del clúster se mantiene en ese valor hasta que se realiza otro cambio.

   ![Método de escala manual](media/manage-cluster-horizontal-scaling/manual-scale-method.png)

### <a name="optimized-autoscale-preview"></a>Escalabilidad automática optimizada (versión preliminar)

La escalabilidad automática optimizada es el método recomendado de escalabilidad automática. Este método optimiza el rendimiento y los costes del clúster. Si el clúster se acerca a un estado de infrautilización, se reducirá horizontalmente. Esta acción reduce los costes, pero mantiene el nivel de rendimiento. Si el clúster se acerca a un estado de sobreutilización, se escalará horizontalmente para mantener un rendimiento óptimo. Para configurar la escalabilidad automática optimizada:

1. Seleccione **Optimized autoscale** (Escalabilidad automática optimizada). 

1. Seleccione un recuento de instancias mínimo y máximo. El escalado automático del clúster oscila entre estas dos cifras, en función de la carga.

1. Seleccione **Guardar**.

   ![Método de escalabilidad automática optimizada](media/manage-cluster-horizontal-scaling/optimized-autoscale-method.png)

El escalado automático optimizado empieza a funcionar. Sus acciones ahora están visibles en el registro de actividad de Azure del clúster.

#### <a name="logic-of-optimized-autoscale"></a>Lógica de la escalabilidad automática optimizada 

**Escalado horizontal**

Cuando el clúster se acerca a un estado de sobreutilización, escale horizontalmente para mantener un rendimiento óptimo. La escalabilidad horizontal se producirá cuando:
* El número de instancias de clúster esté por debajo del número máximo de instancias definido por el usuario.
* La utilización de la memoria caché sea alta durante más de una hora.
* La utilización de la CPU sea alta durante más de una hora.
* La utilización de la ingesta sea alta durante más de una hora.


**Reducción horizontal**

Cuando el clúster se acerque a un estado de infrautilización, reduzca horizontalmente para reducir los costos, pero mantener el rendimiento. Se usan varias métricas para comprobar que es seguro reducir horizontalmente en el clúster. Las reglas siguientes se evalúan cada hora durante 6 horas antes de que se realice la reducción horizontal:
* El número de instancias es superior a 2 y al número mínimo de instancias definidas.
* Para garantizar que no haya ninguna sobrecarga en los recursos, deben comprobarse las siguientes métricas antes de llevar a cabo la reducción horizontal: 
    * El uso de la memoria caché no es alto
    * La CPU se usa por debajo del promedio 
    * La utilización de la ingesta está por debajo del promedio 
    * La utilización de ingesta de streaming (si se da el caso) no es alta
    * Los eventos de conexión persistente están por encima de un valor mínimo definido, se procesan correctamente y a tiempo.
    * No existe limitación de consultas 
    * El número de consultas con errores es inferior al mínimo definido.

> [!NOTE]
> La lógica de reducción horizontal requiere actualmente una evaluación de 1 día antes de la implementación de la reducción horizontal optimizada. Esta evaluación tiene lugar una vez cada 6 horas. Si se necesita un cambio inmediato, use el [escalado manual](#manual-scale).

### <a name="custom-autoscale"></a>Escalabilidad automática personalizada

Al usar la escalabilidad automática personalizada, puede escalar dinámicamente el clúster en función de las métricas que especifique. En el gráfico siguiente se muestran el flujo y los pasos para configurar la escalabilidad automática personalizada. Más detalles después del gráfico.

1. En el cuadro **Nombre del ajuste de escalabilidad automática**, escriba un nombre, por ejemplo, *Scale-out: cache utilization* (Escalado horizontal: utilización de caché). 

   ![Regla de escalado](media/manage-cluster-horizontal-scaling/custom-autoscale-method.png)

2. En **Modo de escala**, seleccione **Escalado basado en una métrica**. Este modo proporciona escalado dinámico. También puede seleccionar **Escalar a un número específico de instancias**.

3. Seleccione **+Agregar una regla**.

4. En la sección **Escalar regla** de la derecha, proporcione valores para cada opción.

    **Criterios**

    | Configuración | Descripción y valor |
    | --- | --- |
    | **Agregación de tiempo** | Seleccione un criterio de agregación, como **Promedio**. |
    | **Nombre de la métrica** | Seleccione la métrica en la que quiera que se base la operación de escalado, como **Cache Utilization**. |
    | **Estadísticas de intervalo de agregación** | Elija entre **Promedio**, **Mínimo**, **Máximo** y **Suma**. |
    | **Operador** | Elija la opción adecuada, como **Mayor o igual que**. |
    | **Umbral** | Elija un valor apropiado. Por ejemplo, para el uso de memoria caché, el 80 % es un buen punto inicial. |
    | **Duración (en minutos)** | Elija un período apropiado para que el sistema busque al calcular las métricas. Comience con el valor predeterminado de 10 minutos. |
    |  |  |

    **Acción**

    | Configuración | Descripción y valor |
    | --- | --- |
    | **operación** | Elija la opción adecuada para reducir o escalar horizontalmente. |
    | **Recuento de instancias** | Elija el número de nodos o instancias que quiere agregar o quitar cuando se cumpla una condición de métrica. |
    | **Tiempo de finalización (minutos)** | Elija un intervalo de tiempo adecuado para esperar entre las operaciones de escalado. Comience con el valor predeterminado de cinco minutos. |
    |  |  |

5. Seleccione **Agregar**.

6. En la sección **Límites de instancia** de la izquierda, proporcione valores para cada opción.

    | Configuración | Descripción y valor |
    | --- | --- |
    | **Mínimo** | Número de instancias por debajo del cual el clúster no se escalará, independientemente del uso. |
    | **Máximo** | Número de instancias por encima del cual el clúster no se escalará, independientemente del uso. |
    | **Valor predeterminado** | El número predeterminado de instancias. Esta configuración se utiliza si hay problemas con la lectura de las métricas de recursos. |
    |  |  |

7. Seleccione **Guardar**.

Ha configurado el escalado horizontal para el clúster de Azure Data Explorer. Agregue otra regla para el escalado vertical. Si necesita ayuda por problemas relacionados con el escalado de un clúster, abra una solicitud de soporte técnico en [Azure Portal](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview).

## <a name="next-steps"></a>Pasos siguientes

* [Supervisión del rendimiento, el mantenimiento y el uso de Azure Data Explorer con métricas](using-metrics.md)
* [Administración del escalado vertical de clústeres](manage-cluster-vertical-scaling.md) para ajustar correctamente el tamaño de un clúster.
