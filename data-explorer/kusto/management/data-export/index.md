---
title: 'Exportación de datos: Azure Data Explorer | Microsoft Docs'
description: En este artículo se describe cómo exportar datos en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 09/09/2019
ms.openlocfilehash: f8c05967d684f9723dd26085eddda7261a7eb19b
ms.sourcegitcommit: 283cce0e7635a2d8ca77543f297a3345a5201395
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2020
ms.locfileid: "84011404"
---
# <a name="data-export"></a>Exportación de datos

La exportación de datos es el proceso que ejecuta una consulta de Kusto y escribe sus resultados, para que estos resultados están disponibles para inspeccionarlos posteriormente.

Existen varios métodos para la exportación de datos:

* **Exportación del lado del cliente**: en su forma más sencilla, la exportación de datos se puede realizar en el cliente (el cliente ejecuta una consulta en el servicio, lee los resultados que devuelve esta consulta y, luego, los escribe en otro lugar). Esta forma de exportación de datos depende de la herramienta de cliente para realizar la exportación, normalmente en el sistema de archivos local en el que se ejecuta la herramienta. Entre las herramientas que admiten este modelo se encuentran [Kusto.Explorer](../../tools/kusto-explorer.md) y la [interfaz de usuario web](../../../web-query-data.md), entre otras.

* **Exportación en el lado del servicio (extracción)** : si el destino de la exportación es una tabla de Kusto (en el mismo clúster o base de datos que la consulta, o en otro), use el flujo de "ingesta desde consulta" en la tabla de destino. En este flujo, se ejecuta una consulta y sus resultados se ingieren inmediatamente en una tabla de Kusto. Consulte este artículo acerca de la [ingesta de datos](../../../ingest-data-overview.md).



* **Exportación en el lado del servicio (inserción)** : los métodos anteriores son algo limitados, ya que los resultados de la consulta se deben transmitir por streaming a través de una conexión de red individual entre el productor que realiza la consulta y el consumidor que escribe sus resultados. Para una exportación de datos escalable, Kusto proporciona un modelo de exportación de "inserción", en el que el servicio que ejecuta la consulta también escribe sus resultados de forma optimizada. Este modelo se expone a través de un conjunto de comandos de control `.export`, que permite exportar los resultados de una consulta a una [tabla externa](export-data-to-an-external-table.md), una [tabla SQL](export-data-to-sql.md) o un [almacenamiento de blobs externo](export-data-to-storage.md).
  
  Los comandos de exportación del lado del servicio están limitados por la capacidad de exportación de datos del clúster que haya disponible. 
  Puede ejecutar el comando [show capacity](../../management/diagnostics.md#show-capacity) para ver la capacidad de exportación de datos total, consumida y restante.

## <a name="recommendations-for-secret-management-when-using-data-export-commands"></a>Recomendaciones para la administración de secretos cuando se usan comandos de exportación de datos

Idealmente, la exportación de datos a un destino remoto (como Azure Blob Storage y Azure SQL Database) se realizaría mediante el uso implícito de las credenciales de la entidad de seguridad que ejecuta el comando de exportación de datos. Esto no es posible en algunos escenarios (por ejemplo, Azure Blob Storage no admite la noción de entidad de seguridad, solo sus propios tokens). Por consiguiente, Kusto permite especificar las credenciales necesarias en línea como parte del comando de control de exportación de datos. Estas son algunas recomendaciones para asegurarse de que esto se realiza de forma segura:

Utilice [literales de cadena ofuscados](../../query/scalar-data-types/string.md#obfuscated-string-literals) (como `h@"..."`) cuando envíe secretos a Kusto.
Luego, Kusto borrará estos secretos para que no aparezcan en ningún seguimiento que emita internamente.

Además, tanto las contraseñas como otros secretos similares se deben almacenar de forma segura, y la aplicación debe "extraerlos" a medida que sea necesario.
