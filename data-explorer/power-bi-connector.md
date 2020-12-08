---
title: Power BI 用 Azure Data Explorer コネクタを使用してデータを視覚化する
description: この記事では、Power BI でデータを視覚化するための 3 つのオプションのうち 1 つ (Azure Data Explorer 用 Power BI コネクタ) を使用する方法について説明します。
author: orspod
ms.author: orspodek
ms.reviewer: gabil
ms.service: data-explorer
ms.topic: how-to
ms.date: 12/03/2020
ms.openlocfilehash: d02f9732791bf66a488779e2bc413fa441664ef7
ms.sourcegitcommit: f134d51e52504d3ca722bdf6d33baee05118173a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/03/2020
ms.locfileid: "96563336"
---
# <a name="visualize-data-using-the-azure-data-explorer-connector-for-power-bi"></a>Power BI 用 Azure Data Explorer コネクタを使用してデータを視覚化する

Azure Data Explorer は、ログと利用統計情報データのための高速で拡張性に優れたデータ探索サービスです。 Power BI はビジネス分析ソリューションであり、データを視覚化して、組織全体で結果を共有することができます。 Azure Data Explorer には、Power BI のデータに接続する方法が 3 つ用意されています。ビルトインのコネクタを使用する方法、Azure Data Explorer からクエリをインポートする方法、SQL クエリを使用する方法です。 この記事では、ビルトインのコネクタを使用してデータを取得し、それを Power BI レポートで視覚化する方法について説明します。 Power BI ダッシュボードの作成に Azure Data Explorer ネイティブ コネクタを使用する方法は簡単です。 Power BI コネクタでは、[Import および DirectQuery 接続モード](/power-bi/desktop-directquery-about)がサポートされています。 シナリオ、スケール、およびパフォーマンスの要件に応じて、**Import** または **DirectQuery** モードを使用してダッシュボードを構築できます。 

## <a name="prerequisites"></a>前提条件

この記事を完了するには、以下が必要です。

* Azure サブスクリプションをお持ちでない場合は、開始する前に[無料の Azure アカウント](https://azure.microsoft.com/free/)を作成してください。
* Azure Active Directory のメンバーである組織の電子メール アカウント。このアカウントによって [Azure Data Explorer ヘルプ クラスター](https://dataexplorer.azure.com/clusters/help/databases/samples)に接続できます。
* [Power BI Desktop](https://powerbi.microsoft.com/get-started/) ( **[無料ダウンロード]** を選択)

## <a name="get-data-from-azure-data-explorer"></a>Azure Data Explorer からデータを取得する

まず、Azure Data Explorer ヘルプ クラスターに接続し、*StormEvents* テーブルからデータのサブセットを取り込みます。 [!INCLUDE [data-explorer-storm-events](includes/data-explorer-storm-events.md)]

1. Power BI Desktop の **[ホーム]** タブで、 **[データの取得]** 、 **[詳細]** の順に選択します。

    ![データを取得する](media/power-bi-connector/get-data-more.png)

1. *Azure Data Explorer* を検索し、 **[Azure Data Explorer]** を選択して、 **[接続]** を選択します。

    ![データを検索して取得する](media/power-bi-connector/search-get-data.png)

1. **[Azure Data Explorer (Kusto)]** 画面のフォームに次の情報を入力します。

    ![クラスター、データベース、テーブルのオプション](media/power-bi-connector/cluster-database-table.png)

    | 設定 | 値 | フィールドの説明
    |---|---|---
    | クラスター | *https://help.kusto.windows.net* | ヘルプ クラスターの URL。 その他のクラスターの場合、URL は *https://\<ClusterName\>.\<Region\>.kusto.windows.net* の形式になります。 |
    | データベース | 空白 | 接続先のクラスターでホストされるデータベース。 これは後の手順で選択します。 |
    | テーブル名 | 空白 | データベース内のいずれかのテーブル、または <code>StormEvents \| take 1000</code> のようなクエリ。 これは後の手順で選択します。 |
    | [詳細オプション] | 空白 | クエリのオプション (結果セットのサイズなど)。
    | データ接続モード | *DirectQuery* | Power BI がデータをインポートするか、直接データ ソースに接続するかを決定します。 このコネクタでは、どちらかのオプションを使用できます。 |

    > [!NOTE]
    > **Import** モードでは、データは Power BI に移行されます。 **DirectQuery** モードでは、Azure Data Explorer クラスターからデータのクエリが直接実行されます。
    >
    > 次の場合は **Import** モードを使用します。
    >
    > * データ セットが小さい。
    > * ほぼリアルタイムのデータが必要ない。
    > * データは既に集計済みか、[Kusto で集計](kusto/query/summarizeoperator.md#list-of-aggregation-functions)を実行している。
    >
    > 次の場合は **DirectQuery** モードを使用します。
    > * データ セットが非常に大きい。
    > * ほぼリアルタイムのデータが必要。

    ### <a name="advanced-options"></a>[詳細オプション]

    | 設定 | 値の例 | フィールドの説明
    |---|---|---
    | クエリ結果のレコード数を制限する| `300000` | 結果で返されるレコードの最大数 |
    | クエリ結果のデータ サイズを制限する | `4194304` | 結果で返される最大データ サイズ (バイト単位) |
    | 結果セットの切り詰めを無効にする | `true` | notruncation 要求オプションを使用して、結果の切り詰めを有効化または無効化します |
    | 追加の SET ステートメント | `set query_datascope=hotcache` | クエリの実行中のクエリ オプションを設定します。 クエリ オプションは、クエリの実行方法とクエリが結果を返す方法を制御します。 |

1. ヘルプ クラスターへの接続がまだない場合は、サインインしてください。 組織アカウントでサインインし、 **[接続]** を選択します。

    ![サインイン](media/power-bi-connector/sign-in.png)

1. **[ナビゲーター]** 画面で **[Samples]** データベースを展開し、 **[StormEvents]** を選択して **[データの変換]** を選択します。

    ![テーブルの選択](media/power-bi-connector/select-table.png)

    テーブルが Power Query エディターで開き、データをインポートする前に行と列を編集できます。

1. Power Query エディターで、 **[DamageCrops]** 列の横の矢印を選択して、 **[降順に並べ替え]** を選択します。

    ![DamageCrops を降順に並べ替え](media/power-bi-connector/sort-descending.png)

1. **[ホーム]** タブで **[行の保持]** を選択し、 **[上位の行を保持]** を選択します。 並べ替えたテーブルの上位 1,000 行を取り込むために、値として「*1000*」を入力します。

    ![上位の行を保持](media/power-bi-connector/keep-top-rows.png)

1. **[ホーム]** タブの **[Close & Apply]\(閉じて適用\)** を選択します。

    ![閉じて適用する](media/power-bi-connector/close-apply.png)

## <a name="visualize-data-in-a-report"></a>レポートでデータを視覚化する

[!INCLUDE [data-explorer-power-bi-visualize-basic](includes/data-explorer-power-bi-visualize-basic.md)]

## <a name="clean-up-resources"></a>リソースをクリーンアップする

この記事で作成したレポートが不要になったら、Power BI Desktop (.pbix) ファイルを削除してください。

## <a name="next-steps"></a>次のステップ

[Power BI 用 Azure Data Explorer コネクタを使用してデータのクエリを実行するためのヒント](power-bi-best-practices.md#tips-for-using-the-azure-data-explorer-connector-for-power-bi-to-query-data)