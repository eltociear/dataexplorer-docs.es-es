---
title: Bases de datos - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describen las bases de datos en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/30/2019
ms.openlocfilehash: a94b168d8aa8443251f3a01dc659e114b0aacaf5
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81509442"
---
# <a name="databases"></a>Bases de datos

Kusto sigue un modelo de relación de almacenamiento de `database`los datos donde la entidad de nivel superior es un archivo . 

El clúster de Kusto puede hospedar varias bases de datos, donde cada base de datos hospedará su propia colección de [tablas,](tables.md) [funciones almacenadas](stored-functions.md)y [tablas externas.](externaltables.md)
Cada base de datos tiene su propio conjunto de permisos, basado en el modelo de control de acceso basado en roles [(RBAC)](../../management/access-control/index.md)

**Notas**  

* Los nombres de base de datos no distinguen mayúsculas de minúsculas.
* Los nombres de base de datos deben seguir las reglas de los nombres de [entidad.](./entity-names.md)
* El límite máximo de bases de datos por clúster es de 10.000.