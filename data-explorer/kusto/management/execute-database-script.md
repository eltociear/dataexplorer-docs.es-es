---
title: '. ejecutar script de base de datos: Azure Explorador de datos'
description: En este artículo se describe la funcionalidad de script de base de datos. Execute en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
ms.date: 06/14/2020
ms.openlocfilehash: 4eaa7e8cc6f3c0f321abb9744bfe1608521e7b0e
ms.sourcegitcommit: 8e097319ea989661e1958efaa1586459d2b69292
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/15/2020
ms.locfileid: "84784538"
---
# <a name="execute-database-script"></a>. ejecutar script de base de datos

Ejecuta el lote de comandos de control en el ámbito de una sola base de datos.

## <a name="syntax"></a>Sintaxis

`.execute` `database` `script`  
[ `with` `(` *PropertyName* `=` *PropertyValue* [ `,` ...] `)` ]   
`<|`  
 *Script de control de comandos*

### <a name="parameters"></a>Parámetros

* *Control-Commands-script*: texto con uno o más comandos de control.
* *Ámbito de base de datos*: el script se aplica en el *ámbito de base de datos* especificado como parte del contexto de la solicitud.

### <a name="optional-properties"></a>Propiedades opcionales

| Propiedad            | Tipo            | Descripción                          |
|---------------------|-----------------|---------------------------------------------------------------------------------------------------|
| `ContinueOnErrors`            | `bool`        | Si se establece en `false` , el script se detendrá en el primer error. Si se establece en `true` , la ejecución del script continúa. Valor predeterminado: `false`. |

## <a name="output"></a>Resultados

Cada comando que aparece en el script se informará como un registro independiente en la tabla de salida. Cada registro tiene los siguientes campos:

|Parámetro de salida |Tipo |Descripción
|---|---|--- 
|OperationId  |GUID |Identificador del comando.
|CommandType  |String |Tipo del comando.
|CommandText  |String |Texto del comando específico.
|Resultado|String|Resultado de la ejecución del comando específico.
|Motivo|String|Información detallada sobre el resultado de la ejecución del comando.

>[!NOTE]
>* El texto del script puede incluir líneas vacías y comentarios entre los comandos.
>* Los comandos se ejecutan secuencialmente, en el orden en que aparecen en el script de entrada.
>* La ejecución del script es no transaccional y no se realiza ninguna reversión cuando se produce un error. Se recomienda usar la forma idempotente de comandos cuando se usa `.execute database script` .
>* Comportamiento predeterminado del comando: error en el primer error, se puede cambiar mediante el argumento de propiedad.
>* Los comandos de control de solo lectura (comandos. Show) no se ejecutan y se registran con el estado `Skipped` .

## <a name="example"></a>Ejemplo

```kusto
.execute database script <|

// Create tables
.create-merge table T(a:string, b:string)

// Apply policies
.alter-merge table T policy retention softdelete = 10d 

// Create functions
.create-or-alter function
  with (skipvalidation = "true") 
  SampleT1(myLimit: long) { 
    T1 | limit myLimit
}
```

|OperationId|CommandType|CommandText|Resultado|Motivo|
|---|---|---|---|---|
|1d28531b-58c8-4023-a5d3-16fa73c06cfa|TableCreate|. Create: Merge Table T (a:String, b:String)|Completado||
|67d0ea69-baa4-419a-93d3-234c03834360|RetentionPolicyAlter|. Alter-Merge tabla T Directiva retención softdelete = 10D|Completado||
|0b0e8769-d4e8-4ff9-adae-071e52a650c7|FunctionCreateOrAlter|.create-or-alter function<br>with (skipvalidation = "true")<br>SampleT1 (allimit: Long) {<br>Límite de T1 \|<br>}|Completado||
