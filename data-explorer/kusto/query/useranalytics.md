---
title: 'Análisis de usuarios: Azure Explorador de datos'
description: En este artículo se describe el análisis de usuarios en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 11/17/2019
ms.openlocfilehash: a63f68564bc0e2fec8b6e2d5faa12e50b7adcc7c
ms.sourcegitcommit: 4f68d6dbfa6463dbb284de0aa17fc193d529ce3a
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/04/2020
ms.locfileid: "82741651"
---
# <a name="user-analytics"></a>Análisis de usuario

En esta sección se describen las extensiones de Kusto (complementos) para escenarios de análisis de usuarios.

|Escenario|Complemento|Detalles|Experiencia del usuario|
|--------|------|--------|-------|
| Recuento de nuevos usuarios a lo largo del tiempo | [activity_counts_metrics](activity-counts-metrics-plugin.md)|Devuelve recuentos/dcounts/nuevos recuentos para cada ventana de tiempo. Cada ventana de tiempo se compara con *las* ventanas de tiempo anteriores|Kusto. Explorer: Galería de informes|
| Período en período de tiempo: tasa de retención/renovación y nuevos usuarios | [activity_metrics](activity-metrics-plugin.md)|Devuelve `dcount`, tasa de retención/renovación para cada ventana de tiempo. Cada ventana de tiempo se compara con la ventana de tiempo *anterior*|Kusto. Explorer: Galería de informes|
| Recuento de `dcount` usuarios y sobre ventana deslizante | [sliding_window_counts](sliding-window-counts-plugin.md)|Para cada ventana de tiempo, devuelve Count `dcount` y en un período de lookback en una ventana deslizante.|
| New-users cohorte: tasa de retención/renovación y nuevos usuarios | [new_activity_metrics](new-activity-metrics-plugin.md)|Compara entre cohortes de nuevos usuarios (todos los usuarios que se vieron por primera vez en el período de tiempo). Cada cohorte se compara con todos los cohortes anteriores. La comparación tiene en cuenta *todas las* ventanas de tiempo anteriores|Kusto. Explorer: Galería de informes|
|Usuarios activos: recuentos distintivos |[active_users_count](active-users-count-plugin.md)|Devuelve usuarios distintos para cada ventana de tiempo. Un usuario solo se tiene en cuenta si aparece en al menos X distintos períodos en un período de lookback especificado.|
|Interacción con el usuario: DAU/WAU/MAU|[activity_engagement](activity-engagement-plugin.md)|Compara entre una ventana de tiempo interna (por ejemplo, diariamente) y una externa (por ejemplo, semanalmente) para calcular la interacción (por ejemplo, DAU/WAU).|Kusto. Explorer: Galería de informes|
|Sesiones: recuento de sesiones activas|[session_count](session-count-plugin.md)|Recuento de sesiones, donde una sesión se define en un período de tiempo: un registro de usuario se considera una nueva sesión, si no se ha detectado en el período de lookback desde el registro actual.|
||||
|Embudos: Análisis de la secuencia de estado anterior y siguiente | [funnel_sequence](funnel-sequence-plugin.md)|Cuenta los usuarios distintos que han realizado una secuencia de eventos y los eventos anteriores o siguientes que llevaron a cabo o que estaban seguidos de la secuencia. Útil para construir [diagramas de Sankey](https://en.wikipedia.org/wiki/Sankey_diagram)||
|Embudos: Análisis de la finalización de secuencias|[funnel_sequence_completion](funnel-sequence-completion-plugin.md)|Calcula el recuento distintivo de usuarios que han *completado* una secuencia especificada en cada ventana de tiempo.|
||||