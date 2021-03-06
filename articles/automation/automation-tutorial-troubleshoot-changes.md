---
title: Azure 仮想マシンの変更に関する問題を解決する | Microsoft Docs
description: 変更履歴を使用して、Azure 仮想マシンの変更に関する問題を解決します。
services: automation
ms.service: automation
ms.component: change-inventory-management
keywords: 変更, 追跡, オートメーション
author: jennyhunter-msft
ms.author: jehunte
ms.date: 02/28/2018
ms.topic: tutorial
ms.custom: mvc
manager: carmonm
ms.openlocfilehash: 62d34f82749900e161bebdb7a1a8d470b2e85bbf
ms.sourcegitcommit: d28bba5fd49049ec7492e88f2519d7f42184e3a8
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/11/2018
ms.locfileid: "34054692"
---
# <a name="troubleshoot-changes-in-your-environment"></a>環境の変更に関する問題を解決する

このチュートリアルでは、Azure 仮想マシンの変更に関する問題を解決する方法について説明します。 変更履歴を有効にして、コンピューター上のソフトウェア、ファイル、Linux デーモン、Windows サービス、Windows レジストリ キーの変更を追跡することができます。
このような構成の変更を識別することで、環境全体の運用上の問題を特定できるようになります。

このチュートリアルで学習する内容は次のとおりです。

> [!div class="checklist"]
> * 変更履歴とインベントリのために VM をオンボードする
> * 停止したサービスの変更ログを検索する
> * 変更の追跡を構成する
> * アクティビティ ログの接続を有効にする
> * イベントをトリガーする
> * 変更を表示する

## <a name="prerequisites"></a>前提条件

このチュートリアルを完了するには、次のものが必要です。

* Azure サブスクリプション。 まだお持ちでない場合は、[MSDN サブスクライバーの特典を有効にする](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/)か、[無料アカウント](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)にサインアップしてください。
* 監視およびアクションの Runbook と監視タスクを保持する、[Automation アカウント](automation-offering-get-started.md)。
* オンボードする[仮想マシン](../virtual-machines/windows/quick-create-portal.md)。

## <a name="log-in-to-azure"></a>Azure にログインする

Azure Portal (http://portal.azure.com) にログインします。

## <a name="enable-change-tracking-and-inventory"></a>変更履歴とインベントリを有効にする

このチュートリアルでは、まず VM の変更履歴とインベントリを有効にする必要があります。 VM の別の Automation ソリューションで有効にした場合、この手順は不要です。

1. 左側のメニューで **[仮想マシン]** を選択し､一覧から VM を選択します。
1. 左側のメニューの **[操作]** セクションで、**[インベントリ]** をクリックします。 **[変更履歴]** ページが表示されます。

![変更履歴の有効化](./media/automation-tutorial-troubleshoot-changes/enableinventory.png) **[変更履歴]** 画面が表示されます。 使用する場所、Log Analytics ワークスペース、Automation アカウントを構成し、**[有効にする]** をクリックします。 フィールドが淡色表示されている場合は、その VM で別の Automation ソリューションが有効になっているため、同じワークスペースと Automation アカウントを使用する必要があることを示します。

[Log Analytics](../log-analytics/log-analytics-overview.md?toc=%2fazure%2fautomation%2ftoc.json) ワークスペースは、インベントリのような機能およびサービスによって生成されるデータを収集するために使用されます。
ワークスペースには、複数のソースからのデータを確認および分析する場所が 1 つ用意されています。

オンボーディング時には、VM と共に Microsoft Monitoring Agent (MMA) とハイブリッド worker がプロビジョニングされます。
このエージェントは VM との通信に使用され、インストールされているソフトウェアに関する情報を取得します。

ソリューションを有効にするには最大 15 分かかります。 この処理中はブラウザーのウィンドウは閉じないでください。
ソリューションが有効になると、VM にインストールされているソフトウェアと変更に関する情報が Log Analytics に送られます。
データの分析に使用できるようになるまでに、30 分から 6 時間かかる場合があります。

## <a name="using-change-tracking-in-log-analytics"></a>Log Analytics で変更履歴を使用する

変更の追跡は、Log Analytics に送信されるログ データを生成します。 クエリを実行してログを検索するには、**[変更の追跡]** ウィンドウの上部にある **[Log Analytics]** ウィンドウを選択します。
変更履歴データは、型 **ConfigurationChange** に格納されます。 次のサンプル Log Analytics クエリは、停止しているすべての Windows サービスを返します。

```
ConfigurationChange
| where ConfigChangeType == "WindowsServices" and SvcState == "Stopped"
```

Log Analytics でのログ ファイルの実行と検索については、[Azure Log Analytics ](https://docs.loganalytics.io/index) に関するページを参照してください。

## <a name="configure-change-tracking"></a>変更履歴を構成する

変更履歴を使用すると、VM の構成の変更を追跡できます。 次にレジストリ キーとファイルの追跡を構成する手順について説明します。
 
収集および追跡するファイルとレジストリ キーを選択するには、**[変更履歴]** ページの上部にある **[設定の編集]** を選択します。

> [!NOTE]
> インベントリと変更履歴で同じ収集設定を使用し、ワークスペース レベルの構成を設定します。

**[ワークスペースの構成]** ウィンドウでは、次の 3 つのセクションで説明するように、追跡する Windows レジストリ キー、Windows ファイル、または Linux ファイルを追加します。

### <a name="add-a-windows-registry-key"></a>Windows レジストリ キーを追加する

1. **[Windows レジストリ]** タブで **[追加]** を選択します。
    **[変更履歴用の Windows レジストリを追加する]** ウィンドウが開きます。

3. **[変更履歴用の Windows レジストリを追加する]** で、追跡するキーの情報を入力し、**[保存]** をクリックします。

|プロパティ  |[説明]  |
|---------|---------|
|有効     | 設定が適用されるかどうかを決定します。        |
|Item Name     | 追跡するファイルのフレンドリ名。        |
|グループ     | ファイルを論理的にグループ化するためのグループ名。        |
|Windows レジストリ キー   | ファイル確認のためのパス (例: "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders\Common Startup")。      |

### <a name="add-a-windows-file"></a>Windows ファイルを追加する

1. **[Windows ファイル]** タブで **[追加]** を選択します。 **[変更履歴用の Windows ファイルを追加する]** ウィンドウが開きます。

1. **[変更履歴用の Windows ファイルを追加する]** で、追跡するファイルまたはディレクトリの情報を入力し、**[保存]** をクリックします。

|プロパティ  |[説明]  |
|---------|---------|
|有効     | 設定が適用されるかどうかを決定します。        |
|Item Name     | 追跡するファイルのフレンドリ名。        |
|グループ     | ファイルを論理的にグループ化するためのグループ名。        |
|パスの入力     | ファイル確認のためのパス。例: "c:\temp\myfile.txt"       |

### <a name="add-a-linux-file"></a>Linux ファイルを追加する

1. **[Linux ファイル]** タブで **[追加]** を選択します。 **[変更履歴用の Linux ファイルを追加する]** ウィンドウが開きます。

1. **[変更履歴用の Linux ファイルを追加する]** で、追跡するファイルまたはディレクトリの情報を入力し、**[保存]** をクリックします。

|プロパティ  |[説明]  |
|---------|---------|
|有効     | 設定が適用されるかどうかを決定します。        |
|Item Name     | 追跡するファイルのフレンドリ名。        |
|グループ     | ファイルを論理的にグループ化するためのグループ名。        |
|パスの入力     | ファイル確認のためのパス (例: "/etc/*.conf")。       |
|パスの種類     | 追跡する項目の種類。"ファイル" または "ディレクトリ" を指定できます。        |
|再帰     | 追跡する項目を検索するときに、再帰を使用するかどうかを決定します。        |
|sudo の使用     | この設定により、項目を確認するときに、sudo を使用するかどうかが決まります。         |
|リンク     | この設定により、ディレクトリを走査するときの、シンボリック リンクの処理方法が決まります。<br> **無視** - シンボリック リンクを無視し、参照先のファイル/ディレクトリを含めません。<br>**フォロー** - 再帰中、シンボリック リンクをフォローします。参照先のファイル/ディレクトリも含めます。<br>**管理** - シンボリック リンクをフォローします。また、返された内容の処理を変更することができます。      |

   > [!NOTE]   
   > "管理" リンク オプションはお勧めしません。 ファイルのコンテンツの取得はサポートされていません。

## <a name="enable-activity-log-connection"></a>アクティビティ ログの接続を有効にする

VM の **[変更履歴]** ページから **[アクティビティ ログ接続の管理]** を選択します。 このタスクで、**[Azure アクティビティ ログ]** ページが開きます。 **[接続]** を選択して、変更履歴を VM の Azure アクティビティ ログに接続します。

この設定を有効にして、VM の **[概要]** ページに移動し、**[停止]** を選択して VM を停止します。 メッセージが表示されたら、**[はい]** を選択して VM を停止します。 割り当てが解除されたら、**[スタート]** を選択して VM を再起動します。

VM の起動時と停止時には、イベントがアクティビティ ログに記録されます。 **[変更履歴]** ページに戻ります。 ページの下部にある **[イベント]** タブを選択します。 しばらくすると、グラフと表にイベントが表示されます。 前の手順と同様に、各イベントを選択してイベントの詳細情報を表示することができます。

![ポータルで変更の詳細を表示する](./media/automation-tutorial-troubleshoot-changes/viewevents.png)

## <a name="view-changes"></a>変更を表示する

変更履歴とインベントリ ソリューションが有効になると、**[変更履歴]** ページで結果を表示できます。

VM 内から **[操作]** の **[変更履歴]** を選択します。

![VM に加えられた変更の一覧を表示するスクリーン ショット](./media/automation-tutorial-troubleshoot-changes/change-tracking-list.png)

グラフには、時間の経過とともに発生した変更が表示されます。
アクティビティ ログ接続を追加すると、上部の折れ線グラフに Azure アクティビティ ログ イベントが表示されます。
棒グラフの各行は、さまざまな追跡可能な変更の種類を表します。
具体的には、Linux デーモン、ファイル、Windows レジストリ キー、ソフトウェア、および Windows サービスです。
[変更] タブには、変更が発生した時刻の降順 (最新のものから順に) で、変更の詳細が図示されます。
**[イベント]** タブの表には、接続されたアクティビティ ログ イベントとそれに対応する詳細が最新の情報から表示されます。

結果を見ると、サービスとソフトウェアの変更を含め、システムに複数の変更があったことがわかります。 ページ上部にあるフィルターを使用し、**[型の変更]** または期間を指定して結果を絞り込むことができます。

**WindowsServices** の変更を選択すると、**[変更の詳細]** ウィンドウが開きます。 [変更の詳細] ウィンドウには、変更の詳細と、変更前の値と変更後の値が表示されます。 この例では、Software Protection サービスが停止されました。

![ポータルで変更の詳細を表示する](./media/automation-tutorial-troubleshoot-changes/change-details.png)

## <a name="next-steps"></a>次の手順

このチュートリアルで学習した内容は次のとおりです。

> [!div class="checklist"]
> * 変更履歴とインベントリのために VM をオンボードする
> * 停止したサービスの変更ログを検索する
> * 変更の追跡を構成する
> * アクティビティ ログの接続を有効にする
> * イベントをトリガーする
> * 変更を表示する

さらに詳しく学ぶには、変更履歴とインベントリ ソリューションの概要に進んでください。

> [!div class="nextstepaction"]
> [変更管理とインベントリ ソリューション](automation-change-tracking.md)
