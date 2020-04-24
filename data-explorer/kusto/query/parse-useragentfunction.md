---
title: parse_user_agent() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe parse_user_agent() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2020
ms.openlocfilehash: 2ebe85c622dda116366e30ba22e2e495e4cb76ac
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81511737"
---
# <a name="parse_user_agent"></a>parse_user_agent()

Interpreta una cadena de agente de usuario, que identifica el navegador del usuario y proporciona ciertos detalles del sistema a los servidores que alojan los sitios web que visita el usuario. El resultado se [`dynamic`](./scalar-data-types/dynamic.md)devuelve como . 

**Sintaxis**

`parse_user_agent(`*user-agent-string*, *look-for*`)`

**Argumentos**

* *user-agent-string*: Una `string`expresión de tipo , que representa una cadena de agente de usuario.

* *look-for*: Una `string` expresión `dynamic`de tipo o , que representa lo que la función debe buscar en la cadena de agente de usuario (destino de análisis). Las opciones posibles: "navegador", "os", "dispositivo". Si solo se requiere un único destino de `string` análisis, se le puede pasar un parámetro.
Si se requieren dos o tres, `dynamic array`se pueden pasar como un archivo .

**Devuelve**

Objeto de `dynamic` tipo que contiene la información sobre los destinos de análisis solicitados.

Navegador: Familia, MajorVersion, MinorVersion, Parche                 

OperatingSystem: Familia, MajorVersion, MinorVersion, Patch, PatchMinor             

Dispositivo: Familia, Marca, Modelo

> [!WARNING]
> La implementación de la función se basa en comprobaciones regex de la cadena de entrada con un gran número de patrones predefinidos. Por lo tanto, el tiempo esperado y el consumo de CPU es alto.
Cuando la función se utiliza en una consulta, asegúrese de que se ejecuta de forma distribuida en varios equipos.
Si las consultas con esta función se utilizan con frecuencia, es posible que desee crear previamente los resultados a través de la directiva de [actualización,](../management/updatepolicy.md)pero debe tener en cuenta que el uso de esta función dentro de la directiva de actualización aumentará la latencia de ingesta.
 
**Ejemplo**

```kusto
print useragent = "Mozilla/5.0 (Windows; U; en-US) AppleWebKit/531.9 (KHTML, like Gecko) AdobeAIR/2.5.1"
| extend x = parse_user_agent(useragent, "browser") 
```

El resultado esperado es un objeto dinámico:

•Navegador: "Familia": "AdobeAIR", "MajorVersion": "2", "MinorVersion": "5", "Patch": "1"

```kusto
print useragent = "Mozilla/5.0 (SymbianOS/9.2; U; Series60/3.1 NokiaN81-3/10.0.032 Profile/MIDP-2.0 Configuration/CLDC-1.1 ) AppleWebKit/413 (KHTML, like Gecko) Safari/4"
| extend x = parse_user_agent(useragent, dynamic(["browser","os","device"])) 
```

El resultado esperado es un objeto dinámico:

•Navegador: "Familia": "Navegador OSS de Nokia", "MajorVersion": "3", "MinorVersion": "1", "Patch": "", "OperatingSystem": "Family": "Symbian OS", "MajorVersion": "9", "MinorVersion": "2", "Patch": "", "PatchMinor": "" ", "Device": "Family": "Nokia N81", "Brand": "Nokia", "