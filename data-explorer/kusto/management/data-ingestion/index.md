---
title: 'Ingesta de datos: Azure Data Explorer | Microsoft Docs'
description: En este artículo se describe la ingesta de datos en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/30/2020
ms.openlocfilehash: f0fb68395ec5ed647e1f28cc1c93d46083d405c6
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83373440"
---
# <a name="data-ingestion"></a>Ingesta de datos

Es el proceso por el que se agregan datos a una tabla para que se puedan consultar.
En función de si la tabla existe de antemano o no, el proceso requiere [permisos de administrador de base de datos, de agente de ingesta de base de datos, de usuario de base de datos o de administrador de tablas](../access-control/role-based-authorization.md).

El proceso de ingesta de datos consta de varios pasos:

1. Recuperar los datos del origen de datos.
1. Analizar y validar los datos.
1. Establecer la correspondencia entre el esquema de los datos y el esquema de la tabla de Kusto de destino, o bien crear la tabla de destino si no existe.
1. Organizar los datos en columnas.
1. Indexar los datos.
1. Codificar y comprimir los datos.
1. Conservar los artefactos de almacenamiento de Kusto resultantes en el almacenamiento.
1. Ejecutar todas las directivas de actualización pertinentes, si hay alguna.
1. "Confirmar" la ingesta de datos para que estén disponibles para la realización de consultas.

> [!NOTE]
> En determinados escenarios, se pueden omitir algunos de los pasos anteriores.
> Por ejemplo, los datos ingeridos mediante el punto de conexión de ingesta de streaming omiten los pasos 4, 5, 6 y 9. Estos se ingieren en segundo plano a medida que se van "limpiando" los datos.
> En otro ejemplo, si el origen de datos es el resultado de una consulta de Kusto al mismo clúster, no es preciso analizar y validar los datos.

> [!WARNING]
> Los datos ingeridos en una tabla de Kusto están sujetos a la **directiva de retención** vigente de la tabla.
> Salvo que la directiva de retención vigente se establezca explícitamente en una tabla, deriva de la directiva de retención de la base de datos. Por consiguiente, al introducir datos en Kusto, asegúrese de que la directiva de retención de la base de datos se ajusta a sus necesidades. Si no es así, anúlela explícitamente en el nivel de tabla. De no hacerlo, sus datos se podrían eliminar de forma "silenciosa" debido a la directiva de retención de la base de datos. Para información más detallada, consulte [Directiva de retención](https://kusto.azurewebsites.net/docs/concepts/retentionpolicy.html).

Para ver las propiedades de la ingesta de datos, consulte el artículo en el que se informa acerca de las [propiedades de la ingesta de datos en Azure Data Explorer](../../../ingestion-properties.md).
Para ver una lista de los formatos compatibles para la ingesta de datos, consulte el artículo en el que se informa acerca de los [formatos de datos](../../../ingestion-supported-formats.md).



## <a name="ingestion-methods"></a>Métodos de ingesta

Hay varios métodos de ingesta de datos y cada uno de ellos tiene sus propias características:

1. **Ingesta insertada (inserción)** : se envía un comando de control ([.ingest inline](./ingest-inline.md)) al motor y los datos que se van a ingerir forman parte del propio texto del comando.
   Este método está pensado principalmente para pruebas ad hoc y no debe usarse para producción.
1. **Ingesta desde consulta**: se envía un comando de control ([.set, .append, .set-or-append o .set-or-replace](./ingest-from-query.md)) se envía al motor y los datos se especifican indirectamente como los resultados de una consulta o un comando.
   Este método es útil para generar tablas de informes a partir de tablas de datos sin procesar o para crear tablas temporales pequeñas para su posterior análisis.
1. **Ingesta desde almacenamiento (extracción)** : se envía al motor un comando de control ([.ingest into](./ingest-from-storage.md)) con los datos almacenados en algún almacenamiento externo (por ejemplo, Azure Blob Storage) al que el motor puede acceder y al que el comando señala.
   Este método permite una eficaz ingesta en bloque de los datos, pero impone cierta carga al cliente que realiza la ingesta, ya que no puede sobrecargar el clúster con ingestas simultáneas (porque se arriesga a consumir todos los recursos del clúster por la ingesta de datos, lo que reduciría el rendimiento de las consultas).
1. **Ingesta en cola**: Los datos se cargan en el almacenamiento externo (por ejemplo, Azure Blob Storage). Se envía una notificación a una cola (por ejemplo, cola de Azure o centro de eventos).
   Este es el método principal que se usa en producción, porque tiene una disponibilidad muy alta, no requiere que el cliente administre la capacidad y controla bien las anexiones masivas. A veces, este método se denomina "ingesta nativa".


|Método             |Latencia                 |Producción|En bloque|Disponibilidad|Sincronicidad|
|-------------------|------------------------|----------|----|------------|-------------|
|Ingesta insertada   |Segundos + tiempo de ingesta   |No        |No  |Motor de Kusto|Sincrónica  |
|Ingesta desde consulta  |Tiempo de consulta + tiempo de ingesta|Sí       |Sí |Motor de Kusto|Sincrónica  |
|Ingesta desde almacenamiento|Segundos + tiempo de ingesta   |Sí       |Sí |Motor de Kusto|Ambos         |
|Ingesta en cola   |Minutos                 |Sí       |Sí |Storage     |Asincrónica |

## <a name="validation-policy-during-ingestion"></a>Directiva de validación durante la ingesta

Al realizar la ingesta desde el almacenamiento, los datos de origen se validan como parte del análisis.
La directiva de validación indica cómo reaccionar ante los errores de análisis. Consta de dos propiedades:

* `ValidationOptions`: aquí, `0` significa que no se debe realizar ninguna validación, `1` valida que todos los registros tengan el mismo número de campos (útil para archivos CSV y similares), y `2` indica que se omitan los campos que no tengan comillas dobles.

* `ValidationImplications`: `0` indica que los errores de validación deben generar un error en toda la ingesta y `1` indica que se deben omitir los errores de validación.