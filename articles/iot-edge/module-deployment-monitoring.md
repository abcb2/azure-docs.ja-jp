---
title: Azure IoT Edge のモジュールをデプロイする | Microsoft Docs
description: モジュールがエッジ デバイスにどのようにデプロイされるかについて説明します
services: iot-edge
keywords: ''
author: kgremban
manager: timlt
ms.author: kgremban
ms.date: 10/05/2017
ms.topic: article
ms.service: iot-edge
ms.openlocfilehash: ffd3a8e6bde7310f6bdbed0e0f87419c73fcd6fc
ms.sourcegitcommit: d78bcecd983ca2a7473fff23371c8cfed0d89627
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/14/2018
ms.locfileid: "34166337"
---
# <a name="understand-iot-edge-deployments-for-single-devices-or-at-scale---preview"></a>1 台のデバイスまたは一群のデバイスを対象とした IoT Edge デプロイについて - プレビュー

Azure IoT Edge デバイスは、他のタイプの IoT デバイスと同様の[デバイス ライフサイクル][lnk-lifecycle]をたどります。

1. IoT Edge デバイスがプロビジョニングされます。これには、OS でのデバイスのイメージ化と、[IoT Edge ランタイム][lnk-runtime]のインストールが含まれます。
1. [IoT Edge モジュール][lnk-modules]を実行するようにデバイスが構成され、その後、正常性が監視されます。 
1. 最後に、デバイスが置き換えられたり古くなった場合に、必要に応じてデバイスが回収されます。  

Azure IoT Edge では、IoT Edge デバイスで実行するモジュールを構成する方法が 2 つ提供されています。1 つは、単一デバイスでの開発と迅速なイテレーションのための方法 (Azure IoT Edge のチュートリアルで使用した方法) で、もう 1 つは、多数の IoT Edge デバイスを管理するための方法です。 どちらのアプローチも、Azure Portal とプログラムから実行可能です。

この記事では、多数のデバイスの構成ステージと監視ステージ (全体を指して IoT Edge デプロイと呼ばれます) に重点を置いて説明をします。 全体的なデプロイ手順は次のとおりです。   

1. オペレーターが、一連のモジュールとターゲット デバイスを指定したデプロイを定義します。 各デプロイには、この情報を反映した配置マニフェストが指定されます。 
1. IoT Hub サービスが、すべてのターゲット デバイスと通信し、それらを目的のモジュールで構成します。 
1. IoT Hub サービスが IoT Edge デバイスからステータスを取得し、オペレーターがそれらを監視できるようにします。  たとえば、オペレーターは Edge デバイスが正常に構成されているかどうかや、実行時にモジュールでエラーが発生していないかかどうかを確認することができます。 
1. ターゲット条件を満たしている新しい IoT Edge デバイスは、随時デプロイ用に構成されます。 たとえば、ワシントン州にあるすべての IoT Edge デバイスがデプロイのターゲットである場合は、新しい IoT Edge デバイスがプロビジョニングされてワシントン州のデバイス グループに追加されると、そのデバイスが自動的に構成されます。 
 
この記事では、デプロイの構成と監視に関係する各コンポーネントについて説明します。 デプロイの作成と更新に関するチュートリアルについては、「[Deploy and monitor IoT Edge modules at scale (IoT Edge モジュールを大規模にデプロイして監視する)][lnk-howto]」を参照してください。

## <a name="deployment"></a>デプロイ

デプロイでは、実行する IoT Edge モジュール イメージが、一連のターゲット IoT Edge デバイス上のインスタンスとして割り当てられます。 これは、IoT Edge 配置マニフェストを構成し、モジュールのリストと対応する初期化パラメーターを指定することで機能します。 デプロイは、1 つのデバイスに割り当てる (通常、デバイス ID に基づく) こともできますし、デバイスのグループに割り当てる (タグに基づく) こともできます。 IoT Edge デバイスは、配置マニフェストを取得すると各コンテナー リポジトリからモジュール コンテナー イメージをダウンロードしてインストールし、それらを適切に構成します。 デプロイが作成されたら、オペレーターはデプロイ ステータスを監視して、ターゲット デバイスが正しく構成されているかどうかを確認できます。   

デバイスをデプロイで構成するには、そのデバイスを IoT Edge デバイスとしてプロビジョニングする必要があります。 次に示すのは前提要件であり、デプロイには含まれません。
* ベース オペレーティング システム
* Docker 
* IoT Edge ランタイムのプロビジョニング 

### <a name="deployment-manifest"></a>配置マニフェスト

配置マニフェストは、ターゲットの IoT Edge デバイス上で構成されるモジュールを記述した、JSON ドキュメントです。 デプロイ マニフェストには、必要なシステム モジュール (具体的には、IoT Edge エージェントと IoT Edge ハブ) など、すべてのモジュールの構成メタデータが含まれます。  

各モジュールの構成メタデータには、次のものが含まれます。 
* バージョン 
* type 
* ステータス (例: 実行中または停止中) 
* 再起動ポリシー 
* イメージおよびコンテナー リポジトリ 
* データ入出力のルート 

### <a name="target-condition"></a>ターゲット条件

ターゲット条件は、デプロイの有効期間をとおして、要求を満たす新しいデバイスを含めたり、要求を満たさなくなったデバイスを削除したりするために、継続的に評価されます。 サービスがターゲット条件の変化を検出した場合、デプロイが再アクティブ化されます。 たとえば、ターゲット条件が tags.environment = 'prod' であるデプロイ A があるものとします。 デプロイを開始するときは、10 個の prod デバイスが存在します。 モジュールは、これら 10 個のデバイスに正常にインストールされます。 IoT Edge エージェントの状態には、合計デバイス数 10、正常応答数 10、異常応答数 0、保留中応答数 0 と表示されます。 そして、tags.environment = 'prod' を設定したデバイスを 5 個追加します。 サービスは変更を検出し、5 個の新しいデバイスのデプロイを試みるときに、IoT Edge エージェントの状態は、合計デバイス数 15、正常応答数 10、異常応答数 0、保留中応答数 5 と表示されます。

ターゲット デバイスを選ぶには、デバイス ツイン タグまたは deviceId に対する任意のブール条件を使います。 タグで条件を使う場合は、デバイス ツインのプロパティと同じレベルに "タグ":{} セクションを追加する必要があります。 [デバイス ツインのタグに関する詳細](../iot-hub/iot-hub-devguide-device-twins.md)

ターゲット条件の例:
* deviceId ='linuxprod1'
* tags.environment ='prod'
* tags.environment = 'prod' AND tags.location = 'westus'
* tags.environment = 'prod' OR tags.location = 'westus'
* tags.operator = 'John' AND tags.environment = 'prod' NOT deviceId = 'linuxprod1'

ターゲット条件を作成するときは、次のような制約があります。

* デバイス ツインでは、ターゲット条件を作成するときに使うことができるのはタグまたは deviceId だけです。
* ターゲット条件のどの部分でも、二重引用符を使うことはできません。 単一引用符を使ってください。
* 単一引用符は、ターゲット条件の値を表します。 そのため、デバイス名に単一引用符が含まれる場合は、別の単一引用符でエスケープする必要があります。 たとえば、operator'sDevice に対するターゲット条件は、deviceId='operator''sDevice' と記述する必要があります。
* ターゲット条件の値では、数字、文字、および -:.+%_#*?!(),=@;$ の各文字を使うことができます。

### <a name="priority"></a>優先順位

優先順位では、デプロイをターゲット デバイスに適用する際、他のデプロイとの関係を考慮するかどうかを定義します。 デプロイ優先順位は正の整数で指定され、数が大きいほど優先順位が高いことを示します。 IoT Edge デバイスが複数のデプロイのターゲットになっている場合は、優先順位の最も高いデプロイが適用されます。  優先順位の低いデプロイは適用されず、マージもされません。  デバイスが複数のデプロイのターゲットになっていて、それらの優先順位が同じである場合は、直近に作成されたデプロイが適用されます (作成日時のタイムスタンプによって決定されます)。

### <a name="labels"></a>ラベル 

ラベルは、デプロイをフィルター処理してグループ化するために使用できる、文字列のキー/値ペアです。 デプロイには複数のラベルを指定することもできます。 ラベルは省略可能であり、IoT Edge デバイスの実際の構成には影響しません。 

### <a name="deployment-status"></a>[デプロイ ステータス]

デプロイは監視することができます。具体的には、任意のターゲット IoT Edge デバイスについて、デプロイが正常に適用されているかどうかを確認することができます。  ターゲット Edge デバイスは、次の 1 つ以上のステータス カテゴリで表示されます。 
* **ターゲット**: IoT Edge デバイスがデプロイのターゲット条件に一致していることを示します。
* **実際**: ターゲット IoT Edge デバイスが、より優先順位の高い別のデプロイのターゲットになっていないことを示します。
* **正常**: モジュールが正常にデプロイされたことが、IoT Edge デバイスからサービスに報告されたことを示します。 
* **異常**: 1 つ以上のモジュールが正常にデプロイされなかったことが、IoT Edge デバイスからサービスに報告されたことを示します。 エラーについて詳しく調べるには、そのデバイスにリモートで接続し、ログ ファイルを参照します。
* **不明**: IoT Edge デバイスから、このデプロイに関するステータスが報告されなかったことを示します。 詳細を調べるには、サービス情報とログ ファイルを参照します。

## <a name="phased-rollout"></a>フェーズ ロールアウト 

フェーズ ロールアウトとは、IoT Edge デバイスの幅広いセットに対してオペレーターが変更をデプロイする、全体的なプロセスのことです。 その目的は、変更を段階的に適用することで、広範囲な変更によるリスクを減らすことです。  

フェーズ ロールアウトは、次のフェーズと手順に従って実行されます。 
1. IoT Edge デバイスをプロビジョニングし、`tag.environment='test'` などのデバイス ツイン タグを設定することで、IoT Edge デバイスのテスト環境を作成します。 テスト環境では、実際にデプロイのターゲットとなる本番環境を厳密に再現する必要があります。 
1. デプロイを作成します (目的のモジュールや構成など)。 ターゲット条件では、IoT Edge デバイスのテスト環境をターゲットにする必要があります。   
1. テスト環境で新しいモジュール構成を検証します。
1. ターゲット条件に新しいタグを追加してデプロイを更新し、本番用の IoT Edge デバイスのサブセットを含めます。 また、デプロイの優先順位が、現在それらのデバイスをターゲットにしている他のデプロイよりも高いことを確認します 
1. デプロイ ステータスを参照して、ターゲット IoT デバイスについてデプロイが成功したことを確認します。
1. デプロイを更新して、残っているすべての本番用 IoT Edge デバイスをターゲットにします。

## <a name="rollback"></a>ロールバック

エラーや構成ミスが発生した場合には、デプロイをロールバックすることができます。  デプロイでは IoT Edge デバイスの固定的なモジュール構成が定義されるため、追加のデプロイでも、同じデバイスをより低い優先順位でターゲットにする必要があります。このことは、目的が全モジュールの削除である場合も同様です。  

ロールバックは次の順序で実行します。 
1. 2 番目のデプロイでも同じデバイスセットがターゲットになっていることを確認します。 ロールバックの目的が全モジュールの削除である場合、2 番目のデプロイにはモジュールを含めません。 
1. ロールバックするデプロイのターゲット条件式を変更または削除して、デバイスがターゲット条件に一致しなくなるようにします。
1. デプロイ ステータスを参照して、ロールバックが成功したことを確認します。
   * ロールバックされたデプロイでは、ロールバックされたデバイスのステータスが表示されなくなります。
   * 2 番目のデプロイに、ロールバックされたデバイスのデプロイ ステータスが表示されるようになります。


## <a name="next-steps"></a>次の手順

* 「[Deploy and monitor IoT Edge modules at scale (IoT Edge モジュールを大規模にデプロイして監視する)][lnk-howto]」で、デプロイを作成、更新、または削除するための手順を学習してください。
* [IoT Edge ランタイム][lnk-runtime]や [IoT Edge モジュール][lnk-modules]など、IoT Edge のその他の概念について学習してください。

<!-- Links -->
[lnk-lifecycle]: ../iot-hub/iot-hub-device-management-overview.md
[lnk-runtime]: iot-edge-runtime.md
[lnk-modules]: iot-edge-modules.md
[lnk-howto]: how-to-deploy-monitor.md