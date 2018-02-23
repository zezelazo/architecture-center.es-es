---
title: "Soluciones de protección de datos"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 57897c31a8abdcd801874bf92d60360f7a80d1fa
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/14/2018
---
# <a name="securing-data-solutions"></a>Soluciones de protección de datos

Para muchos, hacer que los datos estén accesibles en la nube, especialmente si se parte de trabajar exclusivamente con almacenes de datos locales, puede generar alguna inquietud por el aumento de accesibilidad a esos datos y por los nuevos métodos que se deben emplear para protegerlos.

## <a name="challenges"></a>Desafíos

* Centralizar la supervisión y el análisis de los eventos de seguridad almacenados en numerosos registros.
* Implementar el cifrado y la administración de autorizaciones entre las aplicaciones y servicios.
* Asegurarse de que la administración centralizada de identidades funciona en todos los componentes de la solución, tanto si son locales como si están en la nube.

## <a name="data-protection"></a>Protección de datos

El primer paso para proteger la información es identificar qué hay que proteger. Desarrolle directrices claras, sencillas y bien comunicadas para identificar, proteger y supervisar los conjuntos de datos más importantes sin importar donde residen. Establezca la máxima protección para los recursos que tienen un impacto enorme en la misión o rentabilidad de la organización. A estos se les conoce como recursos de alto valor o HVA (por sus siglas en inglés). Realice análisis estrictos del ciclo de vida de los HVA y de las dependencias de seguridad, y establezca los controles y condiciones de seguridad apropiados. Igualmente, identifique y clasifique los recursos confidenciales, y defina las tecnologías y procesos para aplicar automáticamente los controles de seguridad.

Una vez que se han identificado los datos que necesita proteger, reflexione sobre cómo protegerá los datos *en reposo* y los datos *en tránsito*.

* **Datos en reposo**: datos que existen de forma estática en un soporte físico, ya sea un disco magnético u óptico, local o en la nube.
* **Datos en tránsito**: son aquellos datos que se transfieren entre componentes, ubicaciones o programas. Por ejemplo, a través de la red o un bus de servicio (desde una ubicación local hacia la nube, y viceversa) o durante el proceso de entrada y salida.

Para más información sobre cómo proteger los datos en reposo o en tránsito, consulte [Procedimientos recomendados de cifrado y seguridad de datos en Azure](/azure/security/azure-security-data-encryption-best-practices).

## <a name="access-control"></a>Control de acceso

Lo importante a la hora de proteger los datos en la nube es una combinación de administración de identidades y de control de acceso. Dada la variedad y el tipo de servicios en la nube, así como la creciente popularidad de la [nube híbrida](../scenarios/hybrid-on-premises-and-cloud.md), hay varias prácticas clave que debe seguir en cuanto a la identidad y el control de acceso:

* Centralización de la administración de identidades.
* Habilitación del inicio de sesión único (SSO).
* Implementación de la administración de contraseñas.
* Aplicación de Multi-Factor Authentication (MFA) para los usuarios.
* Uso del control de acceso basado en rol (RBAC).
* Configuración de las directivas de acceso condicional que mejoran el concepto clásico de la identidad de usuario con propiedades adicionales relacionadas con la ubicación del usuario, el tipo de dispositivo, el nivel de revisión, etc.
* Control de las ubicaciones donde se crean los recursos mediante Resource Manager.
* Supervisión activa de actividades sospechosas

Para más información, consulte [Procedimientos recomendados para la administración de identidades y la seguridad del control de acceso en Azure](/azure/security/azure-security-identity-management-best-practices).

## <a name="auditing"></a>Auditoría

Además de la supervisión de identidades y del acceso mencionados anteriormente, los servicios y aplicaciones que usa en la nube generan eventos relacionados con la seguridad que también se pueden supervisar. El principal desafío para supervisar estos eventos es el manejo de numerosos registros para evitar posibles problemas o solucionar aquellos que ya se han producido. Las aplicaciones basadas en la nube suelen contener muchas partes móviles, la mayoría de las cuales genera cierto nivel de registro y telemetría. Utilice la supervisión centralizada y el análisis para ayudarle a administrar y comprender la gran cantidad de información.

Para más información, consulte [Registro y auditoría de Azure](/azure/security/azure-log-audit).



## <a name="securing-data-solutions-in-azure"></a>Soluciones de protección de datos de Azure

### <a name="encryption"></a>Cifrado

**Máquinas virtuales**. Use [Azure Disk Encryption](/azure/security/azure-security-disk-encryption) para cifrar los discos conectados a las máquinas virtuales Windows o Linux. Esta solución se integra con [Azure Key Vault](/azure/key-vault/) para controlar y administrar las claves y secretos del cifrado del disco. 

**Azure Storage**. Use el [cifrado del servicio Azure Storage](/azure/storage/common/storage-service-encryption) para cifrar automáticamente los datos en reposo de Azure Storage. La administración de claves, el cifrado y el descifrado son completamente transparentes para los usuarios. Los datos también se pueden proteger en tránsito mediante el cifrado del lado cliente con Azure Key Vault. Consulte [Cifrado del lado de cliente y Azure Key Vault para Microsoft Azure Storage](/azure/storage/common/storage-client-side-encryption) para más información.

**SQL Database** y **Azure SQL Data Warehouse**. Use el [cifrado de datos transparente](/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql) (TDE) para realizar el cifrado y descifrado de las bases de datos en tiempo real, las copias de seguridad asociadas y los archivos de registro de transacciones sin necesidad de efectuar cambios en las aplicaciones. SQL Database también puede usar [Always Encrypted](/azure/sql-database/sql-database-always-encrypted-azure-key-vault) para ayudar a proteger la información confidencial en reposo en el servidor durante el traslado entre el cliente y el servidor, y mientras los datos están en uso. Puede usar Azure Key Vault para almacenar las claves de cifrado de Always Encrypted. 

### <a name="rights-management"></a>Rights management

[Azure Rights Management](/information-protection/understand-explore/what-is-azure-rms) es un servicio basado en la nube que usa directivas de cifrado, identidad y autorización para proteger archivos y correo electrónico. Funciona en varios dispositivos: teléfonos, tabletas y PC. La información se puede proteger dentro de la organización y fuera de ella ya que la protección permanece con los datos, incluso cuando estos traspasan los límites de su organización.

### <a name="access-control"></a>Control de acceso

Use el [control de acceso basado en rol](/azure/active-directory/role-based-access-control-what-is) (RBAC) para restringir el acceso a los recursos de Azure basados en roles de usuario. Si usa Active Directory de forma local, puede [sincronizar con Azure AD](/azure/active-directory/active-directory-hybrid-identity-design-considerations-directory-sync-requirements) para proporcionar a los usuarios una identidad de nube basada en su identidad local.

Use el [acceso condicional de Azure Active Directory](/azure/active-directory/active-directory-conditional-access-azure-portal) para aplicar controles en el acceso a las aplicaciones de su entorno según condiciones específicas. Por ejemplo, la instrucción de la directiva puede seguir este modelo: _Si los contratistas intentan acceder a nuestras aplicaciones en la nube desde redes que no son de confianza, se bloqueará el acceso_. 

[Azure AD Privileged Identity Management](/azure/active-directory/active-directory-privileged-identity-management-configure) puede ayudarle a administrar, controlar y supervisar los usuarios y el tipo de tareas que ellos realizan con sus privilegios de administración. Este es un paso importante a la hora de limitar qué usuarios de la organización pueden realizar operaciones con privilegios en Azure AD, Office 365 o en aplicaciones SaaS, así como supervisar sus actividades.

### <a name="network"></a>Red

Para proteger los datos en tránsito, utilice siempre SSL o TLS al intercambiar datos entre diferentes ubicaciones. En ocasiones, tiene que aislar todo el canal de comunicación entre su infraestructura local y la nube mediante una red privada virtual (VPN) o [ExpressRoute](/azure/expressroute/). Para más información, consulte [Ampliación de las soluciones de datos locales a la nube](../scenarios/hybrid-on-premises-and-cloud.md).

Use [grupos de seguridad de red](/azure/virtual-network/virtual-networks-nsg) (NSG) para reducir el número de posibles vectores de ataque. Un grupo de seguridad de red contiene una lista de reglas de seguridad que permiten o deniegan el tráfico de red entrante o saliente en función de las direcciones IP de origen o destino, el puerto y el protocolo. 

Use [los puntos de conexión de servicio de Virtual Network](/azure/virtual-network/virtual-network-service-endpoints-overview) para proteger los recursos de Azure SQL o Azure Storage, de forma que solo pueda acceder a estos recursos el tráfico procedente de su red virtual.

Las máquinas virtuales de una instancia de Azure Virtual Network (VNet) se pueden comunicar de forma segura con otras redes virtuales mediante el [emparejamiento de redes virtuales](/azure/virtual-network/virtual-network-peering-overview). El tráfico de red entre redes virtuales emparejadas es privado. El tráfico entre las redes virtuales se mantiene en la red troncal de Microsoft.

Para más información, consulte [Azure Network Security](/azure/security/azure-network-security).

### <a name="monitoring"></a>Supervisión

[Azure Security Center](/azure/security-center/security-center-intro) recopila, analiza e integra automáticamente los datos de registro de los recursos de Azure, la red y las soluciones de asociados conectados como, por ejemplo, las soluciones de firewall, para detectar amenazas reales y reducir los falsos positivos. 

[Log Analytics](/azure/log-analytics/log-analytics-overview) proporciona acceso centralizado a los registros y le ayuda a analizar esos datos y a crear alertas personalizadas.

La [detección de amenazas de Azure SQL Database](/azure/sql-database/sql-database-threat-detection) detecta actividad anómala que indica intentos inusuales y potencialmente peligrosos de acceder a las bases de datos o de vulnerar su seguridad. Los responsables de seguridad u otros administradores designados pueden recibir una notificación inmediata sobre las actividades sospechosas en las bases de datos cuando se producen. Cada notificación proporciona detalles de la actividad sospechosa y recomienda cómo investigar más y mitigar la amenaza.


