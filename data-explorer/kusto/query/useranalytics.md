---
title: 'Análisis de usuarios: Explorador de datos de Azure ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe User Analytics en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 11/17/2019
ms.openlocfilehash: 37b5962b9441c3eb7362a239404189e489b1c3ff
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81504988"
---
# <a name="user-analytics"></a>Análisis de usuario

En esta sección se describen las extensiones de Kusto (complementos) para escenarios de User Analytics.

|Escenario|Complemento|Detalles|Experiencia del usuario|
|--------|------|--------|-------|
| Contando nuevos usuarios a lo largo del tiempo | [activity_counts_metrics](activity-counts-metrics-plugin.md)|Devuelve recuentos/dcounts/new counts para cada ventana de tiempo. Cada ventana de tiempo se compara con *todas las* ventanas de tiempo anteriores|Kusto.Explorer: Galería de informes|
| Período durante el período: tasa de retención/rotación y nuevos usuarios | [activity_metrics](activity-metrics-plugin.md)|Devuelve recuentos, tasa de retención/curn para cada ventana de tiempo. Cada ventana de tiempo se compara con la ventana de tiempo *anterior*|Kusto.Explorer: Galería de informes|
| Los usuarios cuentan y cuentan en la ventana deslizante | [sliding_window_counts](sliding-window-counts-plugin.md)|Para cada ventana de tiempo, devuelve recuentos y recuentos durante un período de búsqueda, de una manera de ventana deslizante|
| Cohorte de nuevos usuarios: tasa de retención/rotación y nuevos usuarios | [new_activity_metrics](new-activity-metrics-plugin.md)|Compara entre cohortes de nuevos usuarios (todos los usuarios que fueron vistos en la 1a ventana de tiempo). Cada cohorte se compara con todas las cohortes anteriores. La comparación tiene en cuenta *todas las* ventanas de tiempo anteriores|Kusto.Explorer: Galería de informes|
|Usuarios activos: recuentos distintos |[active_users_count](active-users-count-plugin.md)|Devuelve usuarios distintos para cada ventana de tiempo, donde un usuario solo se tiene en cuenta si aparece en al menos X períodos distintos en un período de retención esp.p.|
|Participación del usuario: DAU/WAU/MAU|[activity_engagement](activity-engagement-plugin.md)|Compara entre una ventana de tiempo interior (por ejemplo, diaria) y una externa (por ejemplo, semanalmente) para la interacción informática (por ejemplo, DAU/WAU)|Kusto.Explorer: Galería de informes|
|Sesiones: contar sesiones activas|[session_count](session-count-plugin.md)|Cuenta las sesiones, donde una sesión se define por un período de tiempo - un registro de usuario se considera una nueva sesión, si no se ha visto en el período de retención del registro actual|
||||
|Embudos: análisis de secuencia de estado anterior y siguiente | [funnel_sequence](funnel-sequence-plugin.md)|Cuenta los usuarios distintos que han tomado una secuencia de eventos, y los eventos anteriores/siguientes que led / fueron seguidos por la secuencia. Es útil para construir [diagramas de sankey](https://en.wikipedia.org/wiki/Sankey_diagram)||
|Embudos: análisis de finalización de secuencias|[funnel_sequence_completion](funnel-sequence-completion-plugin.md)|Calcula el recuento distinto de usuarios que han *completado* una secuencia especificada en cada ventana de tiempo|
||||