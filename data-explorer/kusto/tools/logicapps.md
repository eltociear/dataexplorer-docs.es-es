---
title: Usar Logic Apps para ejecutar consultas Kusto automáticamente
description: Aprenda a usar Logic Apps para ejecutar consultas y comandos de Kusto automáticamente y programarlos
author: orspod
ms.author: orspodek
ms.reviewer: docohe
ms.service: data-explorer
ms.topic: conceptual
ms.date: 04/14/2020
ms.openlocfilehash: 8765635e0eea8c1d41640bc0393d39a0afa5f971
ms.sourcegitcommit: e66c5f4b833b4f6269bb7bfa5695519fcb11d9fa
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2020
ms.locfileid: "83630180"
---
# <a name="microsoft-logic-app-and-azure-data-explorer"></a>Microsoft Logic App y Azure Explorador de datos

El conector de Azure Kusto Logic apps permite ejecutar consultas y comandos de Kusto automáticamente como parte de una tarea programada o desencadenada mediante el conector de [Microsoft Logic App](https://docs.microsoft.com/azure/logic-apps/logic-apps-what-are-logic-apps) .

Logic apps y Flow se basan en el mismo conector. Por lo tanto, las [limitaciones](flow.md#limitations), [acciones](flow.md#azure-kusto-flow-actions), [autenticación](flow.md#authentication) y [ejemplos de uso](flow.md#azure-kusto-flow-actions) que se aplican a Flow también se aplican a Logic Apps, como se mencionó en la página de [documentación de Flow](flow.md).

## <a name="how-to-create-a-logic-app-with-azure-data-explorer"></a>Creación de una aplicación lógica con Azure Explorador de datos

1. Abra el [Microsoft Azure portal](https://ms.portal.azure.com/). 
1. Búsquelo `logic app` y selecciónelo.

    [![](./Images/logicapps/logicapp-search.png "Search for logic app")](./Images/logicapps/logicapp-search.png#lightbox)

1. Seleccione **+Agregar**.

    ![Agregar aplicación lógica](./Images/logicapps/logicapp-add.png)

1. Escriba los detalles necesarios del formulario:
    * Subscription
    * Resource group
    * Nombre de la aplicación lógica
    * Región o Entorno del servicio de integración
    * Location
    * Activar o desactivar el análisis de registros
1. Seleccione **Revisar + crear**.

    ![Creación de la aplicación lógica](./Images/logicapps/logicapp-create-new.png)

1. Cuando se cree la aplicación lógica, seleccione **Editar**.

    ![Editar el diseñador de aplicaciones lógicas](./Images/logicapps/logicapp-editdesigner.png "logicapp-editdesigner")

1. Cree una aplicación lógica en blanco.

    ![Plantilla en blanco de aplicación lógica](./Images/logicapps/logicapp-blanktemplate.png "logicapp-blanktemplate")

1. Agregue la acción de periodicidad y seleccione **Azure Kusto**.

    ![Conector de flujo de Kusto de Logic apps](./Images/logicapps/logicapp-kustoconnector.png "logicapp-kustoconnector")

## <a name="next-steps"></a>Pasos siguientes

* Para obtener más información sobre la configuración de una acción de periodicidad, consulte la [Página de documentación de Flow](flow.md) .
* Eche un vistazo a algunos [ejemplos de uso](flow.md#azure-kusto-flow-actions) para obtener ideas sobre cómo configurar las acciones de la aplicación lógica
