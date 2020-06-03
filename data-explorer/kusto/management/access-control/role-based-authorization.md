---
title: 'Autorización basada en roles en Kusto: Azure Explorador de datos'
description: En este artículo se describe la autorización basada en roles en Kusto en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/14/2020
ms.openlocfilehash: 8e961d389b365d77c7dddd557a28a158add2e3f3
ms.sourcegitcommit: f7101c6b41ec250d05f4cb6092e2939958b37b40
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/03/2020
ms.locfileid: "84329085"
---
# <a name="role-based-authorization-in-kusto"></a>Autorización basada en roles en Kusto

La **autorización** es el proceso de permitir o denegar el permiso de una entidad de seguridad para llevar a cabo una acción.
Kusto usa un modelo **de control de acceso basado en roles** , en el que las entidades de seguridad autenticadas se asignan a los **roles**y obtienen acceso según los roles a los que están asignadas.

El servicio de **motor Kusto** tiene los siguientes roles:

|Role                       |Permisos                                                                                                                                                  |
|---------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------|
|Administrador de todas las bases de datos        |Puede hacer todo en el ámbito de cualquier base de datos. Puede mostrar y modificar ciertas directivas de nivel de clúster                                                               |
|Administrador de base de datos             |Puede hacer todo en el ámbito de una base de datos determinada                                                                                                         |
|Usuario de la base de datos              |Puede leer todos los datos y metadatos de la base de datos. Además, puede crear tablas y convertirse en el administrador de tablas para esas tablas y crear funciones en la base de datos.|
|Visor de todas las bases de datos       |Puede leer todos los datos y metadatos de cualquier base de datos                                                                                                               |
|Visor de base de datos            |Puede leer todos los datos y metadatos de una base de datos determinada                                                                                                       |
|Agente de ingesta de base de datos          |Puede ingerir datos a todas las tablas existentes en la base de datos, pero no puede consultar los datos.                                                                             |
|Visor sin restricciones de la base de datos|Puede consultar todas las tablas de la base de datos que tienen habilitada la [Directiva RestrictedViewAccess](../restrictedviewaccess-policy.md)                                |
|Supervisor de base de datos           |Puede ejecutar `.show` comandos en el contexto de la base de datos y sus entidades secundarias.                                                                           |
|Administrador de funciones             |Puede modificar funciones, eliminar funciones o conceder permisos de administrador a otra entidad de seguridad                                                                         |
|Administrador de tablas                |Puede hacer todo en el ámbito de una tabla determinada.                                                                                                           |
|Agente de ingesta de tablas             |Puede ingerir datos en el ámbito de una tabla determinada, pero no puede consultar los datos.                                                                                 |
