---
title: イベント ハブから Azure Data Explorer にデータを取り込む
description: この記事では、Event Hub から Azure Data Explorer にデータを取り込む (読み込む) 方法について学習します。
author: orspod
ms.author: orspodek
ms.reviewer: tzgitlin
ms.service: data-explorer
ms.topic: how-to
ms.date: 08/13/2020
ms.localizationpriority: high
ms.openlocfilehash: 798a8b201ee87d5c43aeb31d6af515d41c516bef
ms.sourcegitcommit: f49e581d9156e57459bc69c94838d886c166449e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/01/2020
ms.locfileid: "95512215"
---
# <a name="ingest-data-from-event-hub-into-azure-data-explorer"></a>イベント ハブから Azure Data Explorer にデータを取り込む

> [!div class="op_single_selector"]
> * [ポータル](ingest-data-event-hub.md)
> * [ワンクリック](one-click-event-hub.md)
> * [C#](data-connection-event-hub-csharp.md)
> * [Python](data-connection-event-hub-python.md)
> * [Azure Resource Manager テンプレート](data-connection-event-hub-resource-manager.md)

[!INCLUDE [data-connector-intro](includes/data-connector-intro.md)]

Azure データ エクスプローラーには、Event Hubs からの取り込み (データの読み込み)、ビッグ データのストリーミング プラットフォーム、イベント取り込みサービスの機能があります。 [Event Hubs](/azure/event-hubs/event-hubs-about) は、1 秒あたり数百万件のイベントをほぼリアルタイムで処理できます。 この記事では、イベント ハブを作成し、Azure データ エクスプローラーからそれに接続し、システム経由でデータ フローを確認します。

イベント ハブから Azure Data Explorer への取り込みに関する一般的な情報については、[イベント ハブへの接続](ingest-data-event-hub-overview.md)に関する記事を参照してください。

## <a name="prerequisites"></a>前提条件

* Azure サブスクリプションをお持ちでない場合は、開始する前に[無料の Azure アカウント](https://azure.microsoft.com/free/)を作成してください。
* [テスト用のクラスターとデータベース](create-cluster-database-portal.md)。
* データを生成してイベント ハブに送信する[サンプル アプリ](https://github.com/Azure-Samples/event-hubs-dotnet-ingest)。 ご使用のシステムにサンプル アプリをダウンロードしてください。
* サンプル アプリを実行するための [Visual Studio 2019](https://visualstudio.microsoft.com/vs/)。

## <a name="sign-in-to-the-azure-portal"></a>Azure portal にサインインする

[Azure portal](https://portal.azure.com/) にサインインします。

## <a name="create-an-event-hub"></a>イベント ハブの作成

この記事では、サンプル データを生成してイベント ハブに送信します。 最初の手順は、イベント ハブの作成です。 Azure portal で Azure Resource Manager テンプレートを使用してこの処理を行います。

1. イベント ハブを作成するには、次のボタンを使用してデプロイを開始します。 この記事の残りの手順を実行できるよう、右クリックして **[新しいウィンドウで開く]** を選択してください。

    [![[Azure にデプロイ] ボタン](media/ingest-data-event-hub/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F201-event-hubs-create-event-hub-and-consumer-group%2Fazuredeploy.json)

    **[Deploy to Azure]\(Azure へのデプロイ\)** ボタンをクリックして Azure portal に移動し、デプロイ フォームに入力します。

    ![イベント ハブの作成フォーム](media/ingest-data-event-hub/deploy-to-azure.png)

1. イベント ハブを作成するサブスクリプションを選択し、*test-hub-rg* というリソース グループを作成します。

    ![リソース グループを作成する](media/ingest-data-event-hub/create-resource-group.png)

1. フォームに次の情報を入力します。

    ![デプロイ フォーム](media/ingest-data-event-hub/deployment-form.png)

    次の表に記載されていない設定については、既定値のままにしてください。

    **設定** | **推奨値** | **フィールドの説明**
    |---|---|---|
    | サブスクリプション | 該当するサブスクリプション | イベント ハブに使用する Azure サブスクリプションを選択します。|
    | Resource group | *test-hub-rg* | 新しいリソース グループを作成します。 |
    | 場所 | *米国西部* | この記事では *[米国西部]* を選択します。 運用システムでは、ニーズに最も適したリージョンを選択します。 最良のパフォーマンスを得るためには、Kusto クラスターと同じ場所にイベント ハブの名前空間を作成します (高スループットのイベント ハブの名前空間に最も重要です)。
    | 名前空間名 | 一意の名前空間名 | 名前空間を識別する一意の名前を選択します。 たとえば、*mytestnamespace* と指定します。 指定した名前にドメイン名 *servicebus.windows.net* が付加されます。 この名前には、文字、数字、ハイフンのみを含めることができます。 名前の先頭は英字、末尾は英字または数字にする必要があります。 値の長さは 6 から 50 文字にする必要があります。
    | イベント ハブ名 | *test-hub* | イベント ハブは、固有のスコープ コンテナーを提供する名前空間以下にあります。 イベント ハブ名は、名前空間内で一意にする必要があります。 |
    | コンシューマー グループ名 | *test-group* | コンシューマー グループを使用すると、複数の使用アプリケーションがそれぞれイベント ストリーム ビューを持つことができるようになります。 |
    | | |

1. **[購入]** を選択して、サブスクリプション内にリソースを作成することを確認します。

1. プロビジョニング プロセスを監視するには、ツール バーの **[通知]** を選択します。 デプロイが完了するまでには数分かかることがありますが、すぐに次の手順に進むこともできます。

    ![[通知] アイコン](media/ingest-data-event-hub/notifications.png)

## <a name="create-a-target-table-in-azure-data-explorer"></a>Azure データ エクスプローラーでターゲット テーブルを作成する

次は、Event Hubs からデータを送信する先のテーブルを Azure データ エクスプローラーで作成します。 「**前提条件**」でプロビジョニングしたクラスターとデータベースにテーブルを作成します。

1. Azure portal でクラスターに移動し、 **[クエリ]** を選択します。

    ![アプリケーションの [クエリ] リンク](media/ingest-data-event-hub/query-explorer-link.png)

1. 次のコマンドをウィンドウにコピーし、 **[実行]** を選択して、取り込んだデータを受け取るテーブル (TestTable) を作成します。

    ```Kusto
    .create table TestTable (TimeStamp: datetime, Name: string, Metric: int, Source:string)
    ```

    ![クエリの作成の実行](media/ingest-data-event-hub/run-create-query.png)

1. 次のコマンドをウィンドウにコピーし、 **[実行]** を選択して、テーブル (TestTable) の列名とデータ型に受信 JSON データをマップします。

    ```Kusto
    .create table TestTable ingestion json mapping 'TestMapping' '[{"column":"TimeStamp", "Properties": {"Path": "$.timeStamp"}},{"column":"Name", "Properties": {"Path":"$.name"}} ,{"column":"Metric", "Properties": {"Path":"$.metric"}}, {"column":"Source", "Properties": {"Path":"$.source"}}]'
    ```

## <a name="connect-to-the-event-hub"></a>イベント ハブへの接続

ここで、Azure Data Explorer からイベント ハブに接続します。 この接続が確立されると、イベント ハブに送られてくるデータが、この記事の前の方で作成したテスト テーブルにストリームとして送られます。

1. ツールバーの **[通知]** を選択して、イベント ハブのデプロイが成功したことを確認します。

1. 作成したクラスターの **[データベース]** 、 **[TestDatabase]** の順に選択します。

    ![テスト データベースの選択](media/ingest-data-event-hub/select-test-database.png)

1. **[データ インジェスト]** 、 **[データ接続の追加]** の順に選択します。 

    :::image type="content" source="media/ingest-data-event-hub/event-hub-connection.png" alt-text="イベント ハブでデータ インジェストを選択し、データ接続を追加する - Azure Data Explorer":::

### <a name="create-a-data-connection"></a>データ接続を作成する

1. フォームに次の情報を入力します。

    :::image type="content" source="media/ingest-data-event-hub/data-connection-pane.png" alt-text="イベント ハブのデータ接続ペイン - Azure Data Explorer":::

    **設定** | **推奨値** | **フィールドの説明**
    |---|---|---|
    | データ接続名 | *test-hub-connection* | Azure データ エクスプローラーで作成する接続の名前。|
    | サブスクリプション |      | イベントハブ リソースが配置されているサブスクリプション ID。  |
    | Event Hub 名前空間 | 一意の名前空間名 | 以前に選択した、名前空間を識別する名前。 |
    | イベント ハブ | *test-hub* | 作成したイベント ハブ。 |
    | コンシューマー グループ | *test-group* | 作成したイベント ハブに定義されているコンシューマー グループ。 |
    | イベント システム プロパティ | 関連するプロパティを選択する | [イベント ハブのシステム プロパティ](/azure/service-bus-messaging/service-bus-amqp-protocol-guide#message-annotations)。 1 つのイベント メッセージに複数のレコードがある場合、システム プロパティは最初のものに追加されます。 システム プロパティを追加する場合は、テーブル スキーマと[マッピング](kusto/management/mappings.md)を[作成](kusto/management/create-table-command.md)または[更新](kusto/management/alter-table-command.md)して、選択したプロパティを含めます。 |
    | 圧縮 | *なし* | イベント ハブ メッセージ ペイロードの圧縮の種類。 サポートされている圧縮の種類: *なし、GZip*。|
    
#### <a name="target-table"></a>ターゲット テーブル

挿入したデータをルーティングするには、"*静的*" と "*動的*" という 2 つのオプションがあります。 この記事では、静的ルーティングを使用し、テーブル名、データ形式、およびマッピングを既定値として指定します。 イベントハブ メッセージにデータ ルーティング情報が含まれている場合、このルーティング情報によって既定の設定が上書きされます。

1. 次のルーティング設定を入力します。
  
   :::image type="content" source="media/ingest-data-event-hub/default-routing-settings.png" alt-text="イベント ハブにデータを取り込むための既定のルーティング設定 - Azure Data Explorer":::
        
   |**設定** | **推奨値** | **フィールドの説明**
   |---|---|---|
   | テーブル名 | *TestTable* | **TestDatabase** に作成したテーブル。 |
   | データ形式 | *JSON* | サポートされている形式は、Avro、CSV、JSON、MULTILINE JSON、ORC、PARQUET、PSV、SCSV、SOHSV、TSV、TXT、TSVE、APACHEAVRO、および W3CLOG です。 |
   | マッピング | *TestMapping* | **TestDatabase** に作成した [マッピング](kusto/management/mappings.md)。これにより、受信データを **TestTable** の列名とデータ型にマッピングします。 JSON、MULTILINE JSON、AVRO では必須。その他の形式では省略可能。|
    
   > [!NOTE]
   > * **既定のルーティング設定** をすべて指定する必要はありません。 部分的な設定も受け入れられます。
   > * データ接続の作成後にエンキューされたイベントのみが取り込まれたます。

1. **［作成］** を選択します 

### <a name="event-system-properties-mapping"></a>イベント システム プロパティのマッピング

> [!Note]
> * システム プロパティは、単一レコードのイベントに対してサポートされています。
> * `csv` マッピングの場合、レコードの先頭にプロパティが追加されます。 `json` マッピングの場合、ドロップダウン リストに表示される名前に従ってプロパティが追加されます。

テーブルの **[データ ソース]** セクションで **[イベント システムのプロパティ]** を選択した場合は、テーブル スキーマとマッピングに [システム プロパティ](ingest-data-event-hub-overview.md#system-properties)を含める必要があります。

## <a name="copy-the-connection-string"></a>接続文字列のコピー

前提条件の一覧にある[サンプル アプリ](https://github.com/Azure-Samples/event-hubs-dotnet-ingest)を実行する場合は、イベント ハブの名前空間用の接続文字列が必要です。

1. 作成したイベント ハブ名前空間で、 **[共有アクセス ポリシー]** 、 **[RootManageSharedAccessKey]** の順に選択します。

    ![共有アクセス ポリシー](media/ingest-data-event-hub/shared-access-policies.png)

1. **[接続文字列 - 主キー]** をコピーします これを、次のセクションで貼り付けます。

    ![接続文字列](media/ingest-data-event-hub/connection-string.png)

## <a name="generate-sample-data"></a>サンプル データを作成する

ダウンロードした[サンプル アプリ](https://github.com/Azure-Samples/event-hubs-dotnet-ingest)を使用してデータを生成します。

1. Visual Studio でサンプル アプリ ソリューションを開きます。

1. *program.cs* ファイルで、`eventHubName` 定数をイベント ハブの名前に更新し、`connectionString` 定数をイベント ハブ名前空間からコピーした接続文字列に更新します。

    ```csharp
    const string eventHubName = "test-hub";
    // Copy the connection string ("Connection string-primary key") from your Event Hub namespace.
    const string connectionString = @"<YourConnectionString>";
    ```

1. アプリケーションをビルドし、実行します。 アプリからイベント ハブにメッセージが送信され、10 秒ごとに状態が出力されます。

1. アプリからいくつかメッセージが送信されたら、イベント ハブとテスト テーブルへのデータの流れを確認するという次の手順に進みます。

## <a name="review-the-data-flow"></a>データ フローの確認

データを生成するアプリを使用して、そのデータがイベント ハブからクラスター内のテーブルに送信されるのを確認できるようになりました。

1. アプリの実行中に Azure portal でイベント ハブを確認すると、アクティビティの急上昇が見られます。

    ![イベント ハブ グラフ](media/ingest-data-event-hub/event-hub-graph.png)

1. これまでにデータベースに届いたメッセージ数を確認するには、テスト データベースで次のクエリを実行します。

    ```Kusto
    TestTable
    | count
    ```

1. メッセージの内容を表示するには、次のクエリを実行します。

    ```Kusto
    TestTable
    ```

    結果セットは次のようになっている必要があります。

    ![メッセージの結果セット](media/ingest-data-event-hub/message-result-set.png)

    > [!NOTE]
    > * Azure Data Explorer には、インジェスト プロセスを最適化することを目的とした、データ インジェストの集計 (バッチ処理) ポリシーがあります。 既定では、このポリシーは 5 分または 500 MB のデータに構成されているため、待ち時間が生じることがあります。 集計オプションについては、[バッチ処理のポリシー](kusto/management/batchingpolicy.md)に関するページを参照してください。 
    > * イベント ハブ インジェストには、10 秒または 1 MB のイベント ハブの応答時間が含まれます。 
    > * ストリーミングをサポートし、応答時間でのラグを削除するようにテーブルを構成します。 [ストリーミング ポリシー](kusto/management/streamingingestionpolicy.md)に関するページを参照してください。 

## <a name="clean-up-resources"></a>リソースをクリーンアップする

このイベント ハブを今後使用する予定がない場合は、コストが発生しないように **test-hub-rg** をクリーンアップします。

1. Azure Portal の左端で **[リソース グループ]** を選択し、作成したリソース グループを選択します。  

    左側のメニューが折りたたまれている場合は、 ![展開ボタン](media/ingest-data-event-hub/expand.png) をクリックして展開します。

   ![削除するリソース グループの選択](media/ingest-data-event-hub/delete-resources-select.png)

1. **test-resource-group** で **[リソース グループの削除]** を選択します。

1. 新しいウィンドウで、削除するリソース グループの名前 (*test-hub-rg*) を入力し、 **[削除]** を選択します。

## <a name="next-steps"></a>次のステップ

* [Azure Data Explorer でデータのクエリを実行する](web-query-data.md)
