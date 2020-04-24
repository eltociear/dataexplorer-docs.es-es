---
title: Biblioteca de cliente de ingestión de Kusto - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe la biblioteca de cliente de Ingesta de Kusto en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/24/2020
ms.openlocfilehash: 0fd86579f4ca6471903305ca9249b78bc03b8cdf
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81503135"
---
# <a name="kusto-ingest-client-library"></a>Biblioteca de clientes de Kusto Ingest

## <a name="overview"></a>Información general
La biblioteca Kusto.Ingest es una biblioteca de .NET 4.6.2 que permite enviar datos al servicio Kusto.
Kusto.Ingest toma dependencias en las siguientes bibliotecas y SDKs:

* ADAL para la autenticación de AAD
* Cliente de Azure Storage
* [TBD: lista completa de dependencias externas]

Los métodos de ingesta de Kusto se definen mediante la interfaz [IKustoIngestClient](kusto-ingest-client-reference.md#interface-ikustoingestclient) y permiten la ingesta de datos desde Stream, IDataReader, archivos locales y blobs de Azure en modos sincrónicos y asincrónicos.

## <a name="ingest-client-flavors"></a>Ingeriera sabores de cliente
Conceptualmente, hay dos sabores básicos del cliente Ingest: Queued y Direct.

### <a name="queued-ingestion"></a>Ingestión en cola
Definido por [IKustoQueuedIngestClient](kusto-ingest-client-reference.md#interface-ikustoqueuedingestclient), este modo limita la dependencia del código de cliente en el servicio Kusto. La ingesta se realiza mediante la publicación de un mensaje de ingesta de Kusto en una cola de Azure, que luego se adquiere del servicio Kusto Data Management (Ingestion). El cliente de ingesta creará cualquier artefacto de almacenamiento intermedio mediante los recursos asignados por el servicio de administración de datos de Kusto.

**Las ventajas del modo en cola** incluyen (pero no se limitan a):

* Desacoplamiento del proceso de ingesta de datos del servicio Kusto Engine
* Permite que las solicitudes de ingesta se conserven cuando el servicio Kusto Engine (o Ingestion) no está disponible
* Permite la agregación eficiente y controlable de los datos entrantes por el servicio Ingestion, mejorando el rendimiento
* Permite que el servicio Kusto Ingestion administre la carga de ingesta en el servicio Kusto Engine
* El servicio de ingestión de Kusto volverá a intentarlo según sea necesario en errores de ingesta transitorios (por ejemplo, limitación de XStore)
* Proporciona un mecanismo conveniente para realizar un seguimiento del progreso y el resultado de cada solicitud de ingesta

En el diagrama siguiente se describe la interacción del cliente de ingesta en cola con Kusto:

![texto alternativo](../images/queued-ingest.jpg "en cola-ingest")

### <a name="direct-ingestion"></a>Ingestión directa
Definido por IKustoDirectIngestClient, este modo fuerza la interacción directa con el servicio Kusto Engine. En este modo, el servicio Kusto Ingestion no modera ni administra los datos. Cada solicitud de ingesta en modo Directo `.ingest` se traduce finalmente al comando ejecutado directamente en el servicio Kusto Engine.
En el diagrama siguiente se describe la interacción del cliente de ingesta directa con Kusto:

![texto alternativo](../images/direct-ingest.jpg "ingesta directa")

> [!NOTE]
> El modo Directo no se recomienda para las soluciones de ingesta de grado de producción.

**Las ventajas del modo Directo** incluyen:

* Baja latencia (no hay agregación). Sin embargo, la baja latencia también se puede lograr con la ingesta en cola
* Cuando se utilizan métodos sincrónicos, la finalización del método indica el final de la operación de ingesta

**Las desventajas del modo Directo** incluyen:

* El código de cliente debe implementar cualquier lógica de reintento o control de errores
* Las ingestiones son imposibles cuando el servicio Kusto Engine no está disponible
* El código de cliente puede abrumar el servicio Kusto Engine con solicitudes de ingesta, ya que no es consciente de la capacidad del servicio Engine

## <a name="ingestion-best-practices"></a>Prácticas recomendadas de ingestión

### <a name="general"></a>General
[Las prácticas recomendadas](kusto-ingest-best-practices.md) de ingesta proporcionan COGs y POV de rendimiento sobre la ingesta.

### <a name="thread-safety"></a>Seguridad para subprocesos
Las implementaciones de Cliente de Ingesta de Kusto son seguras para subprocesos y están diseñadas para reutilizarse. No es necesario crear una `KustoQueuedIngestClient` instancia de clase para cada operación de ingesta o incluso varias. Se requiere `KustoQueuedIngestClient` una única instancia de por clúster de Kusto de destino por proceso de usuario. La ejecución de varias instancias es contraproducente y puede hacer el clúster de administración de datos.

### <a name="supported-data-formats"></a>Formatos de datos compatibles
Cuando use la ingesta nativa, si aún no existe, cargue los datos en uno o varios blobs de Azure Storage. Los formatos de blob admitidos actualmente se documentan en el tema [Formatos de datos admitidos.](https://docs.microsoft.com/azure/data-explorer/ingestion-supported-formats)

### <a name="schema-mapping"></a>Asignación de esquemas
[Las asignaciones](../../management/mappings.md) de esquema ayudan a enlazar de forma determinista los campos de datos de origen a las columnas de la tabla de destino.

## <a name="usage-and-further-reading"></a>Uso y lectura posterior

* Como se ha descrito anteriormente, la base recomendada para soluciones de ingesta sostenibles y de alta escala para Kusto debe ser **KustoQueuedIngestClient**.
* Para minimizar la carga innecesaria en el servicio Kusto, se recomienda utilizar una única instancia del cliente de Kusto Ingest (en cola o directo) por proceso por clúster de Kusto. La implementación del cliente de Kusto Ingest es segura para subprocesos y totalmente reentrante.

### <a name="ingestion-permissions"></a>Permisos de ingesta
* Permisos de ingesta de [Kusto](kusto-ingest-client-permissions.md) explica la configuración de permisos necesarios para una ingesta exitosa mediante el paquete Kusto.Ingest

### <a name="kustoingest-library-reference"></a>Referencia de la biblioteca Kusto.Ingest
* [Kusto.Ingest Client Reference](kusto-ingest-client-reference.md) contiene una referencia completa de kusto ingerir interfaces e implementaciones de cliente.<BR>Allí encontrará la información sobre cómo crear clientes de ingesta, aumentar las solicitudes de ingesta, gestionar el progreso de la ingesta y más
* Estado de la [operación Kusto.Ingest](kusto-ingest-client-status.md) explica las instalaciones de **KustoQueuedIngestClient** para el seguimiento del estado de la ingesta
* [Errores de Kusto.Ingest](kusto-ingest-client-errors.md) documenta errores y excepciones de Kusto Ingest Client
* [Kusto.Ingest Examples](kusto-ingest-client-examples.md) presenta fragmentos de código que demuestran varias técnicas de ingesta de datos en Kusto

### <a name="data-ingestion-rest-apis"></a>API REST de ingesta de datos
Ingestión de datos [sin Kusto.Ingest Library](kusto-ingest-client-rest.md) explica cómo implementar la ingesta de Kusto en cola mediante las API REST de Kusto y sin depender de la biblioteca Kusto.Ingest.

