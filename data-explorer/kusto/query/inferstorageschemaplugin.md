---
title: infer_storage_schema complemento - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe infer_storage_schema complemento en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/24/2020
ms.openlocfilehash: 6c4543a3b029017067867bb70d913509941c332e
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81513913"
---
# <a name="infer_storage_schema-plugin"></a>infer_storage_schema plugin

Este complemento deduce el esquema de datos externos y lo devuelve como cadena de esquema CSL que se puede usar al [crear tablas externas.](../management/externaltables.md#create-or-alter-external-table)

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

Un único argumento *Options* es `dynamic` un valor constante de tipo que contiene un contenedor de propiedades que especifica las propiedades de la solicitud:

|Nombre                    |Obligatorio|Descripción|
|------------------------|--------|-----------|
|`StorageContainers`|Sí|Lista de cadenas de conexión de [almacenamiento](../api/connection-strings/storage.md) que representan URI de prefijo para artefactos de datos almacenados|
|`DataFormat`|Sí|Uno de [los formatos](https://docs.microsoft.com/azure/data-explorer/ingestion-supported-formats)de datos compatibles.|
|`FileExtension`|No|Sólo escanear archivos que terminan con esta extensión de archivo. No es necesario, pero especificarlo puede acelerar el proceso (o eliminar problemas de lectura de datos)|
|`FileNamePrefix`|No|Sólo escanee los archivos a partir de este prefijo. No es necesario, pero especificarlo puede acelerar el proceso|
|`Mode`|No|Estrategia de inferencia de `any` `last`esquema, una de: , , `all`. Inferir el esquema de datos de cualquier archivo (encontrado por primera vez), del último archivo escrito o de todos los archivos respectivamente. El valor predeterminado es `last`.|

**Devuelve**

El `infer_storage_schema` complemento devuelve una única tabla de resultados que contiene una sola fila/columna que contiene la cadena de esquema CSL.

> [!NOTE]
> * La estrategia de inferencia de esquema 'all' es una operación muy "costosa", ya que implica leer *todos los* artefactos encontrados y combinar su esquema.
> * Algunos tipos devueltos pueden no ser los reales como resultado de una conjetura de tipo incorrecta (o, como resultado del proceso de combinación de esquemas). Esta es la razón por la que se aconseja revisar el resultado cuidadosamente antes de crear una tabla externa.

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
|app_id:string, user_id:long, event_time:datetime, country:string, city:string, device_type:string, device_vendor:string, ad_network:string, campaign:string, site_id:string, event_type:string, event_name:string, organic:string, days_from_install:int, revenue:real|