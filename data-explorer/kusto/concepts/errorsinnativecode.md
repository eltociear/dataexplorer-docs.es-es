---
title: 'Errores en código nativo: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describen los errores en el código nativo de Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
ms.date: 06/15/2020
ms.openlocfilehash: 5446b98283a6533ceacd1c84fa3043732fe79e6a
ms.sourcegitcommit: a8575e80c65eab2a2118842e59f62aee0ff0e416
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/17/2020
ms.locfileid: "84942733"
---
# <a name="errors-in-native-code"></a>Errores en el código nativo

El motor de Kusto se escribe en código nativo e informa de los errores mediante `HRESULT` valores negativos. Estos errores no suelen aparecer con una API de programación, pero pueden producirse. Por ejemplo, los errores de operación pueden mostrar un estado de " `Exception from HRESULT:` *HRESULT-Code*".

Los códigos de error nativos de Kusto se definen mediante la `MAKE-HRESULT` macro de Windows con:

* Gravedad establecida en`1`
* Instalación establecida en`0xDA`
  
Se definen los siguientes códigos de error:

|Constante de manifiesto                  |Código |Value        |Significado                                                                                                        |
|-----------------------------------|-----|-------------|---------------------------------------------------------------------------------------------------------------|
|`E_EXTENT_LOAD_FAILED`             | `0`  |`0x80DA0000`|No se pudo cargar la partición de datos                                                                                  |
|`E_RUNAWAY_QUERY`                  | `1`  |`0x80DA0001`|Ejecución de la consulta anulada porque superó sus recursos permitidos                                                   |
|`E_STREAM_FORMAT`                  | `2`  |`0x80DA0002`|No se puede analizar un flujo de datos porque tiene un formato incorrecto                                    |
|`E_QUERY_RESULT_SET_TOO_LARGE`     | `3`  |`0x80DA0003`|El conjunto de resultados de esta consulta supera sus límites de tamaño/registro permitidos                                            |
|`E_STREAM_ENCODING_VERSION`        | `4`  |`0x80DA0004`|No se puede analizar un flujo de resultados porque su versión es desconocida                                                   |
|`E_KVDB_ERROR`                     | `5`  |`0x80DA0005`|No se pudo realizar una operación de base de datos de clave/valor                                                              |
|`E_QUERY_CANCELLED`                | `6`  |`0x80DA0006`|Consulta cancelada                 |
|`E_LOW_MEMORY_CONDITION`           | `7`  |`0x80DA0007`|La operación se anuló debido a que la memoria de proceso disponible se está ejecutando bajo                                              |
|`E_WRONG_NUMBER_OF_FIELDS`         | `8`  |`0x80DA0008`|Un documento CSV enviado para la ingesta tiene un registro con un número incorrecto de campos (relativo a otros registros)|
|`E_INPUT_STREAM_TOO_LARGE`         | `9`  |`0x80DA0009`|El documento enviado para la ingesta ha superado la longitud permitida                                           |
|`E_ENTITY_NOT_FOUND`               | `10` |`0x80DA000A`|No se encontró la entidad solicitada.                                                                             |
|`E_CLOSING_QUOTE_MISSING`          | `11` |`0x80DA000B`|Un documento CSV enviado para la ingesta tiene un campo con una comilla que falta                                        |
|`E_OVERFLOW`                       | `12` |`0x80DA000C`|Representa un error de desbordamiento aritmético (el resultado de un cálculo es demasiado grande para el tipo de destino)    |
|`E_RS_BLOCKED_ROWSTOREKEY_ERROR`   | `101`|`0x80DA0065`|Se intentó obtener acceso a una clave almacén bloqueada                                                          |
|`E_RS_SHUTTINGDOWN_ERROR`          | `102`|`0x80DA0066`|Almacén se está cerrando                                                                                     |
|`E_RS_LOCAL_STORAGE_FULL_ERROR`    | `103`|`0x80DA0067`|El almacenamiento en disco local asignado para almacén está lleno.                                                             |
|`E_RS_CANNOT_READ_WRITE_AHEAD_LOG` | `104`|`0x80DA0068`|No se pudo leer el almacenamiento de almacén                                                                      |
|`E_RS_CANNOT_RETRIEVE_VALUES_ERROR`| `105`|`0x80DA0069`|Error al recuperar valores de almacén Storage                                                              |
|`E_RS_NOT_READY_ERROR`             | `106`|`0x80DA006A`|Almacén se está inicializando                                                                                      |
|`E_RS_INSERTION_THROTTLED_ERROR`   | `107`|`0x80DA006B`|La inserción de valores en un almacén se limitó                                                                   |
|`E_RS_READONLY_ERROR`              | `108`|`0x80DA006C`|Almacén está adjuntado en estado de solo lectura                                                                       |
|`E_RS_UNAVAILABLE_ERROR`           | `109`|`0x80DA006D`|Almacén no está disponible en este momento                                                                             |
|`E_RS_DOES_NOT_EXIST_ERROR`           | `110`|`0x80DA006E`| Almacén no existe.                        |
|`E_SB_QUERY_THROTTLED_ERROR`           | `200`|`0x80DA00C8`|La consulta en espacio aislado se limitó                                                                           |
|`E_SB_QUERY_CANCELLED`           | `201`|`0x80DA00C9`|Se canceló la consulta en espacio aislado                                                                          |
|`E_UNSUPPORTED_INSTRUCTION_SET`           | `202`|`0x80DA00CA`|Esta CPU no admite el conjunto de instrucciones necesario del motor                                                                            |
