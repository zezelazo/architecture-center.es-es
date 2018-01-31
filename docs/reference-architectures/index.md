---
title: Arquitecturas de referencia de Azure
description: "Arquitecturas de referencia, planos y guía de implementación preceptiva para cargas de trabajo comunes en Azure."
layout: LandingPage
ms.openlocfilehash: 0cc03f87dae39517e1a72a65d4767dcc21879d8f
ms.sourcegitcommit: 9998334bebccb86be0f715ac7dffc0c3175aea68
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/26/2018
---
# <a name="azure-reference-architectures"></a>Arquitecturas de referencia de Azure

Nuestras arquitecturas de referencia se organizan por escenario, agrupando juntas aquellas arquitecturas relacionadas. Cada arquitectura incluye procedimientos recomendados junto con consideraciones sobre escalabilidad, disponibilidad, capacidad de administración y seguridad. La mayoría también incluye una solución que se puede implementar.

<section class="series">
    <ul class="panelContent">
    <!--Windows VM -->
    <li>
        <a href="./virtual-machines-windows/index.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                                <img src="./virtual-machines-windows/images/n-tier.svg" height="140px" />
                            </div>
                        </div>
                        <div class="cardText">
                            <h3>Cargas de trabajo de máquinas virtuales Windows</h3>
                            <p>Esta serie comienza con los procedimientos recomendados para ejecutar una sola máquina virtual Windows, posteriormente, varias máquinas virtuales con equilibrio de carga y, por último, una aplicación de n niveles de varias regiones.</p>
                        </div>
                    </div>
                </div>
            </div>
        </a>
    </li>
    <!-- Linux VM -->
    <li>
        <a href="./virtual-machines-linux/index.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                                <img src="./virtual-machines-linux/images/n-tier.svg" height="140px" />
                            </div>
                        </div>
                        <div class="cardText">
                            <h3>Cargas de trabajo de máquinas virtuales Linux</h3>
                            <p>Esta serie comienza con los procedimientos recomendados para ejecutar una sola máquina virtual Linux, posteriormente, varias máquinas virtuales con equilibrio de carga y, por último, una aplicación de n niveles de varias regiones.</p>
                        </div>
                    </div>
                </div>
            </div>
        </a>
    </li>
    <!-- Hybrid network -->
    <li>
        <a href="./hybrid-networking/index.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                                <img src="./hybrid-networking/images/vpn.svg" height="140px" />
                            </div>
                        </div>
                        <div class="cardText">
                            <h3>Red híbrida</h3>
                            <p>Esta serie muestra opciones para crear una conexión de red entre una red local y Azure.</p>
                        </div>
                    </div>
                </div>
            </div>
        </a>
    </li>
    <!-- DMZ -->
    <li>
        <a href="./dmz/index.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                                <img src="./dmz/images/secure-vnet-dmz.svg" height="140px" />
                            </div>
                        </div>
                        <div class="cardText">
                            <h3>Red perimetral</h3>
                            <p>Esta serie muestra cómo crear una red perimetral para proteger el límite entre una red virtual de Azure y una red local o Internet.</p>
                        </div>
                    </div>
                </div>
            </div>
        </a>
    </li>
    <!-- Identity -->
    <li>
        <a href="./identity/index.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                                <img src="./identity/images/adds-extend-domain.svg" height="140px" >
                            </div>
                        </div>
                        <div class="cardText">
                            <h3>Administración de identidades</h3>
                            <p>Esta serie muestra opciones para integrar el entorno local de Active Directory (AD) con una red de Azure.</p>
                        </div>
                    </div>
                </div>
            </div>
        </a>
    </li>
    <!-- Managed web app -->
    <li>
        <a href="./app-service-web-app/index.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                                <img src="./app-service-web-app/images/scalable-web-app.svg" height="140px" />
                            </div>
                        </div>
                        <div class="cardText">
                            <h3>Aplicación web de App Service</h3>
                            <p>Esta serie muestra los procedimientos recomendados para las aplicaciones web que utilizan Azure App Service.</p>
                        </div>
                    </div>
                </div>
            </div>
        </a>
    </li>
    </ul>

<ul class="panelContent cardsI">
<li>
    <a href="./jenkins/index.md">
    <div class="cardSize">
        <div class="cardPadding">
            <div class="card">
                <div class="cardImageOuter">
                    <div class="cardImage">
                        <img src="./jenkins/images/logo.svg" alt="Jenkins" height="100%" />
                    </div>
                </div>
                <div class="cardText">
                    <h3>Servidor de compilación de Jenkins</h3>
                    <p>Implemente y haga funcionar un servidor Jenkins escalable y de nivel empresarial en Azure.</p>
                </div>
            </div>
        </div>
    </div>
    </a>
</li>

<li>
    <a href="./sharepoint/index.md">
    <div class="cardSize">
        <div class="cardPadding">
            <div class="card">
                <div class="cardImageOuter">
                    <div class="cardImage">
                        <img src="./sharepoint/images/sharepoint.svg" alt="SharePoint Server 2016" height="100%" />
                    </div>
                </div>
                <div class="cardText">
                    <h3>Granja de SharePoint Server 2016</h3>
                    <p>Implemente y ejecute una granja de alta disponibilidad de SharePoint Server 2016 en Azure con los grupos de disponibilidad AlwaysOn de SQL Server.</p>
                </div>
            </div>
        </div>
    </div>
    </a>
</li>

<li>
    <a href="./sap/index.md">
    <div class="cardSize">
        <div class="cardPadding">
            <div class="card">
                <div class="cardImageOuter">
                    <div class="cardImage">
                        <img src="./sap/images/sap.svg" alt="SAP NetWeaver and SAP HANA" width="100%" />
                    </div>
                </div>
                <div class="cardText">
                    <h3>SAP NetWeaver y SAP HANA</h3>
                    <p>Implemente y ejecute SAP NetWeaver y SAP HANA en un entorno de alta disponibilidad en Azure.</p>
                </div>
            </div>
        </div>
    </div>
    </a>
</li>
</ul>


</section>

