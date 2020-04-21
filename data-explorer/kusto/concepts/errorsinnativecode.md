---
title: 'Errores en el código nativo: Explorador de Azure Data Explorer . Microsoft Docs'
description: En este artículo se describen los errores en código nativo en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 07/12/2019
ms.openlocfilehash: fa36bec3586a5e316b08ebc2949617268f0270ab
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81523263"
---
# <a name="errors-in-native-code"></a>Errores en el código nativo

El motor de Kusto se escribe en `HRESULT` código nativo e informa de errores mediante el uso de valores negativos. Estos errores no suelen aparecer con una API mediante programación, pero pueden producirse. Por ejemplo, los errores de operación pueden representar un estado de "`Exception from HRESULT:` *HRESULT-CODE*".

Los códigos de error nativos de Kusto se definen mediante la macro de Windows `MAKE-HRESULT` con:

* Gravedad establecida en`1`
* Facilidad establecida para`0xDA`
  
Se definen los siguientes códigos de error:

|Constante de manifiesto                  |Código |Value        |Significado                                                                                                        |
|-----------------------------------|-----|-------------|---------------------------------------------------------------------------------------------------------------|
|`E_EXTENT_LOAD_FAILED`             | `0`  |`0x80DA0000`|No se pudo cargar el fragmento de datos                                                                                  |
|`E_RUNAWAY_QUERY`                  | `1`  |`0x80DA0001`|La ejecución de consultas se anuló al superar sus recursos permitidos                                                   |
|`E_STREAM_FORMAT`                  | `2`  |`0x80DA0002`|Un flujo de datos no se puede analizar ya que está mal formateado                                                      |
|`E_QUERY_RESULT_SET_TOO_LARGE`     | `3`  |`0x80DA0003`|El conjunto de resultados para esta consulta supera sus límites de registro/tamaño permitidos                                            |
|`E_STREAM_ENCODING_VERSION`        | `4`  |`0x80DA0004`|Un flujo de resultados no se puede analizar ya que su versión es desconocida                                                   |
|`E_KVDB_ERROR`                     | `5`  |`0x80DA0005`|Error al realizar una operación de base de datos clave/valor                                                              |
|`E_QUERY_CANCELLED`                | `6`  |`0x80DA0006`|La consulta fue cancelada                                                                                            |
|`E_LOW_MEMORY_CONDITION`           | `7`  |`0x80DA0007`|La operación se anuló debido a que la memoria de proceso disponible se agota                                              |
|`E_WRONG_NUMBER_OF_FIELDS`         | `8`  |`0x80DA0008`|Un documento csv enviado para la ingesta tiene un registro con el número incorrecto de campos (en relación con otros registros)|
|`E_INPUT_STREAM_TOO_LARGE`         | `9`  |`0x80DA0009`|El documento presentado para la ingestión ha superado la longitud permitida                                           |
|`E_ENTITY_NOT_FOUND`               | `10` |`0x80DA000A`|No se encontró la entidad solicitada                                                                             |
|`E_CLOSING_QUOTE_MISSING`          | `11` |`0x80DA000B`|Un documento csv enviado para la ingestión tiene un campo con una cotización faltante                                        |
|`E_OVERFLOW`                       | `12` |`0x80DA000C`|Representa un error de desbordamiento aritmético (el resultado de un cálculo es demasiado grande para el tipo de destino)    |
|`E_RS_BLOCKED_ROWSTOREKEY_ERROR`   | `101`|`0x80DA0065`|Se ha intentado acceder a una clave de almacén de filas bloqueada                                                          |
|`E_RS_SHUTTINGDOWN_ERROR`          | `102`|`0x80DA0066`|almacén de filas se está cerrando                                                                                     |
|`E_RS_LOCAL_STORAGE_FULL_ERROR`    | `103`|`0x80DA0067`|El espacio en disco asignado para el almacenamiento del almacén de filas está lleno                                                             |
|`E_RS_CANNOT_READ_WRITE_AHEAD_LOG` | `104`|`0x80DA0068`|Error al leer desde el almacenamiento del almacén de filas                                                                      |
|`E_RS_CANNOT_RETRIEVE_VALUES_ERROR`| `105`|`0x80DA0069`|Error al recuperar valores del almacenamiento del almacén de filas                                                              |
|`E_RS_NOT_READY_ERROR`             | `106`|`0x80DA006A`|almacén de filas está inicializando                                                                                      |
|`E_RS_INSERTION_THROTTLED_ERROR`   | `107`|`0x80DA006B`|La inserción de valor en un almacén de filas se limitó                                                                   |
|`E_RS_READONLY_ERROR`              | `108`|`0x80DA006C`|almacén de filas se adjunta en estado de solo lectura                                                                       |
|`E_RS_UNAVAILABLE_ERROR`           | `109`|`0x80DA006D`|tienda de filas no está disponible actualmente                                                                             |