---
title: 'Funciones: Azure Data Explorer | Microsoft Docs'
description: En este artículo se describen las funciones de Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: 6c6d9aedee17592ac1eb1b43e93dead80eb9fc61
ms.sourcegitcommit: 4f576c1b89513a9e16641800abd80a02faa0da1c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/22/2020
ms.locfileid: "85128859"
---
# <a name="function-types"></a>Tipos de función

Las **funciones** son consultas o partes de consultas reutilizables. Kusto admite varios tipos de funciones:

* **Funciones almacenadas**: son funciones definidas por el usuario que se almacenan y administran en una clase de entidades de esquema de una base de datos.
  Consulte [Funciones almacenadas](../schema-entities/stored-functions.md).
* **Funciones definidas por consultas**: son funciones definidas por el usuario que se definen y se usan dentro del ámbito de una consulta individual. La definición de estas funciones se realiza mediante una [instrucción Let](../letstatement.md).
  Consulte [Funciones definidas por el usuario](./user-defined-functions.md).
* **Funciones integradas**: están codificadas de forma rígida (las define Kusto y los usuarios no pueden modificarlas).