---
title: ExpressRoute 用の仮想ネットワーク ゲートウェイについて | Microsoft Docs
description: ExpressRoute 用の仮想ネットワーク ゲートウェイについて説明します。
services: expressroute
documentationcenter: na
author: cherylmc
manager: carmonm
editor: ''
tags: azure-resource-manager, azure-service-management
ms.assetid: 7e0d9658-bc00-45b0-848f-f7a6da648635
ms.service: expressroute
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/22/2018
ms.author: cherylmc
ms.openlocfilehash: ae971e7743d6dd3269c0a4f976bd2a5316300f58
ms.sourcegitcommit: d74657d1926467210454f58970c45b2fd3ca088d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/28/2018
ms.locfileid: "30235327"
---
# <a name="about-virtual-network-gateways-for-expressroute"></a>ExpressRoute 用の仮想ネットワーク ゲートウェイについて
仮想ネットワーク ゲートウェイは、Azure 仮想ネットワークとオンプレミスの場所の間でネットワーク トラフィックの送信に使用されます。 ExpressRoute 接続を構成するときに、仮想ネットワーク ゲートウェイ、および仮想ネットワーク ゲートウェイの接続を作成して構成する必要があります。

仮想ネットワーク ゲートウェイを作成する場合、設定をいくつか指定します。 必須の設定の 1 つとして、ExpressRoute またはサイト間 VPN トラフィック用ゲートウェイのどちらを使用するかを指定します。 Resource Manager デプロイメント モデルでは、設定は "-GatewayType" です。

プライベート接続でネットワーク トラフィックを送信する場合、ゲートウェイの種類は 'ExpressRoute' を使用します。 これは、ExpressRoute ゲートウェイとも呼ばれます。 パブリック インターネットでネットワーク トラフィックを暗号化して送信する場合、ゲートウェイの種類 "Vpn" を使用します。 これは、VPN ゲートウェイと呼ばれます。 サイト間接続、ポイント対サイト接続、VNet 間接続のすべてで、VPN ゲートウェイが使用されます。

各仮想ネットワークに配置できる仮想ネットワーク ゲートウェイは、ゲートウェイの種類ごとに 1 つに限られています。 たとえば、-GatewayType が Vpn の仮想ネットワーク ゲートウェイと -GatewayType が ExpressRoute の仮想ネットワーク ゲートウェイをそれぞれ 1 つ配置できます。 この記事では、ExpressRoute の仮想ネットワーク ゲートウェイについて説明します。

## <a name="gwsku"></a>ゲートウェイの SKU
[!INCLUDE [expressroute-gwsku-include](../../includes/expressroute-gwsku-include.md)]

ゲートウェイをより強力なゲートウェイ SKU にアップグレードする場合、ほとんどの場合 "Resize-AzureRmVirtualNetworkGateway" PowerShell コマンドレットを使用できます。 これは、Standard および HighPerformance SKU へのアップグレードの場合でも機能します。 ただし、UltraPerformance SKU へのアップグレードでは、ゲートウェイを再作成する必要があります。

### <a name="aggthroughput"></a>ゲートウェイ SKU の推定パフォーマンス
次の表は、ゲートウェイの種類と、予測されるパフォーマンスを示したものです。 この表は、リソース マネージャーとクラシック デプロイ モデルの両方に適用されます。

[!INCLUDE [expressroute-table-aggthroughput](../../includes/expressroute-table-aggtput-include.md)]

> [!IMPORTANT]
> アプリケーションのパフォーマンスは複数の要因によって異なります。これらの要因には、エンド ツー エンドの待機時間、アプリケーションが起動するトラフィック フローの数などがあります。 テーブルの数値は、アプリケーションが理想的な環境で理論上達成できる上限を表しています。 
> 
>

## <a name="resources"></a>REST API および PowerShell コマンドレット
仮想ネットワーク ゲートウェイの構成に対して REST API および PowerShell コマンドレットを使用する場合のテクニカル リソースおよび特定構文の要件については、次のページを参照してください。

| **クラシック** | **Resource Manager** |
| --- | --- |
| [PowerShell](https://msdn.microsoft.com/library/mt270335.aspx) |[PowerShell](https://msdn.microsoft.com/library/mt163510.aspx) |
| [REST API](https://msdn.microsoft.com/library/jj154113.aspx) |[REST API](https://msdn.microsoft.com/library/mt163859.aspx) |

## <a name="next-steps"></a>次の手順
使用可能な接続構成の詳細については、 [ExpressRoute の概要](expressroute-introduction.md) に関するページを参照してください。

ExpressRoute ゲートウェイの作成の詳細については、[ExpressRoute 用の仮想ネットワーク ゲートウェイの作成](expressroute-howto-add-gateway-resource-manager.md)に関するページを参照してください。
