---
title: Pago por adelantado del incremento de Azure Data Explorer para ahorrar dinero
description: Obtenga información sobre cómo comprar la capacidad reservada de Azure Data Explorer para ahorrar en los costos de Azure Data Explorer.
author: avneraa
ms.author: avnera
ms.reviewer: orspodek
ms.service: data-explorer
ms.topic: conceptual
ms.date: 11/03/2019
ms.openlocfilehash: 6475a0190546ac577ae86ac1fde4adc41cd47595
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81494304"
---
# <a name="prepay-for-azure-data-explorer-markup-units-with-azure-data-explorer-reserved-capacity"></a>Pago por adelantado de las unidades de incremento de Azure Data Explorer con la capacidad reservada de Azure Data Explorer

Ahorre dinero con Azure Data Explorer al pagar por adelantado las unidades de incremento en lugar de los precios de pago por uso. Con la capacidad reservada de Azure Data Explorer, se compromete a usar Azure Data Explorer durante un período de uno a tres años para obtener un descuento considerable en los costos de incremento de Azure Data Explorer. Para comprar la capacidad reservada de Azure Data Explorer, solo tiene que especificar el periodo, que se aplicará a todas las implementaciones de Azure Data Explorer en todas las regiones.

Al comprar una reserva, se adelanta el pago de los costos de incremento durante un periodo de uno a tres años. En cuanto se compra una reserva, los costos de incremento de Azure Data Explorer que coincidan con los atributos de reserva dejan de pagarse en las tarifas de pago por uso. Los clústeres de Azure Data Explorer que ya están en ejecución o los recién implementados obtendrán la ventaja automáticamente. Esta reserva no cubre los cargos de proceso, de red o de almacenamiento asociados con los clústeres. La capacidad reservada para estos recursos se debe comprar por separado. Al final del plazo de reserva, la ventaja en la facturación expira y las unidades de incremento Azure Data Explorer se facturan según los precios de pago por uso. Las reservas no se renuevan automáticamente. Para obtener información sobre precios, vea la [página de precios de Azure Data Explorer](https://azure.microsoft.com/pricing/details/data-explorer/).

Puede comprar capacidad reservada de Azure Data Explorer en [Azure Portal](https://portal.azure.com). Para comprar capacidad reservada de Azure Data Explorer:

* Debe ser el propietario de al menos una suscripción Enterprise o de Pago por uso.
* En el caso de las suscripciones Enterprise, la opción **Agregar instancias reservadas** debe estar habilitada en el [portal de EA](https://ea.azure.com). O bien, si esa opción está deshabilitada, debe ser un administrador de EA en la suscripción.
* En el caso del programa del Proveedor de soluciones en la nube (CSP), los únicos que pueden comprar la capacidad reservada de Azure Data Explorer son los agentes de administración o de ventas.

Para obtener detalles sobre cómo se cargan los clientes empresariales y los de Pago por uso por las compras de reserva, vea lo siguiente:
* [Información sobre el uso de reservas de Azure para la inscripción Enterprise](/azure/cost-management-billing/reservations/understand-reserved-instance-usage-ea). 
* [Información sobre el uso de reservas de Azure para suscripciones de Pago por uso](/azure/cost-management-billing/reservations/understand-reserved-instance-usage).

## <a name="determine-the-right-markup-usage-before-purchase"></a>Determinación del uso del incremento adecuado antes de la compra

El tamaño de la reserva debe basarse en el número total de unidades de marcado de Azure Data Explorer que usan los clústeres de Azure Data Explorer existentes o los que se van a implementar pronto. El número de unidades de marcado es igual al número de núcleos de clúster del motor de Azure Data Explorer en producción (sin incluir la SKU de desarrollo y pruebas). 

## <a name="buy-azure-data-explorer-reserved-capacity"></a>Compra de capacidad reservada de Azure Data Explorer

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione **Todos los servicios** > **Reservas** > **Comprar ahora**. Seleccione **Agregar**.
1. En el panel **Seleccionar tipo de producto**, seleccione **Azure Data Explorer** con el fin de comprar una nueva reserva para las unidades de marcado de Azure Data Explorer. 
1. Seleccione **Comprar**.
1. Rellene todos los campos obligatorios. 

    ![Página de compra](media/pricing-reserved-capacity/purchase-page.png)

1. Revise el costo de la reserva de capacidad reservada de incremento de Azure Data Explorer en la sección **Costos**.
1. Seleccione **Comprar**.
1. Haga clic en **Ver esta reserva** para conocer el estado de la compra.

## <a name="cancellations-and-exchanges"></a>Cancelaciones e intercambios

Si tiene que cancelar la reserva de capacidad reservada de Azure Data Explorer, podrían aplicarse unos honorarios del 12 % por cancelación anticipada. Los reembolsos se basan en el precio más bajo de su precio de compra o en el precio actual de la reserva. Los reembolsos están limitados a 50 000 dólares al año. El reembolso que recibe es el saldo prorrateado restante menos la tarifa de terminación anticipada del 12 %. Para solicitar una cancelación, vaya a la reserva de Azure Portal y seleccione **Reembolso** para crear una solicitud de soporte técnico.

Si necesita cambiar la reserva de capacidad reservada de Azure Data Explorer a otro periodo, puede cambiarla por otra reserva de igual o mayor valor. La fecha de inicio del período de la nueva reserva no se extiende desde la reserva intercambiada. Al crear la nueva reserva empieza el período de 1 o 3 años. Para solicitar un intercambio, vaya a la reserva en Azure Portal y seleccione **Cambio** para crear una solicitud de soporte técnico.

Para más información acerca de cómo cambiar o reembolsar las reservas, consulte [Cambios de reserva y reembolsos](/azure/cost-management-billing/reservations/exchange-and-refund-azure-reservations).

## <a name="manage-your-reserved-capacity-reservation"></a>Administración de la reserva de capacidad reservada

El descuento por la reserva de unidades de incremento de Azure Data Explorer se aplica automáticamente al número de unidades de incremento que coincida con el ámbito y los atributos de la reserva de capacidad reservada de Azure Data Explorer. 


> [!NOTE]
> * Se puede actualizar el ámbito de la reserva de capacidad reservada de Azure Data Explorer a través de [Azure Portal](https://portal.azure.com), PowerShell, la CLI o la API.
> * Para obtener información sobre cómo administrar la reserva de capacidad reservada de Azure Data Explorer, vea [Administración de la capacidad reservada de Azure Data Explorer](/azure/cost-management-billing/reservations/understand-azure-data-explorer-reservation-charges).

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información acerca de Azure Reservations, consulte los siguientes artículos:

* [¿Qué es Azure Reservations?](/azure/cost-management-billing/reservations/save-compute-costs-reservations)
* [Administración de Azure Reservations](/azure/cost-management-billing/reservations/manage-reserved-vm-instance)
* [Información sobre el descuento de Azure Reservations](/azure/cost-management-billing/reservations/understand-reservation-charges)
* [Información sobre el uso de reservas para suscripciones de pago por uso](/azure/cost-management-billing/reservations/understand-reserved-instance-usage)
* [Información sobre el uso de reservas para la inscripción Enterprise](/azure/cost-management-billing/reservations/understand-reserved-instance-usage-ea)
* [Azure Reservations en el programa del Proveedor de soluciones en la nube (CSP) del Centro de partners](https://docs.microsoft.com/partner-center/azure-reservations)

## <a name="need-help-contact-us"></a>¿Necesita ayuda? Ponerse en contacto con nosotros

Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).
