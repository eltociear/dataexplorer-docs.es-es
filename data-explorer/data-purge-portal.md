---
title: Uso de la purga de datos para eliminar datos personales de un dispositivo o servicio en Azure Data Explorer
description: Aprenda a purgar (eliminar) datos de Azure Data Explorer mediante la purga de datos.
author: orspod
ms.author: orspodek
ms.reviewer: kedamari
ms.service: data-explorer
ms.topic: conceptual
ms.date: 05/12/2020
ms.openlocfilehash: 3f170a48f31f9d842e39fcf42fcfce0c383c71ed
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83232367"
---
# <a name="enable-data-purge-on-your-azure-data-explorer-cluster"></a>Habilitar la purga de datos en el clúster de Azure Data Explorer

[!INCLUDE [gdpr-intro-sentence](includes/gdpr-intro-sentence.md)]

Azure Data Explorer admite la posibilidad de eliminar registros individuales. La eliminación de datos mediante el comando `.purge` protege los datos personales y no debe usarse en otros escenarios. No se ha diseñado para admitir solicitudes de eliminación frecuentes o la eliminación de grandes cantidades de datos, y puede tener un impacto significativo en el rendimiento del servicio.

La ejecución de un comando `.purge` desencadena un proceso que puede tardar varios días en completarse. Si la "densidad" de los registros en los que se aplica el `predicate` es grande, el proceso volverá a ingerir todos los datos de la tabla. Este proceso tiene un impacto significativo en el rendimiento y en la categoría Costo de productos vendidos (COGS). Para más información, consulte [Purga de datos en Azure Data Explorer](kusto/concepts/data-purge.md).

## <a name="methods-of-invoking-purge-operations"></a>Métodos para invocar operaciones de purga 

Azure Data Explorer (Kusto) admite la eliminación de registros individuales y la purga de una tabla completa. El comando `.purge` se puede [invocar de dos maneras](kusto/concepts/data-purge.md#purge-table-tablename-records-command) para distintos escenarios de uso:

* Invocación mediante programación: Un solo paso diseñado para que lo invoquen las aplicaciones. La llamada a este comando desencadena directamente la secuencia de ejecución de la purga.

* Invocación manual: Proceso de dos pasos que requiere una confirmación explícita como paso independiente. La invocación del comando devuelve un token de comprobación que se debe proporcionar para ejecutar la purga real. Este proceso reduce el riesgo de eliminar accidentalmente datos incorrectos. El uso de esta opción puede tardar mucho tiempo en completarse en tablas grandes con una cantidad significativa de datos en una caché inactiva. 

## <a name="prerequisites"></a>Prerrequisitos

* Si no tiene una suscripción a Azure, cree una [cuenta gratuita de Azure](https://azure.microsoft.com/free/) antes de empezar.
* Inicie sesión en la [interfaz de usuario web](https://dataexplorer.azure.com/).
* Cree un [clúster y una base de datos de Azure Data Explorer](create-cluster-database-portal.md).

## <a name="enable-data-purge-on-your-cluster"></a>Habilitar la purga de datos en el clúster

> [!WARNING]
> * Para habilitar la purga de datos, es necesario reiniciar el servicio, lo que podría provocar la anulación de la consulta.
> * Revise las [limitaciones](#limitations) antes de habilitar la purga de datos.

1. En Azure Portal, vaya al clúster de Azure Data Explorer. 
1. En **Configuración**, seleccione **Configuraciones**. 
1. En el panel **Configuraciones**, seleccione **Activado** para activar la opción **Habilitar purga**.
1. Seleccione **Guardar**.
 
    ![Habilitar purga](media/data-purge-portal/enable-purge-on.png)

## <a name="disable-data-purge-on-your-cluster"></a>Deshabilitar la purga de datos en el clúster

1. En Azure Portal, vaya al clúster de Azure Data Explorer. 
1. En **Configuración**, seleccione **Configuraciones**. 
1. En el panel **Configuraciones**, seleccione **Desactivado** para desactivar la opción **Habilitar purga**.
1. Seleccione **Guardar**.

    ![Deshabilitar purga](media/data-purge-portal/enable-purge-off.png)

## <a name="limitations"></a>Limitaciones

* El proceso de purga es final e irreversible. No es posible "deshacer" este proceso ni recuperar los datos que se han purgado. Por lo tanto, los comandos como [undo table drop](kusto/management/undo-drop-table-command.md) no pueden recuperar los datos purgados y la reversión de los datos a una versión anterior no puede ser "anterior" a la última purga.
* El comando `.purge` se ejecuta en el punto de conexión de administración de datos: *https://ingest- [nombreDelClúster].[Región].kusto.windows.net*. El comando requiere permisos de [administración de base de datos](kusto/management/access-control/role-based-authorization.md) en las bases de datos pertinentes. 
* Debido al impacto en el rendimiento del proceso de purga, se espera que el autor de la llamada modifique el esquema de datos para que las tablas mínimas incluyan los datos pertinentes y los comandos por lotes por tabla para reducir el importante impacto en la categoría Costo de productos vendidos del proceso de purga.
* El parámetro `predicate` del comando purge se usa para especificar los registros que se van a purgar. El tamaño de `Predicate` está limitado a 63 KB. 

## <a name="next-steps"></a>Pasos siguientes

* [Purga de datos en Azure Data Explorer](kusto/concepts/data-purge.md)
