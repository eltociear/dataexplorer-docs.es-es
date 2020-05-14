---
title: 'Ingesta de consultas de Kusto (establecer, anexar, reemplazar): Azure Explorador de datos'
description: En este artículo se describe la introducción de consultas (. Set,. Append,. set-o-Append,. Set-or-Replace) en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/30/2020
ms.openlocfilehash: bfa44859987d8f3c4f11221fd8370290f08f9a67
ms.sourcegitcommit: fd3bf300811243fc6ae47a309e24027d50f67d7e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83382053"
---
# <a name="ingest-from-query-set-append-set-or-append-set-or-replace"></a>Introducción de la consulta (. Set,. Append,. set-o-Append,. Set-or-Replace)

Estos comandos ejecutan una consulta o un comando de control e ingestan los resultados de la consulta en una tabla. La diferencia entre estos comandos es cómo tratan las tablas y los datos existentes o inexistentes:

|Comando          |Si la tabla existe                     |Si la tabla no existe                    |
|-----------------|------------------------------------|------------------------------------------|
|`.set`           |Se produce un error en el comando.                  |Se crea la tabla y se introducen los datos.|
|`.append`        |Los datos se anexan a la tabla.      |Se produce un error en el comando.                        |
|`.set-or-append` |Los datos se anexan a la tabla.      |Se crea la tabla y se introducen los datos.|
|`.set-or-replace`|Los datos reemplazan a los datos de la tabla.|Se crea la tabla y se introducen los datos.|

**Sintaxis**

`.set`[ `async` ] *TableName* [ `with` `(` *PropertyName* `=` *PropertyValue* [ `,` ...] `)` ] `<|` *QueryOrCommand*

`.append`[ `async` ] *TableName* [ `with` `(` *PropertyName* `=` *PropertyValue* [ `,` ... `])` ] `<|` *QueryOrCommand*

`.set-or-append`[ `async` ] *TableName* [ `with` `(` *PropertyName* `=` *PropertyValue* [ `,` ...] `)` ] `<|` *QueryOrCommand*

`.set-or-replace`[ `async` ] *TableName* [ `with` `(` *PropertyName* `=` *PropertyValue* [ `,` ...] `)` ] `<|` *QueryOrCommand*

**Argumentos**

* `async`: Si se especifica, el comando se devolverá inmediatamente y continuará la ingesta en segundo plano. Los resultados del comando incluirán un `OperationId` valor que se puede usar con el `.show operation` comando para recuperar el estado y los resultados de la finalización de la ingesta.
* *TableName*: nombre de la tabla a la que se van a ingerir datos.
  El nombre de la tabla siempre es relativo a la base de datos en contexto.
* *PropertyName*, *PropertyValue*: cualquier número de propiedades de ingesta que afecten al proceso de ingesta.

 Propiedades de ingesta admitidas:

|Propiedad.        |Descripción|
|----------------|-----------------------------------------------------------------------------------------------------------------------------|
|`creationTime`   | Valor DateTime (con formato de cadena ISO8601) que se va a usar en el momento de la creación de las extensiones de datos ingeridas. Si no se especifica, se utilizará el valor actual (Now ()).|
|`extend_schema`  | Valor booleano que, si se especifica, indica al comando que extienda el esquema de la tabla (el valor predeterminado es false). Esta opción solo se aplica a los comandos. Append. set-o-Append y set-OR-REPLACE. Las únicas extensiones de esquema permitidas tienen columnas adicionales agregadas a la tabla al final.|
|`recreate_schema`  | Valor booleano que, si se especifica, describe si el comando puede volver a crear el esquema de la tabla (el valor predeterminado es false). Esta opción solo se aplica al comando set-OR-REPLACE. Esta opción tiene prioridad sobre la propiedad extend_schema si ambos están establecidos.|
|`folder`         | Carpeta que se va a asignar a la tabla. Si la tabla ya existe, esta propiedad invalidará la carpeta de la tabla.|
|`ingestIfNotExists`   | Valor de cadena que, si se especifica, impide que la ingesta se realice correctamente si la tabla ya tiene datos etiquetados con el mismo valor.|
|`policy_ingestiontime`   | valor booleano que, si se especifica, describe si se habilita la [directiva de tiempo de ingesta](../../management/ingestiontime-policy.md) en una tabla que este comando crea. El valor predeterminado es true.|
|`tags`   | cadena JSON que indica las validaciones que se ejecutan durante la ingesta.|
|`docstring`   | Cadena que documenta la tabla.|

  Además, hay una propiedad que controla el comportamiento del propio comando:

|Propiedad.        |Tipo    |Description|
|----------------|--------|-----------------------------------------------------------------------------------------------------------------------------|
|`distributed`   |`bool`  |Indica que el comando ingeri de todos los nodos que ejecutan la consulta en paralelo. (El valor predeterminado es `false` .)  Vea la sección Comentarios a continuación.|

* *QueryOrCommand*: el texto de una consulta o un comando de control cuyos resultados se utilizarán como datos para la ingesta.

> [!NOTE]
> Solo `.show` se admiten comandos de control.

**Comentarios:**

* `.set-or-replace`reemplaza los datos de la tabla si existe (quita las particiones de datos existentes) o crea la tabla de destino si aún no existe.
  El esquema de la tabla se conservará a menos que una de `extend_schema` o una `recreate_schema` propiedad de ingesta esté establecida en `true` . Si se modifica el esquema, esto sucede antes de la ingesta de datos real en su propia transacción, por lo que un error al introducir los datos no significa que el esquema no se haya modificado.
* `.set-or-append``.append`los comandos y conservarán el esquema a menos que la `extend_schema` propiedad de ingesta esté establecida en `true` . Si se modifica el esquema, esto sucede antes de la ingesta de datos real en su propia transacción, por lo que un error al introducir los datos no significa que el esquema no se haya modificado.
* Se **recomienda encarecidamente** que los datos para la ingesta se limiten a menos de 1 GB por operación de ingesta. Se pueden usar varios comandos de ingesta, si es necesario.
* La ingesta de datos es una operación que consume muchos recursos que puede afectar a las actividades simultáneas en el clúster, incluidas las consultas en ejecución. Evite ejecutar "demasiados" comandos de este tipo al mismo tiempo.
* Al hacer coincidir el esquema del conjunto de resultados con el de la tabla de destino, la comparación se basa en los tipos de columna. No hay ninguna coincidencia de nombres de columna, por lo que debe asegurarse de que las columnas de esquema del resultado de la consulta están en el mismo orden que la tabla; de lo contrario, los datos se inscribirán en la columna equivocada.
* Establecer la `distributed` marca en `true` es útil cuando la cantidad de datos que se producen en la consulta es grande (supera los 1 GB de datos) **y** la consulta no requiere serialización (para que varios nodos puedan producir resultados en paralelo).
  Cuando los resultados de la consulta son pequeños, no se recomienda usar esta marca, ya que puede generar una gran cantidad de particiones de datos pequeñas innecesariamente.

**Ejemplos** 

Cree una nueva tabla denominada "RecentErrors" en la base de datos actual que tenga el mismo esquema que "LogsTable" y contenga todos los registros de error de la última hora:

```kusto
.set RecentErrors <|
   LogsTable
   | where Level == "Error" and Timestamp > now() - time(1h)
```

Cree una nueva tabla denominada "OldExtents" en la base de datos actual que tenga una sola columna ("ExtentId") y que contenga los identificadores de extensión de todas las extensiones de la base de datos que se han creado hace más de 30 días, en función de una tabla existente denominada "extensiones":

```kusto
.set async OldExtents <| 
   MyExtents 
   | where CreatedOn < now() - time(30d) 
   | project ExtentId     
```

Anexar datos a una tabla existente denominada "OldExtents" en la base de datos actual que tiene una sola columna ("ExtentId") y contiene los identificadores de extensión de todas las extensiones de la base de datos que se han creado hace más de 30 días, mientras se etiqueta la nueva extensión con etiquetas `tagA` y `tagB` , basándose en una tabla existente denominada "extensiones":

```kusto
.append OldExtents with(tags='["TagA","TagB"]') <| 
   MyExtents 
   | where CreatedOn < now() - time(30d) 
   | project ExtentId     
```
 
Anexe los datos a la tabla "OldExtents" de la base de datos actual (o cree la tabla si aún no existe), mientras se etiqueta la nueva extensión con `ingest-by:myTag` . Hágalo solo si la tabla ya no contiene una extensión etiquetada con `ingest-by:myTag` , basada en una tabla existente denominada "extensiones":

```kusto
.set-or-append async OldExtents with(tags='["ingest-by:myTag"]', ingestIfNotExists='["myTag"]') <| 
   MyExtents 
   | where CreatedOn < now() - time(30d) 
   | project ExtentId     
```

Reemplace los datos de la tabla "OldExtents" en la base de datos actual (o cree la tabla si aún no existe), mientras se etiqueta la nueva extensión con `ingest-by:myTag` .

```kusto
.set-or-replace async OldExtents with(tags='["ingest-by:myTag"]', ingestIfNotExists='["myTag"]') <| 
   MyExtents 
   | where CreatedOn < now() - time(30d) 
   | project ExtentId     
```

Anexe los datos a la tabla "OldExtents" de la base de datos actual, mientras establece la hora de creación de la extensión (s) creada en una fecha y hora específica en el pasado:

```kusto
.append async OldExtents with(creationTime='2017-02-13T11:09:36.7992775Z') <| 
   MyExtents 
   | where CreatedOn < now() - time(30d) 
   | project ExtentId     
```

**Devolver salida**
 
Devuelve información sobre las extensiones creadas como resultado del `.set` `.append` comando o.

**Salida del ejemplo**

|ExtentId |OriginalSize |ExtentSize |CompressedSize |IndexSize |RowCount | 
|--|--|--|--|--|--|
|23a05ed6-376D-4119-b1fc-6493bcb05563 |1291 |5882 |1568 |4314 |10 |