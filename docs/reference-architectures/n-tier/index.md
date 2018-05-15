---
title: N 层应用程序参考体系结构
description: 介绍部署 VM 用于在 Azure 中托管企业级应用程序时可用的一些常用体系结构。
layout: LandingPage
ms.openlocfilehash: 288acc36e7c310e70240caa3ed9f2095bbb8bc58
ms.sourcegitcommit: d08f6ee27e1e8a623aeee32d298e616bc9bb87ff
ms.contentlocale: zh-CN
ms.lasthandoff: 05/10/2018
---
<!-- This file is generated! -->
<!-- See the templates in ./build/reference-architectures  -->
<!-- See data in index.json -->

# <a name="n-tier-application-reference-architectures"></a>N 层应用程序参考体系结构

<section class="series">
    <ul class="panelContent">

<!-- N-tier Windows -->
<li style="display: flex; flex-direction: column;">
    <a href="./n-tier-sql-server.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/n-tier-sql-server.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>使用 SQL Server 的 N 层应用程序</h3>
                        <p>部署已为一个使用 Windows 上的 SQL Server 的 N 层应用程序配置的 VM 和虚拟网络。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>

<!-- Multi-region Windows -->
<li style="display: flex; flex-direction: column;">
    <a href="./multi-region-sql-server.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/multi-region-application.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>使用 SQL Server 的多区域 N 层应用程序</h3>
                        <p>将一个使用 SQL Server Always On 可用性组的 N 层应用程序部署到两个区域，以实现高可用性。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>

<!-- N-tier Linux -->
<li style="display: flex; flex-direction: column;">
    <a href="./n-tier-cassandra.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/n-tier-cassandra.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>使用 Cassandra 的 N 层应用程序</h3>
                        <p>部署已为一个使用 Apache Cassandra 的 N 层应用程序配置的 VM 和虚拟网络。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>

<ul class="panelContent cardsI">
<li style="display: flex; flex-direction: column;">
    <a href="./windows-vm.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/Windows.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Windows VM</h3>
                        <p>有关在 Azure 中运行 Windows VM 的基准建议。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>

<li style="display: flex; flex-direction: column;">
    <a href="./linux-vm.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/LinuxPenguin.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Linux VM</h3>
                        <p>有关在 Azure 中运行 Linux VM 的基准建议。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>

</ul>