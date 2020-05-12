---
title: 'complemento de infer_storage_schema: Azure Explorador de datos'
description: En este artículo se describe infer_storage_schema complemento en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/24/2020
ms.openlocfilehash: 1b4a917101ad3a35f8fdbc1cccb257b6f3724b69
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83224874"
---
# <a name="infer_storage_schema-plugin"></a>complemento de infer_storage_schema

Este complemento deduce el esquema de datos externos y lo devuelve como una cadena de esquema de CSL. La cadena se puede usar al [crear tablas externas](../management/external-tables-azurestorage-azuredatalake.md#create-or-alter-external-table).

```kusto
let options = dynamic({
  'StorageContainers': [
    h@'https://storageaccount.blob.core.windows.net/container1;secretKey'
  ],
  'DataFormat': 'parquet',
  'FileExtension': '.parquet'
});
evaluate infer_storage_schema(options)
```

**Sintaxis**

`evaluate` `infer_storage_schema(` *Opciones* `)`

**Argumentos**

Un solo argumento *Options* es un valor constante de tipo `dynamic` que contiene un contenedor de propiedades que especifica las propiedades de la solicitud:

|Nombre                    |Obligatorio|Descripción|
|------------------------|--------|-----------|
|`StorageContainers`|Sí|Lista de [cadenas de conexión de almacenamiento](../api/connection-strings/storage.md) que representan el URI del prefijo para los artefactos de datos almacenados|
|`DataFormat`|Sí|Uno de los [formatos de datos](https://docs.microsoft.com/azure/data-explorer/ingestion-supported-formats)admitidos.|
|`FileExtension`|No|Examinar solo los archivos que terminen con esta extensión de archivo. No es necesario, pero si se especifica, se puede acelerar el proceso (o eliminar problemas de lectura de datos).|
|`FileNamePrefix`|No|Solo se examinan los archivos que empiezan con este prefijo. No es necesario, pero si se especifica, se puede acelerar el proceso.|
|`Mode`|No|Estrategia de inferencia de esquemas, uno de los siguientes: `any` , `last` , `all` . Inferir el esquema de datos de cualquier archivo (primero encontrado), del último archivo escrito o de todos los archivos, respectivamente. El valor predeterminado es `last`.|

**Devuelve**

El `infer_storage_schema` complemento devuelve una única tabla de resultados que contiene una sola fila o columna que contiene una cadena de esquema de CSL.

> [!NOTE]
> * La estrategia de inferencia de esquema ' All ' es una operación muy "costosa", ya que implica la lectura de *todos los* artefactos encontrados y la combinación de su esquema.
> * Es posible que algunos tipos devueltos no sean los reales como resultado de una estimación de tipo incorrecto (o, como resultado del proceso de combinación de esquemas). Este es el motivo por el que debe revisar detenidamente el resultado antes de crear una tabla externa.

**Ejemplo**

```kusto
let options = dynamic({
  'StorageContainers': [
    h@'https://storageaccount.blob.core.windows.net/MovileEvents/2015;secretKey'
  ],
  'FileExtension': '.parquet',
  'FileNamePrefix': 'part-',
  'DataFormat': 'parquet'
});
evaluate infer_storage_schema(options)
```

*Resultado*

|CslSchema|
|---|
|app_id: String, user_id: Long, event_time: DateTime, Country: String, City: String, device_type: String, device_vendor: String, ad_network: String, Campaign: String, site_id: String, event_type: String, event_name: String, orgánica: String, days_from_install: int, Revenue: real|