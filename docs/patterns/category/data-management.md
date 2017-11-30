---
title: "Patrones de administración de datos"
description: "La administración de datos es el elemento clave de las aplicaciones en la nube e influye en la mayoría de los atributos de calidad. Los datos se hospedan normalmente en distintas ubicaciones y entre varios servidores por motivos tales como el rendimiento, la escalabilidad o la disponibilidad, lo cual puede conllevar varios desafíos. Por ejemplo, se debe mantener la coherencia de los datos y estos deben estar sincronizados entre las diferentes ubicaciones."
keywords: "Patrón de diseño"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: a009a06268f114ab7be4544dd81710612dabd8f4
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2017
---
# <a name="data-management-patterns"></a>Patrones de administración de datos

[!INCLUDE [header](../../_includes/header.md)]

La administración de datos es el elemento clave de las aplicaciones en la nube e influye en la mayoría de los atributos de calidad. Los datos se hospedan normalmente en distintas ubicaciones y entre varios servidores por motivos tales como el rendimiento, la escalabilidad o la disponibilidad, lo cual puede conllevar varios desafíos. Por ejemplo, se debe mantener la coherencia de los datos y estos deben estar sincronizados entre las diferentes ubicaciones.

| Patrón | Resumen |
| ------- | ------- |
| [Cache-Aside](../cache-aside.md) | Carga datos a petición en una memoria caché desde un almacén de datos |
| [CQRS](../cqrs.md) | Segrega las operaciones de lectura de datos de las de actualización de datos mediante interfaces independientes. |
| [Event Sourcing](../event-sourcing.md) | Usa un almacén de solo anexar para registrar la serie completa de eventos que describen las acciones realizadas en los datos de un dominio. |
| [Index Table](../index-table.md) | Crea índices en los campos de los almacenes de datos a los que suelen hacer referencia las consultas. |
| [Materialized View](../materialized-view.md) | Genera vistas rellenadas previamente de los datos en uno o más almacenes de datos cuando los datos no tienen el formato idóneo para las operaciones de consulta requeridas. |
| [Sharding](../sharding.md) | Divide un almacén de datos en un conjunto de particiones horizontales o particiones de base de datos. |
| [Static Content Hosting](../static-content-hosting.md) | Implementa contenido estático en un servicio de almacenamiento basado en la nube que puede enviarlo directamente al cliente. |
| [Valet Key](../valet-key.md) | Usa un token o clave que proporciona a los clientes acceso directo restringido a un recurso o servicio específico. |