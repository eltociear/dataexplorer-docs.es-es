---
title: 'Ingesta de streaming y cambios de esquema: Azure Explorador de datos'
description: En este artículo se describen las opciones de administración de cambios de esquema con la ingesta de streaming en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 05/20/2020
ms.openlocfilehash: f1ab3d012be7751eba33447b325748859509af00
ms.sourcegitcommit: 8e097319ea989661e1958efaa1586459d2b69292
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/15/2020
ms.locfileid: "84784572"
---
# <a name="streaming-ingestion-and-schema-changes"></a>Ingesta de streaming y cambios de esquema

## <a name="background"></a>Información previa

Los nodos de clúster almacenan en caché el esquema de las bases de datos que reciben datos a través de la ingesta de streaming. Este proceso optimiza el rendimiento y la utilización de los recursos del clúster, pero puede causar retrasos en la propagación cuando se producen cambios en el esquema.

Ejemplos de cambios de esquema:

* Creación y eliminación de bases de datos y tablas
* Agregar, quitar, volver a escribir o cambiar el nombre de las columnas de la tabla
* Agregar o quitar asignaciones de ingesta creadas previamente
* Adición, eliminación o modificación de directivas

Si los cambios de esquema y los flujos de ingesta de streaming no están coordinados, es posible que se produzcan errores en algunas de las solicitudes de ingesta de streaming. Los errores podrían incluir errores relacionados con el esquema o la inserción de datos incompletos o distorsionados en la tabla.
Al implementar la aplicación de ingesta personalizada, se recomienda encarecidamente controlar los errores relacionados con el esquema mediante la realización de reintentos durante un tiempo limitado o mediante el redireccionamiento de los datos de las solicitudes con error a través de los métodos de ingesta en cola.

## <a name="clearing-the-schema-cache"></a>Borrar la caché de esquema

Reducir los efectos del retraso de propagación al borrar explícitamente la caché de esquema en los nodos del clúster.
Borre la caché de esquema con uno de los comandos [Borrar caché de esquema para la administración de ingesta de streaming](clear-schema-cache-command.md) .
Si el flujo de ingesta de streaming y los cambios de esquema se coordinan, puede eliminar por completo los errores y su distorsión de datos asociada. 

**Ejemplo de flujo coordinado:**

1. Suspender la ingesta de streaming.
1. Espere hasta que se completen todas las solicitudes de ingesta de streaming pendientes>
1. Realizar cambios de esquema.
1. Emita uno o varios `.clear cache streaming ingestion` comandos de esquema. 
    * Repetir hasta que sea correcto y todas las filas del resultado del comando indiquen que se ha realizado correctamente
1. Reanudación de la ingesta de streaming.

> [!NOTE]
> El uso frecuente de comandos de esquema de ingesta de streaming de caché puede tener un efecto adverso en el rendimiento de la ingesta de streaming.
