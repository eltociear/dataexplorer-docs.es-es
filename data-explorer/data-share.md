---
title: Uso de Azure Data Share para compartir datos con Azure Data Explorer (versión preliminar)
description: Aprenda a compartir datos con Azure Data Explorer y Azure Data Share.
author: manojraheja
ms.author: maraheja
ms.reviewer: orspodek
ms.service: data-explorer
ms.topic: conceptual
ms.date: 04/23/2020
ms.openlocfilehash: c935b6f888d40cfcd2208e8857dc3d794f9a50cb
ms.sourcegitcommit: b4d6c615252e7c7d20fafd99c5501cb0e9e2085b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2020
ms.locfileid: "83865931"
---
# <a name="use-azure-data-share-to-share-data-with-azure-data-explorer-preview"></a>Uso de Azure Data Share para compartir datos con Azure Data Explorer (versión preliminar)

Hay muchas formas tradicionales de compartir datos, como a través de recursos compartidos de archivos, FTP, correo electrónico y API. Estos métodos requieren que las dos partes creen y mantengan una canalización de datos que mueva los datos entre los equipos y organizaciones. Con Azure Data Explorer, puede compartir sus datos de forma fácil y segura tanto con el personal de su empresa como con asociados externos. El uso compartido se produce casi en tiempo real, y sin necesidad de crear ni mantener una canalización de datos. Todos los cambios en la base de datos, incluidos el esquema y los datos, que se realizan en el lado del proveedor están disponibles al instante en el lado del consumidor.

[![Vídeo de Azure Friday](https://img.youtube.com/vi/Q3MJv90PegE/0.jpg)](https://www.youtube.com/watch?v=Q3MJv90PegE?&autoplay=1)

Azure Data Explorer desacopla el almacenamiento y el proceso, lo que permite a los clientes ejecutar varias instancias del proceso (solo lectura) en el mismo almacenamiento subyacente. Puede adjuntar una base de datos como [base de datos seguidora](follower.md), que es una base de datos de solo lectura en un clúster remoto.

## <a name="configure-data-sharing"></a>Configuración del uso compartido de datos 

Para configurar el uso compartido de datos se puede usar cualquiera de los siguientes métodos:

* Use [Azure Data Share](/azure/data-share/) para enviar y administrar invitaciones y recursos compartidos en toda la compañía o con clientes y asociados externos. Azure Data Share usa una [base de datos de seguidores](follower.md) para crear un vínculo simbólico entre el clúster de Azure Data Explorer del proveedor y el del consumidor. Esta opción proporciona un único panel para ver y administrar todos los recursos compartidos de datos en los clústeres de Azure Data Explorer y otros servicios de datos. Azure Data Share también permite compartir datos entre organizaciones de distintos inquilinos de Azure Active Directory.
* Configure directamente la [base de datos de seguidores](follower.md) solo para aquellos escenarios en que necesite un proceso adicional para escalar horizontalmente para las necesidades de generación de informes y usted sea el administrador en ambos clústeres.

> [!Note] 
> Cuando se establece la relación de uso compartido, Azure Data Share crea un vínculo simbólico entre el clúster de Azure Data Explorer del consumidor y del proveedor. Si el proveedor de datos revoca el acceso, se elimina el vínculo simbólico y las bases de datos compartidas dejan de estar disponibles para el consumidor de datos.

:::image type="content" source="media/data-share/adx-datashare-image.png" alt-text="Uso compartido de datos de Azure Data Explorer":::

El proveedor de datos puede compartir los datos en el nivel de base de datos o en el nivel de clúster. El clúster que comparte la base de datos es el clúster responsable, mientras que el que la recibe es el clúster seguidor. Los clústeres seguidores pueden seguir una o varias bases de datos de clústeres responsables. El clúster seguidor se sincroniza periódicamente para comprobar si hay cambios. El tiempo de retardo entre ambos clústeres varía entre unos segundos y unos minutos, en función del tamaño total de los metadatos y los datos. Los datos se almacenan en la caché del clúster consumidor y solo están disponibles para las operaciones de lectura o consulta, con una excepción, para invalidar la directiva de almacenamiento en caché activo y los permisos de base de datos. Las consultas que se ejecutan en el clúster seguidor usan la caché local y no usan los recursos del clúster responsable.

## <a name="prerequisites"></a>Requisitos previos

1. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.
1. [Cree el clúster del proveedor y la base de datos](create-cluster-database-portal.md) del responsable y el seguidor.
1. [Ingiera datos](ingest-sample-data.md) en la base de datos del responsable mediante uno de los diferentes métodos descritos en la [información general sobre la ingesta](ingest-data-overview.md).

## <a name="data-provider---share-data"></a>Proveedor de datos: uso compartido de datos

Siga las instrucciones del vídeo para crear una cuenta de uso compartido de datos, agregar un conjunto de datos y enviar una invitación.

[![Proveedor de datos: uso compartido de datos](https://img.youtube.com/vi/QmsTnr90_5o/0.jpg)](https://youtu.be/QmsTnr90_5o?&autoplay=1)

## <a name="data-consumer---receive-data"></a>Consumidor de datos: recepción de datos

Siga las instrucciones del vídeo para aceptar la invitación, crear una cuenta de uso compartido de datos y asignarla al clúster de consumidores.

[![Consumidor de datos: recepción de datos](https://img.youtube.com/vi/vBq6iFaCpdA/0.jpg)](https://youtu.be/vBq6iFaCpdA?&autoplay=1)

El consumidor de datos ahora puede ir a su clúster de Azure Data Explorer para conceder permisos de usuario a las bases de datos compartidas y acceder a los datos. Los datos ingeridos mediante el uso del modo por lotes en el clúster de Azure Data Explorer de origen se mostrarán en el clúster de destino en unos segundos o unos minutos.

## <a name="limitations"></a>Limitaciones

* [Limitaciones de la base de datos de seguidores](follower.md#limitations)

## <a name="next-steps"></a>Pasos siguientes

* [Documentación de Azure Data Share](/azure/data-share/)
* Para obtener información sobre el clúster seguidor, consulte [clúster seguidor](follower.md)
