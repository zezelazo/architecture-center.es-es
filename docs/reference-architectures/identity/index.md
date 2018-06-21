---
title: Administración de identidades
description: Explica y compara los distintos métodos disponibles para la administración de identidades en sistemas híbridos que abarcan el límite local y de la nube con Azure.
layout: LandingPage
ms.openlocfilehash: de98ee30306f5e712759ab7140bd430cb6d4cd75
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/23/2018
ms.locfileid: "29478245"
---
<!-- This file is generated! -->
<!-- See the templates in ./build/reference-architectures  -->
<!-- See data in index.json -->

# <a name="identity-management"></a>Administración de identidades

Estas arquitecturas de referencia muestran opciones para integrar su entorno de Active Directory (AD) local con una red de Azure. <br/>[¿Qué debo elegir?](./considerations.md)

<section class="series">
    <ul class="panelContent">
    <!-- Integrate with Azure Active Directory -->
<li style="display: flex; flex-direction: column;">
    <a href="./azure-ad.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/azure-ad.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Integración con Azure Active Directory</h3>
                        <p>Integre dominios y bosques locales de Active Directory con Azure AD.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- Extend AD DS to Azure -->
<li style="display: flex; flex-direction: column;">
    <a href="./adds-extend-domain.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/adds-extend-domain.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Extensión de AD DS a Azure</h3>
                        <p>Extienda el entorno de Active Directory a Azure mediante Active Directory Domain Services.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- Create an AD DS forest in Azure -->
<li style="display: flex; flex-direction: column;">
    <a href="./adds-forest.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/adds-forest.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Creación de un bosque de AD DS en Azure</h3>
                        <p>Cree un dominio de AD independiente en Azure que sea de confianza para los dominios del bosque local.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- Extend AD FS to Azure -->
<li style="display: flex; flex-direction: column;">
    <a href="./adfs.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/adfs.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Extensión de AD FS a Azure</h3>
                        <p>Use Active Directory Federation Services para la autenticación federada y la autorización en Azure.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    </ul>
</section>

<ul class="panelContent cardsI">
</ul>