---
title: Solución de problemas de errores de conexión del clúster de Azure Data Explorer
description: En este artículo se describen los pasos de solución de problemas para conectarse a un clúster en Azure Data Explorer.
author: orspod
ms.author: orspodek
ms.reviewer: mblythe
ms.service: data-explorer
ms.topic: conceptual
ms.date: 09/24/2018
ms.openlocfilehash: c71af799f614e9cd28221d79634666cbc3b2c987
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81492344"
---
# <a name="troubleshoot-failure-to-connect-to-a-cluster-in-azure-data-explorer"></a>Solución de problemas: error al conectarse a un clúster en el Explorador de datos de Azure

Si no es capaz de conectarse a un clúster en el Explorador de datos de Azure, siga estos pasos.

1. Asegúrese de que la cadena de conexión sea correcta. Debe tener el formato: `https://<ClusterName>.<Region>.kusto.windows.net`, como en el ejemplo siguiente: `https://docscluster.westus.kusto.windows.net`.

1. Asegúrese de que tiene los permisos adecuados. Si no es así, obtendrá una respuesta de *no autorizado*.

    Para más información acerca de los permisos, consulte [Administración de permisos de bases de datos](manage-database-permissions.md). Si es necesario, hable con su administrador de clústeres para que pueda agregarlo al rol adecuado.

1. Compruebe que el clúster todavía no se ha eliminado: revise el registro de actividad en su suscripción.

1. Compruebe el [panel de mantenimiento de los servicios de Azure](https://azure.microsoft.com/status/). Busque el estado del Explorador de datos de Azure en la región donde intenta conectarse a un clúster.

    Si el estado no es **Bueno** (marca de verificación verde), intente conectarse al clúster cuando mejore el estado.

1. Si aún necesita ayuda para solucionar el problema, abra una solicitud de soporte técnico en [Azure Portal](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview).