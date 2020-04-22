---
title: Supervisión de Azure Data Explorer con Resource Health
description: Use Azure Resource Health para supervisar los recursos de Azure Data Explorer.
author: orspod
ms.author: orspodek
ms.reviewer: prvavill
ms.service: data-explorer
ms.topic: conceptual
ms.date: 03/31/2020
ms.openlocfilehash: 9b8b1b40e972d7719ef2cf495ed12cb4ed3478fd
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81492416"
---
# <a name="monitor-azure-data-explorer-using-resource-health-preview"></a>Supervisión de Azure Data Explorer con Resource Health (versión preliminar)

[Resource Health](/azure/service-health/resource-health-overview) para Azure Data Explorer le informa del estado de su recurso de Azure Data Explorer y proporciona recomendaciones útiles para solucionar los problemas. Resource Health se actualiza cada 1 a 2 minutos y notifica el estado actual y anterior de los recursos. 

Resource Health determina el estado del recurso de Azure Data Explorer al analizar varias comprobaciones de estado de mantenimiento como las siguientes:
* Disponibilidad de recursos
* Errores de consulta

## <a name="access-resource-health-reporting"></a>Acceso a los informes de Resource Health

1. En [Azure Portal](https://portal.azure.com/), seleccione **Monitor** de la lista de servicios.
1. Seleccione **Service Health** > **Estado de los recursos**.
1. En la lista desplegable **Suscripción**, seleccione la suscripción. En la lista desplegable **Tipo de recurso**, seleccione **Azure Data Explorer**.
1. La tabla resultante muestra todos los recursos de la suscripción elegida. Cada recurso incluirá un icono de estado de mantenimiento que indica el estado del recurso.
1. Seleccione el recurso para ver su [estado de mantenimiento de recurso](#resource-health-status) y recomendaciones.

    ![Información general](media/monitor-with-resource-health/resource-health-overview.png)

## <a name="resource-health-status"></a>Estado de mantenimiento de los recursos

El estado de un recurso se muestra con uno de los siguientes estados: Disponible, No disponible y Desconocido.

### <a name="available"></a>Disponible

El estado de mantenimiento **Disponible** indica que el recurso de Azure Data Explorer está en buen estado y no tiene ningún problema.

![Disponible](media/monitor-with-resource-health/available.png)

### <a name="unavailable"></a>No disponible

El estado de mantenimiento **No disponible** indica que actualmente hay un problema con el recurso de Azure Data Explorer que hace que no esté disponible para consultas e ingesta. Por ejemplo, es posible que los nodos del recurso de Azure Data Explorer se hayan reiniciado de forma inesperada. Si el recurso de Azure Data Explorer permanece en este estado durante un largo período, póngase en contacto con [soporte técnico]().

![No disponible](media/monitor-with-resource-health/unavailable.png)

### <a name="unknown"></a>Desconocido

El estado de mantenimiento **Desconocido** indica que **Resource Health** no ha recibido información sobre este recurso de Azure Data Explorer durante más de 10 minutos. Este estado no es una indicación definitiva del estado del recurso de Azure Data Explorer, aunque es un punto de datos importante en el proceso de solución de problemas. Si el clúster de Azure Data Explorer funciona según lo previsto, el estado cambiará a **Disponible** en unos minutos. El estado **Desconocido** podría sugerir que un evento de la plataforma afecta al recurso. 

> [!TIP]
> El estado del recursos de clúster de Azure Data Explorer será **Desconocido** si está detenido.

![Desconocido](media/monitor-with-resource-health/unknown.png)

## <a name="historical-information"></a>Información histórica

En el panel **Estado de los recursos** > **Historial de estado**, tiene acceso a un máximo de cuatro semanas de información sobre el estado de mantenimiento de los recursos. Seleccione la flecha para obtener información adicional sobre los problemas de eventos de estado que se muestran en este panel. 

![Historial](media/monitor-with-resource-health/healthhistory.png)

## <a name="next-steps"></a>Pasos siguientes

* [Configuración de alertas de Resource Health](https://docs.microsoft.com/azure/service-health/resource-health-alert-arm-template-guide)
* [Tutorial: Ingesta y consulta de datos de supervisión en Azure Data Explorer](ingest-data-no-code.md)
* [Supervisión del rendimiento, el mantenimiento y el uso de Azure Data Explorer con métricas](using-metrics.md)