---
title: Arquitecturas de referencia de Azure
description: Arquitecturas de referencia, planos y guía de implementación preceptiva para cargas de trabajo comunes en Azure.
layout: LandingPage
ms.topic: landing-page
ms.date: 08/30/2018
ms.openlocfilehash: e7bbd797ba4b8bfc193b146f080cedf299dd47b4
ms.sourcegitcommit: 19a517a2fb70768b3edb9a7c3c37197baa61d9b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/26/2018
ms.locfileid: "52295521"
---
# <a name="azure-reference-architectures"></a>Arquitecturas de referencia de Azure

Nuestras arquitecturas de referencia se organizan por escenario, agrupando juntas aquellas arquitecturas relacionadas. Cada arquitectura incluye procedimientos recomendados junto con consideraciones sobre escalabilidad, disponibilidad, capacidad de administración y seguridad. La mayoría también incluye una solución que se puede implementar.

Vaya a: [AI](#ai-and-machine-learning) | [Macrodatos](#big-data-solutions) | [Sin servidor](#serverless-applications) | [Redes virtuales](#virtual-networks) | [Cargas de trabajo de máquinas virtuales](#vm-workloads) | [SAP](#sap) | [Aplicaciones web](#web-applications) | [Active Directory](#extend-on-premises-active-directory-to-azure)

## <a name="ai-and-machine-learning"></a>AI y aprendizaje automático

<ul  class="panelContent cardsF">
<!-- Batch scoring for deep learning models -->
<li style="display: flex; flex-direction: column;">
    <a href="./ai/batch-scoring-deep-learning.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/batch-ai.svg" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Puntuación de Batch para modelos de aprendizaje profundo</h3>
                        <p>Automatización de la ejecución de trabajos por lotes que aplican transferencias de estilo neuronal a un vídeo.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Batch scoring for deep learning models -->
<li style="display: flex; flex-direction: column;">
    <a href="./ai/realtime-scoring-python.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/python-powered-h.svg" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Puntuación en tiempo real de los modelos de Python</h3>
                        <p>Implementación de modelos de Python como servicios web para realizar predicciones en tiempo real mediante modelos habituales de Python o modelos de aprendizaje profundo.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="big-data-solutions"></a>Soluciones de Big data

<ul  class="panelContent cardsF">
<!-- SQL Data Warehouse -->
<li style="display: flex; flex-direction: column;">
    <a href="./data/enterprise-bi-sqldw.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/data-guide.svg" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Inteligencia empresarial con SQL Data Warehouse</h3>
                        <p>Canalización de extracción, carga y transformación (ELT) para trasladar datos de una base de datos local a SQL Data Warehouse.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./data/enterprise-bi-adf.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/data-guide.svg" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Inteligencia empresarial automatizada con Azure Data Factory</h3>
                        <p>Automatice una canalización de ELT para realizar la carga incremental desde una base de datos local.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Databricks -->
<li style="display: flex; flex-direction: column;">
    <a href="./data/stream-processing-databricks.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/databricks.png" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Procesamiento de flujos de datos con Azure Databricks</h3>
                        <p>Canalización de procesamiento de flujos que combina los registros de dos flujos, enriquece el resultado y calcula una media móvil.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Stream Analytics -->
<li style="display: flex; flex-direction: column;">
    <a href="./data/stream-processing-stream-analytics.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/azure-analysis-service.svg" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Procesamiento de flujos de datos con Azure Stream Analytics</h3>
                        <p>Canalización de procesamiento de flujos de un extremo a otro que correlaciona registros de dos flujos de datos para calcular una media acumulada.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="serverless-applications"></a>Aplicaciones sin servidor

<ul class="panelContent cardsF">
<!-- Serverless web application -->
<li style="display: flex; flex-direction: column;">
    <a href="./serverless/web-app.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/functions.svg" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Aplicación web sin servidor</h3>
                        <p>Una aplicación web sin servidor proporciona contenido estático desde Blob Storage e implementa una API con Azure Functions.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Serverless web application -->
<li style="display: flex; flex-direction: column;">
    <a href="./serverless/event-processing.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/functions.svg" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Procesamiento de eventos con Azure Functions</h3>
                        <p>Una arquitectura basada en eventos que ingiere un flujo de datos y usa Functions para procesar los datos.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="virtual-networks"></a>Redes virtuales

<ul  class="panelContent cardsF">
<li style="display: flex; flex-direction: column;">
    <a href="./hybrid-networking/vpn.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/vpn.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Red híbrida mediante una red virtual privada (VPN)</h3>
                        <p>Conecte una red local a una instancia de Azure Virtual Network.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- ExpressRoute -->
<li style="display: flex; flex-direction: column;">
    <a href="./hybrid-networking/expressroute.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/expressroute.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Red híbrida mediante ExpressRoute</h3>
                        <p>Use una conexión privada dedicada para extender una red local a Azure.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- ExpressRoute + VPN -->
<li style="display: flex; flex-direction: column;">
    <a href="./hybrid-networking/expressroute-vpn-failover.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/expressroute.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Red híbrida mediante ExpressRoute con conmutación por error de VPN</h3>
                        <p>Use ExpressRoute con una VPN como una conexión de conmutación por error para lograr alta disponibilidad.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Hub spoke -->
<li style="display: flex; flex-direction: column;">
    <a href="./hybrid-networking/hub-spoke.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/gateway.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Topología de red en estrella tipo hub-and-spoke</h3>
                        <p>Cree un punto central de conectividad a la red local mientras aísla las cargas de trabajo.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Shared services -->
<li style="display: flex; flex-direction: column;">
    <a href="./hybrid-networking/shared-services.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/gateway.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Topología en estrella tipo hub-and-spoke con servicios compartidos</h3>
                        <p>Amplíe una topología en estrella tipo hub-and-spoke incluyendo servicios compartidos como Active Directory.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Hybrid DMZ -->
<li style="display: flex; flex-direction: column;">
    <a href="./dmz/secure-vnet-hybrid.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/vnet.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Red perimetral entre Azure y una red local</h3>
                        <p>Use aplicaciones virtuales de red para crear una red híbrida segura.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Internet DMZ -->
<li style="display: flex; flex-direction: column;">
    <a href="./dmz/secure-vnet-dmz.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/vnet.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Red perimetral entre Internet y Azure</h3>
                        <p>Use aplicaciones virtuales de red para crear una red segura que acepta tráfico de Internet.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="vm-workloads"></a>Cargas de trabajo de máquinas virtuales

<ul  class="panelContent cardsF">
<!-- n-tier windows -->
<li style="display: flex; flex-direction: column;">
    <a href="./n-tier/n-tier-sql-server.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/windows.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Aplicación de n niveles con SQL Server</h3>
                        <p>Máquinas virtuales configuradas para una aplicación de n niveles mediante SQL Server en Windows.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Multi-region n-tier windows -->
<li style="display: flex; flex-direction: column;">
    <a href="./n-tier/multi-region-sql-server.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/windows.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Aplicación de n niveles para varias regiones</h3>
                        <p>Aplicación de n niveles en dos regiones para lograr alta disponibilidad mediante los Grupos de disponibilidad Always On de SQL Server.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- N-tier Linux -->
<li style="display: flex; flex-direction: column;">
    <a href="./n-tier/n-tier-cassandra.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/linux-penguin.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Aplicación de n niveles con Cassandra</h3>
                        <p>Máquinas virtuales configuradas para una aplicación de n niveles mediante Apache Cassandra en Linux.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Jenkins -->
<li style="display: flex; flex-direction: column;">
    <a href="./jenkins/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/jenkins.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Servidor de compilación de Jenkins</h3>
                        <p>Servidor Jenkins escalable y de nivel empresarial en Azure.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- SharePoint -->
<li style="display: flex; flex-direction: column;">
    <a href="./sharepoint/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/sharepoint.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Granja de SharePoint Server 2016</h3>
                        <p>Granja de alta disponibilidad de SharePoint Server 2016 en Azure con Grupos de disponibilidad AlwaysOn de SQL Server.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="sap"></a>SAP

<ul  class="panelContent cardsF">
<!-- SAP -->
<li style="display: flex; flex-direction: column;">
    <a href="./sap/sap-netweaver.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/sap.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>SAP NetWeaver</h3>
                        <p>SAP NetWeaver en Windows, en un entorno de alta disponibilidad que admite recuperación ante desastres.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./sap/sap-s4hana.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/sap.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>SAP S/4HANA</h3>
                        <p>SAP S/4HANA en Linux, en un entorno de alta disponibilidad que admite recuperación ante desastres.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./sap/hana-large-instances.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/sap.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>SAP HANA en Azure (instancias grandes)</h3>
                        <p>Las instancias de gran tamaño de HANA se implementan en servidores físicos en regiones de Azure.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="extend-on-premises-active-directory-to-azure"></a>Extensión de Active Directory local a Azure

<ul class="panelContent cardsF">
<!-- Azure AD -->
<li style="display: flex; flex-direction: column;">
    <a href="./identity/azure-ad.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/azure-active-directory.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Integración con Azure Active Directory</h3>
                        <p>Integre dominios de AD locales con Azure Active Directory.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- AD DS -->
<li style="display: flex; flex-direction: column;">
    <a href="./identity/adds-extend-domain.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/active-directory-vm.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Ampliación del dominio de Active Directory local a Azure</h3>
                        <p>Implementación de Active Directory Domain Services (AD DS) en Azure para ampliar el dominio local.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- AD DS Forest -->
<li style="display: flex; flex-direction: column;">
    <a href="./identity/adds-forest.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/active-directory-vm.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Creación de un bosque de AD DS en Azure</h3>
                        <p>Cree un dominio de AD independiente en Azure que sea de confianza para los bosques locales de AD.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- AD FS -->
<li style="display: flex; flex-direction: column;">
    <a href="./identity/adfs.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/active-directory-vm.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Extensión de Servicios de federación de Active Directory (AD FS) a Azure</h3>
                        <p>Use los Servicios de federación de Active Directory (AD FS) para la autenticación federada y la autorización de los componentes que se ejecutan en Azure.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="web-applications"></a>Aplicaciones web

<ul  class="panelContent cardsF">
<li style="display: flex; flex-direction: column;">
    <a href="./app-service-web-app/basic-web-app.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/app-service.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Aplicación web básica</h3>
                        <p>Aplicación web con Azure App Service y Azure SQL Database.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./app-service-web-app/scalable-web-app.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/app-service.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Aplicación web altamente escalable</h3>
                        <p>Procedimientos recomendados de mejora de la escalabilidad en una aplicación web.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./app-service-web-app/multi-region.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/app-service.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Aplicación web de alta disponibilidad</h3>
                        <p>Ejecute una aplicación web de App Service en varias regiones para lograr alta disponibilidad.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

