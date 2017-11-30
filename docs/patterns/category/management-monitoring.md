---
title: "Patrones de administración y supervisión"
description: "Las aplicaciones en la nube se ejecutan en un centro de datos remoto en el que no tiene un control completo de la infraestructura ni, en algunos casos, del sistema operativo. Esto puede hacer que la administración y la supervisión sean más complejas que en una implementación local. Las aplicaciones deben exponer la información del entorno de ejecución que los administradores y operadores pueden usar para administrar y supervisar el sistema, así como dar soporte a los cambios en los requisitos empresariales y de personalización sin necesidad de detener o volver a implementar la aplicación."
keywords: "Patrón de diseño"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: 6281f7e5c62593cc60727c994d5dba2c52976b75
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2017
---
# <a name="management-and-monitoring-patterns"></a>Patrones de administración y supervisión

Las aplicaciones en la nube se ejecutan en un centro de datos remoto en el que no tiene un control completo de la infraestructura ni, en algunos casos, del sistema operativo. Esto puede hacer que la administración y la supervisión sean más complejas que en una implementación local. Las aplicaciones deben exponer la información del entorno de ejecución que los administradores y operadores pueden usar para administrar y supervisar el sistema, así como dar soporte a los cambios en los requisitos empresariales y de personalización sin necesidad de detener o volver a implementar la aplicación.

| Patrón | Resumen |
| ------- | ------- |
| [Ambassador](../ambassador.md) | Crea servicios de aplicaciones auxiliares que envíen solicitudes de red en nombre de una aplicación o servicio de consumidor. |
| [Anti-Corruption Layer](../anti-corruption-layer.md) | Implementa una capa de fachada o de adaptador entre una aplicación moderna y un sistema heredado. |
| [External Configuration Store](../external-configuration-store.md) | Extrae la información de configuración del paquete de implementación de la aplicación y la lleva a una ubicación centralizada. |
| [Gateway Aggregation](../gateway-aggregation.md) | Usa una puerta de enlace para agregar varias solicitudes individuales en una sola solicitud. |
| [Gateway Offloading](../gateway-offloading.md) | Descarga una funcionalidad compartida o especializada en un proxy de puerta de enlace. |
| [Gateway Routing](../gateway-routing.md) | Enruta las solicitudes a varios servicios mediante un solo punto de conexión. |
| [Health Endpoint Monitoring](../health-endpoint-monitoring.md) | Implementa comprobaciones funcionales en una aplicación a la que pueden acceder herramientas externas a través de los puntos de conexión expuestos en intervalos regulares. |
| [Sidecar](../sidecar.md) | Implementa componentes de una aplicación en un proceso o contenedor independiente para proporcionar aislamiento y encapsulación. |
| [Strangler](../strangler.md) | Migra de forma incremental un sistema heredado reemplazando gradualmente funciones específicas por los servicios y aplicaciones nuevas. |
