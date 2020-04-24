---
title: Autorización basada en roles en Kusto - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe la autorización basada en roles en Kusto en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/14/2020
ms.openlocfilehash: fe7013e3ee4b842dc2dcb2e251c342897fa4e489
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81522634"
---
# <a name="role-based-authorization-in-kusto"></a>Autorización basada en roles en Kusto



**La autorización** es el proceso de permitir o no permitir que una entidad de seguridad realice una acción.
Kusto usa un modelo de control de **acceso basado en roles,** en el que las entidades de seguridad autenticadas se asignan a **roles**y obtienen acceso según los roles que se les asignan.

El servicio **Kusto Engine** tiene los siguientes roles:

|Role                       |Permisos                                                                                                                                                  |
|---------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------|
|Administración de todas las bases de datos        |Puede hacer "cualquier cosa" en el ámbito de cualquier base de datos; Puede mostrar y modificar ciertas directivas de nivel de clúster.                                                           |
|Administrador de base de datos             |Puede hacer "cualquier cosa" en el ámbito de una base de datos determinada.                                                                                                     |
|Usuario de la base de datos              |Puede leer todos los datos y metadatos de la base de datos; además, puede crear tablas (convirtiéndose así en el administrador de tabla spara esa tabla) y funciones en la base de datos.|
|Visor de todas las bases de datos       |Puede leer todos los datos y metadatos de cualquier base de datos.                                                                                                              |
|Visor de base de datos            |Puede leer todos los datos y metadatos de una base de datos determinada.                                                                                                     |
|Agente de ingesta de base de datos          |Puede ingerir datos de todas las tablas existentes de la base de datos, pero no consultar los datos.                                                                              |
|Visor sin restricciones de la base de datos|Puede consultar todas las tablas de la base de datos que tienen habilitada la [directiva RestrictedViewAccess.](../restrictedviewaccess-policy.md)                                |
|Supervisor de base de datos           |Puede `.show` ejecutar comandos en el contexto de la base de datos y sus entidades secundarias.                                                                          |
|Administrador de funciones             |Puede modificar la función, eliminar la función o conceder permisos de administrador a otra entidad de seguridad.                                                                         |
|Administrador de tablas                |Puede hacer cualquier cosa en el ámbito de una tabla determinada.                                                                                                          |
|Agente de ingesta de tablas             |Puede ingerir datos en el ámbito de una determinada tabla, pero no consultar los datos.                                                                                  |
