---
title: 診断ログを使用して Azure Data Explorer のインジェスト、コマンド、およびクエリを監視する
description: Azure Data Explorer の診断ログを設定してインジェスト コマンド、およびクエリ操作を監視する方法について説明します。
author: orspod
ms.author: orspodek
ms.reviewer: guregini
ms.service: data-explorer
ms.topic: how-to
ms.date: 09/16/2020
ms.openlocfilehash: 61e0a4f13aba871a92bdcf36a8af9ed78bb09624
ms.sourcegitcommit: 97404e9ed4a28cd497d2acbde07d00149836d026
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/21/2020
ms.locfileid: "90832571"
---
# <a name="monitor-azure-data-explorer-ingestion-commands-and-queries-using-diagnostic-logs"></a>診断ログを使用して Azure Data Explorer のインジェスト、コマンド、およびクエリを監視する

Azure Data Explorer は、アプリケーション、Web サイト、IoT デバイスなどからの大量のデータ ストリーミングをリアルタイムに分析するためのフル マネージドのデータ分析サービスです。 [Azure Monitor 診断ログ](/azure/azure-monitor/platform/diagnostic-logs-overview)は、Azure リソースの動作状況に関するデータを提供します。 Azure Data Explorer は、診断ログを使用して、インジェストの成功、インジェストの失敗、コマンド、およびクエリ操作に関する分析情報を取得します。 操作ログを Azure Storage、イベント ハブ、または Log Analytics にエクスポートして、インジェスト、コマンド、およびクエリの状態を監視することができます。 Azure Storage と Azure イベント ハブからのログを Azure Data Explorer クラスター内のテーブルにルーティングして詳細な分析を行うことができます。

> [!IMPORTANT] 
> 診断ログ データには機密データが含まれている場合があります。 監視のニーズに応じて、ログの出力先のアクセス許可を制限します。 

## <a name="prerequisites"></a>前提条件

* Azure のサブスクリプションがない場合は、[Azure の無料アカウント](https://azure.microsoft.com/free/)を作成します。
* [Azure portal](https://portal.azure.com/) にサインインします。
* [クラスターとデータベース](create-cluster-database-portal.md)を作成します。

## <a name="set-up-diagnostic-logs-for-an-azure-data-explorer-cluster"></a>Azure Data Explorer クラスターの診断ログを設定する

診断ログを使用すると、以下のログ データの収集を構成できます。

# <a name="ingestion"></a>[データの取り込み](#tab/ingestion)

* **成功したインジェスト操作**:ログには、正常に完了したインジェスト操作に関する情報が含まれています。
* **失敗したインジェスト操作**:ログには、失敗したインジェスト操作に関する詳細情報 (エラーの詳細を含む) が含まれています。 

# <a name="commands-and-queries"></a>[コマンドとクエリ](#tab/commands-and-queries)

* **コマンド**:これらのログには、最終的な状態に達した管理者コマンドに関する情報が含まれています。
* **クエリ**:これらのログには、最終的な状態に達したクエリに関する詳細情報が含まれています。 

    > [!NOTE]
    > クエリ ログのデータには、クエリのテキストは含まれていません。

---

その後、データはストレージ アカウントにアーカイブされるか、イベント ハブにストリーミングされるか、または仕様に応じて Log Analytics に送信されます。

### <a name="enable-diagnostic-logs"></a>診断ログを有効にする

既定では、診断ログは無効になっています。 診断ログを有効にするには、次のステップを実行します。

1. [Azure portal](https://portal.azure.com) で、監視する Azure Data Explorer クラスター リソースを選択します。
1. **[監視]** で **[診断設定]** を選択します。
  
    ![診断ログを追加する](media/using-diagnostic-logs/add-diagnostic-logs.png)

1. **[診断設定の追加]** を選択します。
1. **[診断設定]** ウィンドウで以下を行います。

    :::image type="content" source="media/using-diagnostic-logs/configure-diagnostics-settings.png" alt-text="診断設定の構成":::

    1. 診断設定の **[名前]** を選択します。
    1. ストレージ アカウント、イベント ハブ、Log Analytics から 1 つ以上のターゲットを選択します。
    1. 収集するログ (`SucceededIngestion`、 `FailedIngestion`、 `Command`、または `Query`) を選択します。
    1. 収集する [[メトリック]](using-metrics.md#supported-azure-data-explorer-metrics) を選択します (省略可能)。  
    1. **[保存]** を選択して、新しい診断ログの設定とメトリックを保存します。

新しい設定は数分で設定されます。 ログは、構成したアーカイブ ターゲット (ストレージ アカウント、イベント ハブ、Log Analytics) に表示されます。 

    > [!NOTE]
    > If you send logs to Log Analytics, the `SucceededIngestion`, `FailedIngestion`, `Command`, and `Query` logs will be stored in Log Analytics tables named: `SucceededIngestion`, `FailedIngestion`, `ADXCommand`, `ADXQuery`, respectively.

## <a name="diagnostic-logs-schema"></a>診断ログのスキーマ

すべての [Azure Monitor 診断ログは、共通の上位スキーマを使用します](/azure/azure-monitor/platform/diagnostic-logs-schema)。 Azure Data Explorer には、独自のイベントに固有のプロパティがあります。 すべてのログは JSON 形式で格納されます。

# <a name="ingestion"></a>[データの取り込み](#tab/ingestion)

### <a name="ingestion-logs-schema"></a>インジェスト ログ スキーマ

ログの JSON 文字列には、次の表に示す要素が含まれます。

|名前               |説明
|---                |---
|time               |レポートされた時刻
|resourceId         |Azure Resource Manager リソース ID
|operationName      |操作の名前:'MICROSOFT.KUSTO/CLUSTERS/INGEST/ACTION'
|operationVersion   |スキーマのバージョン:'1.0' 
|category           |操作のカテゴリ。 `SucceededIngestion` または `FailedIngestion`。 [正常な運用](#successful-ingestion-operation-log)と[運用の失敗](#failed-ingestion-operation-log)それぞれでプロパティは異なります。
|properties         |操作の詳細情報。

#### <a name="successful-ingestion-operation-log"></a>成功したインジェスト操作のログ

**例:**

```json
{
    "time": "",
    "resourceId": "",
    "operationName": "MICROSOFT.KUSTO/CLUSTERS/INGEST/ACTION",
    "operationVersion": "1.0",
    "category": "SucceededIngestion",
    "properties":
    {
        "succeededOn": "2019-05-27 07:55:05.3693628",
        "operationId": "b446c48f-6e2f-4884-b723-92eb6dc99cc9",
        "database": "Samples",
        "table": "StormEvents",
        "ingestionSourceId": "66a2959e-80de-4952-975d-b65072fc571d",
        "ingestionSourcePath": "https://kustoingestionlogs.blob.core.windows.net/sampledata/events8347293.json",
        "rootActivityId": "d0bd5dd3-c564-4647-953e-05670e22a81d"
    }
}
```
**成功した操作の診断ログのプロパティ**

|名前               |説明
|---                |---
|succeededOn        |インジェストの完了時刻
|operationId        |Azure Data Explorer インジェスト操作 ID
|database           |ターゲット データベースの名前
|table              |ターゲット テーブルの名前
|ingestionSourceId  |インジェスト データ ソースの ID
|ingestionSourcePath|インジェスト データ ソースまたは BLOB URI のパス
|rootActivityId     |アクティビティ ID

#### <a name="failed-ingestion-operation-log"></a>インジェスト操作の失敗ログ

**例:**

```json
{
    "time": "",
    "resourceId": "",
    "operationName": "MICROSOFT.KUSTO/CLUSTERS/INGEST/ACTION",
    "operationVersion": "1.0",
    "category": "FailedIngestion",
    "properties":
    {
        "failedOn": "2019-05-27 08:57:05.4273524",
        "operationId": "5956515d-9a48-4544-a514-cf4656fe7f95",
        "database": "Samples",
        "table": "StormEvents",
        "ingestionSourceId": "eee56f8c-2211-4ea4-93a6-be556e853e5f",
        "ingestionSourcePath": "https://kustoingestionlogs.blob.core.windows.net/sampledata/events5725592.json",
        "rootActivityId": "52134905-947a-4231-afaf-13d9b7b184d5",
        "details": "Permanent failure downloading blob. URI: ..., permanentReason: Download_SourceNotFound, DownloadFailedException: 'Could not find file ...'",
        "errorCode": "Download_SourceNotFound",
        "failureStatus": "Permanent",
        "originatesFromUpdatePolicy": false,
        "shouldRetry": false
    }
}
```

**失敗した操作の診断ログのプロパティ**

|名前               |説明
|---                |---
|failedOn           |インジェストの完了時刻
|operationId        |Azure Data Explorer インジェスト操作 ID
|database           |ターゲット データベースの名前
|table              |ターゲット テーブルの名前
|ingestionSourceId  |インジェスト データ ソースの ID
|ingestionSourcePath|インジェスト データ ソースまたは BLOB URI のパス
|rootActivityId     |アクティビティ ID
|details            |エラーとエラーメッセージの詳しい説明
|errorCode          |エラー コード 
|failureStatus      |`Permanent` または `Transient`。 一時的なエラーは、再試行することで成功する可能性があります。
|originatesFromUpdatePolicy|エラーが更新ポリシーによって発生した場合は True です
|shouldRetry        |再試行が成功した場合は True です

# <a name="commands-and-queries"></a>[コマンドとクエリ](#tab/commands-and-queries)

### <a name="commands-and-queries-logs-schema"></a>コマンドとクエリのログ スキーマ

ログの JSON 文字列には、次の表に示す要素が含まれます。

|名前               |説明
|---                |---
|time               |レポートされた時刻
|resourceId         |Azure Resource Manager リソース ID
|operationName      |操作の名前:'MICROSOFT.KUSTO/CLUSTERS/COMMAND/ACTION' または "MICROSOFT.KUSTO/CLUSTERS/QUERY/ACTION"。 プロパティは、コマンドとクエリのログで異なります。
|operationVersion   |スキーマのバージョン:'1.0' 
|category           |操作のカテゴリ (`Command` または `Query`)。 プロパティは、コマンドとクエリのログで異なります。
|properties         |操作の詳細情報。

#### <a name="command-log"></a>コマンド ログ

**例:**

```json
{
    "time": "",
    "resourceId": "",
    "operationName": "MICROSOFT.KUSTO/CLUSTERS/COMMAND/ACTION",
    "operationVersion": "1.0",
    "category": "Command",
    "properties":
    {
        "RootActivityId": "d0bd5dd3-c564-4647-953e-05670e22a81d",
        "StartedOn": "2020-09-12T18:06:04.6898353Z",
        "LastUpdatedOn": "2020-09-12T18:06:04.6898353Z",
        "Database": "DatabaseSample",
        "State": "Completed",
        "FailureReason": "XXX",
        "TotalCpu": "00:01:30.1562500",
        "CommandType": "ExtentsDrop",
        "Application": "Kusto.WinSvc.DM.Svc",
        "ResourceUtilization": "{\"CacheStatistics\":{\"Memory\":{\"Hits\":0,\"Misses\":0},\"Disk\":{\"Hits\":0,\"Misses\":0},\"Shards\":{\"Hot\":{\"HitBytes\":0,\"MissBytes\":0,\"RetrieveBytes\":0},\"Cold\":{\"HitBytes\":0,\"MissBytes\":0,\"RetrieveBytes\":0},\"BypassBytes\":0}},\"TotalCpu\":\"00:00:00\",\"MemoryPeak\":0,\"ScannedExtentsStatistics\":{\"MinDataScannedTime\":null,\"MaxDataScannedTime\":null,\"TotalExtentsCount\":0,\"ScannedExtentsCount\":0,\"TotalRowsCount\":0,\"ScannedRowsCount\":0}}",
        "Duration": "00:03:30.1562500",
        "User": "AAD app id=0571b364-eeeb-4f28-ba74-90a8b4132b53",
        "Principal": "aadapp=0571b364-eeeb-4f28-ba74-90a3b4136b53;5c443533-c927-4410-a5d6-4d6a5443b64f"
    }
}
```
**コマンド診断ログのプロパティ**

|名前               |説明
|---                |---
|RootActivityId |ルート アクティビティ ID
|StartedOn        |このコマンドが開始された時刻 (UTC)
|LastUpdatedOn        |このコマンドが終了した時刻 (UTC)
|データベース          |コマンドが実行されたデータベースの名前
|State              |コマンドが終了した状態
|FailureReason  |エラーの理由
|TotalCpu |合計 CPU 時間
|CommandType     |[コマンドの種類]
|Application     |コマンドを呼び出したアプリケーション名
|ResourceUtilization     |コマンド リソース使用率
|Duration     |コマンドの実行時間
|User     |クエリを呼び出したユーザー
|プリンシパル     |クエリを呼び出したプリンシパル

#### <a name="query-log"></a>クエリ ログ

**例:**

```json
{
    "time": "",
    "resourceId": "",
    "operationName": "MICROSOFT.KUSTO/CLUSTERS/QUERY/ACTION",
    "operationVersion": "1.0",
    "category": "Query",
    "properties": {
        "RootActivityId": "3e6e8814-e64f-455a-926d-bf16229f6d2d",
        "StartedOn": "2020-09-04T13:45:45.331925Z",
        "LastUpdatedOn": "2020-09-04T13:45:45.3484372Z",
        "Database": "DatabaseSample",
        "State": "Completed",
        "FailureReason": "[none]",
        "TotalCpu": "00:00:00",
        "ApplicationName": "MyApp",
        "MemoryPeak": 0,
        "Duration": "00:00:00.0165122",
        "User": "AAD app id=0571b364-eeeb-4f28-ba74-90a8b4132b53",
        "Principal": "aadapp=0571b364-eeeb-4f28-ba74-90a8b4132b53;5c823e4d-c927-4010-a2d8-6dda2449b6cf",
        "ScannedExtentsStatistics": {
            "MinDataScannedTime": "2020-07-27T08:34:35.3299941",
            "MaxDataScannedTime": "2020-07-27T08:34:41.991661",
            "TotalExtentsCount": 64,
            "ScannedExtentsCount": 64,
            "TotalRowsCount": 320,
            "ScannedRowsCount": 320
        },
        "CacheStatistics": {
            "Memory": {
                "Hits": 192,
                "Misses": 0
          },
            "Disk": {
                "Hits": 0,
                "Misses": 0
      },
            "Shards": {
                "Hot": {
                    "HitBytes": 0,
                    "MissBytes": 0,
                    "RetrieveBytes": 0
        },
                "Cold": {
                    "HitBytes": 0,
                    "MissBytes": 0,
                    "RetrieveBytes": 0
        },
            "BypassBytes": 0
      }
    },
    "ResultSetStatistics": {
        "TableCount": 1,
        "TablesStatistics": [
        {
          "RowCount": 1,
          "TableSize": 9
        }
      ]
    }
  }
}
```

**クエリ診断ログのプロパティ**

|名前               |説明
|---                |---
|RootActivityId |ルート アクティビティ ID
|StartedOn        |このコマンドが開始された時刻 (UTC)
|LastUpdatedOn           |このコマンドが終了した時刻 (UTC)
|データベース              |コマンドが実行されたデータベースの名前
|State  |コマンドが終了した状態
|FailureReason|エラーの理由
|TotalCpu     |合計 CPU 時間
|ApplicationName            |クエリを呼び出したアプリケーション名
|MemoryPeak          |メモリのピーク
|Duration      |コマンドの実行時間
|User|クエリを呼び出したユーザー
|プリンシパル        |クエリを呼び出したプリンシパル
|ScannedExtentsStatistics        | スキャンされたエクステント統計情報を含む
|MinDataScannedTime        |最小データ スキャン時間
|MaxDataScannedTime        |最大データ スキャン時間
|TotalExtentsCount        |合計エクステント数
|ScannedExtentsCount        |スキャンされたエクステント数
|TotalRowsCount        |合計行数
|ScannedRowsCount        |スキャンされた行数
|CacheStatistics        |キャッシュ統計情報を含む
|メモリ        |キャッシュ メモリ統計情報を含む
|ヒット数        |メモリ キャッシュ ヒット数
|[Misses] \(ミス数)        |メモリ キャッシュ ミス数
|ディスク        |キャッシュ ディスク統計情報を含む
|ヒット数        |ディスク キャッシュ ヒット数
|[Misses] \(ミス数)        |ディスク キャッシュ ミス数
|シャード        |ホットおよびコールド シャード キャッシュ統計情報を含む
|ホット        |ホット シャード キャッシュ統計情報を含む
|HitBytes        |シャード ホット キャッシュ ヒット数
|MissBytes        |シャード ホット キャッシュ ミス数
|RetrieveBytes        | シャード ホット キャッシュの取得バイト数
|アイス        |コールド シャード キャッシュ統計情報を含む
|HitBytes        |シャード コールド キャッシュ ヒット数
|MissBytes        |シャード コールド キャッシュ ミス数
|RetrieveBytes        |シャード コールド キャッシュの取得バイト数
|BypassBytes        |シャード キャッシのバイパス バイト数
|ResultSetStatistics        |結果セットの統計情報を含む
|TableCount        |結果セット テーブル数
|TablesStatistics        |結果セット テーブルの統計情報を含む
|RowCount        | 結果セット テーブルの行数
|TableSize        |結果セット テーブルの行数

---

## <a name="next-steps"></a>次のステップ

* [メトリックを使用してクラスターの正常性を監視する](using-metrics.md)
* [チュートリアル:インジェスト診断ログのために Azure Data Explorer で監視データを取り込んでクエリを実行する](ingest-data-no-code.md)
