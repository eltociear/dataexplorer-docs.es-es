---
title: Administración de diarios- Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe la administración de diarios en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 08/19/2019
ms.openlocfilehash: bbe5ab4bda42fdfd9382c7e71da85789c13f6987
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81520798"
---
# <a name="journal-management"></a>Gestión de revistas

 `Journal`contiene información sobre las operaciones de metadatos realizadas en la base de datos Kusto.

Las operaciones de metadatos podrían ser el resultado del comando de control ejecutado por comandos de control interno o de usuario ejecutados por el sistema (como las extensiones de colocación por retención).

**Notas:**

- Las operaciones de metadatos que `.append`abarcan `.move` *la adición* de nuevas `Journal`extensiones (como `.ingest`, , y otras) no tendrán eventos coincidentes que se muestran en el archivo .
- Los datos de las columnas del conjunto de resultados, así como el formato en el que se presentan, *no* son contractuales y, por lo tanto, *no* se recomienda depender de ellos.

|Evento        |EventTimestamp     |Base de datos  |EntityName|UpdatedEntityName|EntityVersion|EntityContainerName|
|-------------|-------------------|----------|----------|-----------------|-------------|-------------------|
|CREATE-TABLE |2017-01-05 14:25:07|InternalDb|MyTable1  |MyTable1         |v7.0         |InternalDb         |
|RENOMBRAR-TABLE |2017-01-13 10:30:01|InternalDb|MyTable1  |MyTable2         |v8.0         |InternalDb         |  

|OriginalEntityState|UpdatedEntityState                                              |ChangeCommand                                                                                                          |Principal            |
|-------------------|----------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------|---------------------|
|.                  |Nombre: MyTable1, Atributos: Nombre '[MiTabla1]. [col1]', Tipo 'I32'|Tabla .create MyTable1 (col1:int)                                                                                      |imike@fabrikam.com
|.                  |Las propiedades db (demasiado largas para mostrarse aquí)               |.create base de datoshttps://imfbkm.blob.core.windows.net/mdTestDB persistir (-" ", " ")https://imfbkm.blob.core.windows.net/data|Id.aID app id-76263cdb-abcd-545644e9c404
|Nombre: MyTable1, Atributos: Nombre '[MiTabla1]. [col1]', Tipo 'I32'|Nombre: MyTable2, Atributos: Nombre '[MiTabla1]. [col1]', Tipo 'I32'|Tabla .rename MyTable1 a MyTable2|rdmik@fabrikam.com


Event: el nombre del evento de metadatos.

EventTimestamp - la marca de tiempo del evento.

Base de datos: los metadatos de esta base de datos se cambiaron después del evento.

EntityName: el nombre de la entidad en la que se ejecutó la operación antes del cambio.

UpdatedEntityName - el nuevo nombre de entidad después del cambio.

EntityVersion - la nueva versión de metadatos (db/cluster) después del cambio.

EntityContainerName - el nombre del contenedor de entidades (es decir: entidad-columna, contenedor-tabla).

OriginalEntityState - el estado de la entidad (propiedades de entidad) antes del cambio.

UpdatedEntityState - el nuevo estado después del cambio.

ChangeCommand - el comando de control ejecutado que desencadenó el cambio de metadatos.

Principal: la entidad de seguridad (usuario/aplicación) que ejecutó el comando de control.
                    
## <a name="show-journal"></a>Diario .show

El `.show journal` comando devuelve una lista de cambios de metadatos, en bases de datos/cluster el usuario tiene acceso de administrador.

**Permisos**

Todos los usuarios (acceso al clúster) pueden ejecutar el comando. 

Los resultados devueltos incluirán: 
1. Todas las entradas de diario del usuario que ejecuta el comando. 
2. Todas las entradas de diario de bases de datos a las que el usuario que ejecuta el comando tiene acceso de administrador. 
3. Todas las entradas del diario del clúster si el usuario que ejecuta el comando es un administrador de clúster. 

## <a name="show-database-databasename-journal"></a>.show base de datos *DatabaseName* diario 

El `.show` `database` comando *DatabaseName* `journal` devuelve journal para los cambios de metadatos de base de datos específicos.

**Permisos**

Todos los usuarios (acceso al clúster) pueden ejecutar el comando. Los resultados devueltos incluirán: 
1. Todas las entradas de diario de *databaseName* si el usuario que ejecuta el comando es un administrador de base de datos en *DatabaseName*. 
2. De lo contrario, todas las entradas de diario de la base de datos *DatabaseName* y del usuario que ejecuta el comando. 

