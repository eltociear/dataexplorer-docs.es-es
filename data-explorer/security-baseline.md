---
title: Línea de base de seguridad de Azure para Data Explorer
description: Línea de base de seguridad de Azure para Data Explorer
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 03/25/2020
ms.author: mbaldwin
ms.custom: security-benchmark
ms.openlocfilehash: e67621e92af12c45ac7e66bd6055f90245f705d2
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81494176"
---
# <a name="azure-security-baseline-for-data-explorer"></a>Línea de base de seguridad de Azure para Data Explorer

La línea de base de seguridad de Azure para Data Explorer contiene recomendaciones que le ayudarán a mejorar el nivel de seguridad de la implementación.

La línea de base de este servicio se extrae de [Azure Security Benchmark versión 1.0](https://docs.microsoft.com/azure/security/benchmarks/overview), que proporciona recomendaciones sobre cómo puede proteger las soluciones en la nube en Azure con nuestra guía de procedimientos recomendados.

Para obtener más información, consulte [Introducción a las líneas de base de seguridad de Azure](https://docs.microsoft.com/azure/security/benchmarks/security-baselines-overview).

## <a name="network-security"></a>Seguridad de redes

*Para obtener más información, consulte [Control de seguridad: seguridad de red](https://docs.microsoft.com/azure/security/benchmarks/security-control-network-security).*

### <a name="11-protect-resources-using-network-security-groups-or-azure-firewall-on-your-virtual-network"></a>1.1: Proteja los recursos mediante grupos de seguridad de red o Azure Firewall en su red virtual

**Guía**: Azure Data Explorer admite la implementación de un clúster en una subred de la red virtual. Esta funcionalidad le permite aplicar reglas de grupo de seguridad de red (NSG) en el tráfico del clúster de Azure Data Explorer, conectar la red local a la subred del clúster de Azure Data Explorer y proteger los orígenes de conexión de datos (Event Hub y Event Grid) con puntos de conexión de servicio.

Procedimiento para implementar el clúster de Azure Data Explorer en una red virtual: https://docs.microsoft.com/azure/data-explorer/vnet-deployment


**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-vnets-subnets-and-nics"></a>1.2: Supervisión y registro de la configuración y el tráfico de redes virtuales, subredes y NIC

**Guía**: habilite los registros de flujo del grupo de seguridad de red (NSG) y envíelos a una cuenta de Storage para auditar el tráfico.

Procedimiento para habilitar los registros de flujo de NSG: https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-portal

Descripción de la seguridad de red proporcionada por Azure Security Center: https://docs.microsoft.com/azure/security-center/security-center-network-recommendations


**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="13-protect-critical-web-applications"></a>1.3: Protección de las aplicaciones web críticas

**Guía**: no aplicable; la recomendación está pensada para las aplicaciones web que se ejecutan en Azure App Service o en recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4: Denegar las comunicaciones con direcciones IP malintencionadas conocidas

**Guía**: habilite Protección contra DDoS de Azure estándar en la red virtual que protege los clústeres de Data Explorer para la protección frente a ataques DDoS. Use la inteligencia sobre amenazas integrada de Azure Security Center para denegar las comunicaciones con direcciones IP malintencionadas conocidas o no utilizadas.

Procedimientos para configurar la protección contra DDoS: https://docs.microsoft.com/azure/virtual-network/manage-ddos-protection

Descripción de la inteligencia sobre amenazas integrada de Azure Security Center: https://docs.microsoft.com/azure/security-center/security-center-alerts-service-layer


**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="15-record-network-packets-and-flow-logs"></a>1.5: Registro de los paquetes de red y registros de flujo

**Guía**: habilite los registros de flujo en el grupo de seguridad de red (NSG) que se usa para proteger el clúster de Azure Data Explorer y envíe los registros a una cuenta de Storage para la auditoría de tráfico.

Procedimiento para habilitar los registros de flujo de NSG: https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-portal


**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems"></a>1.6: Implementación de sistemas de prevención de intrusiones y detección de intrusiones basados en la red

**Guía**: no aplicable; este control se realiza en el punto de conexión o firewall.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="17-manage-traffic-to-your-web-applications"></a>1.7: Administración del tráfico a las aplicaciones web

**Guía**: No aplicable; esta recomendación está pensada para las aplicaciones web que se ejecutan en Azure App Service o en recursos de proceso.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8: Minimice la complejidad y la sobrecarga administrativa de las reglas de seguridad de red

**Guía**: use etiquetas de servicio de red virtual para definir controles de acceso a la red en los grupos de seguridad de red o las instancias de Azure Firewall asociadas a los clústeres de Azure Data Explorer. Puede utilizar etiquetas de servicio en lugar de direcciones IP específicas al crear reglas de seguridad. Al especificar el nombre de la etiqueta de servicio (por ejemplo, ApiManagement) en el campo de origen o destino apropiado de una regla, puede permitir o denegar el tráfico para el servicio correspondiente. Microsoft administra los prefijos de direcciones que la etiqueta de servicio incluye y actualiza automáticamente dicha etiqueta a medida que las direcciones cambian.

Descripción y uso de las etiquetas de servicio: https://docs.microsoft.com/azure/virtual-network/service-tags-overview

Requisitos de configuración de etiquetas de servicio para Azure Data Explorer: https://docs.microsoft.com/azure/data-explorer/vnet-deployment#dependencies-for-vnet-deployment


**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9: Mantenimiento de las configuraciones de seguridad estándar para dispositivos de red

**Guía**: el cliente debe definir e implementar configuraciones de seguridad estándar para los recursos de red con Azure Policy.

El cliente también puede usar Azure Blueprints para simplificar las implementaciones de Azure a gran escala mediante el empaquetado de artefactos de entorno clave, como plantillas de ARM, controles de RBAC y directivas, en una única definición de plano técnico. Aplique fácilmente el plano técnico a nuevas suscripciones y entornos, y ajuste el control y la administración mediante el control de versiones.

Configuración y administración de Azure Policy: https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Creación de un plano técnico de Azure: https://docs.microsoft.com/azure/governance/blueprints/create-blueprint-portal


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="110-document-traffic-configuration-rules"></a>1.10: Documentación de las reglas de configuración de tráfico

**Guía**: use etiquetas para grupos de seguridad de red (NSG) y otros recursos relacionados con la seguridad de red y el flujo de tráfico para los clústeres de Data Explorer. En el caso de las reglas de NSG individuales, use el campo "Descripción" para especificar las necesidades empresariales o la duración (etc.) de las reglas que permiten que entre o salga el tráfico en una red.

Creación y uso de etiquetas: https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11: Uso de herramientas automatizadas para supervisar las configuraciones de los recursos de red y detectar cambios

**Guía**: Use Azure Policy para validar (y/o corregir) la configuración de los recursos de red.

Cómo configurar y administrar Azure Policy: https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

## <a name="logging-and-monitoring"></a>Registro y supervisión

*Para obtener más información, consulte [Control de seguridad: registro y supervisión](https://docs.microsoft.com/azure/security/benchmarks/security-control-logging-monitoring).*

### <a name="21-use-approved-time-synchronization-sources"></a>2.1: Uso de orígenes de sincronización de hora aprobados

**Guía**: Microsoft mantiene los orígenes de hora de los recursos de Azure; los clientes pueden actualizar la sincronización de hora de las implementaciones de proceso de su propiedad.

Configuración de la sincronización de hora de los recursos de proceso de Azure: https://docs.microsoft.com/azure/virtual-machines/windows/time-sync

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Microsoft

### <a name="22-configure-central-security-log-management"></a>2.2: Configuración de la administración central de registros de seguridad

**Guía**: Azure Data Explorer usa registros de diagnóstico para extraer conclusiones sobre los errores y aciertos de la ingesta. Puede exportar los registros de operaciones a Azure Storage, Event Hub o Log Analytics para supervisar el estado de ingesta.

Procedimiento para supervisar operaciones de ingesta de Azure Data Explorer:

https://docs.microsoft.com/azure/data-explorer/using-diagnostic-logs

Procedimiento de ingesta y consulta de datos de supervisión en Azure Data Explorer:

https://docs.microsoft.com/azure/data-explorer/ingest-data-no-code

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3: Habilitación del registro de auditoría para los recursos de Azure

**Guía**: habilite la configuración de diagnóstico de Azure Data Explorer para el acceso y el registro de operaciones y registros específicos del servicio. Los registros de actividad de Azure dentro de Azure Monitor, que incluye un registro general sobre el recurso, están habilitados de forma predeterminada.

Procedimiento para supervisar operaciones de ingesta de Azure Data Explorer: https://docs.microsoft.com/azure/data-explorer/using-diagnostic-logs

Recopilación de registros y métricas de plataforma con Azure Monitor: https://docs.microsoft.com/azure/azure-monitor/platform/diagnostic-settings

Introducción a los registros de plataforma de Azure: https://docs.microsoft.com/azure/azure-monitor/platform/platform-logs-overview


**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="24-collect-security-logs-from-operating-system"></a>2.4: Recopilación de los registros de seguridad del sistema operativo

**Guía**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="25-configure-security-log-storage-retention"></a>2.5: Configuración de la retención del almacenamiento de registros de seguridad

**Guía**: En Azure Monitor, establezca el período de retención del área de trabajo de Log Analytics de acuerdo con la normativa de cumplimiento de su organización. Use cuentas de Azure Storage para el almacenamiento de archivos a largo plazo.

Configuración de parámetros de retención de registros de áreas de trabajo de Log Analytics: https://docs.microsoft.com/azure/azure-monitor/platform/manage-cost-storage#change-the-data-retention-period


**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="26-monitor-and-review-logs"></a>2.6: Supervisión y revisión de registros

**Guía**: analice y supervise los registros en busca de comportamientos anómalos y revise los resultados con regularidad. Después de habilitar la configuración de diagnóstico para Azure Data Explorer, use el área de trabajo de Log Analytics de Azure Monitor para revisar los registros y realizar consultas en los datos del registro.

Descripción del área de trabajo de Log Analytics: https://docs.microsoft.com/azure/azure-monitor/log-query/get-started-portal

Procedimiento para realizar consultas personalizadas en Azure Monitor: https://docs.microsoft.com/azure/azure-monitor/log-query/get-started-queries


**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="27-enable-alerts-for-anomalous-activity"></a>2.7: Habilitación de alertas para actividades anómalas

**Guía**: no aplicable; Azure Data Explorer no tiene esta capacidad.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="28-centralize-anti-malware-logging"></a>2.8: Centralización del registro antimalware

**Guía**: no aplicable; Azure Data Explorer no procesa el registro antimalware.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="29-enable-dns-query-logging"></a>2.9: Habilitación del registro de consultas de DNS

**Guía**: No aplicable: la consulta de DNS no es una función de Azure Data Explorer.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="210-enable-command-line-audit-logging"></a>2.10: Habilitación del registro de auditoría de la línea de comandos

**Guía**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

## <a name="identity-and-access-control"></a>Identidad y Access Control

*Para obtener más información, consulte [Control de seguridad: identidad y control de acceso](https://docs.microsoft.com/azure/security/benchmarks/security-control-identity-access-control).*

### <a name="31-maintain-inventory-of-administrative-accounts"></a>3.1: Mantenimiento de un inventario de cuentas administrativas

**Guía**: en Azure Data Explorer, los roles de seguridad definen qué entidades de seguridad (usuarios y aplicaciones) tienen permisos para operar en un recurso protegido, como una base de datos o una tabla, y qué operaciones se permiten.  Puede aprovechar la consulta Kusto para enumerar los principios del rol de administrador para los clústeres y las bases de datos de Azure Data Explorer.
[Administración de roles de seguridad en Azure Data Explorer mediante la consulta de Kusto](kusto/management/security-roles.md)



**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="32-change-default-passwords-where-applicable"></a>3.2: Cambio de las contraseñas predeterminadas cuando proceda

**Guía**: Azure AD no tiene el concepto de contraseñas predeterminadas. Otros recursos de Azure que requieren una contraseña obligan a crear una contraseña con requisitos de complejidad y una longitud mínima, que cambia en función del servicio. Es responsable de las aplicaciones de terceros y de los servicios de Marketplace que pueden usar contraseñas predeterminadas.


**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="33-ensure-the-use-of-dedicated-administrative-accounts"></a>3.3: Aseguramiento del uso de cuentas administrativas dedicadas

**Guía**: cliente para crear procedimientos operativos estándar en torno al uso de cuentas administrativas dedicadas. Use la administración de identidad y acceso de Azure Security Center para supervisar el número de cuentas administrativas.

Los clientes también puede habilitar el acceso Just-in-Time/Just-Enough mediante roles de Azure AD Privileged Identity Management con privilegios para los servicios de Microsoft y Azure ARM. 

¿Qué es Azure AD Privileged Identity Management?: https://docs.microsoft.com/azure/active-directory/privileged-identity-management/pim-configure

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="34-utilize-single-sign-on-sso-with-azure-active-directory"></a>3.4: Uso del inicio de sesión único (SSO) con Azure Active Directory

**Guía**: siempre que sea posible, el cliente debe usar el SSO con Azure Active Directory (Azure AD) en lugar de configurar credenciales independientes individuales por cada servicio. Use las recomendaciones de administración de identidades y acceso de Azure Security Center.

Descripción del SSO con Azure AD: https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on


**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="35-use-multifactor-authentication-for-all-azure-active-directory-based-access"></a>3.5: Uso de la autenticación multifactor para todo el acceso basado en Azure Active Directory

**Guía**: Habilite la autenticación multifactor (MFA) de Azure Active Directory (Azure AD) y siga las recomendaciones de administración de identidades y acceso de Azure Security Center.

Cómo habilitar la MFA en Azure: https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-getstarted 

Procedimiento para supervisar la identidad y el acceso en Azure Security Center: https://docs.microsoft.com/azure/security-center/security-center-identity-access


**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="36-use-of-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6: Uso de máquinas dedicadas (estaciones de trabajo de acceso con privilegios) para todas las tareas administrativas

**Guía**: use estaciones de trabajo de acceso con privilegios (PAW) con la autenticación multifactor (MFA) configurada para iniciar sesión en recursos de Azure y configurarlos.

Más información sobre las estaciones de trabajo con privilegios de acceso: https://docs.microsoft.com/windows-server/identity/securing-privileged-access/privileged-access-workstations

Procedimiento para habilitar la MFA en Azure: https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-getstarted


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="37-log-and-alert-on-suspicious-activity-on-administrative-accounts"></a>3.7: Registro y alerta de la actividad sospechosa en cuentas administrativas

**Guía**: use los informes de seguridad de Azure Active Directory (Azure AD) para la generación de registros y alertas cuando se producen actividades sospechosas o no seguras en el entorno. Use Azure Security Center para supervisar la actividad de identidad y acceso.

Procedimiento para identificar usuarios de Azure AD marcados por una actividad de riesgo: https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-user-at-risk

Supervisión de la actividad de identidad y acceso de los usuarios en Azure Security Center: https://docs.microsoft.com/azure/security-center/security-center-identity-access


**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="38-manage-azure-resource-from-only-approved-locations"></a>3.8: Administración de los recursos de Azure solo desde ubicaciones aprobadas

**Guía**: el cliente debe usar ubicaciones con nombre de acceso condicional para permitir el acceso solo desde agrupaciones lógicas específicas de intervalos de direcciones IP o países o regiones.

Configuración de ubicaciones con nombre en Azure: https://docs.microsoft.com/azure/active-directory/reports-monitoring/quickstart-configure-named-locations


**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="39-utilize-azure-active-directory"></a>3.9: Uso de Azure Active Directory

**Guía**: Azure Active Directory (Azure AD) es el método preferido para autenticarse en Azure Data Explorer. Admite varios escenarios de autenticación:

Autenticación de usuario (inicio de sesión interactivo): se usa para autenticar entidades de seguridad humanas.

Autenticación de aplicación (inicio de sesión no interactivo): se usa para autenticar aquellos servicios y aplicaciones que tienen que ejecutarse o autenticarse sin que haya usuarios humanos presentes.

[Introducción al control de acceso de Azure Data Explorer](kusto/management/access-control/index.md)

[Autenticación con Azure Active Directory](kusto/management/access-control/aad.md)



**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10: Revisión y conciliación de manera periódica del acceso de los usuarios

**Guía**: Azure Active Directory (Azure AD) proporciona registros para ayudar a descubrir cuentas obsoletas. Además, use las revisiones de acceso de identidad de Azure para administrar de forma eficiente las pertenencias a grupos, el acceso a las aplicaciones empresariales y las asignaciones de roles. El acceso de los usuarios se puede revisar de forma periódica para asegurarse de que solo las personas adecuadas tengan acceso continuado. 

[Autenticación con Azure AD para el acceso a Azure Data Explorer](kusto/management/access-control/how-to-authenticate-with-aad.md)

[Informes de Azure AD](https://docs.microsoft.com/azure/active-directory/reports-monitoring/)

[Procedimiento para usar las revisiones de acceso de identidad de Azure](https://docs.microsoft.com/azure/active-directory/governance/access-reviews-overview)


**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="311-monitor-attempts-to-access-deactivated-accounts"></a>3.11: Supervisión de los intentos de acceso a cuentas desactivadas

**Guía**: puede usar orígenes de registro de eventos de actividad de inicio de sesión de Azure Active Directory (Azure AD), auditoría y riesgos para la supervisión, lo que permite realizar la integración con cualquier herramienta de administración de eventos e información de seguridad (SIEM) y de supervisión.

 Puede simplificar este proceso creando una configuración de diagnóstico para las cuentas de usuario de Azure Active Directory y enviando los registros de auditoría y de inicio de sesión a un área de trabajo de Log Analytics. El cliente puede configurar las alertas deseadas en el área de trabajo de Log Analytics.

Procedimiento para integrar los registros de actividad de Azure en Azure Monitor: https://docs.microsoft.com/azure/active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics


**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="312-alert-on-account-login-behavior-deviation"></a>3.12: Alerta de las desviaciones de comportamiento en los inicios de sesión de las cuentas

**Guía**: use las características de protección de identidad y detección de riesgo de Azure Active Directory (Azure AD) para configurar respuestas automatizadas a las acciones sospechosas detectadas relacionadas con las identidades de los usuarios. Además, puede ingerir datos en Azure Sentinel para investigarlos más a fondo.

Procedimiento para ver los inicios de sesión de riesgo de Azure AD: https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-risky-sign-ins

Procedimiento para configurar y habilitar directivas de riesgo de protección de la identidad: https://docs.microsoft.com/azure/active-directory/identity-protection/howto-identity-protection-configure-risk-policies


**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13: Proporcione a Microsoft acceso a los datos pertinentes del cliente durante los escenarios de soporte técnico

**Guía**: en escenarios de soporte técnico en los que Microsoft necesita acceder a los datos del cliente, la Caja de seguridad del cliente proporciona una interfaz para que los clientes revisen y aprueben o rechacen las solicitudes de acceso a los datos del cliente.

Descripción de la Caja de seguridad del cliente:

https://docs.microsoft.com/azure/security/fundamentals/customer-lockbox-overview

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

## <a name="data-protection"></a>Protección de datos

*Para obtener más información, consulte [Control de seguridad: protección de datos](https://docs.microsoft.com/azure/security/benchmarks/security-control-data-protection).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1: Mantenimiento de un inventario de información confidencial

**Guía**: use etiquetas para ayudar a realizar el seguimiento de los recursos de Azure que almacenan o procesan información confidencial.

Creación y uso de etiquetas:

https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2: Aislamiento de los sistemas que almacenan o procesan información confidencial

**Instrucciones**: Implemente suscripciones y/o grupos de administración independientes para los entornos de desarrollo, prueba y producción. los clústeres de Azure Data Explorer se deben separar de otros recursos por red virtual o subred, etiquetarse correctamente y protegerse dentro de un grupo de seguridad de red (NSG) o Azure Firewall. Los clústeres de Azure Data Explorer que almacenan o procesan datos confidenciales deben estar suficientemente aislados.

Procedimiento para crear suscripciones adicionales de Azure: https://docs.microsoft.com/azure/billing/billing-create-subscription

Procedimiento para crear grupos de administración: https://docs.microsoft.com/azure/governance/management-groups/create

Creación y uso de etiquetas: https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

Procedimiento para implementar el clúster de Azure Data Explorer en una red virtual: https://docs.microsoft.com/azure/data-explorer/vnet-deployment

Procedimiento para crear un grupo de seguridad de red con una configuración de seguridad: https://docs.microsoft.com/azure/virtual-network/tutorial-filter-network-traffic


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3: Supervisión y bloqueo de transferencias no autorizadas de información confidencial

**Guía**: no aplicable; en el caso de la plataforma subyacente administrada por Microsoft, Microsoft trata todo el contenido de los clientes como confidencial y hace grandes esfuerzos para proteger a los clientes contra la pérdida y exposición de sus datos. Para garantizar la seguridad de los datos de los clientes dentro de Azure, Microsoft ha implementado y mantiene un conjunto de controles y funcionalidades eficaces de protección de datos.

Descripción de la protección de datos de los clientes en Azure: https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4: Cifrado de toda la información confidencial en tránsito

**Guía**: el clúster de Azure Data Explorer negocia TLS 1.2 de forma predeterminada. Asegúrese de que los clientes que se conectan a los recursos de Azure pueden negociar TLS 1.2 o superior.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Compartido

### <a name="45-divuse-an-active-discovery-tool-to-identify-sensitive-datadiv"></a>4.5: <div>Uso de una herramienta de detección activa para identificar datos confidenciales</div>

**Guía**: las características de identificación, clasificación y prevención de pérdida de datos todavía no están disponibles para Azure Data Explorer. Implemente una solución de terceros, si es necesario, para fines de cumplimiento.

En el caso de la plataforma subyacente administrada por Microsoft, Microsoft trata todo el contenido de los clientes como confidencial y hace grandes esfuerzos para proteger a los clientes contra la pérdida y exposición de sus datos. Para garantizar la seguridad de los datos de los clientes dentro de Azure, Microsoft ha implementado y mantiene un conjunto de controles y funcionalidades eficaces de protección de datos.

Descripción de la protección de datos de los clientes en Azure: https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="46-divuse-azure-rbac-to-control-access-to-resourcesdiv"></a>4.6: <div>Uso de RBAC de Azure para controlar el acceso a los recursos</div>

**Guía**: Azure Data Explorer le permite controlar el acceso a bases de datos y tablas mediante un modelo de control de acceso basado en rol (RBAC). Con este modelo, las entidades de seguridad (usuarios, grupos y aplicaciones) se asignan a roles. Las entidades de seguridad pueden acceder a recursos según los roles que tengan asignados.

Lista de roles y permisos e instrucciones sobre cómo configurar RBAC para Azure Data Explorer: https://docs.microsoft.com/azure/data-explorer/manage-database-permissions


**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="47-use-host-based-data-loss-prevention-to-enforce-access-control"></a>4.7: Uso de la prevención de pérdida de datos basada en host para aplicar el control de acceso

**Guía**: No aplicable; esta recomendación está pensada para los recursos de proceso.

Microsoft administra la infraestructura subyacente para Azure Data Explorer y ha implementado controles estrictos para evitar la pérdida o exposición de los datos de los clientes.

Descripción de la protección de datos de los clientes en Azure: https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8: Cifrado en reposo de la información confidencial

**Guía**: Azure Disk Encryption ayuda a custodiar y proteger sus datos con el fin de satisfacer los compromisos de cumplimiento y seguridad de su organización. Proporciona cifrado de volúmenes tanto para los discos de datos como para los del sistema operativo de las máquinas virtuales del clúster. También se integra con Azure Key Vault, lo que permite controlar y administrar los secretos y las claves de cifrado de discos, y garantiza que todos los datos de los discos de la máquina virtual se cifran en reposo mientras se encuentran en Azure Storage.

Procedimiento para habilitar el cifrado en reposo para los clústeres de Azure Data Explorer: https://docs.microsoft.com/azure/data-explorer/manage-cluster-security


**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9: Registro y alerta de cambios en los recursos críticos de Azure

**Guía**: use Azure Monitor con el registro de actividad de Azure a fin de crear alertas para cuando se produzcan cambios en los clústeres de Azure Data Explorer.

Procedimiento para crear alertas para eventos de registro de actividad de Azure: https://docs.microsoft.com/azure/azure-monitor/platform/alerts-activity-log

Procedimiento para crear alertas para eventos de registro de actividad de Azure: https://docs.microsoft.com/azure/azure-monitor/platform/alerts-activity-log


**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

## <a name="vulnerability-management"></a>Administración de vulnerabilidades

*Para obtener más información, consulte [Control de seguridad: administración de vulnerabilidades](https://docs.microsoft.com/azure/security/benchmarks/security-control-vulnerability-management).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1: Ejecución de herramientas de análisis de vulnerabilidades automatizadas

**Guía**: siga las recomendaciones de Azure Security Center sobre la protección de los recursos de Azure Data Explorer.

Microsoft también realiza la administración de vulnerabilidades en los sistemas subyacentes que admiten Azure Data Explorer.

Descripción de las recomendaciones de Azure Security Center: https://docs.microsoft.com/azure/security-center/recommendations-reference

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Microsoft

### <a name="52-deploy-automated-operating-system-patch-management-solution"></a>5.2: Implementación de una solución de administración de revisiones de sistema operativo automatizada

**Guía**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="53-deploy-automated-third-party-software-patch-management-solution"></a>5.3: Implementación de una solución automatizada de administración de revisiones de software de terceros

**Guía**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="54-compare-back-to-back-vulnerability-scans"></a>5.4: Comparación de los exámenes de vulnerabilidades opuestos

**Guía**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="55-utilize-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5: Uso de un proceso de clasificación de riesgos para priorizar la corrección de las vulnerabilidades detectadas.

**Guía**: Use la clasificación de riesgo predeterminada (puntuación de seguridad) proporcionada por Azure Security Center.
Descripción de la puntuación segura de Azure Security Center: https://docs.microsoft.com/azure/security-center/security-center-secure-score


**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

## <a name="inventory-and-asset-management"></a>Administración de recursos y del inventario

*Para obtener más información, consulte [Control de seguridad: inventario y administración de recursos](https://docs.microsoft.com/azure/security/benchmarks/security-control-inventory-asset-management).*

### <a name="61-utilize-azure-asset-discovery"></a>6.1: Uso de la detección de recursos de Azure

**Guía**: Use Azure Resource Graph para consultar o detectar todos los recursos (por ejemplo, proceso, almacenamiento, red, puertos y protocolos, etc.) dentro de las suscripciones. Asegúrese de que tiene los permisos adecuados (lectura) en el inquilino y enumere todas las suscripciones de Azure, así como los recursos de las suscripciones.

Aunque los recursos clásicos de Azure se pueden detectar a través de Resource Graph, se recomienda encarecidamente crear y usar los recursos de Azure Resource Manager que figuran a continuación.

Creación de consultas con Azure Resource Graph: https://docs.microsoft.com/azure/governance/resource-graph/first-query-portal

Visualización de las suscripciones de Azure: https://docs.microsoft.com/powershell/module/az.accounts/get-azsubscription?view=azps-3.0.0

Descripción de Azure RBAC: https://docs.microsoft.com/azure/role-based-access-control/overview



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="62-maintain-asset-metadata"></a>6.2: Mantenimiento de metadatos de recursos

**Guía**: Aplique etiquetas a los recursos de Azure que proporcionan metadatos para organizarlos de forma lógica en una taxonomía.

Creación y uso de etiquetas: https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3: Eliminación de recursos de Azure no autorizados

**Guía**: puede usar las convenciones de nomenclatura adecuadas, etiquetado, grupos de administración o suscripciones independientes, si procede, para organizar y realizar el seguimiento de los recursos. Puede usar Azure Resource Graph para conciliar el inventario de forma periódica y asegurarse de que los recursos no autorizados se eliminan de la suscripción de manera oportuna. 

Procedimiento para crear suscripciones adicionales de Azure: https://docs.microsoft.com/azure/billing/billing-create-subscription

Creación de grupos de administración: https://docs.microsoft.com/azure/governance/management-groups/create

Creación y uso de etiquetas: https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

Creación de consultas con Azure Resource Graph: https://docs.microsoft.com/azure/governance/resource-graph/first-query-portal



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="64-maintain-inventory-of-approved-azure-resources-and-software-titles"></a>6.4: Mantenimiento de un inventario de los recursos de Azure y títulos de software aprobados

**Guía**: tendrá que crear un inventario de los recursos de Azure aprobados y el software aprobado para los recursos de proceso según las necesidades de la organización.  


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5: Supervisión de recursos de Azure no aprobados

**Guía**: puede usar directivas de Azure para establecer restricciones en el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas:

    - Tipos de recursos no permitidos

    - Tipos de recursos permitidos

Podrá supervisar los eventos generados por la directiva mediante los 

registros de actividad que se pueden supervisar con Azure Monitor.

Además, puede usar Azure Resource Graph para consultar o detectar recursos dentro de las suscripciones.

Tutorial: Creación y administración de directivas para aplicar el cumplimiento: https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Inicio rápido: Ejecución de la primera consulta de Resource Graph mediante Azure Resource Graph Explorer: https://docs.microsoft.com/azure/governance/resource-graph/first-query-portal

Creación, visualización y administración de alertas del registro de actividad mediante Azure Monitor: https://docs.microsoft.com/azure/azure-monitor/platform/alerts-activity-log



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="66-monitor-for-unapproved-software-applications-within-compute-resources"></a>6.6: Supervisión de aplicaciones de software no aprobadas en recursos de proceso

**Guía**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7: Eliminación de aplicaciones de software y recursos de Azure no aprobados

**Guía**: No aplicable; esta recomendación está destinada a los recursos de proceso y a Azure en su conjunto.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="68-utilize-only-approved-applications"></a>6.8: Uso exclusivo de aplicaciones aprobadas

**Guía**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="69-utilize-only-approved-azure-services"></a>6.9: Uso exclusivo de servicios de Azure aprobados

**Guía**: puede usar directivas de Azure para establecer restricciones en el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas:

    - Tipos de recursos no permitidos

    - Tipos de recursos permitidos

Tutorial: Creación y administración de directivas para aplicar el cumplimiento: https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Ejemplos de Azure Policy: https://docs.microsoft.com/azure/governance/policy/samples/not-allowed-resource-types



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="610-implement-approved-application-list"></a>6.10: Implementación de la lista de aplicaciones aprobadas

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="611-limit-users-ability-to-interact-with-azure-resources-manager-via-scripts"></a>6.11: Limitación de la capacidad de los usuarios para interactuar con Azure Resource Manager mediante scripts

**Guía**: Use el acceso condicional de Azure para limitar la capacidad de los usuarios de interactuar con Azure Resource Manager configurando "Bloquear acceso" en la aplicación Microsoft Azure Management. Esto impedirá la creación y los cambios en los recursos de las suscripciones de Azure. 

Administración el acceso a la administración de Azure con acceso condicional: https://docs.microsoft.com/azure/role-based-access-control/conditional-access-azure-management



**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="612-limit-users-ability-to-execute-scripts-within-compute-resources"></a>6.12: Limitación de la capacidad de los usuarios para ejecutar scripts en los recursos de proceso

**Guía**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="613-physically-or-logically-segregate-high-risk-applications"></a>6.13: Segregación física o lógica de aplicaciones de alto riesgo

**Guía**: No aplicable; esta recomendación está pensada para las aplicaciones web que se ejecutan en Azure App Service o en recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

## <a name="secure-configuration"></a>Configuración segura

*Para obtener más información, consulte [Control de seguridad: configuración segura](https://docs.microsoft.com/azure/security/benchmarks/security-control-secure-configuration).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1: Establecimiento de configuraciones seguras para todos los recursos de Azure

**Guía**: use alias de Azure Policy para crear directivas personalizadas con el fin de auditar o aplicar la configuración de los recursos de Azure. También puede usar definiciones de Azure Policy integradas.

Además, Azure Resource Manager tiene la capacidad de exportar la plantilla en notación de objetos JavaScript (JSON), que se debe revisar para asegurarse de que las configuraciones cumplan o superen los requisitos de seguridad de la organización.

También puede usar las recomendaciones de Azure Security Center como línea de base de configuración segura para los recursos de Azure.

Visualización de los alias de Azure Policy disponibles: https://docs.microsoft.com/powershell/module/az.resources/get-azpolicyalias?view=azps-3.3.0

Tutorial: Creación y administración de directivas para aplicar el cumplimiento: https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Exportación de uno y varios recursos a una plantilla en Azure Portal: https://docs.microsoft.com/azure/azure-resource-manager/templates/export-template-portal

Guía de referencia sobre las recomendaciones de seguridad: https://docs.microsoft.com/azure/security-center/recommendations-reference



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="72-establish-secure-configurations-for-your-operating-system"></a>7.2: Establecimiento de configuraciones seguras del sistema operativo

**Guía**: No aplicable; esta guía está pensada para recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="73-maintain-secure-configurations-for-all-azure-resources"></a>7.3: Mantenimiento de configuraciones seguras para todos los recursos de Azure

**Guía**: Use la directiva de Azure [denegar] e [implementar si no existe] para aplicar la configuración segura en los recursos de Azure.  Puede usar soluciones como Change Tracking, el panel de cumplimiento de directivas o una solución personalizada para identificar fácilmente los cambios de seguridad en el entorno.

Descripción de los efectos de Azure Policy: https://docs.microsoft.com/azure/governance/policy/concepts/effects

Creación y administración de directivas para aplicar el cumplimiento: https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Seguimiento de cambios en el entorno con la solución Change Tracking: https://docs.microsoft.com/azure/automation/change-tracking

Obtención de datos de cumplimiento de los recursos de Azure: https://docs.microsoft.com/azure/governance/policy/how-to/get-compliance-data



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="74-maintain-secure-configurations-for-operating-systems"></a>7.4: Mantenimiento de configuraciones seguras para los sistemas operativos

**Guía**: No aplicable; esta guía está pensada para recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5: Almacenamiento seguro de la configuración de los recursos de Azure

**Guía**: use Azure Repos para almacenar y administrar de forma segura el código, como directivas de Azure personalizadas, plantillas de Azure Resource Manager, scripts de Desired State Configuration, etc. Para acceder a los recursos que administra en Azure DevOps, puede conceder o denegar permisos a usuarios específicos, grupos de seguridad integrados o grupos definidos en Azure Active Directory (Azure AD) si se integran con Azure DevOps, o Active Directory si se integran con TFS.

Procedimiento para almacenar código en Azure DevOps: https://docs.microsoft.com/azure/devops/repos/git/gitworkflow?view=azure-devops

Acerca de los permisos y los grupos en Azure DevOps: https://docs.microsoft.com/azure/devops/organizations/security/about-permissions



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="76-securely-store-custom-operating-system-images"></a>7.6: Almacenamiento seguro de las imágenes de sistema operativo personalizadas

**Guía**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="77-deploy-system-configuration-management-tools"></a>7.7: Implementación de herramientas de administración de la configuración del sistema

**Guía**: defina e implemente configuraciones de seguridad estándar para los recursos de Azure con Azure Policy. Use alias de Azure Policy para crear directivas personalizadas con el fin de auditar o aplicar la configuración de red de los recursos de Azure. También puede usar definiciones de directivas integradas relacionadas con recursos concretos.  Además, puede usar Azure Automation para implementar los cambios de configuración.

Configuración y administración de Azure Policy: https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Procedimiento para usar alias: https://docs.microsoft.com/azure/governance/policy/concepts/definition-structure#aliases



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="78-deploy-system-configuration-management-tools-for-operating-systems"></a>7.8: Implementación de herramientas de administración de la configuración del sistema para sistemas operativos

**Guía**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="79-implement-automated-configuration-monitoring-for-azure-services"></a>7.9: Implementación de la supervisión de la configuración automatizada para los servicios de Azure

**Guía**: use alias de Azure Policy para crear directivas personalizadas con el fin de auditar y aplicar las configuraciones del sistema y enviar alertas sobre ellas. Use la directiva de Azure [audit], [deny] y [deploy if not exist] para aplicar automáticamente las configuraciones en los recursos de Azure.

Configuración y administración de Azure Policy: https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10: Implementación de la supervisión de la configuración automatizada para los sistemas operativos

**Guía**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="711-securely-manage-azure-secrets"></a>7.11: Administración segura de los secretos de Azure

**Guía**: Azure Disk Encryption proporciona cifrado de volúmenes tanto para los discos de datos como para los del sistema operativo de las máquinas virtuales del clúster de Azure Data Explorer. También se integra con Azure Key Vault, lo que permite controlar y administrar los secretos y las claves de cifrado de discos, y garantiza que todos los datos de los discos de la máquina virtual se cifran en reposo mientras se encuentran en Azure Storage.

Protección de clústeres en Azure Data Explorer: https://docs.microsoft.com/azure/data-explorer/manage-cluster-security


**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Microsoft

### <a name="712-securely-and-automatically-manage-identities"></a>7.12: Administración de identidades de forma segura y automática

**Guía**: Use las identidades administradas para proporcionar a los servicios de Azure una identidad administrada automáticamente en Azure AD. Las identidades administradas le permiten autenticarse en cualquier servicio que admita la autenticación de Azure AD, incluyendo Key Vault, sin necesidad de credenciales en el código. Procedimiento para configurar identidades administradas: https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm

Configuración de identidades administradas para el clúster de Azure Data Explorer: https://docs.microsoft.com/azure/data-explorer/managed-identities


**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13: Elimine la exposición de credenciales no intencionada

**Instrucciones**: Implemente el escáner de credenciales para identificar las credenciales en el código. El escáner de credenciales también fomenta el traslado de credenciales detectadas a ubicaciones más seguras, como Azure Key Vault. 

Cómo configurar el escáner de credenciales: https://secdevtools.azurewebsites.net/helpcredscan.html

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="malware-defense"></a>Defensa contra malware

*Para obtener más información, consulte [Control de seguridad: defensa contra malware](https://docs.microsoft.com/azure/security/benchmarks/security-control-malware-defense).*

### <a name="81-utilize-centrally-managed-anti-malware-software"></a>8.1: Uso de software antimalware administrado centralmente

**Guía**: No aplicable; esta recomendación está pensada para los recursos de proceso.

Microsoft Antimalware está habilitado en el host subyacente que admite los servicios de Azure (por ejemplo, Azure Data Explorer), pero no se ejecuta en el contenido del cliente.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2: Examine previamente los archivos que se van a cargar en recursos de Azure que no son de proceso

**Guía**: Microsoft Antimalware está habilitado en el host subyacente que admite los servicios de Azure (por ejemplo, Azure Data Explorer), pero no se ejecuta en el contenido del cliente.

Examine previamente el contenido que se carga en recursos de Azure que no son de proceso, como Azure Data Explorer, Data Lake Storage, Blob Storage, Azure Database for PostgreSQL, etc. Microsoft no tiene acceso a los datos de estas instancias.



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="83-ensure-anti-malware-software-and-signatures-are-updated"></a>8.3: Aseguramiento de la actualización del software y las firmas antimalware

**Guía**: No aplicable; esta recomendación está pensada para los recursos de proceso.

Microsoft Antimalware está habilitado en el host subyacente que admite los servicios de Azure, pero no se ejecuta en el contenido del cliente.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

## <a name="data-recovery"></a>Recuperación de datos

*Para obtener más información, consulte [Control de seguridad: recuperación de datos](https://docs.microsoft.com/azure/security/benchmarks/security-control-data-recovery).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1: Garantía de copias de seguridad automáticas periódicas

**Guía**: los datos de la cuenta de almacenamiento de Microsoft Azure que usa el clúster de Azure Data Explorer se replican siempre para garantizar la durabilidad y la alta disponibilidad. Azure Storage copia los datos para protegerlos de eventos previstos e imprevistos, como errores transitorios del hardware, interrupciones del suministro eléctrico o de la red y desastres naturales masivos. Puede optar por replicar los datos en el mismo centro de datos, en centros de datos zonales que estén en la misma región o en regiones geográficamente separadas.

Descripción de la redundancia de Azure Storage y los acuerdos de nivel de servicio: https://docs.microsoft.com/azure/storage/common/storage-redundancy

[Exportación de datos al almacenamiento](kusto/management/data-export/export-data-to-storage.md)



**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2: Realización de copias de seguridad completas del sistema y copia de seguridad de las claves administradas por el cliente

**Guía**: Azure Data Explorer cifra todos los datos en una cuenta de almacenamiento en reposo. De manera predeterminada, los datos se cifran con claves administradas por Microsoft. Para tener un mayor control sobre las claves de cifrado, puede proporcionar claves administradas por el cliente para el cifrado de datos. Las claves administradas por el cliente se deben almacenar en Azure Key Vault. 

Configuración de claves administradas por el cliente mediante C#: https://docs.microsoft.com/azure/data-explorer/customer-managed-keys-csharp

Configuración de claves administradas por el cliente mediante la plantilla de Azure Resource Manager: https://docs.microsoft.com/azure/data-explorer/customer-managed-keys-resource-manager

Realización de copias de seguridad de certificados de Azure Key Vault: https://docs.microsoft.com/powershell/module/azurerm.keyvault/backup-azurekeyvaultcertificate?view=azurermps-6.13.0


**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3: Validación de todas las copias de seguridad, incluidas las claves administradas por el cliente

**Guía**: pruebe periódicamente la restauración de datos de los secretos de Azure Key Vault.

Procedimiento para restaurar certificados de Azure Key Vault:

https://docs.microsoft.com/powershell/module/azurerm.keyvault/restore-azurekeyvaultcertificate?view=azurermps-6.13.0 

Configuración de claves administradas por el cliente mediante C#: https://docs.microsoft.com/azure/data-explorer/customer-managed-keys-csharp

Configuración de claves administradas por el cliente mediante la plantilla de Azure Resource Manager: https://docs.microsoft.com/azure/data-explorer/customer-managed-keys-resource-manager



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4: Aseguramiento de la protección de las copias de seguridad y las claves administradas por el cliente

**Guía**: el cliente puede habilitar la eliminación temporal en Key Vault para proteger las claves contra la eliminación accidental o malintencionada.  También puede habilitar la protección de purgas de forma que, si un almacén o un objeto está en el estado eliminado, no se pueda purgar hasta que haya transcurrido el período de retención.  

Procedimiento para habilitar la eliminación temporal y la protección de purga en Key Vault: https://docs.microsoft.com/azure/key-vault/key-vault-ovw-soft-delete

Configuración de claves administradas por el cliente mediante C#: https://docs.microsoft.com/azure/data-explorer/customer-managed-keys-csharp

Configuración de claves administradas por el cliente mediante la plantilla de Azure Resource Manager: https://docs.microsoft.com/azure/data-explorer/customer-managed-keys-resource-manager



**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

## <a name="incident-response"></a>Respuesta a los incidentes

*Para obtener más información, consulte [Control de seguridad: respuesta ante incidentes](https://docs.microsoft.com/azure/security/benchmarks/security-control-incident-response).*

### <a name="101-create-incident-response-guide"></a>10.1: Creación de una guía de respuesta ante incidentes

**Guía**: Cree una guía de respuesta a incidentes para su organización. Asegúrese de que haya planes de respuesta a incidentes escritos que definan todos los roles del personal, así como las fases de administración y gestión de los incidentes, desde la detección hasta la revisión posterior a la incidencia.
    

    Guidance on building your own security incident response process: https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/

    

    Microsoft Security Response Center's Anatomy of an Incident: https://msrc-blog.microsoft.com/2019/06/27/inside-the-msrc-anatomy-of-a-ssirp-incident/

    

    Customer may also leverage NIST's Computer Security Incident Handling Guide to aid in the creation of their own incident response plan: https://csrc.nist.gov/publications/detail/sp/800-61/rev-2/final

    



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="102-create-incident-scoring-and-prioritization-procedure"></a>10.2: Creación de un procedimiento de priorización y puntuación de incidentes

**Guía**: Security Center asigna una gravedad a cada alerta para ayudarle a priorizar aquellas que se deben investigar en primer lugar. La gravedad se basa en la confianza que tiene Security Center en la búsqueda o en el análisis utilizados para emitir la alerta, así como en el nivel de confianza de que ha habido un intento malintencionado detrás de la actividad que ha provocado la alerta. 
    

    Additionally, clearly mark subscriptions (for ex. production, non-prod) using tags and create a naming system to clearly identify and categorize Azure resources, especially those processing sensitive data.  It is your responsibility to prioritize the remediation of alerts based on the criticality of the Azure resources and environment where the incident occurred.

    

    Security alerts in Azure Security Center: https://docs.microsoft.com/azure/security-center/security-center-alerts-overview

    

    Use tags to organize your Azure resources: https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags



**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="103-test-security-response-procedures"></a>10.3: Prueba de los procedimientos de respuesta de seguridad

**Guía**: Realice ejercicios para probar las funcionalidades de respuesta a los incidentes de los sistemas periódicamente; así, ayudará a proteger los recursos de Azure. Identifique puntos débiles y brechas y revise el plan según sea necesario.
    

    Refer to NIST's publication: Guide to Test, Training, and Exercise Programs for IT Plans and Capabilities: https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-84.pdf



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-nbspfor-security-incidents"></a>10.4: Provisión de detalles de contacto de incidentes de seguridad y configuración de notificaciones de alerta&nbsp; para incidentes de seguridad

**Guía**: La información de contacto del incidente de seguridad la utilizará Microsoft para ponerse en contacto con usted si Microsoft Security Response Center (MSRC) detecta que un tercero no autorizado o ilegal ha accedido a los datos. Revise los incidentes después del hecho para asegurarse de que se resuelven los problemas.
    

Cómo establecer el contacto de seguridad de Azure Security Center: https://docs.microsoft.com/azure/security-center/security-center-provide-security-contact-details

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5: Incorporación de alertas de seguridad en el sistema de respuesta a incidentes

**Guía**: Exporte las alertas y recomendaciones de Azure Security Center y mediante la característica de exportación continua para ayudar a identificar los riesgos para los recursos de Azure. La exportación continua le permite exportar alertas y recomendaciones de forma manual o continua. Puede usar el conector de datos de Azure Security Center para transmitir las alertas a Azure Sentinel.
    

    How to configure continuous export: https://docs.microsoft.com/azure/security-center/continuous-export

    

    How to stream alerts into Azure Sentinel: https://docs.microsoft.com/azure/sentinel/connect-azure-security-center



**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6: Automatización de la respuesta a las alertas de seguridad

**Guía**: use la característica Automatización de flujo de trabajo de Azure Security Center para desencadenar automáticamente las respuestas a través de Logic Apps en las alertas y recomendaciones de seguridad para proteger los recursos de Azure.
    

    How to configure Workflow Automation and Logic Apps: https://docs.microsoft.com/azure/security-center/workflow-automation



**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Pruebas de penetración y ejercicios del equipo rojo

*Para obtener más información, consulte [Control de seguridad: Pruebas de penetración y ejercicios del equipo rojo](https://docs.microsoft.com/azure/security/benchmarks/security-control-penetration-tests-red-team-exercises).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-to-remediate-all-critical-security-findings-within-60-days"></a>11.1: Realización de pruebas de penetración periódicas de los recursos de Azure y garantía de la corrección de todos los resultados de seguridad críticos en un plazo de 60 días

**Guía**: siga las reglas de compromiso de Microsoft para asegurarse de que las pruebas de penetración no infrinjan las directivas de Microsoft: https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1.

Puede encontrar más información sobre la estrategia de Microsoft y la ejecución de las pruebas de penetración del equipo rojo y sitios activos en la infraestructura de nube, los servicios y las aplicaciones administradas por Microsoft aquí: https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Compartido

## <a name="next-steps"></a>Pasos siguientes

- Consulte la [prueba comparativa de seguridad de Azure](https://docs.microsoft.com/azure/security/benchmarks/overview).
- Obtenga más información sobre las [líneas de base de seguridad de Azure](https://docs.microsoft.com/azure/security/benchmarks/security-baselines-overview).
