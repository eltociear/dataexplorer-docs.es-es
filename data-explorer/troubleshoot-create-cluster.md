---
title: Solución de problemas de errores de creación de un clúster en Azure Data Explorer
description: En este artículo se describen los pasos de solución de problemas para crear un clúster en el Explorador de datos de Azure.
author: orspod
ms.author: orspodek
ms.reviewer: mblythe
ms.service: data-explorer
ms.topic: conceptual
ms.date: 09/24/2018
ms.openlocfilehash: 863d9ecedf095e0ab284a0d7fd86363b69ae5658
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81494104"
---
# <a name="troubleshoot-failed-cluster-creation-of-azure-data-explorer"></a>Solución de problemas: error al crear un clúster en Azure Data Explorer

En el improbable caso de que la creación de un clúster en el Explorador de datos de Azure produzca un error, siga estos pasos.

1. Asegúrese de que tiene los permisos adecuados. Para crear un clúster, debe ser miembro del rol *colaborador* o *propietario* en la suscripción de Azure. Si es necesario, hable con su administrador de suscripciones para que pueda agregarle el rol adecuado.

1. Asegúrese de que no haya ningún error de validación relacionado con el nombre del clúster que ha escrito en **Crear clúster** en Azure Portal.

1. Compruebe el [panel de mantenimiento de los servicios de Azure](https://azure.microsoft.com/status/). Busque el estado del Explorador de datos de Azure en la región donde intenta crear el clúster.

    Si el estado no es **Bueno** (marca de verificación verde), pruebe a crear el clúster una vez que mejore el estado.

1. Si aún necesita ayuda para solucionar el problema, abra una solicitud de soporte técnico en [Azure Portal](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview).
