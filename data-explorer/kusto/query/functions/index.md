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
ms.openlocfilehash: f1307b54c9f0b7a948925dd4eaa4d1f2e89d8070
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81490317"
---
# <a name="functions"></a>Functions

Las **funciones** son consultas o partes de consultas reutilizables. Kusto admite varios tipos de funciones:

* **Funciones almacenadas**: son funciones definidas por el usuario que se almacenan y administran en una clase de entidades de esquema de una base de datos.
  Consulte [Funciones almacenadas](../schema-entities/stored-functions.md).
* **Funciones definidas por consultas**: son funciones definidas por el usuario que se definen y se usan dentro del ámbito de una consulta individual. La definición de estas funciones se realiza mediante una [instrucción Let](../letstatement.md).
  Consulte [Funciones definidas por el usuario](./user-defined-functions.md).
* **Funciones integradas**: están codificadas de forma rígida (las define Kusto y los usuarios no pueden modificarlas).