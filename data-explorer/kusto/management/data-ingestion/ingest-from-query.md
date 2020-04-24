---
title: Ingerir de la consulta (.set, .append, .set-or-append, .set-or-replace) - Explorador de Azure Data Explorer ? Microsoft Docs
description: En este artículo se describe Ingerir desde la consulta (.set, .append, .set-or-append, .set-or-replace) en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/30/2020
ms.openlocfilehash: fb0bbb06bb8d28dd3951a7faedf55b0d84155301
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81521461"
---
# <a name="ingest-from-query-set-append-set-or-append-set-or-replace"></a>Ingerir de la consulta (.set, .append, .set-or-append, .set-or-replace)

Estos comandos ejecutan una consulta o un comando de control e ingenian los resultados de la consulta en una tabla. La diferencia entre estos comandos es cómo tratan tablas y datos existentes o inexistentes:

|Get-Help          |Si la tabla existe                     |Si la tabla no existe                    |
|-----------------|------------------------------------|------------------------------------------|
|`.set`           |El comando falla.                  |Se crea la tabla y se ingenten datos.|
|`.append`        |Los datos se anexan a la tabla.      |El comando falla.                        |
|`.set-or-append` |Los datos se anexan a la tabla.      |Se crea la tabla y se ingenten datos.|
|`.set-or-replace`|Los datos reemplazan los datos de la tabla.|Se crea la tabla y se ingenten datos.|

**Sintaxis**

`.set`[`async`] *TableName* [`with` `(` *PropertyName* `=` *PropertyValue* [`,` ...] `)`] `<|` *QueryOrCommand*

`.append`[`async`] *TableName* [`with` `(` *PropertyName* `=` *PropertyValue* [`,` ... `])`] `<|` *QueryOrCommand*

`.set-or-append`[`async`] *TableName* [`with` `(` *PropertyName* `=` *PropertyValue* [`,` ...] `)`] `<|` *QueryOrCommand*

`.set-or-replace`[`async`] *TableName* [`with` `(` *PropertyName* `=` *PropertyValue* [`,` ...] `)`] `<|` *QueryOrCommand*

**Argumentos**

* `async`: Si se especifica, el comando volverá inmediatamente y continuará la ingesta en segundo plano. Los resultados del comando `OperationId` incluirán un valor que, a continuación, se puede utilizar con el `.show operation` comando para recuperar el estado de finalización de la ingesta y los resultados.
* *TableName*: el nombre de la tabla en la que se ingiren datos.
  El nombre de la tabla siempre es relativo a la base de datos en contexto.
* *PropertyName*, *PropertyValue*: cualquier número de propiedades de ingesta que afecten al proceso de ingesta.

 Propiedades de ingesta admitidas:

|Propiedad.        |Descripción|
|----------------|-----------------------------------------------------------------------------------------------------------------------------|
|`creationTime`   | El valor datetime (formateado como una cadena ISO8601) que se usará en el momento de la creación de las extensiones de datos ingeridos. Si no se especifica, se utilizará el valor actual (now()).|
|`extend_schema`  | Valor booleano que, si se especifica, indica al comando que extienda el esquema de la tabla (valor predeterminado es false). Esta opción solo se aplica a los comandos .append .set-or-append y set-or-replace. Las únicas extensiones de esquema permitidas tienen columnas adicionales agregadas a la tabla al final.|
|`recreate_schema`  | Valor booleano que, si se especifica, describe si el comando puede volver a crear el esquema de la tabla (valor predeterminado es false). Esta opción solo se aplica al comando set-or-replace. Esta opción tiene prioridad sobre la propiedad extend_schema si se establecen ambas.|
|`folder`         | La carpeta que se va a asignar a la tabla. Si la tabla ya existe, esta propiedad invalidará la carpeta de la tabla.|
|`ingestIfNotExists`   | Valor de cadena que, si se especifica, impide que la ingesta se realice correctamente si la tabla ya tiene datos etiquetados con una etiqueta ingest-by: con el mismo valor.|
|`policy_ingestiontime`   | valor booleano que, si se especifica, describe si se habilita la [directiva de tiempo de ingesta](../../management/ingestiontime-policy.md) en una tabla que este comando crea. El valor predeterminado es true.|
|`tags`   | cadena JSON que indica las validaciones que se ejecutan durante la ingesta.|
|`docstring`   | Una cadena que documenta la tabla.|

  Además, hay una propiedad que controla el comportamiento del propio comando:

|Propiedad        |Tipo    |Descripción|
|----------------|--------|-----------------------------------------------------------------------------------------------------------------------------|
|`distributed`   |`bool`  |Indica que el comando se ingesta de todos los nodos que ejecutan la consulta en paralelo. (El valor `false`predeterminado es .)  Vea los comentarios a continuación.|

* *QueryOrCommand*: el texto de una consulta o un comando de control cuyos resultados se utilizarán como datos para ingerir.

> [!NOTE]
> Solo `.show` se admiten comandos de control.

**Comentarios:**

* `.set-or-replace`reemplaza los datos de la tabla si existen (elimina las particiones de datos existentes) o crea la tabla de destino si aún no existe.
  El esquema de tabla se `extend_schema` conservará `recreate_schema` a menos que `true`una de las propiedades o ingestion esté establecida en . Si se modifica el esquema, esto sucede antes de la ingesta de datos real en su propia transacción, por lo que un error al ingerir los datos no significa que el esquema no se haya modificado.
* `.set-or-append`y `.append` los comandos `extend_schema` conservarán el esquema `true`a menos que la propiedad ingestion esté establecida en . Si se modifica el esquema, esto sucede antes de la ingesta de datos real en su propia transacción, por lo que un error al ingerir los datos no significa que el esquema no se haya modificado.
* Se **recomienda encarecidamente** que los datos para la ingesta se limiten a menos de 1 GB por operación de ingesta. Si es necesario, se pueden utilizar varios comandos de ingesta.
* La ingesta de datos es una operación que consume muchos recursos y que puede afectar a las actividades simultáneas en el clúster, incluidala la ejecución de consultas. Evite ejecutar "demasiados" estos comandos juntos al mismo tiempo.
* Al hacer coincidir el esquema del conjunto de resultados con el de la tabla de destino, la comparación se basa en los tipos de columna. No hay coincidencia de nombres de columna, así que asegúrese de que las columnas de esquema de resultados de consulta están en el mismo orden que la tabla, de lo contrario los datos se ingerirán en la columna incorrecta.
* Establecer `distributed` la `true` marca en es útil cuando la cantidad de datos generados por la consulta es grande (supera 1 GB de datos) **y** la consulta no requiere serialización (para que varios nodos pueden producir resultados en paralelo).
  Cuando los resultados de la consulta son pequeños, no se recomienda usar esta marca, ya que podría generar una gran cantidad de particiones de datos pequeñas innecesariamente.

**Ejemplos** 

Cree una nueva tabla denominada "RecentErrors" en la base de datos actual que tenga el mismo esquema que "LogsTable" y contenga todos los registros de error de la última hora:

```kusto
.set RecentErrors <|
   LogsTable
   | where Level == "Error" and Timestamp > now() - time(1h)
```

Cree una nueva tabla denominada "OldExtents" en la base de datos actual que tenga una sola columna ("ExtentId") y contenga los identificadores de extensión de todas las extensiones de la base de datos que se han creado hace más de 30 días, basándose en una tabla existente denominada "MyExtents":

```kusto
.set async OldExtents <| 
   MyExtents 
   | where CreatedOn < now() - time(30d) 
   | project ExtentId     
```

Anexe datos a una tabla existente denominada "OldExtents" en la base de datos actual que tiene una sola columna ("ExtentId") y contiene los identificadores de `tagA` extensión `tagB`de todas las extensiones de la base de datos que se han creado hace más de 30 días, al etiquetar la nueva extensión con etiquetas y , en función de una tabla existente denominada "MyExtents":

```kusto
.append OldExtents with(tags='["TagA","TagB"]') <| 
   MyExtents 
   | where CreatedOn < now() - time(30d) 
   | project ExtentId     
```
 
Anexe datos a la tabla "OldExtents" de la base de datos actual (o cree `ingest-by:myTag`la tabla si aún no existe), mientras etiqueta la nueva extensión con . Hágalo solo si la tabla aún no `ingest-by:myTag`contiene una extensión etiquetada con , basada en una tabla existente denominada "MyExtents":

```kusto
.set-or-append async OldExtents with(tags='["ingest-by:myTag"]', ingestIfNotExists='["myTag"]') <| 
   MyExtents 
   | where CreatedOn < now() - time(30d) 
   | project ExtentId     
```

Reemplace los datos de la tabla "OldExtents" de la base de datos actual (o cree `ingest-by:myTag`la tabla si aún no existe), mientras etiqueta la nueva extensión con .

```kusto
.set-or-replace async OldExtents with(tags='["ingest-by:myTag"]', ingestIfNotExists='["myTag"]') <| 
   MyExtents 
   | where CreatedOn < now() - time(30d) 
   | project ExtentId     
```

Anexe datos a la tabla "OldExtents" de la base de datos actual, mientras establece la hora de creación de las extensiones creadas en una fecha y hora específica en el pasado:

```kusto
.append async OldExtents with(creationTime='2017-02-13T11:09:36.7992775Z') <| 
   MyExtents 
   | where CreatedOn < now() - time(30d) 
   | project ExtentId     
```

**Salida de retorno**
 
Devuelve información sobre las extensiones creadas como resultado del comando `.set` o. `.append`

**Salida del ejemplo**

|ExtentId |OriginalSize |ExtentSize |CompressedSize |IndexSize |RowCount | 
|--|--|--|--|--|--|
|23a05ed6-376d-4119-b1fc-6493bcb05563 |1291 |5882 |1568 |4314 |10 |