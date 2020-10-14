---
title: クイック スタート:Azure Data Explorer の Web UI でデータのクエリを実行する
description: このクイック スタートでは、Azure Data Explorer の Web UI でデータのクエリと共有を行う方法について説明します。
author: orspod
ms.author: orspodek
ms.reviewer: mblythe
ms.service: data-explorer
ms.topic: quickstart
ms.date: 06/15/2020
ms.openlocfilehash: 78666938417ce02d5320c56b247e1b854b842234
ms.sourcegitcommit: 7fa9d0eb3556c55475c95da1f96801e8a0aa6b0f
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/11/2020
ms.locfileid: "91942337"
---
# <a name="quickstart-query-data-in-azure-data-explorer-web-ui"></a>クイック スタート:Azure Data Explorer の Web UI でデータのクエリを実行する

Azure Data Explorer は、ログと利用統計情報データのための高速で拡張性に優れたデータ探索サービスです。 Azure Data Explorer の Web アプリケーションを使用して、クエリを実行および共有できます。 そのアプリケーションは、Azure portal で利用でき、スタンドアロン Web アプリケーションとしても利用できます。 この記事では、スタンドアロン バージョンを使います。複数のクラスターに接続し、クエリへのディープ リンクを共有することができます。

Azure サブスクリプションをお持ちでない場合は、開始する前に[無料の Azure アカウント](https://azure.microsoft.com/free/)を作成してください。

## <a name="prerequisites"></a>前提条件

このクイック スタートを実行するには、Azure サブスクリプションに加えて、[テスト用のクラスターとデータベース](create-cluster-database-portal.md)が必要です。

## <a name="sign-in-to-the-application"></a>アプリケーションにサインインする

[アプリケーション](https://dataexplorer.azure.com/)にサインインします。

## <a name="add-clusters"></a>クラスターを追加する

最初にアプリケーションを開いたときには、接続はありません。

![クラスターを追加する](media/web-query-data/add-cluster.png)

クエリの実行を始める前に、少なくとも 1 つのクラスターへの接続を追加する必要があります。 このセクションでは、学習補助用に設定されている "*help クラスター*" と、前のクイック スタートで作成したテスト クラスターへの接続を、Azure Data Explorer に追加します。

1. アプリケーションの左上にある **[Add cluster]\(クラスターの追加\)** を選択します。

1. **[Add cluster]\(クラスターの追加\)** ダイアログ ボックスで URI を入力して、 **[Add]\(追加\)** を選択します。

   ヘルプ クラスター URI `https://help.kusto.windows.net` を使用できます。 独自のクラスターがある場合は、そのクラスターの URI を指定します。 たとえば、次の図のように、`https://mydataexplorercluster.westus.kusto.windows.net` と指定します。

    ![ポータルでのサーバー URI](media/web-query-data/server-uri.png)

1. 左側のウィンドウに **help** クラスターが表示されます。 **Samples** データベースを展開し、アクセスできるサンプル テーブルを表示します。

    ![サンプル データベース](media/web-query-data/sample-databases.png)

    このクイック スタートでこの後、および他の Azure Data Explorer 記事では、**StormEvents** テーブルを使用します。

作成したテスト クラスターを追加します。

1. **[Add cluster]\(クラスターの追加\)** を選択します。

1. **[Add cluster]\(クラスターの追加\)** ダイアログ ボックスで、テスト クラスターの URL を `https://<ClusterName>.<Region>.kusto.windows.net/` の形式で入力して､ **[Add]\(追加\)** を選択します。

    次の例では、**help** クラスターと、新しいクラスター **docscluster.westus** (完全な URL は `https://docscluster.westus.kusto.windows.net/`) が表示されています。

    ![テスト クラスター](media/web-query-data/test-cluster.png)

## <a name="run-queries"></a>クエリを実行する

接続したどちらのクラスターに対してもクエリを実行できます (テスト クラスターにデータがあると仮定)。 ここでは **help** クラスターに注目します。

1. 左側のウィンドウで **help** クラスターの下の **Samples** データベースを選択します。

1. 次のクエリをコピーしてクエリ ウィンドウに貼り付けます。 ウィンドウの上部にある **[Run]\(実行\)** を選択します。

    ```Kusto
    StormEvents
    | sort by StartTime desc
    | take 10
    ```
    このクエリでは、**StormEvents** テーブルから最新の 10 レコードが返されます。 結果の左側は次のテーブルのようになります。

    :::image type="content" source="media/web-query-data/result-set-01.png" alt-text="10 件のストーム イベントの開始時刻、終了時刻、エピソード、イベント ID、州、イベントの種類を一覧表示した表のスクリーンショット。" border="false":::

    次の図は、クラスターが追加されてクエリと結果が表示されている、アプリケーションの現在の状態を示しています。

    ![完全なアプリケーション](media/web-query-data/full-application.png)

1. 次のクエリをコピーし、クエリ ウィンドウの最初のクエリの下に貼り付けます。 最初のクエリのように異なる行に書式設定されていないことに注意してください。

    ```Kusto
    StormEvents | sort by StartTime desc | project StartTime, EndTime, State, EventType, DamageProperty, EpisodeNarrative | take 10
    ```

1. ウィンドウで新しいクエリをクリックして、クエリを選択します。 Shift + Alt + F キーを押してクエリを書式設定すると、次の表示のようになります。

    ![書式設定されたクエリ](media/web-query-data/formatted-query.png)

1. Shift + Enter キーを押します。これは、クエリ実行のショートカットです。

   このクエリでは 1 番目と同じレコードが返されますが、`project` ステートメントで指定されている列のみが含まれます。 結果は次のテーブルのようになります。

    :::image type="content" source="media/web-query-data/result-set-02.png" alt-text="10 件のストーム イベントの開始時刻、終了時刻、エピソード、イベント ID、州、イベントの種類を一覧表示した表のスクリーンショット。" border="false":::

1. クエリ ウィンドウの上部にある **[Recall]\(リコール\)** を選択します。

    クエリを再実行する必要なしに、クエリ ウィンドウに最初のクエリの結果セットが表示されます。 分析では複数のクエリを実行することがよくあり、 **[Recall]\(リコール\)** を使用すると前のクエリの結果を再表示することができます。

1. もう 1 つクエリを実行し、異なる種類の出力を見てみましょう。

    ```Kusto
    StormEvents
    | summarize event_count=count(), mid = avg(BeginLat) by State
    | sort by mid
    | where event_count > 1800
    | project State, event_count
    | render columnchart
    ```
    結果は次のグラフのようになります。

    ![縦棒グラフ](media/web-query-data/column-chart.png)

> [!NOTE]
> クエリ式の空白行は、クエリのどの部分が実行されるかに影響を与える可能性があります。
>
> テキストが選択されていない場合は、クエリまたはコマンドが空白行で区切られていると想定されます。
> テキストが選択されている場合は、選択したテキストが実行されます。

## <a name="work-with-the-table-grid"></a>テーブル グリッドを使用する

基本的なクエリの動作を確認したので、次にテーブル グリッドを使用して結果をカスタマイズし、さらに分析を行う方法を見てみましょう。

1. 最初のクエリを再実行します。 **State** 列をマウスでポイントし、メニューを選択して、 **[Group by State]\(State でグループ化\)** を選択します。

    ![Group by State (State でグループ化)](media/web-query-data/group-by.png)

1. グリッドで **California** を展開し、その州のレコードを確認します。

    :::image type="content" source="media/web-query-data/result-set-03.png" alt-text="10 件のストーム イベントの開始時刻、終了時刻、エピソード、イベント ID、州、イベントの種類を一覧表示した表のスクリーンショット。" border="false":::

    この種類のグループ化は、探索的分析を行うときに役に立ちます。

1. **[Group]\(グループ\)** 列をマウスでポイントし、 **[Reset columns]\(列のリセット\)** を選択します。

    ![Reset columns (列のリセット)](media/web-query-data/reset-columns.png)

    グリッドが元の状態に戻ります。

1. 次のクエリを実行します。

    ```Kusto
    StormEvents
    | sort by StartTime desc
    | where DamageProperty > 5000
    | project StartTime, State, EventType, DamageProperty, Source
    | take 10
    ```

1. グリッドの右側にある **[Columns]\(列\)** を選択して、ツール パネルを表示します。

    ![ツール パネル](media/web-query-data/tool-panel.png)

    このパネルは Excel のピボット テーブル フィールド リストと同様に機能し、グリッド自体でさらに分析を実行できます。

1. **[Pivot Mode]\(ピボット モード\)** を選択し、**State** 列を **[Row groups]\(行グループ\)** に、**DamageProperty** 列を **[Values]\(値\)** に、**EventType** 列を **[Column labels]\(列ラベル\)** に、それぞれドラッグします。  

    ![ピボット モード](media/web-query-data/pivot-mode.png)

    結果は次のピボット テーブルのようになります。

    ![ピボット テーブル](media/web-query-data/pivot-table.png)

    Vermont と Alabama はそれぞれ同じカテゴリの下に 2 つのイベントがあるのに対し、Texas は異なるカテゴリに 2 つのイベントがあることに注意してください。 ピボット テーブルを使用すると、このようなことをすばやく見つけることができます。これは、迅速な分析のための優れたツールです。

## <a name="share-queries"></a>クエリを共有する

作成したクエリを共有したいことがよくあります。 

1. クエリ ウィンドウで、最初にコピーしたクエリを選択します。

1. クエリ ウィンドウの上部にある **[Share]\(共有\)** を選択します。 

:::image type="content" source="media/web-query-data/share-menu.png" alt-text="10 件のストーム イベントの開始時刻、終了時刻、エピソード、イベント ID、州、イベントの種類を一覧表示した表のスクリーンショット。":::

ドロップダウンでは、次のオプションを使用できます。
* リンクをクリップボードに保存
* [リンク、クエリをクリップボードに保存](#provide-a-deep-link)
* リンク、クエリ、結果をクリップボードに保存
* [ダッシュボードにピン留めする](#pin-to-dashboard)
* [Power BI へのクエリ](power-bi-imported-query.md)

### <a name="provide-a-deep-link"></a>ディープ リンクを提供する

ディープ リンクを提供して、クラスターにアクセスできる他のユーザーがクエリを実行できるようにすることができます。

1. **[共有]** で、 **[Link, query to clipboard]\(リンク、クエリをクリップボードに\)** を選択します。

1. リンクとクエリをテキスト ファイルにコピーします。

1. リンクを新しいブラウザー ウィンドウに貼り付けます。 クエリを実行した後の結果は次のようになります。

    :::image type="content" source="media/web-query-data/shared-query.png" alt-text="10 件のストーム イベントの開始時刻、終了時刻、エピソード、イベント ID、州、イベントの種類を一覧表示した表のスクリーンショット。":::

### <a name="pin-to-dashboard"></a>[ダッシュボードにピン留めする]

Web UI でクエリを使用してデータ探索を完了し、必要なデータを見つけたら、それをダッシュボードにピン留めして継続的な監視を行うことができます。 

クエリをピン留めするには:

1. **[共有]** で、 **[ダッシュボードにピン留めする]** を選択します。

1. **[ダッシュボードにピン留めする]** ペインで、次のようにします。
    1. **[クエリ名]** を入力します。
    1. **[既存のものを使用]** または **[新規作成]** を選択します。
    1. **[ダッシュボード名]** を入力します。
    1. **[View dashboard after creation]\(作成後にダッシュボードを表示する\)** チェックボックスをオンにします (新しいダッシュボードの場合)。
    1. **[ピン留め]** の選択

    :::image type="content" source="media/web-query-data/pin-to-dashboard.png" alt-text="10 件のストーム イベントの開始時刻、終了時刻、エピソード、イベント ID、州、イベントの種類を一覧表示した表のスクリーンショット。":::
    
> [!NOTE]
> **[ダッシュボードにピン留めする]** では、選択したクエリのみがピン留めされます。 ダッシュボードのデータ ソースを作成し、レンダリング コマンドをダッシュボードでビジュアルに変換するには、データベースの一覧で、関連するデータベースを選択する必要があります。

## <a name="export-query-results"></a>クエリ結果をエクスポートする

クエリ結果を CSV ファイルにエクスポートするには、 **[ファイル]**  >  **[CSV にエクスポート]** を選択します。

:::image type="content" source="media/web-query-data/export-results.png" alt-text="10 件のストーム イベントの開始時刻、終了時刻、エピソード、イベント ID、州、イベントの種類を一覧表示した表のスクリーンショット。":::

## <a name="provide-feedback"></a>フィードバックの提供

1. アプリケーションの右上にあるフィードバック アイコンを選択します ![フィードバック アイコン](media/web-query-data/icon-feedback.png).

1. フィードバックを入力し、 **[Submit]\(送信\)** を選択してください。

## <a name="clean-up-resources"></a>リソースをクリーンアップする

このクイック スタートではリソースは何も作成しませんでしたが、アプリケーションから一方または両方のクラスターを削除したい場合は、クラスターを右クリックして、 **[Remove connection]\(接続を削除\)** を選択します。

## <a name="next-steps"></a>次のステップ

[Azure Data Explorer のクエリを記述する](write-queries.md)
