---
title: Microsoft Logic App y Kusto - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe Microsoft Logic App y Kusto en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/30/2019
ms.openlocfilehash: f7d719ece5df6eb3f6d4060a2fb07e7092902601
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81523824"
---
# <a name="microsoft-logic-app-and-kusto"></a>Microsoft Logic App y Kusto

El conector de Azure Kusto Logic App permite a los usuarios ejecutar consultas y comandos de Kusto automáticamente como parte de una tarea programada o desencadenada, mediante [Microsoft Logic App](https://docs.microsoft.com/azure/logic-apps/logic-apps-what-are-logic-apps).

La aplicación lógica y los conectores de flujo se crean encima del mismo conector, por lo tanto, se aplican las mismas [limitaciones,](flow.md#limitations) [acciones,](flow.md#azure-kusto-flow-actions) [autenticación](flow.md#authentication) y [ejemplos](flow.md#usage-examples) de uso para ambos como se menciona en la página de [documentación de flujo](flow.md).


## <a name="how-to-create-a-logic-app-with-azure-kusto"></a>Cómo crear una aplicación lógica con Azure Kusto

Abra Azure Portal y haga clic en crear un nuevo recurso de aplicación lógica.
Agregue el nombre, la suscripción, el grupo de recursos y la ubicación deseados y haga clic en crear.

![Creación de la aplicación lógica](./Images/KustoTools-LogicApp/logicapp-createlogicapp.png "logicapp-createlogicapp")

Después de crear la aplicación lógica, haga clic en el botón de edición

![Editar diseñador de aplicaciones lógicas](./Images/KustoTools-LogicApp/logicapp-editdesigner.png "logicapp-editdesigner")

Crear una aplicación lógica en blanco

![Plantilla en blanco de la aplicación lógica](./Images/KustoTools-LogicApp/logicapp-blanktemplate.png "logicapp-blanktemplate")

Agregue la acción de periodicidad y, a continuación, elija 'Azure Kusto'

![Conector Kusto Flow de la aplicación lógica](./Images/KustoTools-LogicApp/logicapp-kustoconnector.png "logicapp-kustoconnector")