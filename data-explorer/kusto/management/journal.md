---
title: 'Administración de Journal: Azure Explorador de datos'
description: En este artículo se describe la administración de Journal en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 08/19/2019
ms.openlocfilehash: 64b0f8179a328ce811747b05a90516fd8b6029be
ms.sourcegitcommit: 41cd88acc1fd79f320a8fe8012583d4c8522db78
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2020
ms.locfileid: "84294413"
---
# <a name="journal-management"></a>Administración de diario

 `Journal`contiene información sobre las operaciones de metadatos que se realizan en la base de datos de Azure Explorador de datos.

Las operaciones de metadatos pueden ser el resultado de un comando de control que un usuario ejecutó, o comandos de control interno que el sistema ejecutó, como Drop extents by Retention.

> [!NOTE]
> * Las operaciones de metadatos que abarcan la *adición* de nuevas extensiones, como `.ingest` , `.append` `.move` y otras, no tendrán eventos coincidentes que se muestran en `Journal` .
> * Los datos de las columnas del conjunto de resultados, así como el formato en el que se presentan, no son contractuales. 
  No se recomienda tomar una dependencia en ellas.

|Evento        |EventTimestamp     |Base de datos  |EntityName|UpdatedEntityName|EntityVersion|EntityContainerName|
|-------------|-------------------|----------|----------|-----------------|-------------|-------------------|
|CREATE-TABLE |2017-01-05 14:25:07|InternalDb|MyTable1  |MyTable1         |V7.0         |InternalDb         |
|RENAME-TABLE |2017-01-13 10:30:01|InternalDb|MyTable1  |MyTable2         |v 8.0         |InternalDb         |  

|OriginalEntityState|UpdatedEntityState                                              |ChangeCommand                                                                                                          |Principal            |
|-------------------|----------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------|---------------------|
|.                  |Nombre: MyTable1, atributos: name = ' [MyTable1]. [col1] ', tipo = ' I32 '|. CREATE TABLE MyTable1 (col1: int)                                                                                      |imike@fabrikam.com
|.                  |Propiedades de la base de datos (demasiado tiempo para mostrarse aquí)         |. Create Database TestDB persiste (@ " https://imfbkm.blob.core.windows.net/md ", @ " https://imfbkm.blob.core.windows.net/data ")|Azure AD ID. de aplicación = 76263cdb-ABCD-545644e9c404
|Nombre: MyTable1, atributos: name = ' [MyTable1]. [col1] ', tipo = ' I32 '|Nombre: MyTable2, atributos: name = ' [MyTable1]. [col1] ', tipo = ' I32 '|. cambie el nombre de la tabla MyTable1 a MyTable2|rdmik@fabrikam.com

|Elemento                 |Descripción                                                              |                                
|---------------------|-------------------------------------------------------------------------|
|Evento                |Nombre del evento de metadatos                                                  |
|EventTimestamp       |La marca de tiempo del evento                                                      |                        
|Base de datos             |Los metadatos de esta base de datos se cambiaron después del evento                |
|EntityName           |El nombre de la entidad en la que se ejecutó la operación antes del cambio.    |
|UpdatedEntityName    |El nuevo nombre de entidad después del cambio                                     |
|EntityVersion        |La nueva versión de metadatos (dB/cluster) tras el cambio               |
|EntityContainerName  |El nombre del contenedor de entidades (Entity = Column, Container = Table)               |
|OriginalEntityState  |El estado de la entidad (propiedades de entidad) antes del cambio.            |
|UpdatedEntityState   |Nuevo estado después del cambio                                           |
|ChangeCommand        |Comando de control ejecutado que desencadenó el cambio de metadatos          |
|Principal            |La entidad de seguridad (usuario o aplicación) que ejecutó el comando de control               |
    
## <a name="show-journal"></a>. Mostrar diario

El `.show journal` comando devuelve una lista de cambios de metadatos en las bases de datos o en el clúster al que el usuario tiene acceso de administrador.

**Permisos**

Todos (acceso al clúster) pueden ejecutar el comando. 

Los resultados devueltos incluirán: 
- Todas las entradas del diario del usuario que ejecuta el comando. 
- Todas las entradas del diario de las bases de datos a las que el usuario que ejecuta el comando tiene acceso de administrador. 
- Todas las entradas del diario del clúster si el usuario que ejecuta el comando es un administrador de clústeres. 

## <a name="show-database-databasename-journal"></a>. Mostrar el diario de la base de datos *DatabaseName* 

El `.show` `database` comando *DatabaseName* `journal` devuelve el diario de los cambios de metadatos de base de datos específicos.

**Permisos**

Todos (acceso al clúster) pueden ejecutar el comando. Los resultados devueltos incluyen: 
- Todas las entradas del diario de la base de datos *DatabaseName* si el usuario que ejecuta el comando es un administrador de base de datos en *DatabaseName*. 
- De lo contrario, todas las entradas del diario de la base de datos `DatabaseName` y del usuario que ejecuta el comando. 
